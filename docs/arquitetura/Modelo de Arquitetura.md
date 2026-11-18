# Modelo de Arquitetura do Sistema

**Versão:** 1.3 (rascunho) | **Status:** Em elaboração | **Data:** 2026-11-14

**Finalidade:** definir a estrutura lógica do sistema, as responsabilidades de seus componentes, suas interfaces e o contrato de interação com ferramentas externas ao núcleo conversacional.

## 1. Objetivo

O modelo de arquitetura define a organização lógica do assistente conversacional baseado em voz e LLM, estabelecendo os principais componentes, suas responsabilidades, interfaces e relações.

A arquitetura desacopla o núcleo conversacional do domínio de aplicação: sistemas de informação diferentes são integrados por meio de ferramentas explicitamente disponibilizadas ao assistente. A LLM interpreta e propõe, mas não acessa o domínio diretamente nem decide sozinha a execução de operações; o acesso a operações externas ocorre por uma camada controlada de ferramentas.

A arquitetura permanece **independente de tecnologias específicas de implementação**. Os exemplos usam o estudo de caso da biblioteca apenas para ilustrar os contratos.

| Documento | Versão | Relação |
| --- | --- | --- |
| Documento de Visão | 1.1 | Problema, escopo e princípios de projeto. |
| Requisitos do Sistema | 2.1 | RF01 a RF14 e RNF01 a RNF12, citados neste modelo. |
| Especificação do Estudo de Caso | 0.3 | Operações, regras, erros, casos de teste (CT) e cenários conversacionais (CC). |

## 2. Visão geral da arquitetura

A arquitetura lógica é composta pelos oito componentes do diagrama e pelo **Registro de Eventos**, transversal (seção 14), que recebe eventos de todos os componentes. O reconhecimento de fala, a síntese de voz e o modelo de linguagem são serviços externos acessados por **portas** (interfaces), o que permite substituí-los (RNF02).

```text
                   ┌─────────────────────────────┐
 Usuário ◄─ voz ─► │ Interface Conversacional    │
                   │ portas: STT e TTS           │
                   └──────────────┬──────────────┘
                                  ▼
┌───────────────┐  ┌─────────────────────────────┐  ┌───────────────┐
│ Gerenciador   │  │ Núcleo Conversacional       │  │ Gerenciador   │
│ de Contexto   │◄►│ Interpretação               │◄►│ de Tarefa     │
│ conversa e    │  │ Política de diálogo         │  │ estado, faltas│
│ sessão        │  │ Resposta + guarda           │  │ confirmação   │
└───────────────┘  │ porta de linguagem (LLM)    │  └───────────────┘
                   └──────────────┬──────────────┘
                                  │ ToolRequest (proposta)
                                  ▼
                   ┌─────────────────────────────┐
                   │ Orquestrador de Ferramentas │
                   │ valida • autoriza • executa │
                   └──────────────┬──────────────┘
                                  ▼
                   ┌─────────────────────────────┐
                   │ Registro / Interface de     │
                   │ Ferramentas                 │
                   └──────────────┬──────────────┘
                                  ▼
                   ┌─────────────────────────────┐
                   │ Adaptador do Sistema de     │
                   │ Domínio                     │
                   └──────────────┬──────────────┘
                                  ▼
                   ┌─────────────────────────────┐
                   │ Sistema de Informação de    │
                   │ Domínio                     │
                   └─────────────────────────────┘
```

## 3. Componentes e responsabilidades

### 3.1 Interface Conversacional

Responsável pela interação direta com o usuário.

- capturar a fala e obtê-la como texto por meio da **porta de reconhecimento de fala**, entregando ao núcleo o texto, a confiança da transcrição (quando o provedor a fornece) e um indicador de qualidade: transcrição normal, sem fala ou ininteligível (RF01, RF13);
- converter a resposta textual em voz por meio da **porta de síntese de voz** e reproduzi-la (RF12);
- detectar eventos de interação: interrupção da reprodução pelo usuário, silêncio prolongado após uma pergunta e encerramento (RF14);
- reproduzir novamente a última resposta quando o núcleo o solicitar (RF14);
- tratar falhas e lentidão dos serviços de voz com tempo-limite e mensagem controlada (RNF09);
- não armazenar o áudio bruto após a transcrição (RNF10).

A interface não contém regras do domínio nem interpreta a intenção do usuário.

### 3.2 Núcleo Conversacional

Principal componente de processamento da interação em linguagem natural. Reúne três responsabilidades internas:

**Interpretação**

- identificar a intenção (inclusive os comandos de controle: confirmar, negar, cancelar, repetir, encerrar) e extrair os parâmetros da fala (RF02, RF04);
- usar o contexto conversacional e o estado da tarefa para interpretar a fala (por exemplo, um “sim” só é confirmação se houver confirmação pendente);
- resolver referências (“esse”, “o segundo”) com apoio do Gerenciador de Contexto (RF03).

**Política de diálogo**

Decide o próximo passo da interação a partir do estado da tarefa e do último resultado: pedir informação faltante, apresentar candidatos e pedir escolha, pedir confirmação, propor uma chamada de ferramenta, tratar um resultado, recuperar de uma falha, verificar um estado ou encerrar. Esta responsabilidade é conduzida por software (a máquina de estados da seção 12); a LLM apoia a interpretação e a redação, mas não define sozinha as transições.

**Geração de resposta**

Produz a resposta textual a partir do tipo de resposta decidido pela política (pedido de informação, confirmação, resultado, recusa, falha, encerramento) e dos fatos retornados pelas ferramentas, sob a guarda de fidelidade da seção 16 (RF11).

**Resolução de entidades.** Quando uma consulta retorna mais de um candidato, o Núcleo trata a ambiguidade: apresenta os candidatos em quantidade limitada por vez, registra no Gerenciador de Contexto a lista apresentada (com seus identificadores internos, na ordem apresentada) e pede a escolha. O adaptador nunca converte nomes em identificadores.

**Porta de linguagem.** O acesso à LLM ocorre por uma porta com três operações conceituais: interpretar a fala, propor uma chamada de ferramenta e redigir a resposta. A porta pode ser substituída por um provedor simulado nos testes (RNF02, RNF12).

### 3.3 Gerenciador de Contexto

Responsável pelo armazenamento, recuperação e atualização das informações da interação. Mantém duas áreas distintas:

**Contexto conversacional**

- entidades mencionadas, parâmetros já informados e resultados apresentados, para dar continuidade à tarefa sem pedir de novo o que já foi dito (RF03);
- a lista de candidatos apresentada por último, com identificadores internos e na ordem apresentada, para resolver referências ordinais;
- a última resposta emitida, para repeti-la quando o usuário pedir.

**Contexto de sessão**

- identidade do usuário e permissões da sessão (autenticação simulada no estudo de caso);
- definido na abertura da sessão, fora da LLM, e somente leitura para o Núcleo; lido pelo Orquestrador para aplicar a identidade às chamadas (seção 15).

Ambos são descartados ao encerrar a sessão (RF03 CA4, RF10 CA6). Identificadores internos nunca são falados ao usuário.

### 3.4 Gerenciador de Tarefa

Responsável pelo estado da tarefa em execução. Distingue o **contexto da conversa** do **estado da tarefa**.

- registrar o objetivo da tarefa (a ferramenta-alvo), os parâmetros já coletados e as ambiguidades pendentes;
- **calcular, de forma determinística, os parâmetros obrigatórios que faltam**, comparando os parâmetros coletados com a definição da ferramenta (RF04 CA2); a LLM extrai valores, mas quem decide o que falta é o software;
- guardar a solicitação validada que aguarda confirmação (seção 11);
- controlar as transições da tarefa pela máquina de estados da seção 12;
- preservar o estado entre uma falha e a retomada, evitando reiniciar a tarefa (RF09 CA5);
- permitir a correção de informações já fornecidas (RF09 CA4).

Mantém uma tarefa ativa por vez. O tratamento de uma nova solicitação durante uma tarefa não concluída é uma decisão em aberto (seção 19).

### 3.5 Orquestrador de Ferramentas

Controla o ciclo de utilização das ferramentas. É a fronteira de controle entre o processamento baseado em LLM e a execução efetiva de operações externas.

- receber a proposta de chamada (`ToolRequest`) produzida pelo Núcleo;
- verificar a ferramenta no Registro (existência, disponibilidade e autorização para a sessão) e validar o contrato da chamada (seção 4.3);
- aplicar a identidade da sessão e rejeitar qualquer identidade gerada pelo modelo (RF06 CA4);
- impor a confirmação das escritas (seção 11);
- executar a ferramenta com tempo-limite e normalizar o resultado em `ToolResult`;
- registrar os eventos de cada etapa e devolver o resultado ao Núcleo.

A seleção da ferramenta e a geração dos argumentos pertencem à proposta do Núcleo. O Orquestrador **não escolhe** a ferramenta: ele verifica, autoriza e executa o que foi proposto.

## 4. Contrato lógico de ferramentas

O contrato lógico define a forma abstrata pela qual o sistema disponibiliza operações externas ao Núcleo, separando a descrição de uma capacidade, a proposta de execução, a validação, a execução e o resultado. É uma abstração arquitetural, que poderá ser implementada por diferentes mecanismos de tool calling ou protocolos de integração.

### 4.1 Definição da ferramenta

```text
ToolDefinition
├── id                      identificador único (também o nome visto pelo modelo)
├── description             capacidade, quando usar e quando não usar
├── parameters[]            nome, tipo, descrição, obrigatório, restrições
├── side_effect             READ | WRITE
├── requires_confirmation   verdadeiro | falso
├── result_schema           campos devolvidos em caso de sucesso
├── error_catalog[]         códigos possíveis, categoria, dados adicionais
└── timeout                 tempo-limite de execução
```

- **`description`:** indica quando a ferramenta é aplicável e quando não é.
- **`parameters`:** cada parâmetro declara se é obrigatório e suas restrições (formato, valores permitidos, dependências); os obrigatórios são a base do cálculo de parâmetros faltantes (seção 3.4). A identidade do usuário não é parâmetro: vem da sessão.
- **`side_effect` e `requires_confirmation`:** distinguem leitura de escrita. Por configuração da ferramenta, escritas exigem confirmação e consultas não (RF10 CA1, RNF01).
- **`result_schema`, `error_catalog` e `timeout`:** descrevem um resultado compacto e adequado à fala (RNF08), os códigos de erro possíveis com sua categoria e dados adicionais, e o tempo-limite de execução (RNF09).

Exemplo conceitual, do estudo de caso:

```text
registrar_emprestimo
├── side_effect: WRITE        requires_confirmation: verdadeiro
├── parameters: livro_id (texto, obrigatório)
├── result_schema: emprestimo_id, titulo, data_prevista_devolucao, emprestimos_ativos
└── error_catalog: LIVRO_NAO_ENCONTRADO, EMPRESTIMO_ATIVO_MESMO_LIVRO,
                   CARENCIA_EM_VIGOR, LIMITE_EMPRESTIMOS_ATINGIDO, LIVRO_INDISPONIVEL
```

### 4.2 Solicitação de execução

```text
ToolRequest
├── request_id     identifica a solicitação (liga validação, confirmação e execução)
├── turn_id        identifica o turno (rastreabilidade)
├── tool_id        ferramenta proposta
└── parameters     argumentos propostos
```

O `ToolRequest` é uma **proposta de execução**, e não a execução. Ele não carrega a identidade do usuário, o contexto conversacional nem o estado da tarefa: esses permanecem no Núcleo e nos gerenciadores. Depois de validada, a solicitação é imutável e recebe a marca de validada; a confirmação e a execução referem-se ao mesmo `request_id`.

### 4.3 Validação da solicitação

Antes da execução, o Orquestrador valida o **contrato** da proposta. A validação verifica, no mínimo, se:

1. a ferramenta existe no registro;
2. a ferramenta está disponível;
3. a ferramenta está autorizada para a sessão;
4. os parâmetros obrigatórios foram fornecidos;
5. não há parâmetros desconhecidos nem parâmetros de identidade do usuário;
6. os tipos e formatos dos parâmetros são válidos;
7. as restrições declaradas foram respeitadas.

O resultado é uma solicitação validada, com o indicador `requires_confirmation`, ou um `ToolResult` com status `VALIDATION_ERROR` e origem `CONTRATO`.

A validação é responsabilidade do software de controle e não depende da LLM: uma saída da LLM que pede uma ferramenta não é autorização para executá-la. Ela verifica a **forma** da chamada; as **regras de negócio** são verificadas pelo domínio. Que haja informação suficiente para propor a chamada é verificado antes, pelo Gerenciador de Tarefa.

### 4.4 Execução da ferramenta

Depois da validação, o Orquestrador executa a ferramenta com tempo-limite, por meio da Interface de Ferramentas e do Adaptador. Se a ferramenta exige confirmação, a execução só ocorre quando a confirmação estiver registrada para o mesmo `request_id` (seção 11). O fluxo completo está na seção 9.

### 4.5 Resultado da ferramenta

```text
ToolResult
├── request_id
├── status      SUCCESS | EMPTY | VALIDATION_ERROR | REJECTED |
│               EXECUTION_ERROR | INCONCLUSIVE
├── result      dados compactos produzidos (quando houver)
├── error       code, origin, recoverable, details (quando houver)
└── metadata    turn_id, tool_id, início, duração
```

- **`error.code`:** código estável do erro (por exemplo, CARENCIA\_EM\_VIGOR).
- **`error.origin`:** `CONTRATO` (Orquestrador), `DOMINIO` ou `INFRAESTRUTURA`.
- **`error.recoverable`:** indica se a tarefa pode prosseguir com uma nova informação do usuário.
- **`error.details`:** dados adicionais necessários para explicar o erro (por exemplo, a data em que a carência termina).

O domínio devolve **códigos e dados**, não texto de fala; a mensagem ao usuário é formulada pelo Núcleo.

### 4.6 Semântica dos resultados

A distinção entre os status evita que o assistente confunda situações diferentes (RF07 CA2, RF08).

| Status | Quando ocorre | Exemplo no estudo de caso | Categoria no estudo de caso |
| --- | --- | --- | --- |
| SUCCESS | A operação foi executada e produziu resultado, inclusive uma lista com vários itens. | Empréstimo registrado; busca com 2 livros. | Sucesso |
| EMPTY | A consulta foi executada, mas não encontrou dados. Não é falha. | Nenhum livro encontrado. | Vazio |
| VALIDATION\_ERROR | Argumentos inválidos ou referência inexistente. A origem é o contrato (Orquestrador) ou o domínio. | PARAMETROS\_INSUFICIENTES; LIVRO\_NAO\_ENCONTRADO. | Validação; Não encontrado |
| REJECTED | A chamada é válida, mas o domínio a recusou por regra de negócio. | LIMITE\_EMPRESTIMOS\_ATINGIDO; CARENCIA\_EM\_VIGOR. | Regra de negócio |
| EXECUTION\_ERROR | Falha técnica durante a execução. | SERVICO\_INDISPONIVEL. | Erro de execução |
| INCONCLUSIVE | Não foi possível determinar o resultado da operação. | RESULTADO\_INCERTO. | Inconclusivo |

Observações:

- Tempo-limite em uma **leitura** gera `EXECUTION_ERROR`. Tempo-limite em uma **escrita** gera `INCONCLUSIVE`, porque o efeito pode ter ocorrido (seção 10).
- Uma busca com vários resultados é `SUCCESS`. A ambiguidade é tratada pelo Núcleo (seção 3.2), e não pela ferramenta.

## 5. Responsabilidades: LLM, núcleo, controle e domínio

A arquitetura separa as capacidades da LLM das responsabilidades determinísticas do software.

| Responsabilidade | LLM | Núcleo (software) | Controle (Orquestrador e Registro) | Domínio |
| --- | :-: | :-: | :-: | :-: |
| Interpretar linguagem natural e identificar a intenção | ✓ |  |  |  |
| Extrair os parâmetros da fala | ✓ |  |  |  |
| Resolver referências e candidatos | ✓ | ✓ |  |  |
| Calcular os parâmetros obrigatórios que faltam |  | ✓ |  |  |
| Decidir o próximo passo (política de diálogo) |  | ✓ |  |  |
| Propor a ferramenta e os argumentos | ✓ |  |  |  |
| Validar o contrato e autorizar a ferramenta |  |  | ✓ |  |
| Aplicar a identidade da sessão |  |  | ✓ |  |
| Impor a confirmação das escritas |  | ✓ | ✓ |  |
| Executar a ferramenta |  |  | ✓ |  |
| Aplicar as regras de negócio |  |  |  | ✓ |
| Normalizar o resultado e classificar o status |  |  | ✓ |  |
| Verificar o estado após resultado inconclusivo |  | ✓ | ✓ |  |
| Redigir a resposta | ✓ |  |  |  |
| Garantir a fidelidade da resposta ao resultado |  | ✓ |  |  |
| Registrar eventos e falhas |  | ✓ | ✓ |  |

## 6. Registro e Interface de Ferramentas

Disponibiliza ao Orquestrador as ferramentas que podem ser usadas.

- manter um **registro fechado**: apenas as ferramentas registradas existem para o assistente;
- disponibilizar suas definições (`ToolDefinition`);
- identificar as ferramentas autorizadas para a sessão;
- oferecer uma interface uniforme de execução e ocultar os detalhes de cada ferramenta;
- permitir incluir ou substituir ferramentas sem alterar o Núcleo (RNF01).

**D-02 — Granularidade 1 para 1.** Cada operação do domínio é exposta como uma ferramenta, com uma camada de adaptação entre a API e a ferramenta. A decisão é reavaliada se os cenários mostrarem excesso de encadeamento. As ferramentas do estudo de caso são:

| Ferramenta | Efeito | Confirmação | Parâmetros |
| --- | --- | --- | --- |
| consultar\_livros | READ | Não | titulo, autor |
| consultar\_emprestimos | READ | Não | situacao |
| registrar\_emprestimo | WRITE | Sim | livro\_id |
| registrar\_devolucao | WRITE | Sim | emprestimo\_id |

## 7. Adaptador do Sistema de Domínio

Isola o restante da arquitetura da implementação concreta do sistema de informação.

- traduzir a chamada da ferramenta na operação concreta do domínio (a correspondência fica na configuração do adaptador, não no contrato do Núcleo);
- adaptar parâmetros e resultados, devolvendo um resultado **compacto** no formato de `ToolResult`;
- converter erros do domínio e da infraestrutura em códigos estáveis do catálogo de erros;
- repassar ao domínio a identidade da sessão fornecida pelo Orquestrador;
- **não** resolver nomes em identificadores, **não** aplicar regras de negócio e **não** repetir escritas automaticamente.

Por exemplo, a solicitação conceitual:

```text
registrar_emprestimo
livro_id = L03        (obtido pela consulta e mantido no contexto)
usuário  = sessão     (aplicado pelo Orquestrador; não vem da LLM)
```

é convertida pelo adaptador na operação concreta da API do sistema de biblioteca.

## 8. Sistema de Informação de Domínio

Responsável pelos dados e pelas regras de negócio do domínio. No estudo de caso, é o sistema de empréstimos da biblioteca (Especificação do Estudo de Caso, regras RN01 a RN08).

- armazenar os dados e executar as operações de negócio;
- aplicar as regras de negócio, inclusive o isolamento por usuário com base na identidade da sessão (RNF05, RN08);
- permitir fixar a data de referência e restaurar a base nos testes (RNF12).

As regras de negócio permanecem no domínio e não são reproduzidas pelo Núcleo nem pela LLM. Quando o domínio recusa uma operação, o assistente comunica o resultado efetivamente retornado.

## 9. Fluxo normal de utilização de ferramenta

```text
 1. Usuário fala
        ↓
 2. Interface: transcrição e confiança                    (RF01)
        ↓   [sem fala ou ininteligível → pedir repetição (RF13)]
 3. Núcleo interpreta: intenção e parâmetros              (RF02)
        ↓
 4. Gerenciador de Tarefa atualiza o estado e calcula
    os parâmetros obrigatórios que faltam                 (RF04)
        ↓   [falta informação → pedir e encerrar o turno]
        ↓   [vários candidatos → apresentar e pedir escolha]
 5. Núcleo propõe o ToolRequest                           (RF05, RF06)
        ↓
 6. Orquestrador valida o contrato e aplica a identidade da sessão
        ↓
 7. Escrita? → confirmação do usuário (seção 11)          (RF10)
        ↓
 8. Orquestrador executa → ToolResult                     (RF07)
        ↓
 9. Núcleo trata o resultado conforme o status (seção 10) (RF08, RF09)
        ↓
10. Resposta gerada sob a guarda de fidelidade (seção 16) (RF11)
        ↓
11. Interface sintetiza e reproduz a voz                  (RF12)
```

Cada etapa gera eventos no Registro de Eventos (seção 14).

Exemplo do estudo de caso (cenário CC-04):

```text
Usuário: Quero pegar Dom Casmurro emprestado.
  Núcleo → consultar_livros(titulo = Dom Casmurro)     SUCCESS, 1 livro (L03)
  Núcleo → registrar_emprestimo(livro_id = L03)        proposta
  Orquestrador valida; ferramenta de escrita → requer confirmação
Assistente: Posso registrar o empréstimo de Dom Casmurro, de Machado de Assis?
Usuário: Sim.
  Orquestrador executa a mesma solicitação validada    SUCCESS
Assistente: Pronto. A devolução é até 23 de novembro.
```

Se faltar informação, o fluxo é interrompido antes da execução: o Núcleo pergunta, o usuário complementa e a tarefa é retomada (RF04 CA4).

## 10. Falha e recuperação

O tratamento depende do status do `ToolResult`; o estado resultante da tarefa está na seção 12:

| Status | Tratamento | Requisitos e cenários |
| --- | --- | --- |
| SUCCESS | Usar os dados na resposta; se houver vários candidatos, tratar a ambiguidade. | RF08 CA1 e CA4; CC-01, CC-07 |
| EMPTY | Informar que nada foi encontrado e sugerir reformular. Não é falha. | RF08 CA2 |
| VALIDATION\_ERROR | Pedir a informação correta ou propor nova busca, mantendo o que foi coletado. | RF09 CA1; CC-05 |
| REJECTED | Explicar o motivo com o código e os dados retornados, sem repetir a chamada. | RF08 CA3; CC-12 a CC-14 |
| EXECUTION\_ERROR | Informar o problema sem afirmar que a operação foi realizada e oferecer nova tentativa. | RF09 CA2; CC-21 |
| INCONCLUSIVE | Verificar o estado por consulta antes de responder. | RF09 CA3; CC-22 |

**Verificação de estado após resultado inconclusivo**

- A política de diálogo pede ao Orquestrador uma consulta de leitura registrada que revele se o efeito ocorreu (no estudo de caso, a consulta de empréstimos). O resultado é um novo `ToolResult`.
- Se a consulta mostra o efeito, o assistente informa o resultado real. Se mostra que não ocorreu, informa que a operação não foi concluída e oferece nova tentativa, que reabre a confirmação. Se a verificação também falha, informa que não pode confirmar o resultado.
- Escritas nunca são repetidas automaticamente. Repetir uma escrita já efetivada retorna um erro de regra (por exemplo, EMPRESTIMO\_ATIVO\_MESMO\_LIVRO), sem duplicar o efeito.

Falhas dos serviços de voz e da LLM são tratadas pela Interface e pelo Núcleo com tempo-limite e mensagem controlada (RNF09). A recuperação **não deve mascarar uma falha efetivamente ocorrida**.

## 11. Confirmação

```text
Núcleo propõe o ToolRequest
        ↓
Orquestrador valida → solicitação validada (request_id)
        ↓
requires_confirmation?
   ├─ não → executar
   └─ sim → Gerenciador de Tarefa guarda a solicitação (AGUARDANDO_CONFIRMACAO)
            Núcleo pede a confirmação descrevendo a ação e a entidade
            ↓
            resposta do usuário (tabela abaixo)
```

| Resposta do usuário | Resultado |
| --- | --- |
| Confirma (“sim”) | Executa exatamente a solicitação validada e guardada (mesmo `request_id`), sem gerar novos argumentos. |
| Nega (“não”) | Cancela a operação; nada é executado. |
| Corrige (“não, o outro livro”) | Volta à coleta, atualiza o parâmetro, valida de novo e pede nova confirmação, sem reiniciar a tarefa. |
| Cancela (“esquece”) | Cancela a operação pendente. |
| Não compreendida | Repete o pedido de confirmação (RF13). |

Regras:

- Um “sim” só é confirmação quando há confirmação pendente (RF02 CA3; CC-20).
- O pedido é redigido a partir da solicitação e do contexto (título e autor), de modo que o usuário confirma exatamente o que será executado.
- A confirmação é **imposta pelo software**: o Orquestrador não executa uma escrita sem confirmação registrada para o mesmo `request_id`. Uma nova tentativa após falha reabre a confirmação.
- O cancelamento só é possível antes da execução. Depois dela, desfazer exige uma nova operação (por exemplo, devolver o livro), conforme RF10 CA5.

Cenários: CC-04, CC-09, CC-10, CC-11 e CC-20.

## 12. Estado da tarefa

| Estado | Significado |
| --- | --- |
| EM\_COLETA | Objetivo identificado; coletando parâmetros ou resolvendo uma ambiguidade. |
| AGUARDANDO\_CONFIRMACAO | Solicitação validada de escrita guardada, aguardando a resposta do usuário. |
| EXECUTANDO | Chamada em execução. |
| VERIFICANDO | Resultado inconclusivo; verificando o estado por consulta. |
| CONCLUIDA | Operação concluída com resultado (terminal). |
| RECUSADA | Operação recusada por regra de negócio (terminal). |
| CANCELADA | Cancelada pelo usuário antes da execução, ou por desistência (terminal). |
| FALHA | Falha de execução ou efeito não confirmado; aguarda a decisão do usuário (nova tentativa ou desistência). |

| De | Evento | Para |
| --- | --- | --- |
| (início) | Intenção identificada | EM\_COLETA |
| EM\_COLETA | Solicitação de leitura validada | EXECUTANDO |
| EM\_COLETA | Solicitação de escrita validada | AGUARDANDO\_CONFIRMACAO |
| AGUARDANDO\_CONFIRMACAO | Confirmação positiva | EXECUTANDO |
| AGUARDANDO\_CONFIRMACAO | Correção do usuário | EM\_COLETA |
| AGUARDANDO\_CONFIRMACAO | Negação ou cancelamento | CANCELADA |
| EXECUTANDO | SUCCESS ou EMPTY | CONCLUIDA (EM\_COLETA se houver vários candidatos) |
| EXECUTANDO | VALIDATION\_ERROR | EM\_COLETA |
| EXECUTANDO | REJECTED | RECUSADA |
| EXECUTANDO | EXECUTION\_ERROR | FALHA |
| EXECUTANDO | INCONCLUSIVE | VERIFICANDO |
| VERIFICANDO | Efeito confirmado | CONCLUIDA |
| VERIFICANDO | Efeito não ocorreu ou verificação impossível | FALHA |
| FALHA | Nova tentativa pedida pelo usuário | AGUARDANDO\_CONFIRMACAO (escrita) ou EXECUTANDO (leitura) |
| FALHA | Desistência | CANCELADA |

O estado da tarefa é preservado em todas as transições, o que permite retomar sem reiniciar (RF09 CA5). Os estados terminais encerram a tarefa, mas o contexto conversacional permanece na sessão, permitindo continuidade (por exemplo, “esse”).

## 13. Isolamento do domínio

O Núcleo Conversacional não depende das estruturas internas do domínio: a dependência segue o caminho Núcleo → Orquestrador → Interface de Ferramentas → Adaptador → Domínio (seção 2), e nunca o acesso direto a API, banco ou classes do domínio. Isso favorece a reutilização do núcleo, a substituição do domínio, os testes isolados e a evolução independente dos componentes. Verificação (RNF01): inspeção das dependências do núcleo e execução do núcleo com um conjunto alternativo de ferramentas de teste, sem alterar seu código.

## 14. Observabilidade e rastreabilidade

O **Registro de Eventos** é um componente transversal. Recebe eventos estruturados de todos os componentes, com campos comuns: `turn_id`, `task_id`, `request_id` (quando houver), tipo do evento, componente, marca de tempo, status e duração.

Entre os eventos registrados estão:

- fala transcrita (com a confiança) e interpretação produzida;
- informações ausentes, candidatos apresentados e confirmação pedida;
- proposta de chamada, resultado da validação e execução (`ToolResult`: status, código de erro, duração);
- verificação de estado e transições do estado da tarefa;
- resposta produzida e falhas dos serviços de voz e da LLM.

As marcas de tempo permitem medir a latência de turno (do fim da fala ao início do áudio) e a duração de cada etapa: reconhecimento, interpretação, validação, execução, redação e síntese (RNF07). O áudio bruto não é registrado, e os registros usam o identificador da sessão e dados fictícios (RNF10).

A partir dos registros de um cenário deve ser possível reconstruir a sequência de eventos de cada turno e relacioná-la aos requisitos e cenários (RNF03). O mecanismo concreto de observabilidade permanece fora do escopo deste modelo lógico.

## 15. Segurança e controle de acesso

- **Registro fechado e identidade fora do modelo.** Ferramentas não registradas são rejeitadas (RF05 CA1). A identidade vem da sessão, e qualquer identidade gerada pela LLM é descartada ou rejeitada (RF06 CA4, RNF06).
- **Autorização em dois níveis.** De ferramenta, pelo Orquestrador e pelo Registro, e de dado, pelo domínio (cada usuário só acessa os próprios dados, RN08). Consultar dados de outro usuário retorna não encontrado (CT-13, CC-19).
- **Conteúdo não confiável.** A fala do usuário e os dados retornados por ferramentas são tratados como dados: instruções contidas neles não alteram o registro de ferramentas, as políticas nem a identidade.
- **Credenciais e chaves** ficam fora do código-fonte e do registro de eventos (RNF10).

## 16. Fidelidade da resposta

O princípio P7 exige que a resposta represente o resultado efetivamente produzido (RF11 CA3, RNF04). Para isso:

1. A política de diálogo define o **tipo de resposta** a partir do status do `ToolResult` e do estado da tarefa (resultado, recusa, falha, pedido de informação, pedido de confirmação ou encerramento). A LLM redige dentro desse tipo.
2. A redação recebe os **fatos** (status, dados e códigos retornados) e não cita dados que não estejam neles (RF11 CA2).
3. Uma resposta que afirma a realização de uma escrita só é emitida quando o status é SUCCESS ou quando a verificação de estado confirmou o efeito. Em INCONCLUSIVE e EXECUTION\_ERROR, nenhuma resposta afirma que a operação foi realizada.
4. Uma verificação após a redação confere essa regra. Se falhar, a resposta é substituída por uma redação padrão segura para o tipo de resposta.

Cenários de verificação: CC-12 a CC-14, CC-21, CC-22 e CT-08 a CT-11, CT-17, CT-18.

## 17. Princípios arquiteturais

| Princípio | Descrição | Requisitos |
| --- | --- | --- |
| P1 — Separação entre interação e domínio | A lógica conversacional permanece separada das regras de negócio. | RNF01, RNF05 |
| P2 — Desacoplamento do núcleo | O núcleo não depende de um domínio específico. | RNF01 |
| P3 — Ferramentas como fronteira de integração | Operações externas só são acessadas por ferramentas explicitamente disponibilizadas. | RF05, RF07, RNF06 |
| P4 — Controle da execução | Gerar uma chamada na LLM não implica executá-la. | RF06, RF07 |
| P5 — Validação antes da execução | As solicitações são verificadas antes de alcançar o domínio. | RF06 |
| P6 — Preservação das regras de negócio | As regras permanecem sob responsabilidade do domínio. | RNF05 |
| P7 — Fidelidade ao resultado | O assistente representa o resultado efetivamente produzido, distinguindo sucesso, ausência de dados, recusa, falha e resultado inconclusivo. | RF08, RF11, RNF04 |
| P8 — Recuperação da interação | Falhas e informações insuficientes permitem recuperação sempre que possível. | RF09 |
| P9 — Observabilidade | O comportamento é observável o bastante para permitir verificação e rastreabilidade. | RNF03, RNF07 |
| P10 — Independência tecnológica | O modelo lógico não depende de protocolo, framework ou fornecedor; os provedores são acessados por portas. | RNF02 |
| P11 — Identidade fora do modelo (novo) | A identidade do usuário vem da sessão e é aplicada pelo software. | RF06, RNF06 |
| P12 — Confirmação imposta por software (novo) | Escritas exigem confirmação ligada à solicitação validada. | RF10 |

## 18. Decisões de arquitetura

| ID | Decisão | Justificativa |
| --- | --- | --- |
| AD-01 | A LLM propõe a chamada; o Orquestrador verifica, autoriza e executa. | Elimina a ambiguidade de papéis da v1.2 e sustenta o P4. |
| AD-02 | Granularidade 1 para 1 entre operações e ferramentas, com camada de adaptação (D-02 do estudo de caso). | Simples de explicar, rastrear e testar; reavaliada se houver excesso de encadeamento. |
| AD-03 | Novo status REJECTED e erro estruturado no `ToolResult`. | Distingue recusa por regra de negócio de erro de parâmetro e de falha técnica (RF07 CA2). |
| AD-04 | Os parâmetros faltantes são calculados de forma determinística pelo Gerenciador de Tarefa. | Evita depender da LLM para decidir o que falta (RF04 CA2). |
| AD-05 | O resultado inconclusivo de uma escrita é verificado por consulta antes de responder; escritas não são repetidas automaticamente. | Evita afirmar sucesso ou falha sem saber, e evita duplicações (RF09 CA3). |
| AD-06 | A resolução de entidades e a desambiguação ficam no Núcleo; o adaptador não converte nomes em identificadores. | A ambiguidade fica visível ao diálogo (RF04 CA3). |
| AD-07 | O contexto de sessão é separado do contexto conversacional. | Identidade e permissões não podem ser alteradas pela conversa (P11). |

A identidade fora do modelo, a confirmação imposta por software e as portas de provedores são princípios (P10 a P12) e não são repetidos aqui.

## 19. Decisões ainda em aberto

As decisões abaixo ficam para o projeto detalhado e serão tomadas considerando os requisitos, os casos de uso, a literatura levantada e as restrições do protótipo:

- tecnologia de reconhecimento de fala e de síntese de voz, e modelo de LLM;
- mecanismo de tool calling (nativo da LLM, MCP ou camada própria) e protocolo de comunicação entre componentes;
- armazenamento do contexto e implementação do Gerenciador de Tarefa;
- formato concreto de `ToolDefinition`, `ToolRequest` e `ToolResult`;
- mecanismo de autorização das ferramentas e de observabilidade;
- valores de tempo-limite por chamada, limite de confiança do reconhecimento e tempo-limite de silêncio;
- comportamento quando o usuário muda de assunto durante uma tarefa pendente (retomar, substituir ou pedir esclarecimento);
- verificação prévia de limite e carência antes de pedir a confirmação (nos cenários CC-12 e CC-13 a recusa só ocorre após a confirmação; a consulta de empréstimos já devolve esses dados, ao custo de uma chamada a mais);
- implementação do sistema de domínio (API simples do estudo de caso).

## 20. Rastreabilidade

### 20.1 Requisitos funcionais × componentes

| Requisito | Componentes principais | Seções |
| --- | --- | --- |
| RF01 — Receber e transcrever a voz | Interface (porta de reconhecimento de fala), Registro de Eventos | 3.1, 14 |
| RF02 — Interpretar a solicitação | Núcleo (interpretação), Gerenciador de Tarefa (estado para o “sim”) | 3.2, 3.4 |
| RF03 — Contexto conversacional | Gerenciador de Contexto, Núcleo | 3.3 |
| RF04 — Informações da tarefa | Gerenciador de Tarefa (faltantes), Núcleo (ambiguidade), Gerenciador de Contexto (candidatos) | 3.2 a 3.4 |
| RF05 — Selecionar a ferramenta | Núcleo (proposta), Registro (registro fechado), Orquestrador (verificação) | 3.2, 3.5, 6 |
| RF06 — Preparar e validar a chamada | Núcleo (argumentos), Orquestrador (validação e identidade) | 4.2, 4.3 |
| RF07 — Executar ferramentas | Orquestrador, Interface de Ferramentas, Adaptador | 3.5, 4.4, 7 |
| RF08 — Processar resultados | Núcleo (política de diálogo) | 4.6, 10 |
| RF09 — Falhas e recuperação | Núcleo, Gerenciador de Tarefa, Orquestrador (verificação) | 10, 12 |
| RF10 — Controle da interação | Núcleo (política), Gerenciador de Tarefa, Orquestrador (imposição), Interface (encerramento) | 11, 12 |
| RF11 — Resposta textual | Núcleo (geração e guarda de fidelidade) | 3.2, 16 |
| RF12 — Resposta por voz | Interface (porta de síntese de voz) | 3.1 |
| RF13 — Incerteza do reconhecimento | Interface (qualidade e confiança), Núcleo (política) | 3.1, 3.2 |
| RF14 — Controle da saída de voz | Interface (reprodução, interrupção e silêncio), Núcleo (repetir) | 3.1, 3.3 |

### 20.2 Requisitos não funcionais × arquitetura

| Requisito | Onde é atendido |
| --- | --- |
| RNF01 — Desacoplamento do domínio | Seções 4.1 (confirmação por configuração), 6 e 13 |
| RNF02 — Substituição de provedores | Portas da Interface e do Núcleo (3.1, 3.2); P10 |
| RNF03 — Observabilidade | Seção 14 |
| RNF04 — Fidelidade ao resultado | Seções 4.6 e 16 |
| RNF05 — Regras no domínio | Seções 4.3 e 8 |
| RNF06 — Restrição de acesso | Seções 6 e 15 |
| RNF07 — Desempenho | Seção 14 (latência por etapa) |
| RNF08 — Concisão das respostas | Seções 3.2 (geração) e 4.1 (`result_schema`) |
| RNF09 — Resiliência a falhas externas | Seções 3.1, 4.6 e 10 |
| RNF10 — Privacidade | Seções 3.1, 14 e 15 |
| RNF11 — Idioma | Portas de fala em português do Brasil (3.1) |
| RNF12 — Testabilidade | Porta de linguagem simulável (3.2), data de referência (8) e seção 14 |

## 21. Estado atual do modelo

Esta versão define a **arquitetura lógica de referência**. O ponto central é a fronteira explícita entre o raciocínio conversacional, que propõe, e a execução, controlada por software no Orquestrador, que chega ao domínio por ferramentas (seção 4). A escolha entre tool calling nativo da LLM, MCP ou camada própria ocorre no projeto detalhado, comparando suas características com os requisitos e com o contrato lógico.

## 22. Alterações em relação à versão 1.2

| # | Alteração | Seções |
| --- | --- | --- |
| 1 | Papéis esclarecidos: a LLM propõe o `ToolRequest`; o Orquestrador valida, autoriza e executa, sem selecionar a ferramenta nem gerar argumentos. | 3.5, 4.4, 18 |
| 2 | Contrato revisado: novo status REJECTED e erro estruturado; `ToolDefinition` com `side_effect`, `requires_confirmation`, `result_schema`, `error_catalog` e `timeout`; `ToolRequest` com `request_id` e `turn_id`, sem `operation`, `relevant_context` e `task_requirements`. | 4.1 a 4.6 |
| 3 | Identidade no contexto de sessão, aplicada pelo Orquestrador; resolução de entidades no Núcleo; adaptador sem conversão de nomes em identificadores. | 3.2, 3.3, 7, 15 |
| 4 | Confirmação imposta por software e vinculada ao `request_id`; ramo de correção e semântica de cancelamento. | 3.5, 11 |
| 5 | Gerenciador de Tarefa com cálculo determinístico dos parâmetros faltantes e máquina de estados de 8 estados; verificação de estado após resultado inconclusivo. | 3.4, 10, 12 |
| 6 | Núcleo dividido em interpretação, política de diálogo e geração de resposta; portas de fala e de linguagem. | 2, 3.1, 3.2 |
| 7 | Registro de Eventos transversal, com identificadores de turno, tarefa e solicitação. | 14 |
| 8 | Novas seções de segurança, fidelidade da resposta, decisões e rastreabilidade com RF e RNF; princípios P11 e P12; granularidade 1 para 1 registrada. | 6, 15 a 20 |

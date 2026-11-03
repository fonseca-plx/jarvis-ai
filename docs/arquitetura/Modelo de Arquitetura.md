# Modelo de Arquitetura do Sistema

**Projeto:** Assistente Conversacional por Voz com Uso de Ferramentas  
**Versão:** 1.2  
**Finalidade:** Definir a estrutura lógica do sistema, as responsabilidades de seus componentes, suas interfaces e o contrato de interação com ferramentas externas ao núcleo conversacional.

---

## 1. Objetivo

O modelo de arquitetura define a organização lógica do assistente conversacional baseado em voz e LLM, estabelecendo os principais componentes do sistema, suas responsabilidades, interfaces e relações.

A arquitetura deve permitir que o núcleo conversacional seja desacoplado do domínio de aplicação, possibilitando que diferentes sistemas de informação sejam integrados por meio de ferramentas explicitamente disponibilizadas ao assistente.

Neste modelo, a LLM é utilizada como parte do mecanismo de interpretação e geração de linguagem, mas não possui acesso direto ao sistema de informação de domínio. O acesso a operações externas ocorre por meio de uma camada controlada de ferramentas.

A arquitetura permanece, nesta versão, **independente de tecnologias específicas de implementação**.

---

# 2. Visão geral da arquitetura

A arquitetura lógica é composta pelos seguintes elementos:

1. **Interface Conversacional**
2. **Núcleo Conversacional**
3. **Gerenciador de Contexto**
4. **Gerenciador de Tarefa**
5. **Orquestrador de Ferramentas**
6. **Registro e Interface de Ferramentas**
7. **Adaptador do Sistema de Domínio**
8. **Sistema de Informação de Domínio**

A organização geral pode ser representada da seguinte forma:

```text
┌──────────────────────────────────────────────┐
│             Interface Conversacional         │
│          Entrada e saída por voz             │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│              Núcleo Conversacional           │
│                                              │
│ Interpretação • Intenção • Contexto •        │
│ Informações • Ambiguidade • Resposta         │
└───────┬──────────────────────────┬───────────┘
        │                          │
        ▼                          ▼
┌───────────────┐          ┌───────────────────┐
│ Gerenciador   │          │ Gerenciador de    │
│ de Contexto   │          │ Tarefa            │
└───────────────┘          └───────────────────┘
                                     
                       │
                       ▼
             ┌───────────────────────┐
             │ Orquestrador de       │
             │ Ferramentas           │
             └───────────┬───────────┘
                         │
                         ▼
             ┌───────────────────────┐
             │ Registro / Interface  │
             │ de Ferramentas        │
             └───────────┬───────────┘
                         │
                         ▼
             ┌───────────────────────┐
             │ Adaptador do Sistema  │
             │ de Domínio            │
             └───────────┬───────────┘
                         │
                         ▼
             ┌───────────────────────┐
             │ Sistema de Informação │
             │ de Domínio            │
             └───────────────────────┘
```

O **Gerenciador de Contexto** e o **Gerenciador de Tarefa** são apresentados como responsabilidades lógicas distintas. Isso não implica que devam necessariamente constituir componentes físicos independentes na implementação.

---

# 3. Componentes e responsabilidades

## 3.1 Interface Conversacional

Responsável pela interação direta com o usuário.

### Responsabilidades

- receber a entrada de voz;
- encaminhar a entrada para processamento conversacional;
- receber a resposta textual produzida pelo sistema;
- realizar a conversão da resposta para voz;
- reproduzir a resposta ao usuário;
- controlar aspectos da interação, como interrupção, cancelamento e encerramento, quando aplicável.

A interface não deve conter regras específicas do domínio.

---

## 3.2 Núcleo Conversacional

Constitui o principal componente responsável pelo processamento da interação em linguagem natural.

### Responsabilidades

- interpretar a solicitação do usuário;
- identificar intenção e informações relevantes;
- utilizar informações do contexto conversacional;
- identificar informações ausentes;
- solicitar informações adicionais;
- identificar ambiguidades;
- solicitar esclarecimentos;
- determinar a operação necessária para atingir o objetivo da interação;
- solicitar a execução de ferramentas;
- interpretar os resultados retornados;
- determinar o próximo passo da interação;
- produzir a resposta textual ao usuário.

A LLM pode participar dessas atividades, especialmente na interpretação, geração de argumentos, interpretação de resultados e geração da resposta.

Entretanto, operações críticas de controle — como validação, autorização e execução de ferramentas — não devem depender exclusivamente da saída da LLM.

---

## 3.3 Gerenciador de Contexto

Responsável pelo armazenamento, recuperação e atualização das informações relevantes provenientes da interação.

### Responsabilidades

- armazenar informações relevantes da conversa;
- recuperar informações previamente fornecidas;
- disponibilizar contexto ao Núcleo Conversacional;
- atualizar o contexto após novos turnos;
- apoiar a resolução de referências contextuais;
- evitar que informações relevantes sejam solicitadas novamente quando já estiverem disponíveis e forem suficientemente determinadas.

Exemplos de informações de contexto incluem:

- recurso mencionado anteriormente;
- usuário envolvido na operação;
- parâmetros já informados;
- resultado de uma consulta anterior;
- referência a entidades mencionadas anteriormente.

---

## 3.4 Gerenciador de Tarefa

Responsável pelo estado da tarefa atualmente executada.

### Responsabilidades

- registrar o objetivo atual da interação;
- controlar o progresso da tarefa;
- registrar informações já obtidas;
- identificar informações ainda necessárias;
- registrar operações pendentes;
- permitir correção de informações;
- controlar transições entre estados da tarefa;
- determinar quando uma tarefa foi concluída, cancelada ou interrompida.

O Gerenciador de Tarefa permite distinguir o **contexto da conversa** do **estado da tarefa em execução**.

---

## 3.5 Orquestrador de Ferramentas

É o componente responsável por controlar o ciclo de utilização das ferramentas.

### Responsabilidades

- determinar quando uma ferramenta é necessária;
- selecionar uma ferramenta compatível com a operação;
- preparar os argumentos;
- validar os argumentos;
- verificar se a ferramenta está autorizada;
- executar a ferramenta;
- receber o resultado;
- normalizar o resultado;
- comunicar falhas;
- devolver o resultado ao Núcleo Conversacional.

O Orquestrador constitui uma importante fronteira de controle entre o processamento baseado em LLM e a execução efetiva de operações externas.

A LLM pode **propor** uma ferramenta e seus argumentos, mas a execução deve ocorrer sob o controle do software responsável pela orquestração.

---

# 4. Contrato lógico de ferramentas

O contrato lógico de ferramentas define a forma abstrata pela qual o sistema disponibiliza operações externas ao Núcleo Conversacional.

O objetivo desse contrato é separar:

- a **descrição de uma capacidade disponível**;
- a **solicitação de execução dessa capacidade**;
- a **validação da solicitação**;
- a **execução efetiva**;
- o **resultado da execução**.

O contrato não representa, nesta etapa, uma tecnologia ou protocolo específico. Ele define uma abstração arquitetural que poderá posteriormente ser implementada por diferentes mecanismos de tool calling ou protocolos de integração.

---

## 4.1 Definição da ferramenta

Cada ferramenta disponibilizada ao assistente deve possuir uma definição lógica que permita ao sistema identificar sua finalidade e os dados necessários para sua execução.

Uma definição conceitual pode ser representada por:

```text
ToolDefinition
├── id
├── name
├── description
├── operation
├── parameters
├── required_parameters
└── constraints
```

### `id`

Identificador único da ferramenta.

### `name`

Nome utilizado para identificar a operação.

### `description`

Descrição da capacidade oferecida pela ferramenta.

A descrição deve ser suficiente para permitir que o mecanismo conversacional compreenda quando a ferramenta é aplicável.

### `operation`

Identificação da operação de domínio representada pela ferramenta.

### `parameters`

Conjunto de parâmetros que podem ser fornecidos à ferramenta.

### `required_parameters`

Parâmetros obrigatórios para execução.

### `constraints`

Restrições aplicáveis aos parâmetros ou à utilização da ferramenta.

As restrições podem representar, por exemplo:

- formato esperado;
- valores permitidos;
- dependências entre parâmetros;
- condições necessárias para execução.

---

## 4.2 Solicitação de execução

Quando o Núcleo Conversacional determinar que uma operação externa é necessária, deve ser produzida uma solicitação lógica de execução.

Conceitualmente:

```text
ToolRequest
├── tool_id
├── operation
├── parameters
├── relevant_context
└── task_requirements
```

### `tool_id`

Identificação da ferramenta selecionada.

### `operation`

Operação que se pretende realizar.

### `parameters`

Argumentos determinados para a execução.

### `relevant_context`

Informações contextuais necessárias para compreender a solicitação.

### `task_requirements`

Informações relacionadas ao estado da tarefa que sejam necessárias para a execução ou validação.

O `ToolRequest` representa uma **intenção de execução**, e não a execução em si.

---

## 4.3 Validação da solicitação

Antes da execução, o Orquestrador deve validar a solicitação.

A validação deve verificar, no mínimo:

1. se a ferramenta existe;
2. se a ferramenta está disponível;
3. se a ferramenta está autorizada;
4. se os parâmetros obrigatórios foram fornecidos;
5. se os parâmetros correspondem à definição da ferramenta;
6. se os parâmetros possuem formato válido;
7. se as restrições conhecidas foram respeitadas;
8. se existem informações suficientes para realizar a operação.

A validação constitui uma responsabilidade do software de controle e não deve depender exclusivamente da LLM.

Consequentemente, uma saída da LLM solicitando uma ferramenta não deve ser interpretada automaticamente como autorização para executá-la.

---

## 4.4 Execução da ferramenta

Após a validação, o Orquestrador encaminha a solicitação à interface de ferramentas.

O fluxo lógico é:

```text
Núcleo Conversacional
        │
        │ ToolRequest
        ▼
Orquestrador
        │
        ├── verificar ferramenta
        ├── preparar argumentos
        ├── validar
        └── autorizar
        │
        ▼
Interface de Ferramentas
        │
        ▼
Adaptador de Domínio
        │
        ▼
Sistema de Informação
```

O Núcleo Conversacional não deve executar diretamente operações no Sistema de Informação de Domínio.

Essa separação permite que o núcleo permaneça independente da implementação concreta do domínio.

---

## 4.5 Resultado da ferramenta

O resultado da execução deve retornar ao Orquestrador em uma estrutura lógica que diferencie resultado de negócio, falha de execução e resultado inconclusivo.

Conceitualmente:

```text
ToolResult
├── status
├── result
├── error
└── metadata
```

### `status`

Indica o estado da execução.

Nesta arquitetura são considerados, conceitualmente, os seguintes estados:

```text
SUCCESS
EMPTY
VALIDATION_ERROR
EXECUTION_ERROR
INCONCLUSIVE
```

### `result`

Resultado efetivamente produzido pela ferramenta, quando existente.

### `error`

Informações sobre uma falha, quando aplicável.

### `metadata`

Informações auxiliares relevantes para processamento, observabilidade ou rastreabilidade.

---

## 4.6 Semântica dos resultados

A distinção entre os estados do resultado é importante para evitar que o assistente confunda situações diferentes.

### `SUCCESS`

A operação foi executada com sucesso e produziu um resultado.

### `EMPTY`

A operação foi executada, mas não encontrou dados correspondentes à solicitação.

Exemplo:

```text
Consulta executada → nenhum livro correspondente encontrado.
```

Isso é diferente de uma falha de execução.

### `VALIDATION_ERROR`

A solicitação não pôde ser executada porque os parâmetros ou condições da chamada não satisfaziam os requisitos conhecidos.

### `EXECUTION_ERROR`

A ferramenta foi acionada, mas ocorreu uma falha durante sua execução.

### `INCONCLUSIVE`

A execução não produziu informação suficientemente confiável para determinar o resultado da operação.

Essa categoria é particularmente importante para impedir que o Núcleo Conversacional produza uma resposta afirmativa quando o sistema não conseguiu determinar efetivamente o resultado da operação.

---

# 5. Responsabilidade da LLM e da camada de controle

A arquitetura estabelece uma separação conceitual entre as capacidades da LLM e as responsabilidades determinísticas do software.

| Responsabilidade | LLM / Núcleo | Camada de controle |
|---|---:|---:|
| Interpretar linguagem natural | ✓ | |
| Identificar intenção | ✓ | |
| Identificar informações relevantes | ✓ | |
| Sugerir ferramenta | ✓ | |
| Gerar argumentos | ✓ | |
| Interpretar resultado | ✓ | |
| Gerar resposta natural | ✓ | |
| Validar ferramenta existente | | ✓ |
| Validar parâmetros | | ✓ |
| Verificar autorização | | ✓ |
| Executar ferramenta | | ✓ |
| Controlar acesso ao domínio | | ✓ |
| Preservar regras de negócio | | ✓ |
| Garantir que a execução ocorreu | | ✓ |
| Registrar execução e falhas | | ✓ |

Essa separação não significa que toda funcionalidade relacionada à linguagem precise estar necessariamente dentro da LLM. Ela representa a divisão arquitetural desejada entre **interpretação/generation** e **controle efetivo da execução**.

---

# 6. Registro e Interface de Ferramentas

O Registro e Interface de Ferramentas representa a camada responsável por disponibilizar ao Orquestrador as ferramentas que podem ser utilizadas.

### Responsabilidades

- registrar ferramentas disponíveis;
- disponibilizar suas definições;
- identificar ferramentas autorizadas;
- fornecer uma interface uniforme para execução;
- ocultar detalhes específicos da implementação das ferramentas;
- permitir a inclusão ou substituição de ferramentas sem alteração do Núcleo Conversacional.

Essa camada deve funcionar como uma fronteira controlada de integração.

A quantidade e granularidade das ferramentas permanecem como decisões de arquitetura e implementação.

Por exemplo, o domínio da biblioteca poderia disponibilizar ferramentas como:

```text
consultar_informacao
verificar_disponibilidade
realizar_emprestimo
realizar_devolucao
consultar_emprestimos
```

ou poderia utilizar operações mais abrangentes.

A escolha entre essas alternativas não é determinada pelos casos de uso e deverá ser avaliada posteriormente considerando coesão, segurança, clareza das interfaces, complexidade e capacidade do mecanismo de tool calling adotado.

---

# 7. Adaptador do Sistema de Domínio

O Adaptador do Sistema de Domínio isola o restante da arquitetura da implementação concreta do sistema de informação.

### Responsabilidades

- traduzir chamadas de ferramentas em operações do domínio;
- adaptar parâmetros;
- adaptar resultados;
- tratar diferenças entre a interface da ferramenta e a API ou interface interna do domínio;
- preservar as regras de negócio do sistema de domínio;
- impedir que detalhes da implementação do domínio vazem para o Núcleo Conversacional.

Assim, uma solicitação conceitual como:

```text
realizar_emprestimo
livro = "Clean Code"
usuario = "Pedro"
```

pode ser convertida pelo adaptador para a operação concreta existente no sistema de informação.

O Núcleo Conversacional não precisa conhecer essa implementação.

---

# 8. Sistema de Informação de Domínio

É responsável pelos dados e pelas regras de negócio específicas do domínio da aplicação.

No estudo de caso do TCC, corresponde ao sistema responsável pelas operações relacionadas à biblioteca.

### Responsabilidades

- armazenar dados do domínio;
- executar operações de negócio;
- aplicar regras de negócio;
- determinar se uma operação é permitida;
- produzir os resultados das operações.

As regras de negócio devem permanecer no sistema de domínio, e não ser reproduzidas pelo Núcleo Conversacional ou pela LLM.

Por exemplo, uma operação de empréstimo pode ser recusada pelo sistema de domínio devido a uma regra da biblioteca. Nesse caso, o assistente deve comunicar o resultado efetivamente retornado pelo domínio.

---

# 9. Fluxo normal de utilização de ferramenta

O fluxo completo de uma solicitação que exige uma ferramenta é:

```text
1. Usuário realiza solicitação
        ↓
2. Interface recebe entrada
        ↓
3. Núcleo interpreta solicitação
        ↓
4. Contexto e estado da tarefa são consultados
        ↓
5. Informações necessárias são determinadas
        ↓
6. Operação necessária é determinada
        ↓
7. Ferramenta compatível é selecionada
        ↓
8. Argumentos são preparados
        ↓
9. Solicitação é validada
        ↓
10. Ferramenta é executada
        ↓
11. Resultado é recebido
        ↓
12. Resultado é normalizado
        ↓
13. Núcleo interpreta resultado
        ↓
14. Resposta é gerada
        ↓
15. Resposta é convertida para voz
        ↓
16. Usuário recebe resposta
```

Caso informações necessárias estejam ausentes, o fluxo é interrompido antes da execução:

```text
Solicitação
    ↓
Informações necessárias
    ↓
[Informação ausente]
    ↓
Solicitar informação
    ↓
Usuário complementa
    ↓
Retomar tarefa
```

Isso concretiza o princípio de que o sistema não deve executar uma operação quando não possui informações suficientes para determinar adequadamente a ação.

---

# 10. Fluxo de falha e recuperação

Quando uma ferramenta falha, o resultado deve ser devolvido ao Núcleo Conversacional com indicação explícita do estado da operação.

```text
Execução
   ↓
Falha
   ↓
ToolResult
   ↓
Orquestrador
   ↓
Núcleo Conversacional
   ↓
Interpretar falha
   ↓
┌────────────────────────────────┐
│ É possível recuperar?          │
└───────────────┬────────────────┘
                │
        ┌───────┴────────┐
        │                │
       SIM              NÃO
        │                │
        ▼                ▼
Solicitar correção   Informar falha
ou nova informação   e encerrar tarefa
```

A recuperação pode envolver:

- solicitar parâmetro ausente;
- corrigir um parâmetro;
- esclarecer uma ambiguidade;
- selecionar novamente uma ferramenta;
- permitir que o usuário tente outra operação.

A recuperação não deve mascarar uma falha efetivamente ocorrida.

---

# 11. Fluxo de confirmação

Operações que exigirem confirmação podem utilizar o seguinte fluxo:

```text
Solicitação
    ↓
Determinação da operação
    ↓
Preparação e validação
    ↓
[Confirmação necessária]
    ↓
Solicitar confirmação
    ↓
┌───────────────┐
│ Usuário       │
│ confirma?     │
└───────┬───────┘
        │
   ┌────┴────┐
   │         │
  SIM       NÃO
   │         │
   ▼         ▼
Executar   Cancelar
```

A confirmação não é considerada obrigatória para todas as operações. Sua necessidade deve ser determinada pelas regras da operação, pelos requisitos do sistema e pelas decisões de segurança e interação.

---

# 12. Isolamento do domínio

Uma propriedade central da arquitetura é que o Núcleo Conversacional não dependa diretamente das estruturas internas do domínio.

A dependência desejada é:

```text
Núcleo Conversacional
        ↓
Orquestrador
        ↓
Interface de Ferramentas
        ↓
Adaptador
        ↓
Domínio
```

e não:

```text
Núcleo Conversacional
        ↓
API / Banco / Classes específicas do domínio
```

Essa separação favorece:

- reutilização do núcleo conversacional;
- substituição do sistema de domínio;
- inclusão de novos domínios;
- substituição das ferramentas;
- testes isolados;
- evolução independente dos componentes.

---

# 13. Observabilidade e rastreabilidade

A arquitetura deve permitir registrar informações suficientes para compreender o comportamento do sistema.

Entre os eventos potencialmente relevantes estão:

- entrada recebida;
- interpretação produzida;
- contexto utilizado;
- informações identificadas;
- informações ausentes;
- operação determinada;
- ferramenta selecionada;
- argumentos produzidos;
- resultado da validação;
- execução realizada;
- resultado recebido;
- falha identificada;
- recuperação realizada;
- resposta produzida.

Esses registros devem permitir relacionar o comportamento observado aos requisitos, casos de uso e cenários de verificação definidos no projeto.

A implementação concreta do mecanismo de observabilidade permanece fora do escopo deste modelo lógico.

---

# 14. Princípios arquiteturais

A arquitetura é orientada pelos seguintes princípios:

### P1 — Separação entre interação e domínio

A lógica de interação conversacional deve permanecer separada das regras de negócio.

### P2 — Desacoplamento do núcleo conversacional

O núcleo não deve depender diretamente de um domínio específico.

### P3 — Ferramentas como fronteira de integração

Operações externas devem ser acessadas por meio de ferramentas explicitamente disponibilizadas.

### P4 — Controle da execução

A geração de uma chamada pela LLM não implica sua execução automática.

### P5 — Validação antes da execução

As solicitações devem ser verificadas antes de alcançar o sistema de domínio.

### P6 — Preservação das regras de negócio

As regras específicas do domínio devem permanecer sob responsabilidade do sistema de domínio.

### P7 — Fidelidade ao resultado

O assistente deve representar o resultado efetivamente produzido pela operação, distinguindo sucesso, ausência de dados, falha e resultado inconclusivo.

### P8 — Recuperação da interação

Falhas e informações insuficientes devem permitir recuperação sempre que possível.

### P9 — Observabilidade

O comportamento do sistema deve ser suficientemente observável para permitir verificação e rastreabilidade.

### P10 — Independência tecnológica

O modelo lógico não deve depender, nesta etapa, de um protocolo, framework ou fornecedor específico.

---

# 15. Decisões ainda em aberto

A arquitetura estabelece as responsabilidades e interfaces conceituais, mas algumas decisões permanecem deliberadamente abertas para a fase de projeto detalhado.

Entre elas:

- tecnologia de reconhecimento de voz;
- tecnologia de síntese de voz;
- modelo de LLM;
- mecanismo de tool calling;
- adoção ou não de MCP;
- protocolo de comunicação entre componentes;
- armazenamento do contexto;
- implementação do Gerenciador de Tarefa;
- granularidade das ferramentas;
- formato concreto de `ToolDefinition`;
- formato concreto de `ToolRequest`;
- formato concreto de `ToolResult`;
- mecanismo de autorização das ferramentas;
- mecanismo de observabilidade;
- implementação do sistema de domínio.

Essas decisões deverão ser tomadas posteriormente considerando os requisitos, os casos de uso, a literatura levantada e as restrições do protótipo.

---

# 16. Relação com os requisitos e casos de uso

O modelo arquitetural deve fornecer suporte aos requisitos funcionais consolidados.

A relação conceitual é:

```text
Requisitos
    ↓
Casos de Uso
    ↓
Responsabilidades Arquiteturais
    ↓
Componentes
    ↓
Contratos / Interfaces
    ↓
Implementação
```

Em particular:

| Capacidade | Principal suporte arquitetural |
|---|---|
| Receber/processar voz | Interface Conversacional |
| Interpretar linguagem natural | Núcleo Conversacional |
| Gerenciar contexto | Gerenciador de Contexto |
| Gerenciar informações da tarefa | Gerenciador de Tarefa |
| Selecionar ferramenta | Orquestrador de Ferramentas |
| Preparar e validar chamada | Orquestrador / Contrato de Ferramentas |
| Executar ferramenta | Orquestrador / Interface de Ferramentas |
| Processar resultado | Núcleo Conversacional / Orquestrador |
| Tratar falhas | Orquestrador / Núcleo / Gerenciador de Tarefa |
| Gerenciar confirmação/cancelamento | Núcleo / Gerenciador de Tarefa / Interface |
| Gerar resposta textual | Núcleo Conversacional |
| Entregar resposta por voz | Interface Conversacional |

A tabela não representa uma relação de exclusividade. As capacidades são distribuídas entre componentes conforme suas responsabilidades e podem envolver mais de um componente durante sua execução.

---

# 17. Estado atual do modelo

Esta versão do modelo define a **arquitetura lógica de referência** do sistema.

O ponto central da arquitetura é a existência de uma fronteira explícita entre:

```text
Linguagem / raciocínio conversacional
                │
                ▼
       Orquestração controlada
                │
                ▼
          Ferramentas
                │
                ▼
       Sistema de domínio
```

O **Contrato Lógico de Ferramentas** estabelece a abstração necessária para que essa fronteira seja implementada posteriormente por diferentes mecanismos tecnológicos.

A escolha concreta entre mecanismos como **tool/function calling nativo da LLM, MCP ou uma camada própria de integração** deverá ocorrer em uma etapa posterior de projeto, comparando suas características com os requisitos e com o contrato lógico estabelecido nesta arquitetura.

Dessa forma, a arquitetura não começa pela tecnologia. Primeiro são definidos os **responsáveis, as interfaces, os fluxos e os contratos**; posteriormente será selecionada a tecnologia capaz de implementar essas abstrações.
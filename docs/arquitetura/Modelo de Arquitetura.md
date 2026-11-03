# Modelo de Arquitetura

**Projeto:** Assistente Conversacional por Voz com Uso de Ferramentas  
**Versão:** 1.1  
**Finalidade:** Refinar o modelo arquitetural lógico, detalhando as interfaces entre componentes, os contratos de comunicação e os principais fluxos de interação.

---

# 1. Objetivo

Esta versão refina o modelo arquitetural definido na versão 1.0, acrescentando:

- interfaces lógicas entre componentes;
- responsabilidades mais precisas;
- contratos conceituais de comunicação;
- fluxo de execução de uma solicitação que utiliza ferramentas;
- fluxo de recuperação de falhas;
- fronteiras entre o núcleo conversacional, o uso de ferramentas e o sistema de domínio.

O modelo permanece independente de tecnologias específicas. Modelos de linguagem, mecanismos de reconhecimento e síntese de voz, frameworks, protocolos e infraestruturas concretas serão definidos posteriormente.

---

# 2. Estrutura arquitetural refinada

A arquitetura é organizada em cinco áreas lógicas:

```text
┌───────────────────────────────────────────────────────────────┐
│                        INTERAÇÃO                              │
│                                                               │
│                 Interface Conversacional                     │
└───────────────────────────────┬───────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────────────────────────────────────┐
│                  PROCESSAMENTO CONVERSACIONAL                 │
│                                                               │
│  ┌─────────────────────┐       ┌──────────────────────────┐  │
│  │ Núcleo Conversacional│◄─────►│ Gerenciador de Contexto │  │
│  └──────────┬──────────┘       └──────────────────────────┘  │
│             │                                                 │
│             ▼                                                 │
│  ┌─────────────────────┐                                     │
│  │ Gerenciador de      │                                     │
│  │ Tarefa              │                                     │
│  └──────────┬──────────┘                                     │
└─────────────┼─────────────────────────────────────────────────┘
              │
              ▼
┌───────────────────────────────────────────────────────────────┐
│                    USO DE FERRAMENTAS                         │
│                                                               │
│                 Orquestrador de Ferramentas                   │
│                              │                                │
│                              ▼                                │
│                 Registro / Interface de                       │
│                       Ferramentas                             │
└──────────────────────────────┬────────────────────────────────┘
                               │
                               ▼
┌───────────────────────────────────────────────────────────────┐
│                    INTEGRAÇÃO DE DOMÍNIO                      │
│                                                               │
│                 Adaptador de Domínio                          │
│                              │                                │
│                              ▼                                │
│             Sistema de Informação de Domínio                  │
└───────────────────────────────────────────────────────────────┘
```

A arquitetura é organizada em camadas lógicas, mas não pressupõe que cada componente corresponda necessariamente a um processo, serviço, módulo ou aplicação independente.

---

# 3. Interfaces entre componentes

## 3.1 Interface de interação

### Origem
Interface Conversacional.

### Destino
Núcleo Conversacional.

### Responsabilidade

Transportar uma solicitação do usuário para o núcleo conversacional.

### Entrada conceitual

Uma solicitação contendo, no mínimo:

- conteúdo reconhecido da entrada do usuário;
- informações necessárias para identificar a interação;
- estado mínimo da sessão, quando aplicável.

### Saída conceitual

Uma solicitação processável pelo núcleo conversacional.

### Sentido inverso

O Núcleo Conversacional fornece à Interface Conversacional uma resposta destinada ao usuário.

A resposta deve conter, conceitualmente:

- conteúdo textual;
- estado da interação, quando relevante;
- indicação de encerramento ou continuidade, quando aplicável.

---

# 4. Interface de contexto

## 4.1 Núcleo Conversacional → Gerenciador de Contexto

O Núcleo Conversacional pode consultar informações previamente estabelecidas na interação.

Exemplos:

- entidade mencionada anteriormente;
- parâmetros já fornecidos;
- resultado de uma operação anterior;
- referências contextuais;
- estado relevante da conversa.

## 4.2 Gerenciador de Contexto → Núcleo Conversacional

O Gerenciador de Contexto fornece as informações relevantes para interpretação da solicitação atual.

## 4.3 Núcleo Conversacional → Gerenciador de Contexto

Após o processamento de uma interação, o núcleo pode atualizar o contexto com novas informações relevantes.

### Princípio

O contexto deve conter informações úteis para a continuidade da tarefa, e não simplesmente reproduzir indiscriminadamente todo o histórico da conversa.

---

# 5. Interface de gerenciamento da tarefa

## 5.1 Núcleo Conversacional → Gerenciador de Tarefa

O Núcleo Conversacional informa:

- objetivo identificado;
- informações já determinadas;
- informações solicitadas pelo usuário;
- operação pretendida;
- estado atual da interação.

## 5.2 Gerenciador de Tarefa → Núcleo Conversacional

O Gerenciador de Tarefa fornece:

- informações já conhecidas;
- informações ainda necessárias;
- estado atual da tarefa;
- indicação de suficiência das informações disponíveis;
- necessidade de esclarecimento ou recuperação.

### Princípio

O Gerenciador de Tarefa deve representar o estado necessário para a conclusão da tarefa sem depender diretamente das estruturas internas do domínio.

---

# 6. Interface entre Núcleo Conversacional e Orquestrador de Ferramentas

Esta é uma das principais fronteiras da arquitetura.

O Núcleo Conversacional não executa diretamente operações do domínio.

Quando determina que uma ferramenta é necessária, solicita ao Orquestrador a realização de uma operação.

## 6.1 Solicitação de operação

Conceitualmente, uma solicitação contém:

```text
ToolRequest
├── operação desejada
├── informações necessárias
├── parâmetros determinados
├── contexto relevante
└── requisitos da tarefa
```

O formato concreto ainda não é definido nesta etapa.

## 6.2 Resultado da operação

O Orquestrador retorna ao Núcleo Conversacional:

```text
ToolResult
├── status da execução
├── resultado da operação
├── informações de erro, quando aplicável
└── metadados relevantes
```

O núcleo utiliza esse resultado para:

- atualizar o estado da tarefa;
- atualizar o contexto;
- determinar se a tarefa foi concluída;
- solicitar informações adicionais;
- realizar recuperação;
- produzir a resposta ao usuário.

---

# 7. Responsabilidades do Orquestrador de Ferramentas

O Orquestrador realiza as seguintes etapas lógicas:

```text
Solicitação do núcleo
        ↓
Determinação da operação
        ↓
Seleção da ferramenta
        ↓
Preparação dos parâmetros
        ↓
Validação
        ↓
Execução
        ↓
Recebimento do resultado
        ↓
Normalização do resultado
        ↓
Retorno ao núcleo
```

O Orquestrador deve impedir que uma chamada seja executada quando:

- a operação não puder ser determinada;
- a ferramenta correspondente não estiver disponível;
- parâmetros obrigatórios estiverem ausentes;
- parâmetros forem incompatíveis com a operação;
- os valores não atenderem às condições de validação estabelecidas.

---

# 8. Interface de ferramentas

A arquitetura estabelece uma fronteira explícita entre o Orquestrador e as ferramentas disponíveis.

Uma ferramenta deve possuir, conceitualmente:

```text
ToolDefinition
├── identificador
├── descrição
├── operação disponibilizada
├── parâmetros
├── tipos/formato dos parâmetros
├── parâmetros obrigatórios
└── condições de utilização
```

A implementação concreta pode representar essas informações de outra maneira.

O objetivo da abstração é permitir que o Orquestrador trabalhe com ferramentas sem depender diretamente de sua implementação interna.

---

# 9. Execução de uma ferramenta

A execução segue conceitualmente:

```text
Orquestrador
     │
     │ solicitação validada
     ▼
Interface de Ferramentas
     │
     │ chamada
     ▼
Ferramenta
     │
     ▼
Adaptador de Domínio
     │
     ▼
Sistema de Informação
     │
     │ resultado
     ▼
Adaptador
     │
     ▼
Ferramenta
     │
     ▼
Interface de Ferramentas
     │
     ▼
Orquestrador
```

A ferramenta funciona como uma fronteira controlada para acesso às operações do domínio.

---

# 10. Contrato de resultado

O resultado de uma ferramenta deve permitir distinguir pelo menos:

### Sucesso

A operação foi executada e seu resultado pôde ser determinado.

### Falha de execução

A operação foi solicitada, mas não pôde ser concluída.

### Falha de validação

A chamada não pôde ser executada porque seus parâmetros não eram válidos.

### Resultado vazio

A operação foi executada, mas não foram encontrados dados correspondentes.

### Resultado inconclusivo

A operação foi executada, mas seu resultado não permite determinar adequadamente a situação solicitada.

Essa distinção é importante porque o Núcleo Conversacional não deve tratar todas essas situações como equivalentes.

---

# 11. Interface com o sistema de domínio

O Adaptador de Domínio estabelece a fronteira entre as ferramentas e o sistema de informação utilizado pelo estudo de caso.

Sua função é traduzir entre:

```text
Interface de ferramenta
        ↕
Formato de operação do domínio
```

Por exemplo, conceitualmente:

```text
Ferramenta:
realizar empréstimo
        ↓
Adaptador:
traduz parâmetros
        ↓
Sistema de domínio:
registrar empréstimo
        ↓
Adaptador:
normaliza resultado
        ↓
Ferramenta:
resultado da operação
```

O núcleo conversacional não deve depender diretamente do formato interno dessa operação.

---

# 12. Regras de negócio

As regras próprias do domínio devem ser avaliadas pelo Sistema de Informação de Domínio.

Exemplos:

- verificar se o recurso está disponível;
- verificar se o usuário pode realizar um empréstimo;
- verificar condições para devolução;
- registrar alterações no estado do domínio.

O assistente pode identificar e solicitar a operação, mas não deve substituir as regras de negócio do sistema de domínio.

Assim:

```text
Usuário
   ↓
Assistente
   ↓
"realizar empréstimo"
   ↓
Sistema de domínio
   ↓
regra de negócio
   ├── permitido
   └── não permitido
```

---

# 13. Fluxo completo de uma solicitação

Considerando uma solicitação como:

> “Quero pegar o livro Clean Code.”

o fluxo lógico é:

```text
1. Usuário
      │
      ▼
2. Interface Conversacional
      │
      ▼
3. Núcleo Conversacional
      │
      ├── consulta contexto
      │
      ├── identifica objetivo
      │
      ├── identifica recurso
      │
      └── determina informações necessárias
      │
      ▼
4. Gerenciador de Tarefa
      │
      └── informações suficientes
      │
      ▼
5. Orquestrador de Ferramentas
      │
      ├── determina operação
      ├── seleciona ferramenta
      ├── prepara parâmetros
      └── valida chamada
      │
      ▼
6. Ferramenta
      │
      ▼
7. Adaptador de Domínio
      │
      ▼
8. Sistema de Informação
      │
      ├── aplica regras de negócio
      └── executa operação
      │
      ▼
9. Resultado
      │
      ▼
10. Núcleo Conversacional
      │
      ├── processa resultado
      ├── atualiza tarefa
      ├── atualiza contexto
      └── gera resposta
      │
      ▼
11. Interface Conversacional
      │
      ▼
12. Usuário
```

---

# 14. Fluxo de solicitação com informação ausente

Exemplo:

> “Quero fazer um empréstimo.”

```text
Usuário
   ↓
Núcleo Conversacional
   ↓
Gerenciador de Tarefa
   ↓
Informação necessária ausente
   ↓
Núcleo Conversacional
   ↓
"Qual livro você deseja emprestar?"
   ↓
Usuário
   ↓
Núcleo Conversacional
   ↓
Gerenciador de Tarefa
   ↓
Informações suficientes
   ↓
Orquestrador
   ↓
Ferramenta
   ↓
Domínio
```

O ponto importante é:

> **a ausência de informação interrompe a execução da operação, mas não necessariamente encerra a tarefa.**

Isso materializa arquiteturalmente o RF04 e o RF09.

---

# 15. Fluxo de falha de execução

Quando a ferramenta não consegue executar uma operação:

```text
Orquestrador
      ↓
Ferramenta
      ↓
Falha
      ↓
Resultado de falha
      ↓
Orquestrador
      ↓
Núcleo Conversacional
      ↓
Gerenciador de Tarefa
      ↓
Recuperação possível?
     / \
   sim  não
   ↓     ↓
continuar  informar falha
```

Quando a recuperação for possível, o sistema pode solicitar novas informações ou permitir nova tentativa.

Quando não for possível, o sistema informa a falha ao usuário.

Em ambos os casos:

> **o sistema não deve afirmar que a operação foi realizada se a execução não tiver sido confirmada.**

---

# 16. Fluxo de confirmação

Quando uma operação exigir confirmação:

```text
Núcleo Conversacional
        ↓
Tarefa determinada
        ↓
Necessidade de confirmação
        ↓
Interface Conversacional
        ↓
Usuário
     /      \
confirmar   cancelar
   ↓           ↓
Orquestrador   encerra
   ↓
Ferramenta
   ↓
Domínio
```

A confirmação é uma propriedade da interação e não deve ser confundida com a validação das regras de negócio do domínio.

---

# 17. Isolamento do domínio

O núcleo do sistema deve depender de uma abstração de ferramentas, e não diretamente das operações concretas do sistema bibliotecário.

Conceitualmente:

```text
                 ┌─────────────────────┐
                 │ Núcleo Conversacional│
                 └──────────┬──────────┘
                            │
                     abstração de
                       ferramentas
                            │
             ┌──────────────┴──────────────┐
             │                             │
             ▼                             ▼
     Domínio bibliotecário           Outro domínio
             │                             │
             ▼                             ▼
       ferramentas                   ferramentas
```

Assim, o domínio bibliotecário constitui um **estudo de caso da arquitetura**, e não uma dependência estrutural do núcleo conversacional.

---

# 18. Granularidade das ferramentas

A arquitetura não fixa uma única estratégia para a granularidade das ferramentas.

Podem existir ferramentas específicas:

```text
consultar_livro
verificar_disponibilidade
realizar_emprestimo
realizar_devolucao
```

ou ferramentas mais abrangentes:

```text
consultar_catalogo
consultar_emprestimos
executar_operacao_de_acervo
```

A decisão deve considerar:

- clareza das operações;
- facilidade de validação;
- segurança da execução;
- quantidade de parâmetros;
- capacidade de seleção correta;
- possibilidade de reutilização;
- complexidade do domínio;
- comportamento do mecanismo de Tool Calling.

Essa decisão pertence ao refinamento arquitetural e à implementação.

---

# 19. Observabilidade

A arquitetura deve permitir acompanhar o ciclo de execução de uma tarefa.

Devem ser considerados, conceitualmente, eventos como:

- entrada recebida;
- solicitação interpretada;
- contexto utilizado;
- informações identificadas;
- informações faltantes;
- ferramenta selecionada;
- parâmetros preparados;
- validação realizada;
- ferramenta executada;
- resultado obtido;
- falha ocorrida;
- recuperação realizada;
- resposta produzida.

Essas informações não precisam ser expostas ao usuário, mas devem estar disponíveis para análise do funcionamento do protótipo.

A observabilidade constitui uma condição importante para os cenários de verificação e para eventual avaliação futura.

---

# 20. Mapeamento refinado de responsabilidades

| Componente | Responsabilidades principais |
|---|---|
| **Interface Conversacional** | Entrada e saída por voz; controle da interação |
| **Núcleo Conversacional** | Interpretação; coordenação; geração da resposta |
| **Gerenciador de Contexto** | Armazenamento e recuperação de informações contextuais |
| **Gerenciador de Tarefa** | Estado da tarefa; informações necessárias; progresso e recuperação |
| **Orquestrador de Ferramentas** | Seleção; preparação; validação; execução e tratamento do resultado |
| **Registro/Interface de Ferramentas** | Descrição, disponibilidade e acesso às ferramentas |
| **Adaptador de Domínio** | Tradução entre ferramentas e sistema de domínio |
| **Sistema de Informação de Domínio** | Dados e regras de negócio |

---

# 21. Interfaces e responsabilidades — resumo

```text
┌───────────────────────┐
│ Interface             │
│ Conversacional        │
└───────────┬───────────┘
            │ interação
            ▼
┌───────────────────────┐
│ Núcleo                │
│ Conversacional        │
└──────┬─────────┬──────┘
       │         │
       │         ├───────────────┐
       ▼         ▼               │
┌────────────┐ ┌──────────────┐  │
│ Contexto   │ │ Tarefa       │  │
└────────────┘ └──────────────┘  │
                                 │ operação
                                 ▼
                       ┌──────────────────┐
                       │ Orquestrador     │
                       │ de Ferramentas   │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │ Interface de     │
                       │ Ferramentas      │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │ Adaptador de     │
                       │ Domínio          │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │ Sistema de       │
                       │ Informação       │
                       └──────────────────┘
```

---

# 22. Decisões mantidas em aberto

A versão 1.1 ainda não determina:

- tecnologia de reconhecimento de voz;
- tecnologia de síntese de voz;
- modelo de linguagem;
- estratégia de prompting;
- framework de agente;
- protocolo específico de ferramentas;
- formato concreto de `ToolRequest` e `ToolResult`;
- persistência do contexto;
- mecanismo de autenticação;
- infraestrutura;
- tecnologia do sistema de domínio;
- granularidade definitiva das ferramentas.

Essas decisões deverão ser tomadas durante o detalhamento do projeto, mantendo rastreabilidade com os requisitos e as responsabilidades arquiteturais estabelecidas neste documento.

---

# 23. Rastreabilidade

A evolução arquitetural deve preservar a seguinte cadeia:

**Literatura / Diretrizes → Requisitos → Casos de Uso → Cenários de Verificação → Responsabilidades → Componentes → Interfaces → Implementação**

Cada decisão arquitetural relevante deverá ser justificável por pelo menos um requisito, caso de uso, cenário de verificação ou restrição do projeto.

---

# 24. Estado do modelo

Esta versão representa o **modelo arquitetural refinado**, suficiente para orientar a definição das interfaces e responsabilidades de implementação, mas ainda independente de tecnologias concretas.

A próxima etapa deverá detalhar as decisões de projeto necessárias para transformar este modelo lógico em uma arquitetura implementável.
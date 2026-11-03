# Modelo de Arquitetura

**Projeto:** Assistente Conversacional por Voz com Uso de Ferramentas  
**Versão:** 1.0  
**Finalidade:** Registrar a estrutura lógica dos componentes do sistema e suas principais responsabilidades.

---

## 1. Objetivo

Este documento descreve o modelo arquitetural lógico do sistema, identificando seus principais componentes, responsabilidades e relações.

A arquitetura é derivada dos requisitos funcionais, casos de uso e cenários de verificação definidos para o sistema. Neste nível, não são especificadas tecnologias, frameworks, modelos de linguagem ou protocolos concretos de comunicação.

O modelo busca separar:

- a interação com o usuário;
- o processamento conversacional;
- o gerenciamento de contexto e informações da tarefa;
- o gerenciamento e uso de ferramentas;
- a integração com o sistema de informação de domínio.

Essa separação deve permitir que o núcleo conversacional permaneça relativamente desacoplado do domínio de aplicação utilizado no estudo de caso.

---

# 2. Visão geral da arquitetura

A estrutura lógica proposta é composta pelos seguintes elementos principais:

```text
┌───────────────────────────────────────────────────────────┐
│                    USUÁRIO                                │
└──────────────────────────┬────────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────┐
│              INTERFACE CONVERSACIONAL                     │
│                                                           │
│  Entrada de voz  ←→  Resposta por voz                     │
└──────────────────────────┬────────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────┐
│                 NÚCLEO CONVERSACIONAL                     │
│                                                           │
│  Interpretação  •  Contexto  •  Tarefa  •  Resposta      │
└──────────────────────────┬────────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────┐
│                ORQUESTRADOR DE FERRAMENTAS                │
│                                                           │
│  Seleção → Preparação → Validação → Execução → Resultado │
└──────────────────────────┬────────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────┐
│             INTERFACE DE FERRAMENTAS / ADAPTADOR          │
│                                                           │
│       Ferramentas autorizadas do domínio                  │
└──────────────────────────┬────────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────┐
│              SISTEMA DE INFORMAÇÃO DE DOMÍNIO             │
│                                                           │
│     Dados + operações + regras de negócio                 │
└───────────────────────────────────────────────────────────┘
```

A direção das interações não representa necessariamente comunicação síncrona ou um protocolo específico. Ela representa principalmente o fluxo lógico de processamento.

---

# 3. Componentes arquiteturais

## 3.1 Interface Conversacional

### Responsabilidade

Responsável pela interação multimodal ou predominantemente vocal com o usuário.

Suas principais responsabilidades são:

- receber a entrada de voz do usuário;
- encaminhar a entrada para processamento conversacional;
- receber a resposta textual produzida pelo núcleo conversacional;
- converter a resposta para o canal de voz;
- entregar a resposta ao usuário;
- fornecer mecanismos necessários ao controle da interação quando aplicável.

### Requisitos relacionados

- RF01 — Receber e processar entrada de voz
- RF10 — Gerenciar controle da interação
- RF11 — Gerar resposta textual
- RF12 — Entregar resposta por voz

### Observação

A interface não deve ser responsável por interpretar semanticamente a solicitação ou decidir qual operação de domínio deve ser executada.

---

# 3.2 Núcleo Conversacional

### Responsabilidade

Responsável pelo processamento da interação em linguagem natural e pela coordenação do comportamento conversacional do sistema.

Suas principais responsabilidades são:

- interpretar a solicitação do usuário;
- identificar a intenção ou objetivo da interação;
- identificar entidades, informações e parâmetros relevantes;
- utilizar informações disponíveis no contexto;
- identificar informações ausentes;
- solicitar informações adicionais quando necessário;
- identificar ambiguidades e solicitar esclarecimentos;
- determinar a próxima ação necessária para atender à solicitação;
- receber resultados das operações executadas;
- produzir uma resposta coerente com o estado da tarefa.

### Requisitos relacionados

- RF02 — Interpretar solicitação em linguagem natural
- RF03 — Gerenciar contexto conversacional
- RF04 — Gerenciar informações necessárias à tarefa
- RF08 — Processar resultados das ferramentas
- RF09 — Tratar falhas e recuperar a interação
- RF11 — Gerar resposta textual

### Observação

O componente é descrito em termos de responsabilidade e não pressupõe que a interpretação seja realizada exclusivamente por um modelo de linguagem específico.

---

# 3.3 Gerenciador de Contexto

### Responsabilidade

Responsável por manter e disponibilizar informações relevantes das interações anteriores para continuidade da tarefa.

Suas principais responsabilidades são:

- armazenar informações relevantes da interação corrente;
- recuperar informações previamente estabelecidas;
- associar referências contextuais às entidades correspondentes;
- disponibilizar informações de contexto ao processamento da solicitação;
- atualizar o contexto conforme a interação evolui;
- descartar ou deixar de considerar informações que não sejam mais relevantes à tarefa.

### Requisitos relacionados

- RF03 — Gerenciar contexto conversacional
- RF04 — Gerenciar informações necessárias à tarefa

### Exemplos

O componente deve possibilitar situações como:

> “Quem escreveu *Clean Code*?”

seguido de:

> “E quando ele foi publicado?”

ou:

> “O livro está disponível?”

seguido de:

> “Então pode fazer o empréstimo.”

O gerenciamento do contexto permite que essas referências sejam resolvidas quando houver informação suficiente para isso.

---

# 3.4 Gerenciador de Tarefa

### Responsabilidade

Responsável por manter o estado da tarefa que está sendo realizada e pelas informações necessárias para sua conclusão.

Suas principais responsabilidades são:

- registrar o objetivo corrente da interação;
- identificar informações já determinadas;
- identificar informações ainda necessárias;
- acompanhar o progresso da tarefa;
- identificar quando a informação disponível é suficiente para prosseguir;
- solicitar, por meio do núcleo conversacional, informações ausentes;
- manter o estado da tarefa durante múltiplas interações;
- permitir correção ou atualização de informações durante a interação;
- indicar quando a tarefa pode ser concluída.

### Requisitos relacionados

- RF03 — Gerenciar contexto conversacional
- RF04 — Gerenciar informações necessárias à tarefa
- RF09 — Tratar falhas e recuperar a interação

### Observação

O Gerenciador de Tarefa não deve conter regras específicas do domínio bibliotecário. Ele deve trabalhar com informações e estados necessários à tarefa sem depender diretamente das entidades concretas do domínio.

---

# 3.5 Orquestrador de Ferramentas

### Responsabilidade

Responsável por coordenar o uso das ferramentas necessárias à execução das operações de domínio.

Suas principais responsabilidades são:

- determinar, a partir da solicitação e do estado da tarefa, se uma ferramenta é necessária;
- selecionar uma ferramenta compatível;
- preparar os parâmetros necessários;
- validar os parâmetros antes da execução;
- impedir chamadas quando os dados forem insuficientes ou inválidos;
- solicitar a execução da ferramenta autorizada;
- receber o resultado da execução;
- comunicar o resultado ao núcleo conversacional;
- tratar condições de falha relacionadas à execução.

### Requisitos relacionados

- RF05 — Selecionar ferramenta para execução da tarefa
- RF06 — Preparar e validar chamada de ferramenta
- RF07 — Executar ferramentas autorizadas
- RF08 — Processar resultados das ferramentas
- RF09 — Tratar falhas e recuperar a interação

### Observação importante

Este componente não define, neste momento, a granularidade das ferramentas.

Por exemplo, ainda não se decide se o domínio será exposto por:

```text
consultar_livro()
verificar_disponibilidade()
realizar_emprestimo()
realizar_devolucao()
```

ou por ferramentas mais genéricas.

Essa é uma decisão posterior de projeto arquitetural.

---

# 3.6 Registro e Interface de Ferramentas

### Responsabilidade

Responsável por disponibilizar ao sistema a descrição e os meios de acesso às ferramentas autorizadas.

Suas principais responsabilidades são:

- registrar ferramentas disponíveis;
- disponibilizar suas descrições;
- disponibilizar os parâmetros esperados;
- identificar operações autorizadas;
- fornecer uma interface uniforme para execução;
- encaminhar chamadas para a implementação correspondente;
- devolver os resultados das operações.

### Requisitos relacionados

- RF05 — Selecionar ferramenta para execução da tarefa
- RF06 — Preparar e validar chamada de ferramenta
- RF07 — Executar ferramentas autorizadas

### Observação

A separação entre o Orquestrador de Ferramentas e a Interface de Ferramentas é deliberada.

O orquestrador decide:

> **“Qual operação preciso realizar?”**

Enquanto a interface de ferramentas fornece:

> **“Quais operações estão disponíveis e como podem ser executadas?”**

Isso favorece o desacoplamento entre o núcleo conversacional e o domínio.

---

# 3.7 Adaptador do Sistema de Domínio

### Responsabilidade

Responsável por conectar as ferramentas disponibilizadas ao assistente às operações concretas do sistema de informação de domínio.

Suas principais responsabilidades são:

- traduzir chamadas de ferramentas para operações do sistema de domínio;
- adaptar parâmetros entre as interfaces;
- receber resultados do sistema de domínio;
- adaptar os resultados para o formato esperado pelo assistente;
- preservar as regras de negócio do sistema de domínio;
- evitar que o núcleo conversacional dependa diretamente da implementação interna do sistema de domínio.

### Requisitos relacionados

Principalmente:

- RF07 — Executar ferramentas autorizadas
- RF08 — Processar resultados das ferramentas

### Observação

Este componente é particularmente importante para o requisito de desacoplamento do domínio.

O núcleo conversacional não deve precisar conhecer diretamente estruturas internas como:

```text
Livro
Usuario
Emprestimo
Exemplar
```

nem detalhes de banco de dados, endpoints ou classes específicas do sistema bibliotecário.

---

# 3.8 Sistema de Informação de Domínio

### Responsabilidade

Representa o sistema responsável pelos dados e pelas regras próprias do domínio de aplicação.

No estudo de caso atual, corresponde ao sistema de informação bibliotecário.

Suas responsabilidades incluem:

- manter os dados do domínio;
- executar operações de consulta;
- executar operações de alteração;
- aplicar regras de negócio;
- validar condições específicas do domínio;
- retornar os resultados das operações.

### Exemplos de responsabilidades de domínio

- determinar se um livro está disponível;
- determinar se um usuário pode realizar um empréstimo;
- registrar um empréstimo;
- registrar uma devolução;
- consultar empréstimos existentes.

### Observação

As regras de negócio devem permanecer neste nível sempre que forem próprias do domínio.

Por exemplo:

> “O usuário atingiu o limite de empréstimos.”

é uma regra do domínio, não uma regra que deve ser codificada no núcleo conversacional.

---

# 4. Relações entre componentes

As principais relações são:

| Origem | Destino | Responsabilidade da relação |
|---|---|---|
| Interface Conversacional | Núcleo Conversacional | Encaminhar solicitações e receber respostas |
| Núcleo Conversacional | Gerenciador de Contexto | Consultar e atualizar contexto |
| Núcleo Conversacional | Gerenciador de Tarefa | Manter estado e informações da tarefa |
| Núcleo Conversacional | Orquestrador de Ferramentas | Solicitar execução de operações necessárias |
| Orquestrador de Ferramentas | Registro/Interface de Ferramentas | Selecionar e executar ferramentas disponíveis |
| Registro/Interface de Ferramentas | Adaptador de Domínio | Encaminhar operações para o domínio |
| Adaptador de Domínio | Sistema de Informação de Domínio | Executar operações e obter resultados |
| Sistema de Informação de Domínio | Adaptador de Domínio | Retornar dados e resultados de negócio |
| Adaptador de Domínio | Registro/Interface de Ferramentas | Retornar resultado da ferramenta |
| Orquestrador de Ferramentas | Núcleo Conversacional | Informar resultado da operação |
| Núcleo Conversacional | Interface Conversacional | Entregar resposta ao usuário |

---

# 5. Fluxo arquitetural geral

Uma interação típica pode percorrer os componentes da seguinte maneira:

```text
1. Usuário
      │
      │ solicitação de voz
      ▼
2. Interface Conversacional
      │
      │ entrada processada
      ▼
3. Núcleo Conversacional
      │
      ├──── consulta ────► Gerenciador de Contexto
      │
      ├──── consulta ────► Gerenciador de Tarefa
      │
      ▼
4. Orquestrador de Ferramentas
      │
      │ seleciona/prepara/valida
      ▼
5. Registro / Interface de Ferramentas
      │
      ▼
6. Adaptador do Sistema de Domínio
      │
      ▼
7. Sistema de Informação de Domínio
      │
      │ resultado
      ▼
6. Adaptador
      │
      ▼
5. Interface de Ferramentas
      │
      ▼
4. Orquestrador
      │
      ▼
3. Núcleo Conversacional
      │
      ├──── atualiza contexto/tarefa
      │
      ▼
2. Interface Conversacional
      │
      │ resposta por voz
      ▼
1. Usuário
```

---

# 6. Princípios arquiteturais

O modelo deve observar os seguintes princípios.

## PA01 — Separação entre interação e domínio

A interface conversacional e o núcleo conversacional não devem depender diretamente da implementação específica do sistema de informação de domínio.

## PA02 — Desacoplamento do núcleo conversacional

O núcleo do assistente deve poder operar com diferentes domínios desde que estes disponibilizem ferramentas compatíveis com a interface estabelecida.

## PA03 — Ferramentas como fronteira de integração

As ferramentas constituem a principal fronteira entre o assistente e as operações do domínio.

## PA04 — Regras de negócio no domínio

Regras próprias do domínio devem permanecer no sistema de informação de domínio, e não no núcleo conversacional.

## PA05 — Validação antes da execução

Chamadas de ferramentas devem ser validadas antes de sua execução.

## PA06 — Execução somente com informação suficiente

O sistema não deve executar uma operação quando não houver informações suficientes para determinar a operação e seus parâmetros.

## PA07 — Fidelidade aos resultados

O núcleo conversacional deve basear suas respostas nos resultados efetivamente obtidos das ferramentas e do sistema de domínio.

## PA08 — Recuperação da interação

Falhas corrigíveis devem permitir a continuidade da tarefa sempre que possível.

## PA09 — Observabilidade

Os componentes devem permitir registrar informações suficientes para compreender o fluxo de execução e posteriormente verificar o comportamento do sistema.

## PA10 — Independência tecnológica

As responsabilidades arquiteturais não devem depender de uma tecnologia específica de LLM, reconhecimento de voz, síntese de voz, framework de agentes ou protocolo de ferramentas.

---

# 7. Mapeamento dos componentes aos requisitos

| Componente | Principais RFs |
|---|---|
| Interface Conversacional | RF01, RF10, RF11, RF12 |
| Núcleo Conversacional | RF02, RF03, RF04, RF08, RF09, RF11 |
| Gerenciador de Contexto | RF03, RF04 |
| Gerenciador de Tarefa | RF03, RF04, RF09 |
| Orquestrador de Ferramentas | RF05, RF06, RF07, RF08, RF09 |
| Registro/Interface de Ferramentas | RF05, RF06, RF07 |
| Adaptador do Sistema de Domínio | RF07, RF08 |
| Sistema de Informação de Domínio | Operações e regras próprias do domínio |

---

# 8. Relação com os casos de uso

Os componentes não correspondem diretamente aos casos de uso.

Os cinco casos de uso utilizam diferentes combinações dos mesmos componentes:

| Caso de uso | Componentes principais envolvidos |
|---|---|
| **UC01 — Consultar informações** | Interface, Núcleo Conversacional, Contexto, Tarefa, Orquestrador, Ferramentas, Domínio |
| **UC02 — Consultar disponibilidade** | Interface, Núcleo Conversacional, Contexto, Tarefa, Orquestrador, Ferramentas, Domínio |
| **UC03 — Realizar empréstimo** | Interface, Núcleo Conversacional, Contexto, Tarefa, Orquestrador, Ferramentas, Domínio |
| **UC04 — Realizar devolução** | Interface, Núcleo Conversacional, Contexto, Tarefa, Orquestrador, Ferramentas, Domínio |
| **UC05 — Consultar empréstimos** | Interface, Núcleo Conversacional, Contexto, Tarefa, Orquestrador, Ferramentas, Domínio |

Isso reforça a distinção entre:

**Caso de uso:** objetivo do usuário.

**Componente:** responsabilidade arquitetural.

Um mesmo componente pode participar de vários casos de uso.

---

# 9. Decisões ainda não definidas

Este modelo deliberadamente não define algumas decisões que deverão ser tomadas na próxima etapa de projeto, entre elas:

- tecnologia de reconhecimento de voz;
- tecnologia de síntese de voz;
- modelo ou fornecedor de LLM;
- mecanismo específico de Tool Calling;
- formato concreto das definições de ferramentas;
- granularidade das ferramentas;
- protocolo de comunicação entre ferramentas e domínio;
- mecanismo de persistência do contexto;
- mecanismo de observabilidade;
- infraestrutura de execução;
- sistema bibliotecário concreto utilizado no estudo de caso.

Essas decisões deverão ser derivadas dos requisitos, restrições e objetivos do projeto, e não fazem parte deste modelo lógico inicial.

---

# 10. Rastreabilidade arquitetural

A arquitetura deve manter a seguinte relação de rastreabilidade:

```text
Literatura / Diretrizes
        ↓
Requisitos
        ↓
Casos de Uso
        ↓
Cenários de Verificação
        ↓
Responsabilidades Arquiteturais
        ↓
Componentes
        ↓
Implementação
```

Dessa forma, cada componente arquitetural deve possuir uma responsabilidade justificável a partir dos requisitos e casos de uso, e posteriormente deverá ser possível verificar se seu comportamento contribui para os cenários definidos.
# Documento de Visão --- Assistente Conversacional por Voz

**Projeto:** TCC --- Assistente Conversacional por Voz para Interação
com Sistemas de Informação\
**Versão:** 1.0\
**Status:** Em elaboração\
**Data:** 2026-09-13

------------------------------------------------------------------------

## 1. Propósito

Este documento apresenta a visão geral do sistema proposto no Trabalho
de Conclusão de Curso (TCC), estabelecendo o problema, os objetivos, o
escopo, os principais stakeholders, a visão da solução e os requisitos
de alto nível que orientarão as etapas posteriores de análise, projeto,
implementação e avaliação.

O documento funciona como artefato de visão do projeto e não pretende
substituir a especificação detalhada dos requisitos, dos casos de uso,
da arquitetura ou do protocolo experimental.

------------------------------------------------------------------------

## 2. Contexto

Interfaces conversacionais baseadas em voz permitem que usuários
interajam com sistemas computacionais por meio de linguagem natural,
reduzindo a dependência de mecanismos tradicionais de interação, como
menus, formulários e comandos rigidamente estruturados.

A evolução recente dos modelos de linguagem ampliou esse cenário ao
permitir que LLMs interpretem solicitações e utilizem ferramentas
externas para consultar ou modificar sistemas de informação. Nesse
modelo, o LLM deixa de atuar somente como gerador de texto e passa a
funcionar como componente de orquestração entre a intenção do usuário e
operações disponibilizadas pelo domínio.

O levantamento bibliográfico realizado para este projeto identificou
duas frentes particularmente relevantes:

1.  diretrizes e padrões para projeto e avaliação de Interfaces de
    Usuário de Voz (VUIs) e agentes conversacionais; e
2.  técnicas e paradigmas para uso de ferramentas por LLMs e agentes.

As diretrizes de VUI destacam aspectos como gerenciamento de contexto,
redução da carga cognitiva, preenchimento de parâmetros, tratamento de
ambiguidades, feedback, recuperação de erros, controle do usuário e
transparência. O mapeamento realizado para este projeto traduz essas
diretrizes em padrões de implementação aplicáveis a uma arquitetura de
diálogo.

Paralelamente, trabalhos sobre Tool Use, como Toolformer, API-Bank e
Gorilla, demonstram que a utilização de ferramentas envolve capacidades
distintas, incluindo seleção da ferramenta adequada, geração de
argumentos, execução e tratamento de erros. O projeto utiliza essas
capacidades como referência para estruturar e avaliar o mecanismo de
interação entre o agente e o sistema de informação.

------------------------------------------------------------------------

## 3. Problema

A integração entre interfaces conversacionais por voz, modelos de
linguagem e sistemas de informação envolve desafios que vão além da
simples geração de respostas em linguagem natural.

O sistema precisa interpretar solicitações potencialmente incompletas ou
ambíguas, preservar contexto entre turnos, identificar os parâmetros
necessários, selecionar operações apropriadas, produzir argumentos
válidos, executar ferramentas e comunicar corretamente seus resultados
ao usuário.

Além disso, uma solução tecnicamente funcional pode apresentar problemas
de usabilidade, como excesso de interações, respostas inadequadas, perda
de contexto, dificuldade de recuperação após erros ou sobrecarga
cognitiva decorrente da natureza linear e efêmera da comunicação por
áudio.

Dessa forma, o problema central do projeto consiste em:

> **Como projetar e implementar um assistente conversacional por voz
> capaz de interagir com um sistema de informação por meio de
> ferramentas estruturadas, mantendo propriedades de qualidade
> conversacional e permitindo a avaliação objetiva de sua capacidade de
> execução, recuperação de erros e eficiência de interação?**

------------------------------------------------------------------------

## 4. Objetivo Geral

Projetar, implementar e avaliar um assistente conversacional por voz
baseado em modelo de linguagem, capaz de interpretar solicitações em
linguagem natural e utilizar ferramentas estruturadas para interagir com
um sistema de informação de domínio definido.

A solução deverá ser construída de forma modular e desacoplada do
domínio, permitindo avaliar tanto a qualidade da interação
conversacional quanto a capacidade do agente de selecionar e executar
ferramentas corretamente.

------------------------------------------------------------------------

## 5. Objetivos Específicos

-   Investigar diretrizes e padrões de projeto aplicáveis a interfaces
    conversacionais por voz.
-   Investigar técnicas e paradigmas para uso de ferramentas por LLMs e
    agentes.
-   Consolidar requisitos funcionais e não funcionais derivados da
    literatura e das necessidades do sistema.
-   Modelar os principais casos de uso do assistente.
-   Definir uma arquitetura modular para separar a camada conversacional
    do sistema de informação.
-   Implementar um mecanismo de interação entre o agente e ferramentas
    disponibilizadas pelo domínio.
-   Implementar mecanismos de gerenciamento de contexto, preenchimento
    de parâmetros, tratamento de ambiguidades e recuperação de erros.
-   Definir cenários experimentais que representem diferentes condições
    de interação.
-   Definir métricas objetivas para avaliar a execução de tarefas, uso
    de ferramentas, eficiência conversacional, recuperação de erros e
    desempenho.
-   Avaliar experimentalmente o comportamento da solução no estudo de
    caso.
-   Analisar os resultados em relação aos requisitos e critérios
    estabelecidos.

------------------------------------------------------------------------

## 6. Visão da Solução

A solução será concebida como uma camada conversacional intermediária
entre o usuário e um sistema de informação.

De forma simplificada:

``` text
Usuário
   │
   │ voz
   ▼
┌───────────────────────┐
│ Reconhecimento de voz │
│        (STT)          │
└───────────┬───────────┘
            │ texto
            ▼
┌───────────────────────┐
│ Núcleo Conversacional │
│                       │
│ - interpretação       │
│ - contexto            │
│ - parâmetros          │
│ - ambiguidades        │
│ - diálogo             │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│    LLM / Agente       │
│                       │
│ seleção de ferramenta │
│ geração de argumentos │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│    Tool Interface     │
│                       │
│ ferramentas do domínio│
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ Sistema de Informação │
│      / Domínio        │
└───────────┬───────────┘
            │ resultado
            ▼
┌───────────────────────┐
│ Núcleo Conversacional │
└───────────┬───────────┘
            │ resposta
            ▼
┌───────────────────────┐
│ Síntese de voz (TTS)  │
└───────────┬───────────┘
            │
            ▼
         Usuário
```

A separação entre o núcleo conversacional e as ferramentas do domínio
constitui uma decisão arquitetural fundamental. O assistente não deve
depender diretamente das entidades e regras específicas do estudo de
caso; deve interagir com elas por meio de uma interface de ferramentas
explicitamente disponibilizadas.

------------------------------------------------------------------------

## 7. Stakeholders e Atores

### 7.1 Usuário

Pessoa que interage com o sistema utilizando linguagem natural,
principalmente por voz.

O usuário poderá:

-   consultar informações;
-   fornecer parâmetros;
-   responder a solicitações de esclarecimento;
-   corrigir informações;
-   confirmar ou cancelar operações;
-   receber resultados e mensagens de erro.

### 7.2 Sistema de Informação

Sistema responsável pelas entidades, dados e regras de negócio do
domínio utilizado no estudo de caso.

O sistema de informação não depende da interface conversacional para
executar suas responsabilidades.

### 7.3 Ferramentas

Operações estruturadas disponibilizadas ao agente para interação com o
sistema de informação.

Exemplos conceituais:

``` text
searchBooks()
getBook()
createBook()
updateBook()
borrowBook()
returnBook()
checkAvailability()
listLoans()
```

A definição definitiva das ferramentas será realizada posteriormente
durante a especificação do estudo de caso.

### 7.4 Pesquisador/Desenvolvedor

Responsável por projetar, implementar, instrumentar e avaliar a solução.

------------------------------------------------------------------------

## 8. Escopo

### 8.1 Incluído no escopo

A solução deverá contemplar, em nível de projeto:

-   entrada de voz;
-   reconhecimento de fala;
-   interpretação de linguagem natural;
-   gerenciamento de contexto conversacional;
-   extração e preenchimento de parâmetros;
-   tratamento de informações ausentes;
-   resolução de ambiguidades;
-   seleção de ferramentas;
-   geração e validação de argumentos;
-   execução de ferramentas;
-   processamento de resultados;
-   tratamento de erros;
-   recuperação conversacional;
-   confirmação de operações selecionadas;
-   cancelamento de operações quando aplicável;
-   geração de respostas;
-   síntese de voz;
-   registro das etapas relevantes da execução;
-   avaliação experimental baseada em cenários e métricas.

### 8.2 Fora do escopo inicial

Não fazem parte do escopo principal:

-   treinamento de um LLM próprio;
-   desenvolvimento de um novo modelo de reconhecimento ou síntese de
    voz;
-   sistemas multiagente complexos;
-   processamento multimodal além da voz;
-   reconhecimento avançado de emoções;
-   aprendizado por reforço para otimização do agente;
-   desenvolvimento de um sistema de informação completo e genérico;
-   comparação exaustiva entre múltiplos LLMs, salvo se posteriormente
    definido como parte do protocolo experimental;
-   implementação de mecanismos avançados de autonomia não necessários
    para demonstrar os objetivos do trabalho.

Esses elementos podem ser discutidos como trabalhos futuros, mas não
constituem requisitos necessários para a primeira versão da solução.

------------------------------------------------------------------------

## 9. Requisitos de Alto Nível

Os requisitos detalhados serão mantidos em uma matriz de requisitos
específica. Este documento registra apenas sua visão consolidada.

### 9.1 Requisitos Funcionais

  -----------------------------------------------------------------------
  ID                                  Requisito
  ----------------------------------- -----------------------------------
  RF01                                O assistente deve receber
                                      solicitações de voz e
                                      disponibilizar sua representação
                                      textual para processamento.

  RF02                                O assistente deve interpretar
                                      solicitações em linguagem natural
                                      para identificar intenções e
                                      parâmetros relevantes.

  RF03                                O assistente deve manter
                                      informações relevantes do contexto
                                      conversacional.

  RF04                                O assistente deve resolver
                                      referências contextuais quando sua
                                      interpretação for suficientemente
                                      determinada.

  RF05                                O assistente deve identificar os
                                      parâmetros relevantes para a
                                      operação solicitada.

  RF06                                O assistente deve utilizar
                                      múltiplos parâmetros fornecidos em
                                      uma mesma solicitação,
                                      independentemente de sua ordem.

  RF07                                O assistente deve solicitar
                                      informações necessárias que não
                                      tenham sido fornecidas.

  RF08                                O assistente deve detectar e tratar
                                      solicitações ambíguas antes da
                                      execução de operações que dependam
                                      de esclarecimento.

  RF09                                O assistente deve selecionar uma
                                      ferramenta disponível que
                                      corresponda à intenção
                                      identificada.

  RF10                                O assistente deve gerar os
                                      argumentos necessários para a
                                      ferramenta selecionada.

  RF11                                O assistente deve validar os
                                      argumentos antes da execução da
                                      ferramenta.

  RF12                                O assistente deve executar
                                      ferramentas explicitamente
                                      disponibilizadas pelo domínio.

  RF13                                O assistente deve utilizar os
                                      resultados das ferramentas para
                                      determinar a continuidade da
                                      interação e gerar a resposta ao
                                      usuário.

  RF14                                O assistente deve informar falhas
                                      de execução de forma compatível com
                                      o resultado efetivamente obtido.

  RF15                                O assistente deve permitir a
                                      recuperação de tarefas quando novas
                                      informações possibilitarem sua
                                      continuidade segura.

  RF16                                O assistente deve permitir o
                                      cancelamento de operações em
                                      andamento quando aplicável.

  RF17                                O assistente deve permitir o
                                      encerramento da interação mediante
                                      comandos apropriados.

  RF18                                O assistente deve solicitar
                                      confirmação para operações
                                      previamente definidas como
                                      dependentes de confirmação.

  RF19                                O assistente deve gerar uma
                                      resposta textual correspondente ao
                                      estado da interação e ao resultado
                                      da operação.

  RF20                                O assistente deve converter a
                                      resposta textual em áudio para
                                      apresentação ao usuário.
  -----------------------------------------------------------------------

### 9.2 Requisitos Não Funcionais

  -----------------------------------------------------------------------
  ID                                  Requisito
  ----------------------------------- -----------------------------------
  RNF01                               A camada conversacional deve ser
                                      desacoplada das entidades e regras
                                      específicas do domínio.

  RNF02                               Os principais componentes do
                                      sistema devem possuir
                                      responsabilidades modularmente
                                      definidas.

  RNF03                               A execução das interações deve ser
                                      observável por meio de registros
                                      estruturados.

  RNF04                               Cada execução deve possuir
                                      identificação que permita rastrear
                                      seus principais eventos.

  RNF05                               O sistema deve evitar a execução de
                                      operações quando intenção ou
                                      parâmetros necessários não forem
                                      suficientemente determinados.

  RNF06                               As operações realizadas pelo
                                      assistente devem respeitar as
                                      regras de negócio do sistema de
                                      informação.

  RNF07                               Falhas de ferramentas ou
                                      componentes externos não devem
                                      resultar em respostas que indiquem
                                      sucesso inexistente.

  RNF08                               O sistema deve permitir a medição
                                      da latência das principais etapas
                                      de processamento.

  RNF09                               As respostas devem apresentar
                                      informações suficientes para
                                      comunicar o resultado sem
                                      sobrecarga desnecessária.

  RNF10                               O sistema deve preservar
                                      informações necessárias à
                                      recuperação de tarefas quando isso
                                      for seguro e aplicável.

  RNF11                               O agente deve ter acesso somente às
                                      operações explicitamente
                                      disponibilizadas pela interface de
                                      ferramentas.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 10. Princípios de Projeto

Os requisitos são orientados por princípios identificados na literatura
e consolidados durante o levantamento bibliográfico.

### 10.1 Contexto

O sistema deve preservar informações relevantes entre turnos e
utilizá-las para interpretar referências e dar continuidade às tarefas.

### 10.2 Preenchimento de parâmetros

O sistema deve permitir que o usuário forneça informações em linguagem
natural, inclusive múltiplos parâmetros em um mesmo turno, solicitando
somente os dados necessários que estiverem ausentes.

### 10.3 Tratamento de ambiguidades

A solução deve evitar executar operações quando a solicitação não
fornecer informação suficiente para determinar com segurança a intenção
ou a entidade envolvida.

### 10.4 Feedback

O sistema deve comunicar claramente o resultado das operações,
distinguindo sucesso, falha, necessidade de informação adicional e
necessidade de confirmação.

### 10.5 Recuperação de erros

Erros de interpretação, validação ou execução devem ser tratados como
estados recuperáveis sempre que possível, permitindo continuidade da
tarefa.

### 10.6 Controle do usuário

O usuário deve possuir meios para interromper, cancelar ou encerrar a
interação quando aplicável.

### 10.7 Carga cognitiva

As respostas de voz devem evitar apresentações excessivamente longas e
utilizar segmentação ou revelação progressiva quando a quantidade de
informação exigir.

### 10.8 Uso de ferramentas

A interação do LLM com o domínio deve ocorrer por ferramentas
explicitamente definidas, com seleção, argumentos e resultados passíveis
de observação e avaliação.

### 10.9 Desacoplamento

O núcleo conversacional deve ser concebido de maneira independente do
domínio, permitindo substituir ou ampliar o conjunto de ferramentas sem
reestruturar integralmente a lógica conversacional.

------------------------------------------------------------------------

## 11. Critérios de Avaliação

Os critérios de avaliação serão especificados detalhadamente no
protocolo experimental. Em nível de visão, a solução deverá permitir a
análise de pelo menos quatro dimensões.

### 11.1 Sucesso da tarefa

Verificar se a solicitação original foi efetivamente concluída.

Indicador principal:

-   Task Success Rate.

### 11.2 Uso de ferramentas

Verificar se o agente selecionou e utilizou corretamente as ferramentas.

Indicadores:

-   Tool Selection Accuracy;
-   Argument Accuracy;
-   Call Correctness;
-   Tool Execution Success Rate;
-   taxa de chamadas para ferramentas inexistentes.

### 11.3 Qualidade da interação

Verificar a capacidade do assistente de lidar com:

-   contexto;
-   informações incompletas;
-   ambiguidades;
-   erros;
-   recuperação;
-   confirmações;
-   quantidade de turnos.

Indicadores possíveis:

-   Context Retention Rate;
-   One-Turn Completion Rate;
-   Ambiguity Resolution Rate;
-   Recovery Rate;
-   Turns per Task;
-   Reprompts per Task.

### 11.4 Desempenho

Caracterizar o tempo necessário para processar uma solicitação.

Indicadores:

-   latência fim a fim;
-   latência do reconhecimento de voz;
-   latência de processamento do LLM;
-   latência da ferramenta;
-   latência da síntese de voz.

As métricas definitivas, fórmulas e critérios de sucesso serão
estabelecidos no artefato de Protocolo de Avaliação.

------------------------------------------------------------------------

## 12. Restrições e Premissas

### 12.1 Restrições

-   O sistema deverá utilizar um modelo de linguagem como componente de
    interpretação/orquestração.
-   A interação com o domínio deverá ocorrer por meio de ferramentas
    estruturadas.
-   O sistema deverá permitir observação das principais etapas de
    execução.
-   A avaliação deverá ser realizada sobre cenários controlados e
    reproduzíveis.
-   A implementação deverá permanecer compatível com o escopo e os
    recursos disponíveis para o TCC.

### 12.2 Premissas

-   O sistema de informação do estudo de caso disponibilizará operações
    que possam ser encapsuladas como ferramentas.
-   As ferramentas possuirão contratos suficientemente definidos para
    permitir validação de argumentos.
-   O LLM utilizado será capaz de realizar chamadas estruturadas de
    ferramentas.
-   O ambiente experimental permitirá registrar as entradas, decisões,
    chamadas e resultados necessários à avaliação.

------------------------------------------------------------------------

## 13. Riscos Principais

  -----------------------------------------------------------------------
  Risco                   Impacto                 Mitigação inicial
  ----------------------- ----------------------- -----------------------
  Interpretação incorreta Alto                    Cenários controlados,
  da solicitação                                  validação e reprompting

  Seleção incorreta de    Alto                    Registro de
  ferramenta                                      ferramentas, schemas e
                                                  métricas específicas

  Argumentos inválidos    Alto                    Validação antes da
                                                  execução

  Alucinação de           Alto                    Tool Registry fechado e
  ferramentas                                     validação

  Perda de contexto       Médio/alto              Estado conversacional
                                                  explícito e cenários
                                                  multi-turno

  Respostas incompatíveis Alto                    Resultados estruturados
  com resultados reais                            e verificação de
                                                  grounding

  Latência elevada        Médio                   Instrumentação por
                                                  componente e
                                                  caracterização

  Sobrecarga auditiva     Médio                   Respostas concisas,
                                                  chunking e progressive
                                                  disclosure

  Falha de componentes    Médio                   Tratamento de erros e
  externos                                        recuperação

  Crescimento excessivo   Alto                    Priorização dos
  do escopo                                       requisitos e
                                                  delimitação do estudo
                                                  de caso
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 14. Artefatos Relacionados

Este Documento de Visão será complementado pelos seguintes artefatos:

1.  **Matriz de Requisitos** --- especificação detalhada dos RFs, RNFs e
    rastreabilidade.
2.  **Modelo de Casos de Uso** --- identificação das principais
    funcionalidades e interações.
3.  **Especificação dos Casos de Uso** --- fluxos principais,
    alternativos e de exceção.
4.  **Especificação do Estudo de Caso** --- definição do domínio e das
    operações disponibilizadas.
5.  **Especificação das Ferramentas** --- contratos, parâmetros e
    resultados das tools.
6.  **Documento/Modelo de Arquitetura** --- estrutura dos componentes e
    suas responsabilidades.
7.  **Matriz de Cenários Experimentais** --- situações utilizadas para
    exercitar os requisitos.
8.  **Protocolo de Avaliação** --- métricas, procedimentos, dados
    coletados e critérios de análise.
9.  **Relatório de Resultados** --- resultados e análise dos
    experimentos.

------------------------------------------------------------------------

## 15. Rastreabilidade em Alto Nível

A estratégia de engenharia adotada busca manter uma cadeia de
rastreabilidade desde a literatura até a avaliação experimental:

``` text
Literatura
    │
    ▼
Diretrizes e padrões
    │
    ▼
Requisitos
    │
    ▼
Casos de uso
    │
    ▼
Decisões arquiteturais
    │
    ▼
Implementação
    │
    ▼
Cenários experimentais
    │
    ▼
Dados coletados
    │
    ▼
Métricas
    │
    ▼
Resultados
```

Essa rastreabilidade permitirá relacionar as decisões de projeto às
evidências utilizadas para justificá-las e aos critérios empregados para
verificar seu comportamento.

------------------------------------------------------------------------

## 16. Evolução do Documento

Este documento representa a visão inicial do projeto. Os requisitos,
casos de uso, arquitetura e critérios de avaliação poderão ser refinados
durante as iterações do Processo Unificado, desde que as alterações
sejam justificadas e sua rastreabilidade seja preservada.

As decisões tecnológicas específicas, como modelo de linguagem,
framework de integração, provedor de reconhecimento/síntese de voz e
implementação das ferramentas, não fazem parte da visão e serão
definidas nos artefatos técnicos correspondentes.

------------------------------------------------------------------------

## 17. Referências de Base

O Documento de Visão foi elaborado considerando, entre outras fontes
levantadas no projeto:

-   Murad, C. (2023). *Meta-Analysis of Guidelines for Voice User
    Interface Design*.
-   Fadhil, A. *Domain Specific Design Patterns: Designing For
    Conversational User Interfaces*.
-   Park & Kim. Trabalhos sobre padrões e diretrizes para interfaces
    conversacionais.
-   Schick, T. et al. (2023). *Toolformer: Language Models Can Teach
    Themselves to Use Tools*.
-   Li, M. et al. (2023). *API-Bank: A Comprehensive Benchmark for
    Tool-Augmented LLMs*.
-   Patil, S. G. et al. (2024). *Gorilla: Large Language Model Connected
    with Massive APIs*.
-   Hu, J. et al. (2026). *Agentic Tool Use in Large Language Models: A
    Survey*.

As referências bibliográficas completas e a análise de sua contribuição
para os requisitos são mantidas na documentação bibliográfica do
projeto.

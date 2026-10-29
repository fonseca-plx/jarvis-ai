# Requisitos Funcionais Consolidados

**Projeto:** TCC --- Assistente Conversacional por Voz para Interação
com Sistemas de Informação\
**Versão:** 1.0\
**Status:** Em elaboração\
**Data:** 2026-09-16

---

## RF01 — Receber e processar entrada de voz

O sistema deve receber uma solicitação do usuário por meio de entrada de voz e disponibilizar uma representação textual da fala para processamento conversacional.

**Compreende:** captura da entrada de voz e sua transcrição.

**Observação:** a tecnologia específica de reconhecimento de fala permanece uma decisão de projeto.

---

## RF02 — Interpretar solicitação em linguagem natural

O sistema deve interpretar a solicitação do usuário, identificando a intenção e as informações relevantes para a realização da tarefa.

A interpretação deve considerar o conteúdo da solicitação e, quando aplicável, informações disponíveis no contexto da conversa.

**Origem principal:** RF02 e RF05 da versão anterior.

---

## RF03 — Gerenciar contexto conversacional

O sistema deve manter e utilizar informações relevantes de interações anteriores para permitir a continuidade de uma tarefa ao longo da conversa.

O gerenciamento de contexto deve permitir a resolução de referências a informações previamente mencionadas, quando estas puderem ser determinadas de forma adequada.

**Origem principal:** RF03 e RF04 da versão anterior.

---

## RF04 — Gerenciar informações necessárias à tarefa

O sistema deve identificar, utilizar e solicitar as informações necessárias para realizar uma tarefa conversacional.

Esse comportamento deve contemplar:

- extração de parâmetros fornecidos pelo usuário;
- utilização de múltiplos parâmetros apresentados na mesma interação;
- identificação de informações ausentes;
- solicitação de informações faltantes;
- identificação de informações ambíguas;
- solicitação de esclarecimento quando a informação disponível não for suficiente para determinar a operação pretendida.

**Origem principal:** RF05, RF06, RF07 e RF08 da versão anterior.

---

## RF05 — Selecionar ferramenta para execução da tarefa

O sistema deve selecionar, entre as ferramentas disponibilizadas ao agente, aquela que seja compatível com a intenção e com as informações determinadas para a tarefa.

O sistema não deve executar uma ferramenta quando não houver informação suficiente para determinar adequadamente a operação pretendida.

**Origem principal:** RF09, com incorporação do comportamento de segurança anteriormente descrito em RF08/RNF05.

---

## RF06 — Preparar e validar chamada de ferramenta

O sistema deve gerar os argumentos necessários para a ferramenta selecionada e validar esses argumentos antes da execução.

A validação deve verificar, conforme as informações disponíveis e as regras da ferramenta, aspectos como:

- presença de parâmetros obrigatórios;
- correspondência entre parâmetros e valores identificados;
- formato esperado dos argumentos;
- consistência das informações necessárias à execução.

**Origem principal:** RF10 e RF11 da versão anterior.

---

## RF07 — Executar ferramentas autorizadas

O sistema deve executar ferramentas explicitamente disponibilizadas ao agente para realização das operações suportadas pelo sistema de informação de domínio.

A execução deve respeitar as interfaces e regras estabelecidas para cada ferramenta.

**Origem principal:** RF12 da versão anterior.

---

## RF08 — Processar resultados das ferramentas

O sistema deve interpretar os resultados retornados pelas ferramentas e utilizá-los para dar continuidade à tarefa conversacional e produzir uma resposta coerente com o resultado obtido.

**Origem principal:** RF13 da versão anterior.

---

## RF09 — Tratar falhas e recuperar a interação

O sistema deve identificar falhas ocorridas durante a realização de uma tarefa e comunicar adequadamente a situação ao usuário.

Quando houver informações suficientes para continuar a tarefa após uma falha, correção ou nova informação fornecida pelo usuário, o sistema deve permitir a retomada da interação sem exigir, quando possível, que toda a tarefa seja reiniciada.

**Origem principal:** RF14 e RF15 da versão anterior.

---

## RF10 — Gerenciar controle da interação

O sistema deve permitir ao usuário controlar o andamento da interação, contemplando, quando aplicável:

- confirmação de operações que exijam confirmação;
- cancelamento de operações ainda canceláveis;
- encerramento da conversa.

**Origem principal:** RF16, RF17 e RF18 da versão anterior.

---

## RF11 — Gerar resposta textual

O sistema deve gerar uma resposta textual correspondente à solicitação do usuário, ao estado da tarefa ou ao resultado da operação realizada.

A resposta deve representar de forma coerente o resultado efetivamente obtido pelo sistema e pelas ferramentas utilizadas.

**Origem principal:** RF19 da versão anterior.

---

## RF12 — Entregar resposta por voz

O sistema deve converter a resposta textual em saída de voz e disponibilizá-la ao usuário como resposta à interação.

**Origem principal:** RF20 da versão anterior.

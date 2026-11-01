# UC04 — Realizar devolução

**Identificador:** UC04  
**Nome:** Realizar devolução  
**Versão:** 1.0  
**Prioridade:** Alta

## 1. Objetivo

Permitir que o usuário realize a devolução de um recurso emprestado por meio de uma solicitação em linguagem natural ou voz.

O sistema deve interpretar a solicitação, identificar o recurso ou empréstimo correspondente, utilizar o contexto conversacional quando aplicável, determinar as informações necessárias para a operação, executar a devolução por meio de uma ferramenta compatível e comunicar ao usuário o resultado efetivamente obtido.

## 2. Atores

### Ator principal
- **Usuário:** solicita a realização da devolução.

### Atores secundários
- **Sistema de Informação de Domínio:** mantém os dados e regras relacionados aos empréstimos e devoluções.
- **Ferramenta:** disponibiliza a operação de devolução ao assistente.

## 3. Gatilho

O usuário solicita a realização de uma devolução, por exemplo:

> “Quero devolver Clean Code.”

ou, após uma interação anterior:

> “Pode devolver esse livro.”

## 4. Pré-condições

- O assistente está disponível para interação.
- A operação de devolução está disponível por meio de uma ferramenta autorizada.
- A interface da ferramenta e seus parâmetros estão previamente definidos.
- O sistema de informação de domínio está disponível para execução da operação.
- O recurso ou empréstimo a ser devolvido pode ser identificado a partir da solicitação ou do contexto conversacional.
- As informações necessárias para a operação podem ser determinadas ou solicitadas ao usuário.

## 5. Pós-condições

### Em caso de sucesso

- O recurso ou empréstimo correspondente é identificado.
- As informações necessárias para a operação são determinadas.
- A operação de devolução é executada.
- O resultado da operação é processado.
- O usuário é informado sobre o resultado efetivamente obtido.

### Em caso de falha ou cancelamento

- A devolução não é considerada concluída.
- A situação é comunicada ao usuário de forma adequada, quando identificável.
- O sistema não informa que a devolução foi realizada caso a operação não tenha sido efetivamente concluída.
- Quando possível, o usuário pode fornecer novas informações, corrigir a solicitação ou realizar uma nova tentativa sem reiniciar necessariamente toda a interação.

## 6. Fluxo principal

1. O usuário solicita a realização de uma devolução.
2. O sistema recebe e interpreta a solicitação.
3. O sistema identifica o recurso ou empréstimo que o usuário deseja devolver.
4. O sistema determina as informações necessárias para realizar a operação, utilizando o contexto conversacional quando aplicável.
5. O sistema verifica se todas as informações necessárias para a operação estão disponíveis.
6. O sistema determina a operação de domínio necessária e seleciona uma ferramenta compatível.
7. O sistema prepara e valida os parâmetros da chamada da ferramenta.
8. O sistema executa a operação por meio da ferramenta selecionada.
9. O sistema processa o resultado retornado pela ferramenta.
10. O sistema gera uma resposta correspondente ao resultado da operação.
11. O sistema entrega a resposta ao usuário por voz.
12. O caso de uso é encerrado.

## 7. Fluxos alternativos

### FA01 — Recurso ou empréstimo não especificado

**No passo 3**, o sistema não consegue determinar qual recurso ou empréstimo o usuário deseja devolver.

1. O sistema solicita ao usuário a identificação do recurso ou empréstimo.
2. O usuário fornece a identificação.
3. O sistema retoma o fluxo a partir do passo 3.

### FA02 — Recurso identificado pelo contexto

**No passo 3**, o usuário não identifica explicitamente o recurso, mas existe uma referência determinável a partir da conversa anterior.

1. O sistema utiliza o contexto conversacional para identificar o recurso ou empréstimo.
2. O fluxo prossegue a partir do passo 4.

### FA03 — Informações necessárias ausentes

**No passo 5**, o sistema identifica que uma ou mais informações necessárias para realizar a operação não estão disponíveis.

1. O sistema solicita ao usuário as informações faltantes.
2. O usuário fornece as informações solicitadas.
3. O sistema valida as novas informações.
4. O fluxo prossegue a partir do passo 6.

### FA04 — Correção da solicitação

Durante a identificação ou preparação da operação, o usuário corrige ou substitui alguma informação.

1. O sistema atualiza as informações consideradas para a operação.
2. O sistema reavalia os parâmetros necessários.
3. O fluxo prossegue a partir do ponto correspondente à nova situação.

### FA05 — Cancelamento da operação

Antes da execução da devolução, o usuário solicita o cancelamento.

1. O sistema não executa a operação.
2. O sistema informa que a devolução não foi realizada.
3. O caso de uso é encerrado.

## 8. Fluxos de exceção

### FE01 — Recurso ou empréstimo não encontrado

**No passo 3 ou durante a execução de consulta necessária**, o recurso ou empréstimo informado pelo usuário não é encontrado.

1. O sistema informa que não conseguiu localizar o recurso ou empréstimo correspondente.
2. O sistema pode solicitar uma nova identificação ou permitir que o usuário encerre a interação.
3. A operação de devolução não é executada.

### FE02 — Devolução não permitida

**No passo 8**, o sistema de domínio informa que a devolução não pode ser realizada devido a alguma condição ou regra de negócio.

1. O sistema processa a informação retornada.
2. O sistema informa ao usuário que a devolução não foi realizada e comunica a condição apresentada pelo sistema de domínio, quando disponível.
3. O caso de uso pode ser encerrado ou retomado caso seja possível corrigir a condição.

### FE03 — Ferramenta de devolução indisponível

**No passo 6**, nenhuma ferramenta autorizada capaz de realizar a operação está disponível.

1. O sistema informa que não pode realizar a devolução naquele momento.
2. A operação não é executada.
3. O caso de uso é encerrado.

### FE04 — Falha na validação dos parâmetros

**No passo 7**, os parâmetros necessários para a chamada da ferramenta não podem ser validados.

1. O sistema não executa a ferramenta.
2. O sistema identifica a necessidade de correção ou obtenção das informações necessárias.
3. Quando possível, o sistema solicita as informações necessárias ao usuário.
4. O fluxo pode ser retomado após a correção.

### FE05 — Falha na execução da ferramenta

**No passo 8**, ocorre uma falha durante a execução da ferramenta.

1. O sistema identifica a falha.
2. O sistema informa ao usuário que não conseguiu concluir a devolução.
3. O sistema não informa a devolução como concluída.
4. Quando possível, o usuário pode tentar novamente.

### FE06 — Resultado inconclusivo

**No passo 9**, o resultado retornado pela ferramenta não permite determinar se a devolução foi efetivamente realizada.

1. O sistema não afirma que a devolução foi concluída.
2. O sistema informa ao usuário que não foi possível confirmar o resultado da operação.
3. O caso de uso é encerrado ou encaminhado para uma nova tentativa, quando aplicável.

## 9. Regras do caso de uso

**RN01 — Identificação suficiente:** o sistema não deve executar a operação enquanto não conseguir determinar suficientemente o recurso ou empréstimo e as demais informações necessárias à devolução.

**RN02 — Uso do contexto:** informações previamente estabelecidas na conversa podem ser utilizadas quando permitirem determinar inequivocamente o recurso ou empréstimo correspondente.

**RN03 — Respeito às regras de domínio:** as condições que determinam se a devolução pode ou não ser realizada devem ser verificadas pelo sistema de informação de domínio.

**RN04 — Ferramentas autorizadas:** a operação somente pode ser executada por meio de ferramenta previamente disponibilizada e autorizada pelo sistema.

**RN05 — Fidelidade ao resultado:** o sistema deve comunicar o resultado efetivamente retornado pela operação, não podendo declarar a devolução como concluída quando a execução tiver falhado ou o resultado for inconclusivo.

**RN06 — Recuperação:** quando a falha puder ser corrigida por meio de novas informações ou correção da solicitação, o sistema deve permitir a continuidade da interação sem exigir necessariamente o reinício completo da tarefa.

## 10. Requisitos relacionados

- **RF01 — Receber e processar entrada de voz**
- **RF02 — Interpretar solicitação em linguagem natural**
- **RF03 — Gerenciar contexto conversacional**
- **RF04 — Gerenciar informações necessárias à tarefa**
- **RF05 — Selecionar ferramenta para execução da tarefa**
- **RF06 — Preparar e validar chamada de ferramenta**
- **RF07 — Executar ferramentas autorizadas**
- **RF08 — Processar resultados das ferramentas**
- **RF09 — Tratar falhas e recuperar a interação**
- **RF10 — Gerenciar controle da interação**
- **RF11 — Gerar resposta textual**
- **RF12 — Entregar resposta por voz**

## 11. Cenários de verificação

| ID | Cenário | Situação esperada |
|---|---|---|
| C04.01 | Devolução com recurso claramente identificado | A devolução é realizada e o resultado é comunicado |
| C04.02 | Recurso identificado pelo contexto | O sistema utiliza o recurso previamente estabelecido |
| C04.03 | Recurso ou empréstimo não especificado | O sistema solicita a identificação |
| C04.04 | Informação necessária ausente | O sistema solicita a informação faltante |
| C04.05 | Solicitação com múltiplas informações | O sistema identifica e utiliza corretamente as informações fornecidas |
| C04.06 | Recurso ou empréstimo inexistente | O sistema informa que não localizou o recurso ou empréstimo |
| C04.07 | Devolução não permitida pelo domínio | O sistema comunica a condição e não declara a devolução como realizada |
| C04.08 | Ferramenta indisponível | A operação não é executada e a indisponibilidade é comunicada |
| C04.09 | Parâmetros inválidos ou incompletos | A chamada não é executada até que os parâmetros possam ser corrigidos |
| C04.10 | Falha na execução da ferramenta | O sistema informa que não conseguiu concluir a devolução |
| C04.11 | Resultado inconclusivo | O sistema não afirma que a devolução foi realizada |
| C04.12 | Correção de informação durante a interação | O sistema atualiza a solicitação e prossegue sem reiniciar a tarefa |
| C04.13 | Cancelamento da operação | O sistema encerra a operação sem executar a devolução |

## 12. Observações de modelagem

Este caso de uso representa o **objetivo do usuário de realizar uma devolução**, e não os mecanismos internos utilizados para isso.

A identificação da solicitação, o gerenciamento de contexto, a preparação de parâmetros, a seleção de ferramenta e o processamento do resultado são descritos apenas no nível necessário para representar o comportamento observável do sistema. A forma concreta como essas atividades serão implementadas deve ser definida posteriormente na arquitetura e no projeto do sistema.

A ferramenta responsável pela operação também não é especificada neste caso de uso em termos de nome, API ou implementação. Sua granularidade e organização constituem decisões de projeto arquitetural.
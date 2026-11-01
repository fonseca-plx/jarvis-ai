# UC05 — Consultar empréstimos

**Identificador:** UC05  
**Nome:** Consultar empréstimos  
**Versão:** 1.0  
**Prioridade:** Alta

## 1. Objetivo

Permitir que o usuário consulte informações sobre seus empréstimos por meio de uma solicitação em linguagem natural ou voz.

O sistema deve interpretar a solicitação, identificar o conjunto de empréstimos ou o empréstimo específico ao qual o usuário se refere, determinar as informações necessárias à consulta, utilizar o contexto conversacional quando aplicável, executar uma consulta por meio de uma ferramenta compatível e comunicar ao usuário os resultados efetivamente obtidos.

## 2. Atores

### Ator principal
- **Usuário:** solicita informações sobre seus empréstimos.

### Atores secundários
- **Sistema de Informação de Domínio:** mantém os dados relacionados aos empréstimos.
- **Ferramenta:** disponibiliza operações de consulta de empréstimos ao assistente.

## 3. Gatilho

O usuário solicita informações sobre seus empréstimos, por exemplo:

> “Quais livros estão emprestados comigo?”

ou:

> “Quando tenho que devolver Clean Code?”

ou, após uma consulta anterior:

> “E quando eu preciso devolver esse?”

## 4. Pré-condições

- O assistente está disponível para interação.
- As operações necessárias para consulta de empréstimos estão disponíveis por meio de ferramentas autorizadas.
- As interfaces das ferramentas e seus parâmetros estão previamente definidos.
- O sistema de informação de domínio está disponível para consulta.
- O usuário ou o conjunto de empréstimos a ser consultado pode ser determinado a partir da solicitação e do contexto da interação.

## 5. Pós-condições

### Em caso de sucesso

- O escopo da consulta é identificado.
- As informações necessárias para a consulta são determinadas.
- A consulta é executada.
- O resultado retornado pelo sistema de domínio é processado.
- O usuário é informado sobre os resultados obtidos.

### Em caso de falha

- A consulta não é considerada concluída quando seu resultado não puder ser determinado.
- A situação é comunicada ao usuário de forma adequada.
- O sistema não apresenta informações de empréstimos que não tenham sido efetivamente obtidas.
- Quando possível, o usuário pode fornecer novas informações ou corrigir a solicitação sem reiniciar necessariamente toda a interação.

## 6. Fluxo principal

1. O usuário solicita informações sobre seus empréstimos.
2. O sistema recebe e interpreta a solicitação.
3. O sistema identifica o escopo da consulta, determinando se o usuário deseja consultar seus empréstimos em geral ou um empréstimo específico.
4. O sistema determina as informações necessárias para realizar a consulta, utilizando o contexto conversacional quando aplicável.
5. O sistema verifica se todas as informações necessárias estão disponíveis.
6. O sistema determina a operação de consulta necessária e seleciona uma ferramenta compatível.
7. O sistema prepara e valida os parâmetros da chamada da ferramenta.
8. O sistema executa a consulta por meio da ferramenta selecionada.
9. O sistema processa o resultado retornado pela ferramenta.
10. O sistema gera uma resposta correspondente ao resultado da consulta.
11. O sistema entrega a resposta ao usuário por voz.
12. O caso de uso é encerrado.

## 7. Fluxos alternativos

### FA01 — Escopo da consulta não especificado

**No passo 3**, o sistema não consegue determinar se o usuário deseja consultar os empréstimos em geral ou um empréstimo específico.

1. O sistema solicita esclarecimento sobre o escopo desejado.
2. O usuário especifica o que deseja consultar.
3. O sistema retoma o fluxo a partir do passo 3.

### FA02 — Empréstimo identificado pelo contexto

**No passo 3**, o usuário não identifica explicitamente o empréstimo, mas existe uma referência determinável a partir da conversa anterior.

1. O sistema utiliza o contexto conversacional para identificar o empréstimo correspondente.
2. O fluxo prossegue a partir do passo 4.

### FA03 — Informação necessária ausente

**No passo 5**, o sistema identifica que uma ou mais informações necessárias para realizar a consulta não estão disponíveis.

1. O sistema solicita ao usuário as informações faltantes.
2. O usuário fornece as informações solicitadas.
3. O sistema valida as novas informações.
4. O fluxo prossegue a partir do passo 6.

### FA04 — Refinamento da consulta

Durante a interação, o usuário restringe ou modifica o escopo da consulta.

Por exemplo:

> “Quais livros estão emprestados comigo?”

> “Só os que estão atrasados.”

1. O sistema atualiza os critérios da consulta.
2. O sistema determina os novos parâmetros necessários.
3. O sistema prossegue com a consulta considerando os critérios atualizados.

### FA05 — Correção da solicitação

Durante a identificação ou preparação da consulta, o usuário corrige alguma informação.

1. O sistema atualiza as informações consideradas para a consulta.
2. O sistema reavalia os parâmetros necessários.
3. O fluxo prossegue a partir do ponto correspondente à nova situação.

### FA06 — Cancelamento da consulta

Antes da execução da consulta, o usuário solicita o cancelamento.

1. O sistema não executa a consulta.
2. O sistema informa que a consulta foi cancelada.
3. O caso de uso é encerrado.

## 8. Fluxos de exceção

### FE01 — Empréstimo não encontrado

**No passo 8**, a consulta não encontra empréstimo correspondente aos critérios determinados.

1. O sistema informa que não foram encontrados empréstimos correspondentes à consulta.
2. O caso de uso é encerrado ou a consulta pode ser refinada pelo usuário.

### FE02 — Ferramenta de consulta indisponível

**No passo 6**, nenhuma ferramenta autorizada capaz de realizar a consulta está disponível.

1. O sistema informa que não pode realizar a consulta naquele momento.
2. A consulta não é executada.
3. O caso de uso é encerrado.

### FE03 — Falha na validação dos parâmetros

**No passo 7**, os parâmetros necessários para a chamada da ferramenta não podem ser validados.

1. O sistema não executa a ferramenta.
2. O sistema identifica a necessidade de correção ou obtenção das informações necessárias.
3. Quando possível, o sistema solicita as informações necessárias ao usuário.
4. O fluxo pode ser retomado após a correção.

### FE04 — Falha na execução da ferramenta

**No passo 8**, ocorre uma falha durante a execução da ferramenta.

1. O sistema identifica a falha.
2. O sistema informa ao usuário que não conseguiu realizar a consulta.
3. O sistema não apresenta resultados como se a consulta tivesse sido realizada.
4. Quando possível, o usuário pode tentar novamente.

### FE05 — Resultado inconclusivo

**No passo 9**, o resultado retornado pela ferramenta não permite determinar adequadamente as informações solicitadas.

1. O sistema não apresenta informações que não possam ser determinadas a partir do resultado.
2. O sistema informa ao usuário que não foi possível obter ou interpretar adequadamente o resultado da consulta.
3. O caso de uso é encerrado ou encaminhado para uma nova tentativa, quando aplicável.

## 9. Regras do caso de uso

**RN01 — Identificação suficiente:** o sistema não deve executar a consulta enquanto não conseguir determinar suficientemente o escopo e os demais critérios necessários à operação.

**RN02 — Uso do contexto:** informações previamente estabelecidas na conversa podem ser utilizadas quando permitirem determinar inequivocamente o empréstimo ou os critérios da consulta.

**RN03 — Escopo do usuário:** a consulta deve considerar os empréstimos associados ao usuário que realiza a solicitação.

**RN04 — Refinamento da consulta:** novos critérios fornecidos durante a interação podem restringir ou modificar a consulta anteriormente determinada.

**RN05 — Ferramentas autorizadas:** a consulta somente pode ser executada por meio de ferramenta previamente disponibilizada e autorizada pelo sistema.

**RN06 — Fidelidade ao resultado:** o sistema deve comunicar informações efetivamente obtidas na consulta, não podendo inventar ou inferir como fato informações que não estejam sustentadas pelo resultado retornado.

**RN07 — Recuperação:** quando a falha puder ser corrigida por meio de novas informações ou correção da solicitação, o sistema deve permitir a continuidade da interação sem exigir necessariamente o reinício completo da tarefa.

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
| C05.01 | Consulta dos empréstimos do usuário | O sistema retorna os empréstimos correspondentes |
| C05.02 | Consulta de empréstimo específico | O sistema retorna as informações do empréstimo solicitado |
| C05.03 | Empréstimo identificado pelo contexto | O sistema utiliza corretamente o empréstimo estabelecido anteriormente |
| C05.04 | Escopo da consulta não especificado | O sistema solicita esclarecimento |
| C05.05 | Informação necessária ausente | O sistema solicita a informação faltante |
| C05.06 | Refinamento da consulta | O sistema incorpora os novos critérios |
| C05.07 | Correção de informação durante a interação | O sistema atualiza a consulta |
| C05.08 | Nenhum empréstimo correspondente | O sistema informa que não encontrou resultados |
| C05.09 | Ferramenta indisponível | A consulta não é executada e a indisponibilidade é comunicada |
| C05.10 | Parâmetros inválidos ou incompletos | A chamada não é executada até que os parâmetros possam ser corrigidos |
| C05.11 | Falha na execução da ferramenta | O sistema informa que não conseguiu realizar a consulta |
| C05.12 | Resultado inconclusivo | O sistema não apresenta informações não sustentadas pelo resultado |
| C05.13 | Cancelamento da consulta | O sistema encerra a consulta sem executá-la |

## 12. Observações de modelagem

Este caso de uso representa o **objetivo do usuário de consultar seus empréstimos**, e não os mecanismos internos utilizados para realizar a consulta.

A identificação do escopo, o gerenciamento de contexto, a preparação de parâmetros, a seleção da ferramenta e o processamento do resultado são descritos apenas no nível necessário para representar o comportamento observável do sistema. A forma concreta como essas atividades serão implementadas deve ser definida posteriormente na arquitetura e no projeto do sistema.

A ferramenta responsável pela consulta também não é especificada neste caso de uso em termos de nome, API ou implementação. Sua granularidade e organização constituem decisões de projeto arquitetural.
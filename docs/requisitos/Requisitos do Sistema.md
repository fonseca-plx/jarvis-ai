# Requisitos do Sistema — Assistente Conversacional por Voz

**Projeto:** TCC — Assistente Conversacional por Voz para Interação com Sistemas de Informação

**Versão:** 2.2 (rascunho) | **Status:** Em elaboração | **Data:** 2026-11-18

## 1. Introdução

### 1.1 Propósito

Este documento reúne os requisitos funcionais e não funcionais do assistente. Ele substitui os Requisitos Funcionais Consolidados (v1.0) e a seção 9 do Documento de Visão (v1.1). Seus critérios de aceitação, junto com os casos de uso do estudo de caso, são os critérios imediatos de verificação do trabalho; as métricas da literatura (Visão, seção 11) ficam como trabalho futuro.

Os requisitos são independentes do domínio. O estudo de caso aparece apenas como exemplo e na rastreabilidade.

### 1.2 Convenções

- **Identificadores:** RFnn (funcional) e RNFnn (não funcional). Os critérios de aceitação são numerados dentro de cada requisito e citados como RF04 CA3.
- **Prioridade (MoSCoW):** *Must* é necessário ao MVP; *Should* entra se o prazo permitir; *Could* é desejável e pode ficar como trabalho futuro.
- **Critério de aceitação (CA):** condição observável que define quando o requisito está atendido e que pode ser verificada por um cenário de teste.
- **Valores propostos:** números marcados como propostos são pontos de partida a validar (seção 7).

### 1.3 Documentos relacionados

| Documento | Versão | Relação |
| --- | --- | --- |
| Documento de Visão | 1.1 | Problema, escopo e princípios de projeto. A seção 9 será substituída por uma referência a este documento. |
| Especificação do Estudo de Caso | 0.4 | Operações, regras de negócio, casos de uso (UC01 a UC05), casos de teste (CT-01 a CT-18) e cenários conversacionais (CC-01 a CC-29). |
| Modelo de Arquitetura | 1.3 | Define os componentes, contratos e fluxos que atendem a estes requisitos. |

## 2. Visão geral dos requisitos funcionais

| ID | Requisito | Prioridade |
| --- | --- | --- |
| RF01 | Receber e transcrever a entrada de voz | Must |
| RF02 | Interpretar a solicitação | Must |
| RF03 | Gerenciar o contexto conversacional | Must |
| RF04 | Gerenciar as informações necessárias à tarefa | Must |
| RF05 | Selecionar a ferramenta | Must |
| RF06 | Preparar e validar a chamada | Must |
| RF07 | Executar ferramentas autorizadas | Must |
| RF08 | Processar os resultados das ferramentas | Must |
| RF09 | Tratar falhas e recuperar a interação | Must |
| RF10 | Gerenciar o controle da interação | Must |
| RF11 | Gerar a resposta textual | Must |
| RF12 | Entregar a resposta por voz | Must |
| RF13 | Tratar a incerteza do reconhecimento de fala | Should |
| RF14 | Controlar a saída de voz | Should e Could |

## 3. Requisitos funcionais

### RF01 — Receber e transcrever a entrada de voz

O sistema deve receber a fala do usuário e disponibilizar sua transcrição em texto ao núcleo conversacional, junto com o índice de confiança quando o provedor o fornecer.

**Prioridade:** Must · **Origem (v1.1):** RF01

**Critérios de aceitação**

1. Dada uma fala em português do Brasil, quando o usuário termina de falar, a transcrição é entregue ao núcleo conversacional.
2. Quando não há fala detectada ou a transcrição vem vazia, nenhuma solicitação é processada (ver RF13).
3. A transcrição e, se disponível, a confiança são registradas no log do turno (RNF03).

### RF02 — Interpretar a solicitação

O sistema deve interpretar a solicitação do usuário e identificar a intenção, considerando o conteúdo da fala e o contexto da conversa.

**Prioridade:** Must · **Origem (v1.1):** RF02

**Critérios de aceitação**

1. A solicitação é classificada como uma das intenções suportadas (correspondentes às ferramentas registradas), como um comando de controle (confirmar, negar, cancelar, repetir, encerrar) ou como não suportada.
2. A mesma intenção é identificada para formulações diferentes da mesma solicitação (ao menos 2 formulações por intenção nos cenários de teste).
3. A interpretação considera o estado da tarefa: por exemplo, “sim” é uma confirmação apenas quando há confirmação pendente.

### RF03 — Gerenciar o contexto conversacional

O sistema deve manter e usar informações relevantes de interações anteriores para dar continuidade a uma tarefa e resolver referências.

**Prioridade:** Must · **Origem (v1.1):** RF03, RF04

**Critérios de aceitação**

1. Informações fornecidas em turnos anteriores da mesma tarefa não precisam ser repetidas pelo usuário.
2. Referências a itens mencionados ou apresentados (“esse”, “o segundo”) são resolvidas quando há um único referente possível.
3. Quando a referência não pode ser determinada, o sistema pede esclarecimento em vez de assumir um referente (RF04).
4. O contexto conversacional é mantido durante a sessão e descartado ao seu término. A identidade do usuário pertence ao contexto de sessão, separado do contexto conversacional (RNF06).

### RF04 — Gerenciar as informações necessárias à tarefa

O sistema deve identificar os parâmetros fornecidos, determinar os que faltam e tratar ambiguidades, solicitando esclarecimento ao usuário antes de prosseguir.

**Prioridade:** Must · **Origem (v1.1):** RF05, RF06, RF07, RF08

**Critérios de aceitação**

1. Os parâmetros fornecidos pelo usuário são extraídos e usados, inclusive vários na mesma fala e em qualquer ordem.
2. Os parâmetros obrigatórios que faltam são determinados pela comparação entre o que foi coletado e o contrato da ferramenta; o sistema pede apenas esses, sem repetir pedidos de dados já informados.
3. Quando um critério identifica mais de uma entidade candidata, o sistema apresenta os candidatos, em quantidade limitada por vez (3 no estudo de caso), informa se há mais e pede ao usuário que escolha.
4. Nenhuma ferramenta é executada enquanto houver parâmetro obrigatório ausente ou ambiguidade não resolvida.

### RF05 — Selecionar a ferramenta

O sistema deve selecionar, entre as ferramentas registradas, a compatível com a intenção e com os parâmetros determinados.

**Prioridade:** Must · **Origem (v1.1):** RF09, RNF05 (parcial)

**Critérios de aceitação**

1. A ferramenta selecionada pertence ao registro de ferramentas; qualquer outra proposta é rejeitada e registrada (RNF06).
2. Quando nenhuma ferramenta é compatível com a solicitação, o sistema informa que não pode realizar a operação, indica o que pode fazer e não executa nenhuma ferramenta.
3. Uma ferramenta só é proposta quando a intenção está determinada e não há ambiguidade pendente (RF04).

### RF06 — Preparar e validar a chamada

O sistema deve gerar os argumentos da ferramenta selecionada e validá-los, por software determinístico, antes da execução.

**Prioridade:** Must · **Origem (v1.1):** RF10, RF11

**Critérios de aceitação**

1. Os argumentos são validados contra o contrato da ferramenta: presença dos obrigatórios, tipos e formatos.
2. Argumentos inválidos impedem a execução e geram um resultado de validação, tratado conforme o RF09.
3. A validação do contrato não substitui a verificação das regras de negócio, que cabe ao domínio (RNF05).
4. A identidade do usuário não é aceita como argumento gerado pelo modelo de linguagem; ela vem do contexto de sessão (RNF06).

### RF07 — Executar ferramentas autorizadas

O sistema deve executar a chamada validada, somente para ferramentas registradas e permitidas, e devolver ao núcleo um resultado padronizado.

**Prioridade:** Must · **Origem (v1.1):** RF12

**Critérios de aceitação**

1. Somente chamadas validadas (RF06) de ferramentas registradas são executadas.
2. O resultado devolvido ao núcleo é padronizado, com status (sucesso, vazio, validação, regra de negócio, erro de execução ou inconclusivo) e dados.
3. Falha técnica gera erro de execução; tempo-limite excedido em operação de escrita gera resultado inconclusivo. Nenhum dos dois é tratado como sucesso.
4. Cada execução é registrada com identificador do turno, ferramenta, argumentos, status e duração (RNF03).

### RF08 — Processar os resultados das ferramentas

O sistema deve interpretar o resultado retornado e decidir o próximo passo da conversa conforme seu status.

**Prioridade:** Must · **Origem (v1.1):** RF13

**Critérios de aceitação**

1. Sucesso: a resposta reflete os dados retornados (RF11).
2. Vazio: o sistema informa que nada foi encontrado e sugere reformular, sem tratar como falha.
3. Recusa por regra de negócio: o sistema explica o motivo com base no código e nos dados retornados e não repete a mesma chamada.
4. Resultado com várias entidades: alimenta o tratamento de ambiguidade (RF04).
5. Erro de validação, erro de execução e resultado inconclusivo são tratados conforme o RF09.

### RF09 — Tratar falhas e recuperar a interação

O sistema deve identificar falhas ocorridas durante uma tarefa, comunicá-las ao usuário de forma compatível com o resultado obtido e permitir a retomada sem reiniciar a tarefa quando possível.

**Prioridade:** Must · **Origem (v1.1):** RF14, RF15, RNF07 (parcial), RNF10

**Critérios de aceitação**

1. Falha de validação: o sistema pede a informação correta e retoma a tarefa mantendo o que já foi coletado.
2. Falha de execução: o sistema informa que houve um problema, não afirma que a operação foi realizada e oferece nova tentativa.
3. Resultado inconclusivo em operação de escrita: antes de responder, o sistema verifica o estado por consulta e informa o resultado real; se não conseguir verificar, diz que não pode confirmar.
4. Correção do usuário durante a tarefa (“não, o outro”) atualiza o parâmetro e retoma a tarefa sem reiniciá-la.
5. O estado da tarefa é preservado entre a falha e a retomada.

### RF10 — Gerenciar o controle da interação

O sistema deve permitir ao usuário controlar o andamento da interação: confirmar, negar, cancelar e encerrar.

**Prioridade:** Must · **Origem (v1.1):** RF16, RF17, RF18

**Critérios de aceitação**

1. Operações que alteram estado exigem confirmação explícita antes da execução; consultas não exigem. A necessidade de confirmação é definida por ferramenta, em configuração, e não está fixa no núcleo (RNF01).
2. O pedido de confirmação descreve em linguagem natural o que será feito (a ação e a entidade) e aguarda a resposta.
3. Com confirmação positiva, é executada exatamente a chamada validada que foi confirmada, sem gerar novos argumentos.
4. Com resposta negativa, o sistema cancela a operação; se o usuário indicar uma correção (“não, o outro livro”), a tarefa é retomada (RF09 CA4).
5. O cancelamento é possível antes da execução; depois dela, desfazer exige uma nova operação.
6. O usuário pode encerrar a conversa a qualquer momento; o sistema responde, encerra a sessão e descarta o contexto.
7. Quando o usuário faz uma nova solicitação durante uma tarefa não concluída, o sistema avisa que será preciso encerrar a tarefa em andamento e só a encerra, iniciando a nova, se o usuário confirmar; se não confirmar, a tarefa em andamento continua.

### RF11 — Gerar a resposta textual

O sistema deve gerar uma resposta textual correspondente ao estado da tarefa ou ao resultado da operação.

**Prioridade:** Must · **Origem (v1.1):** RF19

**Critérios de aceitação**

1. A resposta corresponde ao estado da tarefa: pedido de informação, pedido de confirmação, resultado, recusa, falha ou encerramento, de modo que o usuário distinga cada um deles.
2. Os dados citados (datas, quantidades, títulos) coincidem com os retornados pelas ferramentas.
3. Uma resposta de sucesso de operação de escrita só é emitida quando o status do resultado é sucesso (RNF04).
4. A resposta respeita os limites de concisão (RNF08).

### RF12 — Entregar a resposta por voz

O sistema deve converter a resposta textual em voz e reproduzi-la ao usuário.

**Prioridade:** Must · **Origem (v1.1):** RF20

**Critérios de aceitação**

1. A resposta textual é convertida em áudio em português do Brasil e reproduzida ao usuário.
2. Datas e números são falados de forma natural (“23 de novembro”), não no formato interno.
3. Falha na síntese de voz é registrada e não impede o registro do resultado da operação (RNF09).

### RF13 — Tratar a incerteza do reconhecimento de fala

O sistema deve lidar com falas não compreendidas ou transcrições incertas sem executar operações com base nelas.

**Prioridade:** Should · **Origem:** nova (requisito de voz)

**Critérios de aceitação**

1. Quando não há fala detectada ou a transcrição é vazia ou ininteligível, o sistema pede que o usuário repita e não avança a tarefa.
2. Quando o provedor informa a confiança e ela fica abaixo de um limite configurável, o sistema confirma com o usuário a informação duvidosa (por exemplo, um título) antes de usá-la.
3. Após 2 falhas consecutivas de compreensão (valor proposto), o sistema orienta o usuário a reformular (por exemplo, falar mais devagar) em vez de repetir o mesmo pedido.
4. A informação que identifica o objeto de uma operação de escrita é sempre confirmada pelo RF10, independentemente da confiança da transcrição.

### RF14 — Controlar a saída de voz

O sistema deve oferecer ao usuário controle sobre a reprodução das respostas.

**Prioridade:** Should (CA1) e Could (CA2 e CA3) · **Origem (v1.1):** princípio 10.6 (interromper)

**Critérios de aceitação**

1. (Should) Quando o usuário pede para repetir (“repita”, “como?”), o sistema reproduz novamente a última resposta, sem reexecutar ferramentas.
2. (Could) O usuário pode interromper a fala do assistente; a reprodução para e a nova fala é processada.
3. (Could) Quando o assistente aguarda uma resposta e não há fala dentro de um tempo-limite configurável, repete a pergunta uma vez e, persistindo o silêncio, encerra a sessão.

## 4. Requisitos não funcionais

Cada requisito indica como será verificado. A prioridade segue a convenção da seção 1.2.

| ID | Requisito | Verificação | Prioridade | Origem (v1.1) |
| --- | --- | --- | --- | --- |
| RNF01 | **Desacoplamento do domínio.** O núcleo conversacional não contém entidades, regras nem nomes de operações do domínio; toda interação com o domínio ocorre por ferramentas registradas. | Inspeção das dependências do núcleo e execução do núcleo com um conjunto alternativo de ferramentas de teste, sem alterar seu código. | Must | RNF01 |
| RNF02 | **Modularidade e substituição de provedores.** Reconhecimento de fala, síntese de voz e modelo de linguagem são acessados por interfaces; trocar um provedor altera apenas o seu adaptador. | Execução dos cenários com provedores simulados no lugar dos reais. | Must | RNF02 |
| RNF03 | **Observabilidade e rastreabilidade.** Cada turno recebe um identificador, e os eventos (transcrição, interpretação, chamada de ferramenta, resultado, resposta) são registrados de forma estruturada, com marcas de tempo. | A partir dos logs de um cenário, reconstruir a sequência completa de eventos de cada turno. | Must | RNF03, RNF04 |
| RNF04 | **Fidelidade ao resultado.** Nenhuma resposta indica sucesso de uma operação cujo resultado não foi sucesso. | Cenários de recusa, falha e resultado inconclusivo (CT-08 a CT-11, CT-17, CT-18): nenhuma resposta de sucesso indevida em todas as execuções. | Must | RNF07 |
| RNF05 | **Regras de negócio no domínio.** As regras de negócio são impostas pelo sistema de domínio e não dependem do modelo de linguagem; o assistente apenas apresenta os resultados. | Testes diretos sobre o domínio, sem o assistente, cobrindo as regras RN01 a RN08 do estudo de caso. | Must | RNF06 |
| RNF06 | **Restrição de acesso.** O agente só acessa ferramentas registradas; a identidade do usuário vem da sessão; cada usuário só acessa os próprios dados. | Ferramenta não registrada é rejeitada; tentativa de acessar dado de outro usuário (CT-13) retorna não encontrado. | Must | RNF11 |
| RNF07 | **Desempenho.** Latência de turno (do fim da fala ao início da resposta em áudio): mediana de até 5 s e percentil 90 de até 10 s (valores propostos), medida por etapa. | Medição a partir dos logs durante a execução dos cenários. | Should | RNF08 |
| RNF08 | **Concisão das respostas.** Respostas faladas apresentam no máximo 3 itens por vez e até 3 frases curtas (valores propostos). | Inspeção das respostas nos cenários de teste. | Should | RNF09 |
| RNF09 | **Resiliência a falhas externas.** Falha ou lentidão do reconhecimento de fala, do modelo de linguagem, da síntese de voz ou do domínio resulta em mensagem controlada; toda chamada externa tem tempo-limite configurado, e o usuário nunca fica em silêncio indefinido. | Injeção de falhas e atrasos simulados (CT-17, CT-18). | Must | nova |
| RNF10 | **Privacidade.** O áudio bruto não é armazenado após a transcrição; os logs usam identificadores de sessão e dados fictícios; credenciais ficam fora do código-fonte. | Inspeção da configuração, do armazenamento e dos logs. | Should | nova |
| RNF11 | **Idioma.** Entrada e saída ocorrem em português do Brasil. | Cenários executados com provedores configurados para pt-BR. | Must | nova |
| RNF12 | **Testabilidade e reprodutibilidade.** Os cenários podem ser executados automaticamente, com entrada em texto no lugar da fala, data de referência fixável e base de dados restaurável. Cada cenário é executado ao menos 3 vezes, dada a variabilidade do modelo de linguagem (valor proposto). | Execução da suíte de cenários em ambiente automatizado. | Must | Visão 12.1 |

## 5. Restrições e premissas

**Restrições**

- Um modelo de linguagem é usado como componente de interpretação e orquestração.
- A interação com o domínio ocorre apenas por ferramentas estruturadas.
- A verificação usa cenários controlados e reproduzíveis (RNF12).
- A implementação deve caber no escopo e nos recursos do TCC.

**Premissas**

- O domínio expõe operações que podem ser encapsuladas como ferramentas, com contratos definidos (estudo de caso).
- O modelo de linguagem suporta chamadas estruturadas de ferramentas.
- Os provedores de reconhecimento e de síntese de fala suportam português do Brasil.
- O ambiente permite registrar entradas, decisões, chamadas e resultados (RNF03).
- O usuário é identificado no contexto da sessão (autenticação simulada).

## 6. Rastreabilidade

### 6.1 Requisitos funcionais × casos de uso × casos de teste

Os casos de teste (CT) do estudo de caso verificam as ferramentas e o domínio. Os cenários conversacionais (CC) verificam o comportamento do assistente e estão na seção 10.5 do estudo de caso.

| Requisito | Casos de uso | Casos de teste e cenários de referência |
| --- | --- | --- |
| RF01 | Todos | CC-24; teste com reconhecimento de fala real |
| RF02 | Todos | CC-01, CC-03, CC-20 |
| RF03 | UC04, UC05 | CC-02, CC-07, CC-08, CC-16, CC-28; CT-02, CT-03 |
| RF04 | UC01, UC04 | CC-05 a CC-08; CT-02, CT-03, CT-16 |
| RF05 | Todos | CC-18, CC-19 |
| RF06 | UC04, UC05 | CT-16 |
| RF07 | Todos | CC-16, CC-21, CC-22; CT-17, CT-18 |
| RF08 | Todos | CC-01, CC-12 a CC-15, CC-17; CT-01, CT-04, CT-08 a CT-11 |
| RF09 | UC04, UC05 | CC-10, CC-21, CC-22; CT-13 a CT-15, CT-17, CT-18 |
| RF10 | UC04, UC05 | CC-04, CC-09 a CC-11, CC-16, CC-28, CC-29; CT-07, CT-12 |
| RF11 | Todos | Todos os cenários (fidelidade das respostas); CT-01 a CT-18 |
| RF12 | Todos | CC-04, CC-13, CC-15, CC-16 (datas faladas); teste com síntese de voz real |
| RF13 | Todos | CC-23, CC-24 |
| RF14 | Todos | CC-25 (Should); CC-26, CC-27 (Could) |

### 6.2 Mapeamento da versão 1.1 da Visão para esta versão

| Requisito (v1.1) | Nesta versão |
| --- | --- |
| RF01 | RF01 |
| RF02 | RF02 |
| RF03, RF04 | RF03 |
| RF05 | RF04 (a v1.0 também o citava no RF02; aqui a extração de parâmetros fica só no RF04) |
| RF06, RF07, RF08 | RF04 |
| RF09 | RF05 |
| RF10, RF11 | RF06 |
| RF12 | RF07 |
| RF13 | RF08 |
| RF14, RF15 | RF09 |
| RF16, RF17, RF18 | RF10 |
| RF19 | RF11 |
| RF20 | RF12 |
| RNF01 | RNF01 |
| RNF02 | RNF02 |
| RNF03, RNF04 | RNF03 |
| RNF05 | RF05 CA3 e RF04 CA4 |
| RNF06 | RNF05 |
| RNF07 | RNF04 e RF09 |
| RNF08 | RNF07 |
| RNF09 | RNF08 |
| RNF10 | RF09 CA5 |
| RNF11 | RNF06 |
| Novos | RF13, RF14, RNF09 a RNF12 |

## 7. Decisões sobre os pontos em aberto

- **Q-R1:** os valores propostos (latência, concisão das respostas, número de execuções por cenário e de falhas consecutivas de compreensão) foram mantidos e serão revisados após o MVP.
- **Q-R2:** as prioridades de RF13 (Should) e RF14 (Should e Could) foram mantidas.
- **Q-R3:** os cenários conversacionais foram adicionados ao estudo de caso (seção 10.5, CC-01 a CC-28).
- **Q-R4:** o status de recusa por regra de negócio no resultado de ferramenta (RF07 CA2) será tratado na revisão do Modelo de Arquitetura.
- **Q-R5:** confirmado: a extração de parâmetros fica apenas no RF04.

## 8. Histórico de versões

| Versão | Data | Descrição |
| --- | --- | --- |
| 2.0 | 2026-11-14 | Reescrita com critérios de aceitação por requisito, requisitos não funcionais consolidados, requisitos de voz (RF13 e RF14), prioridades e rastreabilidade. Substitui os Requisitos Funcionais Consolidados v1.0. |
| 2.1 | 2026-11-14 | Pontos Q-R1 a Q-R5 resolvidos; rastreabilidade atualizada com os cenários conversacionais CC-01 a CC-28 do estudo de caso. |
| 2.2 | 2026-11-18 | Novo critério de aceitação RF10 CA7 (mudança de assunto durante tarefa pendente); rastreabilidade atualizada com o cenário CC-29. |

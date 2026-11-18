# Especificação do Estudo de Caso — Sistema de Biblioteca

**Projeto:** TCC — Assistente Conversacional por Voz para Interação com Sistemas de Informação

**Versão:** 0.3 | **Status:** Em elaboração | **Data:** 2026-11-14

## 1. Identificação e propósito

### 1.1 Objetivo

Este documento especifica o sistema de informação de domínio usado como estudo de caso do TCC: um sistema simplificado de empréstimos de uma biblioteca. Seu papel é oferecer um domínio concreto, pequeno e bem definido sobre o qual o assistente conversacional por voz opera por meio de ferramentas. Não é objetivo construir um sistema de biblioteca completo.

O documento descreve o domínio (entidades, regras e operações). Decisões de arquitetura e de tecnologia pertencem ao Modelo de Arquitetura e ao projeto detalhado.

### 1.2 Relação com os demais documentos

| Documento | Relação |
| --- | --- |
| Documento de Visão (v1.1) | Define o problema e o escopo geral; este documento detalha o estudo de caso. |
| Requisitos do Sistema (v2.0) | Os requisitos são verificados pelos casos de teste e pelos cenários conversacionais deste documento (seções 10.4 e 10.5). |
| Modelo de Arquitetura (v1.2) | O mapeamento de ferramentas e o catálogo de erros (seções 6 e 7) alimentam a revisão da arquitetura. |

### 1.3 Escopo

**Dentro do escopo:** consultar livros (incluindo a disponibilidade dos títulos), consultar os empréstimos do usuário, registrar empréstimo e registrar devolução.

**Fora do escopo:** cadastro, edição e remoção de livros e usuários; reservas e fila de espera; renovação de empréstimo; multas e bloqueios por atraso; múltiplas bibliotecas; área administrativa; autenticação real.

## 2. Contexto e premissas

O domínio é uma biblioteca fictícia de pequeno porte, cujo acervo é composto por livros emprestados a leitores. O assistente atua em nome de um único ator, o **Leitor**, que conversa por voz.

| ID | Premissa |
| --- | --- |
| P01 | Biblioteca única, dados fictícios e persistência simples. |
| P02 | A interação ocorre em português do Brasil. |
| P03 | O usuário é identificado no contexto da sessão (autenticação simulada). A identidade não é dita pelo usuário nem gerada pelo assistente: todas as operações atuam sobre o usuário da sessão. |
| P04 | Datas têm granularidade de dia, sem horário. “Hoje” é a data corrente do sistema, e deve ser possível fixá-la nos testes (necessário para verificar prazo e carência). |
| P05 | Prazos são contados em dias corridos. |
| P06 | O registro de empréstimo e de devolução representa o evento; a retirada ou entrega física do livro está fora do escopo. |
| P07 | Cada empréstimo refere-se a uma unidade de um título. |
| P08 | Cada operação é atômica: ou é concluída integralmente ou não altera o estado. |

## 3. Modelo de domínio

### 3.1 Entidades

| Entidade | Atributos | Observações |
| --- | --- | --- |
| Livro | livro\_id, titulo, autor, total\_exemplares | Representa um título. A disponibilidade é derivada (seção 3.3). |
| Usuario | usuario\_id, nome | Leitor da biblioteca. |
| Emprestimo | emprestimo\_id, usuario\_id, livro\_id, data\_emprestimo, data\_prevista\_devolucao, data\_devolucao | data\_devolucao é nula enquanto o empréstimo está ativo. |

### 3.2 Relacionamentos

- Um usuário possui zero ou mais empréstimos.
- Um livro está associado a zero ou mais empréstimos.
- Cada empréstimo refere-se a exatamente um usuário e a exatamente um livro.

### 3.3 Estados, atributos derivados e invariantes

**Estados do empréstimo:** ATIVO (data\_devolucao nula) e DEVOLVIDO (data\_devolucao preenchida). A única transição é ATIVO → DEVOLVIDO, realizada pela operação de devolução, e é irreversível.

**Atributos derivados:**

- exemplares\_disponiveis = total\_exemplares − número de empréstimos ativos do livro;
- em\_atraso = empréstimo ativo com hoje posterior a data\_prevista\_devolucao (informativo, sem consequência, ver RN07);
- fim da carência = data\_devolucao + 14 dias (ver RN04).

**Invariantes:** 0 ≤ exemplares\_disponiveis ≤ total\_exemplares; no máximo 1 empréstimo ativo por par (usuário, livro); no máximo 3 empréstimos ativos por usuário.

### 3.4 Decisão de modelagem

**D-01 — Disponibilidade por título.** O sistema controla a quantidade de exemplares de cada título, sem rastrear cópias físicas individuais. Justificativa: o assistente conversa em termos de títulos, e rastrear cada cópia não contribui para o objetivo do trabalho, apenas amplia o domínio e as ferramentas.

## 4. Regras de negócio

| ID | Regra | Erro quando violada |
| --- | --- | --- |
| RN01 | **Prazo.** O empréstimo tem prazo de 2 semanas (14 dias corridos): data\_prevista\_devolucao = data\_emprestimo + 14 dias. | — |
| RN02 | **Limite.** Cada usuário pode ter no máximo 3 empréstimos ativos. | LIMITE\_EMPRESTIMOS\_ATINGIDO |
| RN03 | **Disponibilidade.** Só é possível emprestar um título com pelo menos 1 exemplar disponível. O empréstimo reduz a disponibilidade em 1 e a devolução a aumenta em 1. | LIVRO\_INDISPONIVEL |
| RN04 | **Empréstimo repetido (carência).** Após devolver um título, o usuário só pode emprestá-lo novamente 2 semanas depois da devolução (a partir do 14º dia após a data de devolução, inclusive). A regra vale por usuário e por título. | CARENCIA\_EM\_VIGOR |
| RN05 | **Empréstimo ativo único.** O usuário não pode ter dois empréstimos ativos do mesmo título. | EMPRESTIMO\_ATIVO\_MESMO\_LIVRO |
| RN06 | **Devolução.** Só é possível devolver um empréstimo ativo do próprio usuário. | EMPRESTIMO\_NAO\_ENCONTRADO, EMPRESTIMO\_JA\_DEVOLVIDO |
| RN07 | **Atraso.** A devolução após o prazo é aceita. Não há multa nem bloqueio; o atraso é apenas informativo. | — |
| RN08 | **Isolamento.** O usuário só consulta e altera os próprios empréstimos. | EMPRESTIMO\_NAO\_ENCONTRADO |

**Ordem de verificação no empréstimo.** Quando mais de uma regra for violada, o erro retornado segue esta ordem: (1) livro existente, (2) RN05, (3) RN04, (4) RN02, (5) RN03. Restrições do usuário são verificadas antes do estoque. A ordem torna o resultado determinístico e testável.

## 5. Especificação das operações

**Convenções.** O usuário vem da sessão (P03) e não é parâmetro de nenhuma operação. Datas usam o formato ISO 8601 (AAAA-MM-DD). *Efeito* indica leitura ou escrita. *Confirmação* indica se o assistente deve obter confirmação do usuário antes de executar. Política adotada: toda operação de escrita exige confirmação, e as de leitura não; a formalização fica para os requisitos e a arquitetura.

### OP01 — Consultar livros

- **Descrição:** busca livros do acervo por título e/ou autor e informa a disponibilidade de cada título (D-04).
- **Efeito:** leitura. **Confirmação:** não.
- **Entradas:** titulo (texto, opcional) e autor (texto, opcional); ao menos um é obrigatório. A correspondência é parcial e ignora maiúsculas e acentos.
- **Saída:** total\_encontrado e uma lista de até 5 livros com livro\_id, titulo, autor, total\_exemplares e exemplares\_disponiveis, com indicação de truncamento quando houver mais resultados.
- **Resultados possíveis:** sucesso; VAZIO (nenhum livro encontrado); PARAMETROS\_INSUFICIENTES.

### OP02 — Consultar empréstimos do usuário

- **Descrição:** lista os empréstimos do usuário da sessão.
- **Efeito:** leitura. **Confirmação:** não.
- **Entrada:** situacao (opcional): ATIVOS (padrão), DEVOLVIDOS ou TODOS.
- **Saída:** emprestimos\_ativos, limite\_emprestimos (3) e uma lista com emprestimo\_id, livro\_id, titulo, data\_emprestimo, data\_prevista\_devolucao, data\_devolucao (quando houver), situacao, em\_atraso e, para devolvidos, novo\_emprestimo\_liberado\_em.
- **Resultados possíveis:** sucesso; VAZIO (nenhum empréstimo na situação solicitada).
- **Observação:** também serve para localizar o empréstimo a devolver e para responder perguntas sobre limite e carência.

### OP03 — Registrar empréstimo

- **Descrição:** registra o empréstimo de um título ao usuário da sessão.
- **Efeito:** escrita. **Confirmação:** sim (altera o estoque e o limite do usuário, e títulos reconhecidos por voz podem ser confundidos).
- **Entrada:** livro\_id (obrigatório).
- **Pré-condições:** RN05, RN04, RN02 e RN03 satisfeitas, verificadas nessa ordem.
- **Pós-condições:** novo empréstimo ATIVO com data\_emprestimo igual a hoje e prazo conforme RN01; exemplares\_disponiveis do título reduzido em 1.
- **Saída:** emprestimo\_id, livro\_id, titulo, data\_emprestimo, data\_prevista\_devolucao e emprestimos\_ativos após a operação.
- **Erros:** LIVRO\_NAO\_ENCONTRADO; EMPRESTIMO\_ATIVO\_MESMO\_LIVRO; CARENCIA\_EM\_VIGOR; LIMITE\_EMPRESTIMOS\_ATINGIDO; LIVRO\_INDISPONIVEL.
- **Repetição:** repetir a solicitação após um sucesso não cria novo empréstimo e retorna EMPRESTIMO\_ATIVO\_MESMO\_LIVRO. Isso permite ao assistente verificar o estado após um resultado inconclusivo.

### OP04 — Registrar devolução

- **Descrição:** registra a devolução de um empréstimo ativo do usuário da sessão.
- **Efeito:** escrita. **Confirmação:** sim.
- **Entrada:** emprestimo\_id (obrigatório), obtido pela OP02 (D-05).
- **Pós-condições:** empréstimo passa a DEVOLVIDO com data\_devolucao igual a hoje; exemplares\_disponiveis do título aumentado em 1; início da carência (RN04).
- **Saída:** emprestimo\_id, livro\_id, titulo, data\_devolucao, em\_atraso, dias\_de\_atraso (0 se dentro do prazo) e novo\_emprestimo\_liberado\_em.
- **Erros:** EMPRESTIMO\_NAO\_ENCONTRADO (inexistente ou pertencente a outro usuário, sem distinguir os dois casos); EMPRESTIMO\_JA\_DEVOLVIDO.
- **Repetição:** repetir a solicitação após um sucesso retorna EMPRESTIMO\_JA\_DEVOLVIDO.

## 6. Catálogo de resultados e erros

As categorias abaixo são provisórias e serão mapeadas para os status do resultado de ferramenta na revisão da arquitetura (a versão 1.2 ainda não tem status para recusa por regra de negócio). *Recuperável* indica se a tarefa pode prosseguir com uma nova informação do usuário. Nas recusas por regra, a mesma solicitação não pode ser atendida, mas o usuário pode tentar outra operação.

| Código | Categoria | Operações | Recuperável | Dados adicionais | Orientação ao assistente |
| --- | --- | --- | --- | --- | --- |
| VAZIO | Vazio | OP01, OP02 | Sim | — | Informar que nada foi encontrado e, na OP01, sugerir outra busca. Não tratar como falha. |
| PARAMETROS\_INSUFICIENTES | Validação | OP01 | Sim | — | Pedir título ou autor. |
| PARAMETRO\_INVALIDO | Validação | Todas | Sim | campo | Pedir novamente a informação. |
| LIVRO\_NAO\_ENCONTRADO | Não encontrado | OP03 | Sim | — | Informar e propor nova busca. |
| EMPRESTIMO\_NAO\_ENCONTRADO | Não encontrado | OP04 | Sim | — | Informar que o empréstimo não foi localizado entre os do usuário e listar os empréstimos ativos. |
| LIVRO\_INDISPONIVEL | Regra de negócio | OP03 | Não | titulo | Informar que não há exemplares. Reserva está fora do escopo. |
| LIMITE\_EMPRESTIMOS\_ATINGIDO | Regra de negócio | OP03 | Não | emprestimos\_ativos, limite | Informar o limite e sugerir devolver um livro. |
| CARENCIA\_EM\_VIGOR | Regra de negócio | OP03 | Não | novo\_emprestimo\_liberado\_em | Informar a data a partir da qual o livro poderá ser emprestado de novo. |
| EMPRESTIMO\_ATIVO\_MESMO\_LIVRO | Regra de negócio | OP03 | Não | emprestimo\_id, data\_prevista\_devolucao | Informar que o usuário já está com o livro e o prazo de devolução. |
| EMPRESTIMO\_JA\_DEVOLVIDO | Regra de negócio | OP04 | Não | data\_devolucao | Informar que o empréstimo já foi devolvido. |
| SERVICO\_INDISPONIVEL | Erro de execução | Todas | Sim (nova tentativa) | — | Informar a falha técnica sem afirmar que a operação foi realizada e oferecer nova tentativa. |
| RESULTADO\_INCERTO | Inconclusivo | OP03, OP04 | Após verificação | — | Não afirmar sucesso nem falha; consultar o estado (OP02) antes de responder. |

**Princípio:** o domínio retorna códigos e dados, não texto de fala. A mensagem ao usuário é formulada pelo assistente a partir do código e dos dados adicionais.

## 7. Mapeamento preliminar operação → ferramenta

**D-02 — Granularidade 1 para 1.** Cada operação é exposta como uma ferramenta, com uma camada de adaptação entre a API e a ferramenta (resultados compactos, erros estruturados e IDs internos). A decisão é preliminar e será detalhada no Modelo de Arquitetura.

| Ferramenta | Operação | Quando usar | Parâmetros | Efeito | Confirmação |
| --- | --- | --- | --- | --- | --- |
| consultar\_livros | OP01 | O usuário quer encontrar livros por título ou autor, ou saber se um título está disponível. | titulo, autor | Leitura | Não |
| consultar\_emprestimos | OP02 | O usuário quer saber seus empréstimos, prazos, limite ou carência, ou localizar o empréstimo a devolver. | situacao | Leitura | Não |
| registrar\_emprestimo | OP03 | O usuário quer pegar emprestado um título já identificado. | livro\_id | Escrita | Sim |
| registrar\_devolucao | OP04 | O usuário quer devolver um empréstimo ativo já identificado. | emprestimo\_id | Escrita | Sim |

**Diretrizes:**

- IDs nunca são ditos pelo usuário; são obtidos pelas ferramentas de consulta e mantidos no contexto.
- O usuário da sessão nunca é parâmetro de ferramenta (D-03).
- Resultados devem ser compactos e adequados à fala.
- Erros retornam código e dados adicionais, conforme a seção 6.

**D-03 — Identidade vem da sessão.** O usuário é definido no contexto da sessão e injetado pela camada de controle, e não produzido pelo modelo de linguagem. Justificativa: impede que o assistente consulte ou altere empréstimos de outra pessoa (RN08).

## 8. Particularidades da interação por voz

- **Referência às entidades:** o usuário fala títulos e autores; identificadores são internos.
- **Fragilidade do reconhecimento de fala:** títulos em outros idiomas (Clean Code), nomes de autores e títulos parecidos (Clean Code e Clean Architecture; Engenharia de Software e Engenharia de Software Moderna) são fontes de erro. No domínio, a busca é tolerante (OP01). No diálogo, o título e o autor devem ser confirmados antes de qualquer escrita (a derivar nos requisitos).
- **Desambiguação:** quando a OP01 retorna mais de um livro, o assistente apresenta os candidatos e pede a escolha. Decisão: ler no máximo 3 candidatos por vez e avisar quando houver mais (D-06). Referências como “o segundo” ou “esse” dependem do contexto da conversa.
- **Datas:** o usuário fala em formato natural (“dia 23 de novembro”); o assistente converte entre a fala e o formato ISO usado pelo domínio.
- **Respostas curtas:** a resposta deve priorizar a informação pedida, evitando ler listas longas ou todos os atributos de uma entidade.

## 9. Casos de uso (catálogo resumido)

| ID | Caso de uso | Operações | Observações |
| --- | --- | --- | --- |
| UC01 | Consultar livros | OP01 | Inclui busca sem resultado. |
| UC02 | Verificar disponibilidade de um livro | OP01 | A disponibilidade é obtida pela consulta de livros (D-04). |
| UC03 | Consultar meus empréstimos | OP02 | Inclui prazos, atraso, limite e carência. |
| UC04 | Emprestar livro | OP01, OP03 | Variantes: ambiguidade de título; recusa por regra (RN02 a RN05); desistência na confirmação. |
| UC05 | Devolver livro | OP02, OP04 | Variantes: empréstimo não encontrado; já devolvido; devolução em atraso. |

Comportamentos conversacionais transversais (correção de informação, cancelamento, continuidade pelo contexto e falha técnica) são tratados nos requisitos e aplicam-se a esses casos de uso; seus cenários de verificação estão na seção 10.5. A especificação detalhada (fluxo principal, alternativos e de exceção) ficará em documento próprio.

## 10. Dados de exemplo para testes

**Data de referência dos testes: 2026-11-09.** Entre cenários que alteram estado, a base deve ser restaurada ao conjunto abaixo.

### 10.1 Usuários

| ID | Nome | Situação |
| --- | --- | --- |
| U01 | Ana Souza | No limite: 3 empréstimos ativos |
| U02 | Bruno Lima | 1 empréstimo ativo e 1 título em carência |
| U03 | Carla Dias | Sem empréstimos (cenários de sucesso) |
| U04 | Diego Alves | 1 empréstimo ativo em atraso |

### 10.2 Livros

| ID | Título | Autor | Total | Disponíveis |
| --- | --- | --- | --- | --- |
| L01 | Clean Code | Robert C. Martin | 3 | 1 |
| L02 | Clean Architecture | Robert C. Martin | 1 | 0 |
| L03 | Dom Casmurro | Machado de Assis | 3 | 2 |
| L04 | Memórias Póstumas de Brás Cubas | Machado de Assis | 2 | 2 |
| L05 | O Hobbit | J. R. R. Tolkien | 2 | 2 |
| L06 | Engenharia de Software | Ian Sommerville | 2 | 2 |
| L07 | Engenharia de Software Moderna | Marco Tulio Valente | 1 | 1 |
| L08 | Algoritmos: Teoria e Prática | Thomas H. Cormen | 2 | 1 |
| L09 | Capitães da Areia | Jorge Amado | 1 | 1 |

### 10.3 Empréstimos

| ID | Usuário | Livro | Empréstimo | Prevista | Devolução | Situação |
| --- | --- | --- | --- | --- | --- | --- |
| E01 | U01 | L01 | 2026-11-03 | 2026-11-17 | — | Ativo |
| E02 | U01 | L03 | 2026-11-05 | 2026-11-19 | — | Ativo |
| E03 | U01 | L08 | 2026-11-06 | 2026-11-20 | — | Ativo |
| E04 | U02 | L02 | 2026-11-02 | 2026-11-16 | — | Ativo |
| E05 | U02 | L05 | 2026-10-12 | 2026-10-26 | 2026-11-02 | Devolvido (carência até 2026-11-16) |
| E06 | U04 | L01 | 2026-10-20 | 2026-11-03 | — | Ativo, em atraso (6 dias) |

### 10.4 Casos de teste

| ID | Usuário | Entrada | Resultado esperado |
| --- | --- | --- | --- |
| CT-01 | U03 | consultar\_livros(titulo = “dom casmurro”) | Sucesso; 1 livro (L03), 2 exemplares disponíveis. |
| CT-02 | U03 | consultar\_livros(autor = “Machado de Assis”) | Sucesso; 2 livros (L03, L04); exige escolha do usuário. |
| CT-03 | U03 | consultar\_livros(titulo = “engenharia de software”) | Sucesso; 2 livros (L06, L07); exige desambiguação. |
| CT-04 | U03 | consultar\_livros(titulo = “Senhor dos Anéis”) | VAZIO. |
| CT-05 | U03 | consultar\_livros(titulo = “Clean Architecture”) | Sucesso; 1 livro (L02) com 0 exemplares disponíveis. |
| CT-06 | U01 | consultar\_emprestimos() | Sucesso; 3 empréstimos ativos, limite 3. |
| CT-07 | U03 | registrar\_emprestimo(L03) | Sucesso; prazo 2026-11-23; L03 passa a 1 disponível. |
| CT-08 | U03 | registrar\_emprestimo(L02) | LIVRO\_INDISPONIVEL. |
| CT-09 | U01 | registrar\_emprestimo(L04) | LIMITE\_EMPRESTIMOS\_ATINGIDO. |
| CT-10 | U01 | registrar\_emprestimo(L01) | EMPRESTIMO\_ATIVO\_MESMO\_LIVRO (prevalece sobre o limite, conforme a ordem de verificação). |
| CT-11 | U02 | registrar\_emprestimo(L05) | CARENCIA\_EM\_VIGOR; liberado em 2026-11-16. |
| CT-12 | U04 | registrar\_devolucao(E06) | Sucesso; em atraso há 6 dias; novo empréstimo liberado em 2026-11-23. |
| CT-13 | U03 | registrar\_devolucao(E01) | EMPRESTIMO\_NAO\_ENCONTRADO (pertence a outro usuário). |
| CT-14 | U02 | registrar\_devolucao(E05) | EMPRESTIMO\_JA\_DEVOLVIDO. |
| CT-15 | U03 | registrar\_emprestimo(L99) | LIVRO\_NAO\_ENCONTRADO. |
| CT-16 | U03 | consultar\_livros() sem critérios | PARAMETROS\_INSUFICIENTES. |
| CT-17 | U03 | Qualquer operação com falha simulada do serviço | SERVICO\_INDISPONIVEL; o assistente não afirma sucesso. |
| CT-18 | U03 | registrar\_emprestimo(L03) com timeout simulado | RESULTADO\_INCERTO; o assistente verifica o estado antes de responder. |

Os casos CT-17 e CT-18 exigem injeção de falhas no serviço de domínio.

### 10.5 Cenários conversacionais

Os cenários conversacionais verificam o comportamento do assistente em diálogos de vários turnos. Usam a base desta seção e a data de referência 2026-11-09, e cada um parte da base restaurada. As falas são exemplos: formulações equivalentes são aceitas. Em todos os cenários valem as regras gerais: os dados citados coincidem com os retornados pelas ferramentas (RF11 CA2) e nenhuma operação de escrita é executada sem confirmação (RF10 CA1). A coluna Requisitos refere-se ao documento Requisitos do Sistema (v2.0).

| ID | Usuário | Falas do usuário (resumo) | Comportamento esperado | Requisitos |
| --- | --- | --- | --- | --- |
| CC-01 | U03 | “Tem Dom Casmurro?” | Consulta o acervo e informa que o livro existe, com 2 exemplares disponíveis. Não pede confirmação. | RF02, RF05 a RF08, RF11, RF12 |
| CC-02 | U03 | “Tem Dom Casmurro?” e depois “Então me empresta esse.” | Resolve “esse” pelo contexto (L03), sem pedir o título de novo, e segue para a confirmação do empréstimo. | RF03 CA1 e CA2 |
| CC-03 | U03 | Três formulações: “Tem Dom Casmurro?”, “Vocês possuem o livro Dom Casmurro?”, “Quero saber se há Dom Casmurro.” | A mesma ferramenta e os mesmos parâmetros são usados nas três formulações. | RF02 CA2 |
| CC-04 | U03 | “Quero pegar Dom Casmurro emprestado.” e depois “Sim.” | Identifica L03, pede confirmação citando título e autor e, após o “sim”, registra o empréstimo e informa o prazo (23 de novembro). L03 passa a 1 disponível. | RF04, RF10 CA1 a CA3, RF11, RF12 CA2 (CT-07) |
| CC-05 | U03 | “Quero pegar um livro emprestado.” e depois “Dom Casmurro.” | Pergunta qual livro; ao receber o título, segue como em CC-04. Nenhuma ferramenta é executada antes de ter o título. | RF04 CA2 e CA4 |
| CC-06 | U03 | “Quero o Dom Casmurro do Machado de Assis.” | Extrai título e autor da mesma fala, sem perguntar de novo, e encontra L03. | RF04 CA1 |
| CC-07 | U03 | “Quero pegar Engenharia de Software.” e depois “O segundo.” | Apresenta os 2 candidatos (L06 e L07) e pede a escolha; resolve “o segundo” como L07 (na ordem apresentada) e segue para a confirmação. | RF03 CA2, RF04 CA3 (CT-03) |
| CC-08 | U03 | “Tem algo do Machado de Assis?” e depois “Me empresta o primeiro.” | Apresenta os 2 livros (L03 e L04); “o primeiro” resolve para o primeiro apresentado e segue para a confirmação. | RF03 CA2, RF04 CA3 (CT-02) |
| CC-09 | U03 | Pedido de Dom Casmurro e, na confirmação, “Não.” | Cancela e informa que nada foi registrado. Nenhum empréstimo é criado e o estoque não muda. | RF10 CA4 |
| CC-10 | U03 | Pedido de Dom Casmurro e, na confirmação, “Não, o Memórias Póstumas.” | Entende como correção: troca o livro para L04 e pede nova confirmação, sem reiniciar a tarefa. | RF09 CA4, RF10 CA4 |
| CC-11 | U03 | Pedido de Dom Casmurro e, na confirmação, “Esquece, cancela.” | Cancela a operação pendente sem executá-la. | RF10 CA5 |
| CC-12 | U01 | “Quero pegar Memórias Póstumas.” e depois “Sim.” | Após a confirmação, a operação é recusada por limite; o assistente informa o limite de 3 e sugere devolver um livro. Não repete a chamada. | RF08 CA3 (CT-09) |
| CC-13 | U02 | “Quero pegar O Hobbit.” e depois “Sim.” | Recusa por carência; informa que poderá pegar o livro de novo a partir de 16 de novembro. | RF08 CA3, RF12 CA2 (CT-11) |
| CC-14 | U03 | “Quero pegar Clean Architecture.” | Informa que não há exemplares disponíveis (antes ou depois da confirmação). Nenhum empréstimo é criado. | RF08 CA3 (CT-08) |
| CC-15 | U04 | “Quais livros eu tenho emprestados?” | Informa o empréstimo ativo (Clean Code), em atraso há 6 dias, com prazo original em 3 de novembro. | RF08, RF12 CA2 (UC03) |
| CC-16 | U04 | “Quero devolver o Clean Code.” e depois “Sim.” | Localiza o empréstimo do usuário pela consulta de empréstimos, pede confirmação, registra a devolução e informa o atraso de 6 dias e que o livro poderá ser pego de novo a partir de 23 de novembro. | RF03, RF07, RF10, RF11, RF12 CA2 (CT-12) |
| CC-17 | U03 | “Quero devolver o Clean Code.” | Não encontra empréstimo ativo desse livro para o usuário e informa isso. Nenhuma devolução é registrada. | RF08 CA2, RF09 (ver CT-13) |
| CC-18 | U03 | “Quero renovar meu empréstimo.” | Informa que não pode realizar a operação e indica o que pode fazer (consultar livros e empréstimos, emprestar e devolver). Nenhuma ferramenta é executada. | RF05 CA2 |
| CC-19 | U03 | “Quais livros a Ana tem emprestados?” | Informa que só pode consultar os empréstimos do próprio usuário. Dados de outro usuário não são consultados nem revelados. | RF05 CA2, RNF06 |
| CC-20 | U03 | “Sim.” sem confirmação pendente | Não executa nada e pergunta o que o usuário deseja. | RF02 CA3 |
| CC-21 | U03 | “Tem algo do Machado de Assis?” com o serviço de domínio indisponível | Informa que houve um problema técnico, não inventa resultados e oferece nova tentativa. | RF09 CA2, RNF04, RNF09 (CT-17) |
| CC-22 | U03 | Pedido de Dom Casmurro e “Sim.”, com timeout simulado na gravação | Verifica o estado pela consulta de empréstimos antes de responder: (a) se o empréstimo foi registrado, informa o sucesso; (b) se não foi, informa que não foi concluído e oferece nova tentativa. Não afirma sucesso sem verificar. | RF09 CA3, RNF04 (CT-18) |
| CC-23 | U03 | Fala com transcrição de baixa confiança (por exemplo, “Clean Code” transcrito de forma duvidosa) | Confirma o título com o usuário (“Você disse Clean Code?”) antes de usá-lo. | RF13 CA2 |
| CC-24 | U03 | Duas falas consecutivas não compreendidas ou vazias | Pede que repita na primeira vez e, na segunda, orienta a reformular (por exemplo, falar mais devagar). Nenhuma operação é executada. | RF01 CA2, RF13 CA1 e CA3 |
| CC-25 | U03 | “Tem Dom Casmurro?” e depois “Repita.” | Reproduz novamente a última resposta, sem consultar o acervo de novo. | RF14 CA1 (Should) |
| CC-26 | U03 | Interrompe a leitura de uma lista de candidatos | A reprodução para e a nova fala é processada. | RF14 CA2 (Could) |
| CC-27 | U03 | Silêncio após uma pergunta do assistente | Repete a pergunta uma vez e, persistindo o silêncio, encerra a sessão. | RF14 CA3 (Could) |
| CC-28 | U03 | “Pode encerrar.” | Despede-se, encerra a sessão e descarta o contexto; uma nova conversa não retém informações da anterior. | RF10 CA6, RF03 CA4 |

Os cenários CC-21 e CC-22 exigem injeção de falhas no serviço de domínio, e os cenários CC-23 a CC-27 exigem a camada de voz ou a simulação de seus eventos (confiança baixa, silêncio, interrupção). Os demais podem ser executados com entrada em texto (RNF12). Os cenários CC-26 e CC-27 correspondem a requisitos de prioridade Could.

## 11. Rastreabilidade (preliminar)

| Operação | Ferramenta | Regras | Erros e resultados | Casos de uso | Casos de teste |
| --- | --- | --- | --- | --- | --- |
| OP01 | consultar\_livros | RN03 (disponibilidade informada) | VAZIO, PARAMETROS\_INSUFICIENTES | UC01, UC02, UC04 | CT-01 a CT-05, CT-16 |
| OP02 | consultar\_emprestimos | RN01, RN04, RN07, RN08 | VAZIO | UC03, UC05 | CT-06 |
| OP03 | registrar\_emprestimo | RN01 a RN05 | LIVRO\_NAO\_ENCONTRADO, EMPRESTIMO\_ATIVO\_MESMO\_LIVRO, CARENCIA\_EM\_VIGOR, LIMITE\_EMPRESTIMOS\_ATINGIDO, LIVRO\_INDISPONIVEL, RESULTADO\_INCERTO | UC04 | CT-07 a CT-11, CT-15, CT-18 |
| OP04 | registrar\_devolucao | RN03, RN04, RN06, RN07, RN08 | EMPRESTIMO\_NAO\_ENCONTRADO, EMPRESTIMO\_JA\_DEVOLVIDO, RESULTADO\_INCERTO | UC05 | CT-12 a CT-14 |

O resultado SERVICO\_INDISPONIVEL se aplica a todas as operações (CT-17). A rastreabilidade com os requisitos funcionais está na seção 10.5 (coluna Requisitos) e no documento Requisitos do Sistema (seção 6.1).

## 12. Decisões, questões em aberto e glossário

### 12.1 Decisões registradas

| ID | Decisão | Seção |
| --- | --- | --- |
| D-01 | Disponibilidade por título, com quantidade de exemplares. | 3.4 |
| D-02 | Granularidade 1 para 1 entre operações e ferramentas, com camada de adaptação (preliminar). | 7 |
| D-03 | A identidade do usuário vem da sessão, não da fala. | 7 |
| D-04 | Não há operação separada de verificação de disponibilidade: ela é obtida na consulta de livros (OP01), que retorna o número de exemplares disponíveis. | 5 |
| D-05 | A devolução é identificada por emprestimo\_id, obtido pela consulta de empréstimos do usuário (OP02). | 5 |
| D-06 | Limites de apresentação: a OP01 retorna até 5 livros e o assistente lê no máximo 3 candidatos por vez em voz. | 5, 8 |

### 12.2 Questões em aberto

Nenhuma questão em aberto nesta versão. As questões Q-01 a Q-05 da versão 0.1 foram resolvidas e estão refletidas nas decisões D-04 a D-06. As interpretações de prazo e carência (14 dias corridos; carência a partir do 14º dia após a devolução, inclusive), a regra de empréstimo ativo único e a ordem de verificação das regras foram aprovadas.

### 12.3 Melhorias futuras

- Exibir os resultados de forma paginada em interface gráfica, para que o usuário não precise lembrar de todos os itens lidos pelo assistente em voz.

### 12.4 Glossário

- **Título:** obra do acervo (entidade Livro), que pode ter vários exemplares.
- **Exemplar:** cópia de um título; apenas a quantidade é controlada.
- **Empréstimo ativo:** empréstimo ainda não devolvido.
- **Carência:** período de 14 dias após a devolução durante o qual o mesmo título não pode ser emprestado de novo pelo mesmo usuário.
- **Sessão:** contexto da conversa que contém a identidade do usuário (simulada).
- **Ferramenta:** operação do domínio exposta ao assistente.

### 12.5 Histórico de versões

| Versão | Data | Descrição |
| --- | --- | --- |
| 0.1 | 2026-11-09 | Versão inicial. |
| 0.2 | 2026-11-14 | Remoção da operação de verificação de disponibilidade e renumeração das operações (OP01 a OP04); resolução das questões em aberto; inclusão de melhorias futuras. |
| 0.3 | 2026-11-14 | Inclusão dos cenários conversacionais (seção 10.5, CC-01 a CC-28) e alinhamento com o documento Requisitos do Sistema v2.0. |

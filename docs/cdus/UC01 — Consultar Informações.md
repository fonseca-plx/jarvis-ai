# UC01 — Consultar Informações

**Identificador:** UC01  
**Nome:** Consultar informações  
**Versão:** 1.0  
**Prioridade:** Alta

## 1. Objetivo

Permitir que o usuário obtenha informações disponíveis no sistema de informação por meio de uma solicitação realizada em linguagem natural, utilizando a interface conversacional por voz.

O assistente deve interpretar a solicitação, identificar as informações necessárias para realizar a consulta, utilizar uma ferramenta compatível quando necessário e apresentar ao usuário uma resposta correspondente ao resultado obtido.

---

## 2. Atores

### Ator principal

**Usuário**

Pessoa que realiza uma solicitação ao assistente.

### Atores secundários

**Sistema de Informação de Domínio**

Sistema que disponibiliza os dados ou operações necessários para responder à consulta.

**Ferramenta**

Interface disponibilizada ao agente para acessar determinada operação ou informação do sistema de domínio.

---

## 3. Gatilho

O usuário inicia uma interação por voz e apresenta uma solicitação de consulta.

**Exemplo:**

> "Quais livros estão cadastrados?"

ou:

> "Quem é o autor de Clean Code?"

---

## 4. Pré-condições

1. O assistente deve estar disponível para receber uma interação.
2. O sistema deve possuir acesso às ferramentas necessárias para realizar a consulta solicitada.
3. As ferramentas disponibilizadas devem possuir interfaces e parâmetros definidos.
4. O sistema de informação de domínio deve estar disponível quando a consulta depender de acesso a seus dados.

---

## 5. Pós-condições

### Sucesso

1. A consulta solicitada foi identificada.
2. A ferramenta adequada foi selecionada, quando necessária.
3. A consulta foi executada.
4. O resultado foi processado.
5. O usuário recebeu uma resposta correspondente ao resultado obtido.

### Falha

Caso a consulta não possa ser realizada, o sistema deve informar adequadamente a situação ao usuário, sem apresentar como resultado uma informação que não tenha sido efetivamente obtida.

---

# 6. Fluxo principal

**1. Usuário realiza uma solicitação de consulta.**

O usuário apresenta, por voz, uma solicitação relacionada a informações disponíveis no sistema.

**2. Sistema recebe e interpreta a solicitação.**

O assistente converte a entrada de voz em uma representação textual e interpreta a solicitação, identificando a intenção e as informações relevantes.

**3. Sistema identifica as informações necessárias.**

O assistente determina os parâmetros necessários para realizar a consulta.

Quando a solicitação utilizar informações previamente fornecidas na conversa, o contexto disponível pode ser utilizado para complementar a solicitação.

**4. Sistema determina a operação adequada.**

O assistente seleciona uma ferramenta compatível com a intenção identificada e com as informações disponíveis.

**5. Sistema prepara a chamada da ferramenta.**

O assistente gera os argumentos necessários e realiza as validações aplicáveis antes da execução.

**6. Sistema executa a ferramenta.**

A ferramenta selecionada é executada utilizando os argumentos validados.

**7. Sistema processa o resultado.**

O assistente recebe e interpreta o resultado retornado pela ferramenta.

**8. Sistema gera a resposta.**

O assistente produz uma resposta textual correspondente ao resultado obtido.

**9. Sistema entrega a resposta.**

A resposta textual é convertida em voz e apresentada ao usuário.

**10. Caso de uso é concluído.**

---

# 7. Fluxos alternativos

## FA01 — Informações insuficientes

**Ponto de extensão:** passo 3.

1. O sistema identifica que uma ou mais informações necessárias para realizar a consulta não foram fornecidas.
2. O sistema solicita ao usuário a informação necessária.
3. O usuário fornece a informação solicitada.
4. O sistema retoma o fluxo principal a partir do passo 3.

**Exemplo:**

> Usuário: "Qual a situação do livro?"

> Assistente: "Qual livro você gostaria de consultar?"

> Usuário: "Clean Code."

A informação fornecida no segundo turno é incorporada à tarefa em andamento.

---

## FA02 — Consulta dependente de contexto

**Ponto de extensão:** passo 3.

1. O sistema identifica que a solicitação contém uma referência a uma informação apresentada anteriormente.
2. O sistema utiliza o contexto disponível para determinar a referência.
3. Caso a referência possa ser determinada adequadamente, o sistema prossegue para o passo 4.

**Exemplo:**

> Usuário: "O livro Clean Code está disponível?"

> Assistente: "Sim, há um exemplar disponível."

> Usuário: "Quem é o autor?"

O sistema utiliza o contexto da interação anterior para interpretar que a pergunta se refere a *Clean Code*.

---

## FA03 — Referência ambígua

**Ponto de extensão:** passo 3.

1. O sistema identifica que uma referência ou parâmetro pode corresponder a mais de uma informação.
2. O sistema não seleciona arbitrariamente uma das possibilidades.
3. O sistema solicita ao usuário informações adicionais para esclarecer a solicitação.
4. Após receber a informação, o sistema retoma o fluxo principal.

**Exemplo:**

> Usuário: "Qual é a situação do livro do Silva?"

Caso existam vários livros compatíveis com essa descrição:

> Assistente: "Encontrei mais de um livro de autoria de Silva. Você se refere a qual deles?"

---

## FA04 — Consulta sem necessidade de ferramenta de domínio

**Ponto de extensão:** passo 4.

1. O sistema identifica que a solicitação pode ser respondida utilizando informações disponíveis no próprio contexto conversacional, sem necessidade de consultar o sistema de domínio.
2. O sistema gera a resposta diretamente.
3. O fluxo prossegue para o passo 9.

Este fluxo é aplicável somente quando a informação necessária estiver efetivamente disponível ao assistente e não exigir consulta ao sistema de informação.

---

# 8. Fluxos de exceção

## FE01 — Ferramenta necessária não disponível

**Ponto de extensão:** passo 4.

1. O sistema identifica a intenção do usuário.
2. Não é encontrada entre as ferramentas disponíveis uma operação compatível com a solicitação.
3. O sistema informa que não pode realizar aquela consulta.
4. O caso de uso é encerrado.

O sistema não deve simular uma consulta nem apresentar informações como se a ferramenta tivesse sido executada.

---

## FE02 — Falha na validação dos argumentos

**Ponto de extensão:** passo 5.

1. O sistema identifica que os argumentos necessários não são válidos ou estão incompletos.
2. O sistema evita a execução da ferramenta.
3. Quando a situação puder ser resolvida por interação, o sistema solicita ao usuário a informação necessária.
4. Caso contrário, informa que não foi possível realizar a consulta.

---

## FE03 — Falha na execução da ferramenta

**Ponto de extensão:** passo 6.

1. A ferramenta é chamada com os argumentos válidos.
2. A execução retorna uma falha ou indisponibilidade.
3. O sistema informa ao usuário que não foi possível realizar a consulta.
4. O sistema não apresenta a operação como concluída com sucesso.

---

## FE04 — Resultado não encontrado

**Ponto de extensão:** passo 7.

1. A ferramenta é executada corretamente.
2. A consulta não retorna informações correspondentes aos parâmetros fornecidos.
3. O sistema informa ao usuário que não foram encontrados resultados compatíveis.

---

# 9. Regras e restrições

**RN01 — Não execução sob informação insuficiente**

O sistema não deve executar uma ferramenta quando as informações necessárias para determinar a operação não estiverem suficientemente determinadas.

**RN02 — Fidelidade ao resultado**

A resposta apresentada ao usuário deve ser compatível com o resultado efetivamente retornado pela ferramenta.

**RN03 — Ferramentas autorizadas**

O agente deve utilizar somente ferramentas explicitamente disponibilizadas para o sistema.

**RN04 — Tratamento de ambiguidades**

Quando uma solicitação não puder ser determinada de maneira adequada devido à ambiguidade, o sistema deve solicitar esclarecimento antes de executar uma operação potencialmente incorreta.

---

# 10. Requisitos relacionados

O caso de uso realiza ou exercita os seguintes requisitos funcionais:

| Requisito | Relação |
|---|---|
| RF01 — Receber e processar entrada de voz | Entrada da solicitação |
| RF02 — Interpretar solicitação em linguagem natural | Identificação da intenção |
| RF03 — Gerenciar contexto conversacional | Continuidade e referências |
| RF04 — Gerenciar informações necessárias à tarefa | Parâmetros, ausência e ambiguidade |
| RF05 — Selecionar ferramenta para execução | Determinação da operação |
| RF06 — Preparar e validar chamada de ferramenta | Preparação da consulta |
| RF07 — Executar ferramentas autorizadas | Execução |
| RF08 — Processar resultados das ferramentas | Interpretação do resultado |
| RF09 — Tratar falhas e recuperar a interação | Falhas e recuperação |
| RF11 — Gerar resposta textual | Formulação da resposta |
| RF12 — Entregar resposta por voz | Saída da interação |

O **RF10 — Gerenciar controle da interação** não constitui comportamento obrigatório do fluxo principal deste caso de uso, mas pode ser exercitado em cenários de cancelamento ou encerramento da interação.

---

# 11. Cenários de verificação derivados

Para este caso de uso, inicialmente são previstos os seguintes cenários:

| ID | Cenário | Comportamento esperado |
|---|---|---|
| C01.01 | Consulta simples | Informação solicitada é retornada |
| C01.02 | Consulta com parâmetro | Parâmetro é utilizado corretamente |
| C01.03 | Informação ausente | Sistema solicita informação |
| C01.04 | Consulta contextual | Referência anterior é utilizada |
| C01.05 | Referência ambígua | Sistema solicita esclarecimento |
| C01.06 | Ferramenta inexistente | Sistema informa impossibilidade |
| C01.07 | Argumentos inválidos/incompletos | Execução é evitada |
| C01.08 | Falha da ferramenta | Falha é comunicada sem falso sucesso |
| C01.09 | Resultado vazio | Sistema informa ausência de resultados |
| C01.10 | Cancelamento | Usuário consegue interromper a interação quando aplicável |

Esses cenários têm finalidade de **verificação dos requisitos e comportamentos especificados**, não constituindo, nesta versão do trabalho, um protocolo de avaliação experimental.

---

# 12. Observações

Este caso de uso representa uma capacidade de alto nível do assistente, enquanto os mecanismos de interpretação, gerenciamento de contexto, seleção e execução de ferramentas são tratados como comportamentos internos e transversais.

A implementação concreta desses mecanismos será especificada nos artefatos de projeto e arquitetura.

O domínio de biblioteca utilizado nos exemplos representa um estudo de caso. O caso de uso deve permanecer suficientemente abstrato para que sua realização possa ser adaptada a outros domínios mediante a disponibilização das ferramentas correspondentes.
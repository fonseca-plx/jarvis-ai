# UC02 — Consultar Disponibilidade

**Identificador:** UC02  
**Nome:** Consultar disponibilidade  
**Versão:** 1.0  
**Prioridade:** Alta

## 1. Objetivo

Permitir que o usuário consulte, por meio de uma solicitação em linguagem natural e voz, a disponibilidade de um recurso registrado no sistema de informação de domínio.

O assistente deve identificar o recurso solicitado, determinar os parâmetros necessários para a consulta, utilizar uma ferramenta compatível e apresentar ao usuário o estado de disponibilidade obtido.

---

## 2. Atores

### Ator principal

**Usuário**

Pessoa que deseja consultar a disponibilidade de um recurso.

### Atores secundários

**Sistema de Informação de Domínio**

Sistema responsável por armazenar e disponibilizar as informações sobre os recursos.

**Ferramenta**

Interface disponibilizada ao agente para consultar a disponibilidade do recurso no sistema de domínio.

---

## 3. Gatilho

O usuário inicia uma interação e solicita informações sobre a disponibilidade de um recurso.

**Exemplos:**

> "O livro Clean Code está disponível?"

> "Tem algum exemplar de Dom Casmurro disponível?"

> "Posso pegar o livro que você acabou de consultar?"

---

## 4. Pré-condições

1. O assistente deve estar disponível para receber uma interação.
2. Deve existir uma ferramenta capaz de consultar a disponibilidade dos recursos correspondentes ao domínio.
3. A ferramenta deve possuir uma interface e os parâmetros necessários definidos.
4. O sistema de informação de domínio deve estar disponível quando a consulta depender de seus dados.

---

## 5. Pós-condições

### Sucesso

1. O recurso solicitado foi identificado.
2. Os parâmetros necessários foram determinados.
3. Uma ferramenta compatível foi selecionada.
4. A consulta foi executada.
5. O resultado foi processado.
6. O usuário recebeu uma resposta correspondente à disponibilidade encontrada.

### Falha

Caso a disponibilidade não possa ser determinada, o sistema deve comunicar adequadamente a situação ao usuário e não apresentar uma disponibilidade que não tenha sido efetivamente obtida.

---

# 6. Fluxo principal

**1. Usuário solicita a disponibilidade de um recurso.**

O usuário apresenta, por voz, uma solicitação relacionada à disponibilidade de um recurso.

**2. Sistema recebe e interpreta a solicitação.**

O assistente converte a entrada de voz em uma representação textual e identifica a intenção de consultar disponibilidade.

**3. Sistema identifica o recurso solicitado.**

O assistente extrai da solicitação as informações necessárias para identificar o recurso.

**4. Sistema determina os parâmetros da consulta.**

O assistente verifica se as informações disponíveis são suficientes para realizar a consulta.

Quando aplicável, informações fornecidas anteriormente na conversa podem ser utilizadas para completar a solicitação.

**5. Sistema seleciona a ferramenta adequada.**

O assistente seleciona uma ferramenta compatível com a consulta de disponibilidade.

**6. Sistema prepara e valida os argumentos.**

O assistente gera os argumentos necessários para a ferramenta e realiza as validações aplicáveis.

**7. Sistema executa a consulta.**

A ferramenta selecionada é executada com os argumentos validados.

**8. Sistema processa o resultado.**

O assistente interpreta o resultado retornado pela ferramenta e determina a disponibilidade do recurso.

**9. Sistema gera a resposta.**

O assistente produz uma resposta textual correspondente ao resultado obtido.

**10. Sistema entrega a resposta por voz.**

A resposta textual é convertida em voz e apresentada ao usuário.

**11. Caso de uso é concluído.**

---

# 7. Fluxos alternativos

## FA01 — Recurso não especificado

**Ponto de extensão:** passo 3.

1. O sistema identifica que o usuário solicitou uma consulta de disponibilidade, mas não informou qual recurso deseja consultar.
2. O sistema solicita ao usuário a identificação do recurso.
3. O usuário fornece a informação.
4. O sistema retoma o fluxo a partir do passo 3.

**Exemplo:**

> Usuário: "Esse livro está disponível?"

Caso não exista contexto suficiente para determinar "esse livro":

> Assistente: "Qual livro você gostaria de consultar?"

---

## FA02 — Recurso identificado por contexto

**Ponto de extensão:** passo 3.

1. O sistema identifica que o usuário utiliza uma referência a um recurso mencionado anteriormente.
2. O sistema consulta o contexto da conversa.
3. Caso a referência possa ser determinada de forma adequada, o recurso é associado à solicitação.
4. O sistema prossegue para o passo 5.

**Exemplo:**

> Usuário: "Você conhece o livro Clean Code?"

> Assistente: "Sim."

> Usuário: "Ele está disponível?"

Nesse caso, "ele" pode ser associado ao recurso mencionado anteriormente.

---

## FA03 — Múltiplos parâmetros fornecidos na mesma interação

**Ponto de extensão:** passo 4.

1. O usuário fornece mais de uma informação relevante para identificar o recurso.
2. O sistema identifica os parâmetros independentemente da ordem em que foram apresentados.
3. Os parâmetros são utilizados conjuntamente para determinar o recurso.
4. O sistema prossegue para o passo 5.

**Exemplo:**

> "Quero saber se o livro de Robert Martin, Clean Code, está disponível."

O sistema pode utilizar conjuntamente o título e o autor para identificar o recurso.

---

## FA04 — Informação adicional necessária

**Ponto de extensão:** passo 4.

1. O sistema identifica que os dados fornecidos não são suficientes para determinar de forma adequada o recurso.
2. O sistema solicita uma informação adicional.
3. O usuário fornece a informação.
4. O sistema incorpora a informação à tarefa.
5. O fluxo retorna ao passo 4.

**Exemplo:**

> Usuário: "O livro do Silva está disponível?"

Caso existam vários livros de autores ou títulos compatíveis:

> Assistente: "Qual livro de Silva você quer consultar?"

---

## FA05 — Consulta contextual após identificação anterior

**Ponto de extensão:** passo 4.

1. O usuário solicita inicialmente uma informação sobre determinado recurso.
2. O sistema identifica o recurso e responde.
3. O usuário realiza uma nova solicitação relacionada ao mesmo recurso.
4. O sistema utiliza o contexto para evitar que o usuário precise repetir todas as informações.

**Exemplo:**

> Usuário: "Quem escreveu Clean Code?"

> Assistente: "Robert C. Martin."

> Usuário: "Está disponível?"

O sistema interpreta a segunda solicitação como uma consulta de disponibilidade de *Clean Code*.

---

# 8. Fluxos de exceção

## FE01 — Recurso não encontrado

**Ponto de extensão:** passo 7.

1. A ferramenta é executada com os parâmetros determinados.
2. O sistema de informação não encontra um recurso correspondente.
3. O assistente informa ao usuário que o recurso não foi encontrado.
4. O caso de uso é encerrado.

---

## FE02 — Ferramenta de disponibilidade não disponível

**Ponto de extensão:** passo 5.

1. O sistema identifica a intenção de consultar disponibilidade.
2. Não existe ferramenta disponível capaz de realizar a operação.
3. O assistente informa que não consegue realizar a consulta.
4. O caso de uso é encerrado.

O sistema não deve apresentar uma disponibilidade estimada ou inventada.

---

## FE03 — Falha na validação dos argumentos

**Ponto de extensão:** passo 6.

1. O sistema identifica que os argumentos necessários não são válidos ou estão incompletos.
2. O sistema impede a execução da ferramenta.
3. Quando a situação puder ser corrigida por interação, solicita ao usuário a informação necessária.
4. Caso contrário, informa que não foi possível realizar a consulta.

---

## FE04 — Falha na execução da ferramenta

**Ponto de extensão:** passo 7.

1. A ferramenta é chamada com argumentos válidos.
2. O sistema de informação retorna uma falha ou está indisponível.
3. O assistente comunica ao usuário que não foi possível consultar a disponibilidade.
4. O assistente não apresenta a operação como concluída com sucesso.

---

# 9. Regras e restrições

**RN01 — Identificação suficiente do recurso**

A consulta não deve ser executada enquanto não houver informações suficientes para determinar o recurso pretendido.

**RN02 — Tratamento de ambiguidades**

Quando houver mais de um recurso compatível com as informações fornecidas, o sistema deve solicitar esclarecimento antes de realizar a consulta.

**RN03 — Uso de contexto**

Informações previamente fornecidas pelo usuário podem ser utilizadas para completar a solicitação quando sua associação com a nova solicitação puder ser determinada adequadamente.

**RN04 — Fidelidade ao resultado**

A resposta deve representar a disponibilidade efetivamente retornada pelo sistema de informação.

**RN05 — Ferramentas autorizadas**

A consulta deve utilizar somente ferramentas explicitamente disponibilizadas ao agente.

---

# 10. Requisitos relacionados

| Requisito | Relação |
|---|---|
| RF01 — Receber e processar entrada de voz | Entrada da solicitação |
| RF02 — Interpretar solicitação em linguagem natural | Identificação da intenção |
| RF03 — Gerenciar contexto conversacional | Referências e continuidade |
| RF04 — Gerenciar informações necessárias à tarefa | Identificação, ausência e ambiguidade |
| RF05 — Selecionar ferramenta para execução | Seleção da consulta |
| RF06 — Preparar e validar chamada de ferramenta | Preparação dos argumentos |
| RF07 — Executar ferramentas autorizadas | Execução |
| RF08 — Processar resultados das ferramentas | Interpretação da disponibilidade |
| RF09 — Tratar falhas e recuperar a interação | Tratamento de falhas |
| RF10 — Gerenciar controle da interação | Cancelamento/encerramento, quando aplicável |
| RF11 — Gerar resposta textual | Formulação da resposta |
| RF12 — Entregar resposta por voz | Saída da interação |

---

# 11. Cenários de verificação derivados

| ID | Cenário | Comportamento esperado |
|---|---|---|
| C02.01 | Consulta simples de disponibilidade | Disponibilidade correta é apresentada |
| C02.02 | Recurso não informado | Sistema solicita identificação |
| C02.03 | Recurso identificado por contexto | Referência é corretamente resolvida |
| C02.04 | Múltiplos parâmetros na mesma fala | Parâmetros são utilizados conjuntamente |
| C02.05 | Recurso ambíguo | Sistema solicita esclarecimento |
| C02.06 | Recurso inexistente | Sistema informa que não encontrou o recurso |
| C02.07 | Argumentos insuficientes | Execução é evitada ou informação é solicitada |
| C02.08 | Ferramenta indisponível | Sistema comunica a impossibilidade |
| C02.09 | Falha durante execução | Falha é comunicada sem falso sucesso |
| C02.10 | Cancelamento da consulta | Interação é interrompida quando solicitado |

Esses cenários têm finalidade de **verificação dos requisitos e comportamentos especificados**, não constituindo um protocolo de avaliação experimental.

---

# 12. Observações

Este caso de uso representa uma especialização de consulta orientada à determinação do estado de disponibilidade de um recurso.

A identificação do recurso pode utilizar informações fornecidas na interação atual ou informações preservadas no contexto conversacional.

Os mecanismos concretos utilizados para resolução de contexto, seleção de ferramentas, geração de argumentos e processamento de resultados serão especificados nos artefatos de arquitetura e projeto.

O domínio de biblioteca constitui o estudo de caso utilizado para demonstrar a aplicação do assistente. A capacidade descrita neste caso de uso pode ser adaptada a outros domínios nos quais exista uma operação equivalente de consulta de disponibilidade.
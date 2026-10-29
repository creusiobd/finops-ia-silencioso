---
name: finops-ia-silencioso
description: Camada silenciosa de FinOps de IA: antes de cada tarefa avalia complexidade, nível de esforço, custo em tokens e valor, aplica limites de crédito por nível e escolhe o caminho mais barato que entrega o resultado.
---

# Agente FinOps de IA (modo invisível)

O objetivo é entregar o máximo de valor com o mínimo de tokens, créditos e tempo de espera, sem que o utilizador precise de pensar nisso. Esta camada corre antes e durante cada tarefa e só aparece quando há uma decisão de custo que o utilizador deve tomar.

## Regra de invisibilidade

- Não narrar a triagem, os níveis nem as estimativas. Executar já com o plano certo.
- Falar de custo apenas em quatro casos:
  1. A tarefa é N4 e há uma alternativa bem mais barata com 80% do valor: uma frase, uma pergunta.
  2. O utilizador pede ("quanto gastou?", "foi eficiente?", "relatório de custo").
  3. O limite de crédito do nível foi atingido (ver Limites de crédito).
  4. O custo real vai passar 2x o limite do nível: parar antes e pedir decisão.
- Nunca pedir licença para ser eficiente.

## Passo 1: Triagem em 10 segundos

Antes de qualquer ferramenta, classificar o pedido:

| Nível | Sinais | Exemplos |
|---|---|---|
| N0 Resposta | Conhecimento geral, conversa, explicação | "o que é SLO?" |
| N1 Consulta | Um facto atual, um ficheiro, um dado | preço de um modelo, câmbio, ler um CSV |
| N2 Tarefa | Um entregável simples | email, script curto, tabela |
| N3 Projeto | Pesquisa + síntese + ficheiro formatado | relatório, deck, playbook |
| N4 Programa | Várias fontes, várias saídas, iteração longa | produto completo, auditoria ampla |

Se o pedido estiver entre dois níveis, escolher o mais baixo e subir só se o resultado não chegar.

## Limites de crédito por nível

Os créditos reais da sessão não são visíveis durante a execução, por isso o limite é gerido por estimativa em tokens e chamadas. Os valores reais podem ser confirmados no fim com a skill `explain-usage`.

| Nível | Limite de tokens (estimado) | Chamadas de ferramentas | Pesquisas | Subagentes |
|---|---|---|---|---|
| N0 | ~2 mil | 0 | 0 | 0 |
| N1 | ~10 mil | até 2 | 1 standard | 0 |
| N2 | ~40 mil | até 8 | até 3 standard | 0 |
| N3 | ~150 mil | até 25 | até 8 (máx. 2 extended) | 0, salvo pedido |
| N4 | definido com o utilizador | por fases de ~150 mil | conforme plano | só com confirmação |

Pesos para estimar o gasto:

- Pesquisa standard: ~2-4 mil tokens.
- Pesquisa extended: conta como 4 standard.
- Abrir uma página web (fetch com pergunta dirigida): ~1-3 mil tokens; página inteira sem pergunta: ~5-15 mil.
- Ler um ficheiro: ~1 mil tokens por cada 100 linhas.
- Subagente: ~50 mil tokens de arranque, mais o trabalho dele.
- Gerar documento formatado (docx, deck, página): ~10-30 mil tokens.

Gatilhos durante a execução:

- **80% do limite:** deixar de explorar; priorizar o que falta para entregar.
- **100% do limite:** entregar o que existe, com uma linha a dizer o que ficou de fora e quanto custaria completar (em nível equivalente).
- **Antes de passar 2x o limite:** parar e perguntar. Nunca ultrapassar sem confirmação.
- **N4:** propor o plano em fases com o custo estimado de cada uma; avançar fase a fase.

Ajustes pedidos pelo utilizador (válidos para a tarefa em curso, ou como padrão se ele disser que é para sempre):

- "modo económico": usar os limites do nível abaixo.
- "modo completo": usar os limites do nível acima, sem baixar a regra de 2x.
- Um limite explícito ("no máximo X tokens" ou "no máximo N pesquisas") substitui a tabela.

## Passo 2: Escolher a rota mais barata que entrega

Por ordem de custo, usar a primeira que resolve:

1. Responder do contexto já carregado (conversa, perfil, preferências, resultados de tarefas anteriores da mesma conversa).
2. Uma leitura dirigida (ficheiro ou memória específica, não "para ver").
3. Abrir diretamente a página da fonte, quando o URL já é conhecido (ver Passo 4).
4. Uma pesquisa web standard.
5. Várias pesquisas standard em paralelo, na mesma chamada.
6. Pesquisa extended: só para factos muito recentes ou nichos onde a standard veio fraca.
7. Subagentes: só quando o utilizador pede, ou quando o trabalho paralelo é grande e independente. Cada subagente começa a frio e reprocessa contexto; é a rota mais cara.

## Passo 3: Conhecer o utilizador sem gastar

- Perfil e preferências já carregados contam como lidos. Não reler.
- Ler um ficheiro de memória só quando a descrição indica que contém algo que muda a resposta.
- Não perguntar o que já está em memória ou no contexto.
- No máximo uma pergunta de clarificação, e só quando a resposta muda o que se vai construir. Caso contrário, assumir o padrão razoável e dizer a suposição numa linha.

## Passo 4: Pesquisa eficiente

### Regra dos números (preços, câmbios, tarifas, cotações)

As pesquisas devolvem muitas vezes só títulos e links, sem o número. Para dados numéricos:

1. Se o URL de uma fonte fiável já apareceu na conversa ou numa pesquisa anterior, abrir essa página diretamente, com uma pergunta dirigida ("qual o preço de entrada e saída do modelo X e a data"). Sem nova pesquisa.
2. Se não há URL conhecido, fazer uma única pesquisa e, na ronda seguinte, abrir logo a melhor fonte (oficial primeiro, agregador atualizado como alternativa). Nunca fazer uma segunda pesquisa à espera de que o excerto traga o número.
3. Se a página aberta não tiver o dado exato, aceitar o equivalente mais próximo que ela traz e dizê-lo numa linha, em vez de abrir mais páginas.
4. Abrir várias fontes em paralelo, na mesma ronda, quando são independentes (ex.: preços de três fornecedores).

### Regras gerais

- Planear as consultas antes e enviá-las juntas, em paralelo.
- Consultas específicas (entidade + atributo + ano) em vez de genéricas.
- Regra de paragem: duas fontes credíveis concordantes chegam para um facto; parar.
- Não abrir páginas inteiras quando o excerto da pesquisa já responde.
- Preços, modelos, planos e câmbios mudam: obter sempre de uma fonte atual, nunca citar de memória.
- Reutilizar na mesma conversa um dado já obtido (ex.: câmbio do dia) em vez de o procurar outra vez.

## Passo 5: Controlo de queima durante a execução

Fontes típicas de desperdício, a evitar:

- Ler ficheiros grandes inteiros quando bastava um intervalo ou um grep.
- Reescrever um ficheiro inteiro para mudar uma linha (usar edição pontual).
- Repetir a mesma chamada falhada sem mudar nada.
- Pesquisar números em vez de abrir a página que os tem.
- Carregar skills de formato antes de ter o conteúdo.
- Gerar várias versões quando foi pedida uma.
- Respostas longas com resumo do que o utilizador já viu.

Checkpoint a meio de N3/N4: o valor entregue até aqui justifica continuar ao mesmo ritmo? Se não, reduzir âmbito.

## Passo 6: Custo-benefício (valor entregue)

Antes de entregar, verificar em silêncio:

- O entregável responde ao pedido real, não a uma versão inflacionada?
- Existe uma parte cara que acrescentou pouco? Cortar na próxima vez.
- O formato é o mais útil (preferências: passo a passo, concreto, Markdown/.docx)?

Regra de ROI: só escalar de nível se o ganho esperado for claramente maior que o custo extra (ex.: decisão de negócio, dinheiro em jogo, documento para cliente).

## Modo relatório (só quando pedido)

Quando o utilizador pedir avaliação de custo ou eficiência, entregar:

1. Nível atribuído, limite do nível e se foi respeitado (1-2 linhas).
2. Tabela: etapa | chamadas | tokens estimados | valor gerado.
3. Onde houve desperdício e quanto se poupava.
4. Rota recomendada para tarefas semelhantes.

Se a skill `explain-usage` estiver disponível, usá-la para números reais da sessão em vez de estimativas.

## Modo consultoria FinOps de IA (para o negócio do utilizador)

Quando o pedido for avaliar modelos, APIs ou aplicações de IA para um projeto:

1. Obter preços atuais (input/output por milhão de tokens, cache, batch, planos) pela Regra dos números.
2. Estimar volume: pedidos/dia x tokens por pedido (entrada + saída).
3. Comparar 2-4 opções numa tabela: custo mensal, qualidade adequada à tarefa, latência, riscos.
4. Converter para a moeda local do projeto quando relevante (ex.: Kz), com o câmbio e a data.
5. Recomendar alavancas: modelo menor para tarefas N0-N2, cache de prompts, batch para trabalho não urgente, respostas sem IA quando os dados chegam, limitar tamanho de saída, roteamento por complexidade, limites de gasto por utilizador ou por funcionalidade.
6. Indicar o custo por unidade de valor (por transação, por cliente, por relatório) e o seu peso face à receita ou contribuição dessa unidade.

## Anti-padrões

- Explicar ao utilizador que está a poupar tokens.
- Fazer pesquisa extended por defeito.
- Pesquisar duas vezes o mesmo número em vez de abrir a fonte.
- Lançar subagentes para tarefas que cabem numa sequência de chamadas.
- Perguntar várias coisas antes de começar uma tarefa N1-N2.
- Ultrapassar 2x o limite do nível sem confirmação.
- Sacrificar a correção para poupar: uma resposta errada barata é o custo mais alto.

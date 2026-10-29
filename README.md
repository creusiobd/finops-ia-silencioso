# finops-ia-silencioso

Skill de FinOps de IA que governa o **consumo do próprio agente** enquanto trabalha: classifica cada tarefa por nível de esforço, aplica limites de crédito e escolhe o caminho mais barato que entrega o resultado. Funciona em silêncio e só fala de custo quando há uma decisão a tomar.

Escrita em português, com modo de consultoria para avaliar custos de IA em projetos (incluindo conversão para moeda local, como o kwanza).

## O que a distingue

A maioria das skills de custo de IA otimiza **aplicações** (custo de API em produção, roteamento de modelos, dashboards). Esta atua **dentro da sessão**, sobre o próprio agente:

| | Skills de custo de aplicações | finops-ia-silencioso |
|---|---|---|
| Onde atua | Na app que usa a API | No agente, durante a conversa |
| Classifica tarefas | Não (recomenda roteamento) | Sim, N0 a N4 |
| Limites de gasto | Alertas de orçamento na cloud | Limite por nível, com paragem aos 2x |
| Fala com o utilizador | Sempre (é um guia) | Só quando há decisão de custo |

## Como funciona

1. **Triagem:** classifica o pedido de N0 (resposta direta) a N4 (programa grande).
2. **Limites de crédito:** cada nível tem um teto de tokens, chamadas e pesquisas. Aos 80% foca a entrega; aos 100% entrega o que tem; antes de 2x para e pergunta.
3. **Rota mais barata:** contexto → leitura dirigida → fonte conhecida → pesquisa → pesquisa aprofundada → subagentes.
4. **Regra dos números:** para preços e câmbios, abre a página da fonte em vez de pesquisar duas vezes.
5. **Custo-benefício:** só sobe de nível quando o ganho compensa.

| Nível | Tokens (est.) | Chamadas | Pesquisas |
|---|---|---|---|
| N0 | ~2 mil | 0 | 0 |
| N1 | ~10 mil | até 2 | 1 |
| N2 | ~40 mil | até 8 | até 3 |
| N3 | ~150 mil | até 25 | até 8 |
| N4 | combinado | por fases | conforme plano |

## Instalação

**Claude (claude.ai / app):** Definições → Capacidades → Skills → carregar a pasta `finops-ia-silencioso` em ZIP.

**Claude Code:** copiar a pasta para `~/.claude/skills/finops-ia-silencioso/` (ou `.claude/skills/` no projeto).

A skill entra em ação sozinha. Para ver o gasto, pede no fim: "relatório de custo".

## Comandos úteis

- `modo económico` — usa os limites do nível abaixo
- `modo completo` — usa os limites do nível acima
- `no máximo N pesquisas` — limite explícito

## Exemplos

Ver [`exemplos/bateria-testes.md`](exemplos/bateria-testes.md): uma tarefa por nível e o gasto real face ao limite.

## Limitações

- Os créditos reais não são visíveis durante a sessão; os limites são estimativas. Use `explain-usage` (quando disponível) para números reais.
- Preços de modelos e câmbios mudam: a skill obriga a obtê-los de fontes atuais, nunca de memória.

## Autor

Creusio Kizua · [LinkedIn](https://www.linkedin.com/in/creusio-adolfo-gaspar-kizua-a1961b108) · [YouTube](https://www.youtube.com/@creusiokizua)

Licença: [MIT](LICENSE)

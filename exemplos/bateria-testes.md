# Bateria de testes (29/10/2026)

Uma tarefa por nível, executada com as regras da skill. Valores de preço e câmbio são da data do teste e servem só de exemplo.

| Nível | Tarefa | Limite | Usado | Resultado |
|---|---|---|---|---|
| N0 | "O que é um SLO?" | 0 chamadas | 0 | Resposta direta |
| N1 | "Taxa do dólar em kwanzas hoje" | até 2 | 2 (pesquisa + página) | No limite: a pesquisa não trouxe o número |
| N2 | "Custo de um assistente de IA em Kz" | até 8 | 0 | Reutilizou o câmbio e custos já obtidos |
| N3 | Relatório de custos de 4 modelos para um assistente de poupança comunitária | até 25 / ~150 mil tokens | 19 / ~50 mil | Dentro do limite; 1 ronda de pesquisa desperdiçada |
| N4 | "Construir a plataforma completa" | só com confirmação | 0 | Parou e propôs 4 fases de ~150 mil tokens |

## Lições que geraram a v3

- Pesquisas web devolvem muitas vezes só títulos. Para números, abrir a fonte diretamente.
- Dados obtidos numa tarefa (ex.: câmbio) devem ser reutilizados nas seguintes.

## Exemplo de relatório de custo (N3)

| Item | Limite N3 | Usado |
|---|---|---|
| Chamadas | até 25 | 19 |
| Pesquisas + páginas | até 8 | 6 |
| Pesquisas aprofundadas | máx. 2 | 0 |
| Subagentes | 0 | 0 |
| Tokens (est.) | ~150 mil | ~50 mil |

# Tutorial: Perguntas em Lote (Estatística)

A ferramenta **Perguntas em Lote** vai além de uma simples resposta; ela simula a repetição de uma mesma pergunta centenas ou milhares de vezes para entregar um levantamento matemático da tendência gerada.

## Como Utilizar

1. **Faça sua pergunta:** Na barra de texto, insira a pergunta (preferencialmente algo de resposta clara com Sim/Não). Exemplos: *"Devo focar nesse projeto hoje?"*, *"Houve influência externa aqui?"*.
2. **Determine as repetições (Execuções):** Diga à ferramenta quantas mil vezes ela vai rodar as roletas de probabilidade (por padrão 1000 vezes, até um milhão). Recomendamos manter em 1000 para velocidade, e 10.000 para extrema precisão.
3. **Clique em "Responder" (ou tecle Enter):** A aplicação rolará os algoritmos para classificar os resultados.

## Interpretando os Resultados

A ferramenta gera milhares de respostas categorizadas matematicamente em 4 pilares: "Sim", "Não", "Não Sei", "Reformule". 
Por regra estatística, o limiar é 25% para cada uma.

- **Maioria Estatística (Conclusiva):** Só exibe como aposta definitiva se uma alternativa superar os 30% (+5 pontos percentuais acima do ruído estatístico previsto de 25%).
- **Resultado Inconclusivo:** Se as respostas flutuarem de modo muito equilibrado (exemplo: 26% Sim, 24% Não) e ninguém ganhar pelo limiar de 30%, a aplicação alertará ser inconclusiva.

## Acompanhando os Dados
Um histórico detalhado mantém os dados da atual sessão. Para documentar a pesquisa, ou importar num programa externo (como o Excel ou Pandas), clique no fim da página para **Exportar Relatório CSV**.

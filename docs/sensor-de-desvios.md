# Tutorial: Sensor de Desvios Estatísticos

O **Sensor de Desvios Estatísticos** opera sob um contínuo mecanismo simulatório usando um modelo de Monte Carlo simplificado. Ele detecta variações estatísticas comparando os dados brutos com a margem de 25% esperada de ocorrência.

## Configuração Inicial

1. **Acesse as Configurações:** Ao iniciar a aplicação, a primeira coisa a ajustar é a dimensão do lote, normalmente estabelecido em `1.000.000` (um milhão de gerações).
2. **Defina Nomes (Labels):** Você possui 4 rótulos (Faixa A, B, C e D) para as partições de dados; se o uso for correlacionado a certas condições, defina-as aqui. Por exemplo, pode-se usar "Sim", "Não", "Talvez" e "Não sei" para simular um questionário.
3. **Limiar de Desvio Limitante (%):** Por margem padrão, qualquer quebra de `5.0` pontos vai emitir alerta visual no radar sonoro. Podem ser postos valores críticos (ex `1.0` ou `2.0`) para altíssima sensibilidade a ruídos pseudoaleatórios. Normalmente é recomendado valores acima de 5, mas esses valores podem mudar conforme a natureza do experimento.

## Uso do Monitor do Radar

Uma vez iniciado o monitoramento, a interface altera para a exibição de um "radar" (spider-chart), simulando instrumentos de registro físico ou sismógrafos. A figura desenhada no perímetro é a oscilação matemática daquele segundo e lote.

- **Resultado Neutro:** Permanece o ciano pálido, com pulso cadenciado.
- **Desvio Detectado:** Quando atinge-se limiar configurado nas configurações inicias, a cor rompe ao vermelho, indicando que uma das "faixas" apresentou eventos fora da ordem controlada provável.

É possível gerar relatórios de **exportação de CSV**, pausar dinamicamente e retomar a escuta do radar conforme os eventos na sua mesa de trabalho.

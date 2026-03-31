# Spec: Sensor de Presença Estatística

## Visão Geral

Aplicação HTML standalone que executa lotes contínuos de simulações aleatórias (Monte Carlo simplificado), classifica os resultados em quatro faixas de probabilidade e monitora desvios em relação ao esperado estatístico (25% por faixa). A metáfora visual central é a de um **sensor de presença** ou **radar de campo**, não um dashboard analítico. A interface deve evocar instrumentação científica de precisão — algo entre um osciloscópio e um sonar.

---

## Requisito Técnico

- Arquivo único HTML standalone (sem dependências externas obrigatórias; CDN permitido)
- Sem backend, sem servidor — tudo roda no navegador

---

## Lógica de Execução

### Unidade de execução

Cada "execução" consiste em gerar um número pseudoaleatório inteiro entre 0 e 99 (inclusive) via `Math.random()` e classificá-lo em uma das quatro faixas:

| Faixa | Intervalo | Label padrão |
|-------|-----------|--------------|
| A | 0 – 24 | Configurável pelo usuário |
| B | 25 – 49 | Configurável pelo usuário |
| C | 50 – 74 | Configurável pelo usuário |
| D | 75 – 99 | Configurável pelo usuário |

### Lote

Um lote é um conjunto de N execuções (padrão: 1.000.000). Ao final de cada lote, calcula-se o percentual de ocorrência de cada faixa (contagem / N × 100).

### Execução contínua

Lotes são executados em loop contínuo, sem pausa entre eles, até o usuário interromper manualmente. Não há número máximo de lotes imposto pelo sistema.

### Processamento assíncrono (sem travar a UI)

Cada lote deve ser processado em **chunks assíncronos** usando `setTimeout` ou `requestAnimationFrame` para liberar o event loop entre chunks. Tamanho de chunk sugerido ao agente: entre 50.000 e 100.000 execuções por tick. O agente deve calibrar para que cada lote complete em menos de 3 segundos em hardware médio.

---

## Detecção de Desvio

### Definição

Para cada faixa, o desvio é calculado como:

```
desvio_faixa = percentual_observado - 25.0
```

Um **evento de desvio** ocorre quando `desvio_faixa >= limiar_configurado` para qualquer faixa.

### Comportamento ao detectar desvio

1. Registrar o evento automaticamente (sem pausar a execução)
2. Alterar visualmente o estado do radar (ver seção Visual)
3. Adicionar o evento ao log interno de desvios

### Parâmetro configurável

- **Limiar de desvio**: valor em pontos percentuais, padrão `5.0`, mínimo `0.1`, máximo `25.0`

---

## Tela de Configuração

Exibida antes do início da execução. Campos:

| Campo | Tipo | Padrão | Validação |
|-------|------|--------|-----------|
| Tamanho do lote | Inteiro | 1.000.000 | Mín: 10.000 / Máx: 10.000.000 |
| Limiar de desvio (%) | Decimal | 5.0 | Mín: 0.1 / Máx: 25.0 |
| Label da Faixa A | Texto | "Faixa A" | Máx: 20 caracteres |
| Label da Faixa B | Texto | "Faixa B" | Máx: 20 caracteres |
| Label da Faixa C | Texto | "Faixa C" | Máx: 20 caracteres |
| Label da Faixa D | Texto | "Faixa D" | Máx: 20 caracteres |

Um botão **"Iniciar Monitoramento"** inicia o loop de lotes e transita para a tela principal.

---

## Interface Principal

### Layout geral

A tela principal é dividida em duas regiões:

1. **Região do Radar** (elemento central dominante)
2. **Painel de Log** (lateral ou inferior, discreto)

### Componente Radar

Gráfico do tipo **spider/radar chart** circular com 4 eixos, um por faixa. Cada eixo representa o percentual observado daquela faixa no lote mais recente.

- O ponto neutro de cada eixo (25%) está marcado com uma linha de referência
- O polígono formado pelos 4 pontos anima suavemente a cada novo lote (transição de ~600ms)
- O centro do radar exibe o número do lote atual e um indicador de status ("monitorando" / "desvio detectado")

#### Estados visuais do radar

| Estado | Descrição visual |
|--------|-----------------|
| **Monitorando** | Polígono com cor neutra (ex: ciano pálido), linha de referência visível, pulso suave de "heartbeat" no círculo externo |
| **Desvio ativo** | Polígono muda para cor de alerta (ex: âmbar ou vermelho), o círculo externo pulsa mais intensamente, o eixo da faixa desviante é destacado |
| **Estabilizado** | Retorna ao estado "monitorando" no lote seguinte sem desvio |

#### Animação de "nuvem em movimento"

Entre lotes, o polígono do radar deve animar de forma fluida (não um corte abrupto). O agente pode adicionar um efeito de trail/ghost do polígono anterior que some gradualmente, reforçando a sensação de que a distribuição está "oscilando no espaço".

### Indicadores numéricos

Para cada faixa, exibir discretamente (próximo ao vértice do radar):
- Label configurado pelo usuário
- Percentual do lote atual (ex: `24.87%`)
- Desvio do esperado (ex: `−0.13pp`)

### Painel de Log de Desvios

Lista cronológica reversa dos eventos de desvio registrados. Cada entrada exibe:
- Número do lote
- Timestamp (HH:MM:SS)
- Faixa(s) que desviaram
- Percentual observado e desvio em pp

O painel tem altura máxima com scroll. Máximo de 500 entradas visíveis (as mais antigas são descartadas da UI, mas mantidas em memória para exportação).

### Controles

- Botão **"Pausar / Retomar"**: pausa o loop entre lotes (não interrompe um lote em andamento)
- Botão **"Exportar CSV"**: disponível a qualquer momento, exporta todos os eventos de desvio registrados
- Botão **"Encerrar"**: para o loop e retorna à tela de configuração (CSV é oferecido antes de encerrar se houver eventos registrados)

---

## Exportação CSV

### Gatilho

- Clique manual no botão "Exportar CSV"
- Ao clicar em "Encerrar" (se houver eventos registrados, exibir prompt de confirmação antes de encerrar)

### Conteúdo do arquivo

Apenas lotes que geraram pelo menos um evento de desvio. Colunas:

```
lote,timestamp,faixa_a_pct,faixa_b_pct,faixa_c_pct,faixa_d_pct,faixa_a_desvio,faixa_b_desvio,faixa_c_desvio,faixa_d_desvio,faixas_desviantes
```

- `lote`: número sequencial do lote (inteiro)
- `timestamp`: ISO 8601 local
- `faixa_X_pct`: percentual observado com 4 casas decimais
- `faixa_X_desvio`: desvio em pp com 4 casas decimais (positivo ou negativo)
- `faixas_desviantes`: lista separada por pipe das faixas que ultrapassaram o limiar (ex: `A|C`)

Nome do arquivo sugerido: `sensor_desvios_YYYYMMDD_HHMMSS.csv`

---

## Identidade Visual

### Conceito

**Instrumentação científica de campo, não dashboard corporativo.** A interface deve parecer um aparelho, não um site. Referências estéticas: sonar, osciloscópio, monitor de sinais vitais de laboratório, interface de ficção científica com contenção e precisão.

### Esquema de cores

Base **clara** (fundo off-white ou muito levemente tintado), com elementos de medição em cores frias e precisas. Não usar gradientes roxos ou paletas "SaaS genérico".

Sugestão de paleta (o agente pode variar dentro do conceito):
- Fundo: `#F4F4F0` ou similar (quase branco, levemente creme/cinza)
- Texto primário: `#1A1A2E` (azul muito escuro, quase preto)
- Cor neutra do radar: `#00B4D8` (ciano de precisão)
- Linha de referência: `#AAAAAA`
- Cor de desvio: `#FF6B35` (âmbar-laranja) ou `#E63946` (vermelho alerta)
- Acento de UI: `#023E8A` (azul profundo)

### Tipografia

Evitar fontes genéricas (Inter, Roboto, Arial). Priorizar fontes com caráter técnico/científico. Sugestões (via Google Fonts CDN):
- Display/títulos: **"Chakra Petch"** ou **"Share Tech Mono"**
- Corpo/dados: **"DM Mono"** ou **"IBM Plex Mono"**

### Animações

- Transição do polígono radar: `ease-in-out`, ~600ms
- Pulso do círculo externo em estado "monitorando": animação CSS contínua, sutil (~3s de ciclo)
- Pulso em estado "desvio ativo": mais rápido (~0.8s), com aumento de intensidade/opacidade
- Ghost trail do polígono anterior: fade-out em ~800ms
- Entrada de novos itens no log: slide suave vindo de baixo

### O que evitar

- Gradientes purple/pink
- Cards com sombras excessivas
- Ícones decorativos genéricos
- Qualquer coisa que pareça um dashboard de analytics ou SaaS

---

## Comportamento de Borda e Regras Implícitas

- Se o usuário fechar a aba sem exportar, os dados são perdidos (sem persistência local)
- A aplicação não deve usar `localStorage` ou `IndexedDB`
- Não há autenticação, múltiplos usuários ou sincronização
- Um lote em andamento não pode ser interrompido pelo botão Pausar; a pausa ocorre entre lotes
- O log de desvios em memória pode crescer indefinidamente (sem limite de registros para exportação)
- A UI deve ser responsiva o suficiente para funcionar em telas de laptop (mín. 1024px de largura); mobile não é requisito
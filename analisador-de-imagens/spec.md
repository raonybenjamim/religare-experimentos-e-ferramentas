# Spec: Sentinel — Analisador de Presença por Câmera

## Visão Geral

Sentinel é uma aplicação HTML standalone que captura o fluxo de vídeo da câmera do dispositivo, processa cada frame através de um método de análise configurável, e detecta variações significativas em relação a uma linha de base aprendida automaticamente. A aplicação não exibe o conteúdo da câmera ao usuário: toda a visualização é feita através de elementos de interface abstratos (círculo identificador, gráfico, tabela). Funciona em sessões curtas ou contínuas (horas/dias).

---

## Requisito Técnico

- **Formato**: Arquivo `.html` único, completamente standalone (sem dependências externas obrigatórias; CDN é permitido como fallback para bibliotecas de suporte, ex: Chart.js)
- **Sem backend**: toda lógica roda no browser via JavaScript puro
- **Compatibilidade mínima**: Chrome e Firefox modernos (Chromium ≥ 110, Firefox ≥ 110)

---

## Modos de Operação

A interface expõe um alternador visível com dois modos:

### Modo Livre
- A análise corre continuamente
- Cada detecção é registrada automaticamente na tabela de eventos
- O círculo identificador pulsa/muda de cor ao detectar, mas a análise não para
- Adequado para monitoramento passivo de longa duração

### Modo Alerta
- A análise corre normalmente até que uma detecção ocorra
- Ao detectar, a interface entra em **estado de alerta**: análise pausada, círculo em estado de alerta visual persistente
- Um botão **"Confirmar / Retomar"** aparece explicitamente na tela
- Ao clicar no botão, a interface reseta o estado de alerta e retoma a análise
- Adequado para uso como identificador de presença com necessidade de atenção humana

---

## Métodos de Análise

O usuário escolhe um dos três métodos via seletor na interface. Cada método tem seu próprio limiar configurável (ver seção Calibração).

### 1. Hash de Frame (padrão)
- A cada intervalo configurável (default: 500ms), captura o frame atual no canvas
- Reduz o frame para uma resolução muito baixa (ex: 16×16 pixels, escala de cinza) para criar uma "impressão digital" compacta
- Computa um hash perceptual simplificado (diferença entre pixels adjacentes, convertida em valor numérico escalar)
- Compara com o valor de referência (baseline); variância = `|hash_atual - hash_baseline|`
- Aciona detecção se `variância > limiar_hash`

### 2. Análise de Luminância
- A cada intervalo, captura o frame e calcula a **luminância média** de todos os pixels: `L = 0.299R + 0.587G + 0.114B` (padrão Rec.601)
- Compara com a luminância média do baseline; variância = `|L_atual - L_baseline|`
- Valor numérico entre 0–255
- Aciona detecção se `variância > limiar_luminancia`

### 3. Análise de Espectro de Cores
- A cada intervalo, computa a **média de cada canal** (R, G, B) separadamente
- Calcula a distância euclidiana entre o vetor RGB atual e o vetor RGB do baseline: `variância = √((ΔR² + ΔG² + ΔB²))`
- Valor numérico ≥ 0
- Aciona detecção se `variância > limiar_espectro`

---

## Calibração Automática de Baseline

- Ao iniciar a análise (botão **Iniciar**), a aplicação entra em **fase de calibração** por um período configurável (default: 5 segundos, ajustável entre 3s e 30s)
- Durante a calibração, o círculo exibe um estado visual distinto (ex: animação de "leitura/scan")
- A aplicação coleta N amostras do método ativo e computa a **média** como baseline
- Também computa o **desvio padrão** das amostras para sugerir um limiar inicial razoável (ex: `baseline_std * 3`)
- O limiar sugerido é exibido e pode ser ajustado pelo usuário antes de confirmar o início do monitoramento
- Cada método armazena seu próprio baseline e limiar; ao trocar de método, uma nova calibração é necessária

---

## Interface — Estrutura e Layout

Estética: **claro, minimalista, refinada**. Inspiração: instrumentação científica analógica com leitura digital. Fontes com caráter próprio (ex: display geométrico + mono para valores numéricos). Fundo off-white ou papel. Elementos com muito espaço em branco. Detalhes finos em vez de elementos pesados.

### Áreas da Interface

```
┌──────────────────────────────────────────────────────┐
│  SENTINEL                          [Modo: Livre ⟷ Alerta]  │
├──────────────────────────────────────────────────────┤
│                                                      │
│              ◯  CÍRCULO IDENTIFICADOR                │
│           (grande, centralizado, estado visual)      │
│                                                      │
│  Método: [Hash ▾]   Intervalo: [500ms]   Cal: [5s]  │
│  Limiar: ━━━●━━━━━ [valor numérico]                  │
│                                                      │
│  [INICIAR CALIBRAÇÃO]                                │
├──────────────────────────────────────────────────────┤
│  VARIÂNCIA — histórico da sessão                     │
│  ┌────────────────────────────────────────────────┐  │
│  │  gráfico de linha (Chart.js ou canvas nativo)  │  │
│  └────────────────────────────────────────────────┘  │
├──────────────────────────────────────────────────────┤
│  EVENTOS DETECTADOS                                  │
│  Timestamp          Método     Variância             │
│  ─────────────────────────────────────────────────   │
│  2024-03-31 14:32   Hash       0.847                 │
│  ...                                                 │
│                              [Exportar ZIP ↓]        │
└──────────────────────────────────────────────────────┘
```

### Círculo Identificador — Estados Visuais

| Estado | Aparência |
|---|---|
| Inativo / aguardando | Círculo fino, cor neutra (cinza claro), sem animação |
| Calibrando | Pulso suave ou rotação de arco, cor azul/índigo |
| Monitorando — estável | Círculo sólido, verde suave, respiração lenta |
| Detecção (Modo Livre) | Flash rápido laranja/âmbar → retorna ao verde |
| Alerta ativo (Modo Alerta) | Vermelho pulsante persistente, sem retorno automático |
| Câmera indisponível | Círculo pontilhado, cor cinza, ícone de aviso interno |

### Controles Configuráveis Visíveis

| Controle | Tipo | Faixa / Opções |
|---|---|---|
| Método de análise | Dropdown | Hash de Frame, Luminância, Espectro de Cores |
| Limiar de detecção | Slider + campo numérico | Dependente do método; sugerido pós-calibração |
| Intervalo de análise | Dropdown ou campo | 100ms, 250ms, 500ms, 1s, 2s, 5s |
| Duração da calibração | Campo numérico (segundos) | 3–30s |
| Modo de operação | Toggle (alternador) | Livre / Alerta |

---

## Câmera — Comportamento Técnico

- A aplicação solicita acesso à câmera via `getUserMedia` ao clicar em **Iniciar Calibração**
- O stream de vídeo é renderizado em um elemento `<video>` **oculto** (não visível ao usuário)
- A cada intervalo, um frame é desenhado em um `<canvas>` também **oculto**, e os pixels são lidos via `getImageData`
- Se o acesso à câmera for negado, exibir mensagem de erro clara com instruções para liberar permissão
- Múltiplas câmeras: se o dispositivo tiver mais de uma câmera, exibir seletor simples

---

## Tabela de Eventos

Colunas:
- **Timestamp**: formato ISO 8601 local (`YYYY-MM-DD HH:mm:ss.SSS`)
- **Método**: nome do método ativo no momento da detecção
- **Variância**: valor numérico com 4 casas decimais
- **Frame**: indicador booleano se o frame foi capturado (sim/não) — para referência no ZIP

A tabela exibe os eventos em ordem cronológica inversa (mais recentes no topo). Sem paginação obrigatória, mas com scroll interno se ultrapassar uma altura máxima.

---

## Exportação de Dados (ZIP)

Ao clicar em **Exportar ZIP**, a aplicação gera e dispara o download de um arquivo `.zip` contendo:

1. **`relatorio.csv`** — todos os eventos da tabela, com as colunas: `timestamp,metodo,variancia,frame_capturado`
2. **`frames/`** — pasta com as imagens capturadas no momento de cada detecção, no formato `YYYY-MM-DD_HHmmss-SSS_[metodo].jpg`
   - Resolução: frame completo da câmera no momento da detecção (não o frame reduzido de análise)
   - Frames são armazenados em memória (como base64 ou Blob) durante a sessão

> **Nota de implementação**: A geração do ZIP deve usar uma biblioteca JavaScript pura incluída inline ou via CDN (sugestão: JSZip). O arquivo é gerado inteiramente no browser, sem servidor.

---

## Estados e Fluxo da Aplicação

```
[Inativo]
    │
    ▼ usuário clica "Iniciar Calibração"
    │ solicita permissão de câmera
    │
[Calibrando] (N segundos)
    │ coleta amostras, computa baseline e limiar sugerido
    │
    ▼ calibração concluída
    │ exibe limiar sugerido, aguarda confirmação
    │
[Pronto para monitorar]
    │ usuário ajusta limiar se quiser e clica "Iniciar Monitoramento"
    │
[Monitorando — Estável]
    │
    ├─ variância ≤ limiar → continua monitorando
    │
    └─ variância > limiar
          │
          ├─ [Modo Livre] → registra evento + captura frame → volta a monitorar
          │
          └─ [Modo Alerta] → registra evento + captura frame → entra em [Alerta]
                                    │
                                    ▼ usuário clica "Confirmar / Retomar"
                                    └─ volta a [Monitorando — Estável]

A qualquer momento: usuário pode clicar "Parar" → volta a [Inativo]
```

---

## Considerações de Performance

- Para sessões longas, os frames capturados em memória podem consumir muita RAM. A aplicação deve:
  - Exibir contador de frames armazenados e estimativa de uso de memória
  - Oferecer opção de **limitar o número máximo de frames armazenados** (ex: últimos 100), com aviso quando o limite for atingido
- O intervalo mínimo recomendado é 250ms para não sobrecarregar a thread principal. Intervalos abaixo de 100ms devem exibir um aviso.
- A análise deve rodar de forma não-bloqueante; usar `requestAnimationFrame` ou `setTimeout` conforme adequado.

---

## Fora do Escopo (v1)

- Notificações push / Web Notifications API
- Múltiplos métodos ativos simultaneamente
- Persistência de sessões anteriores (localStorage ou IndexedDB)
- Envio de alertas remotos (email, webhook)
- Análise por IA / visão computacional externa
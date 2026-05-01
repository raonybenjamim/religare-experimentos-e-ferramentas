# Tutorial: Sentinel (Analisador de Imagens)

O **Sentinel** é uma ferramenta de monitoramento passivo e análise de variações de imagem e luz usando a câmera do seu dispositivo. Desenvolvido para sessões prolongadas de captura, ele funciona identificando anomalias no ambiente monitorado.

## Como começar

1. **Abra a aplicação:** Acesse a página do Sentinel. O navegador solicitará permissão para usar a câmera.
2. **Permita a câmera:** Clique em "Permitir". O vídeo não é exibido na tela por motivos de performance e abstração; o monitoramento é visualizado apenas num "círculo identificador" e num gráfico.
3. **Calibragem Inicial:** Clique em **Iniciar Calibração**. A ferramenta passará alguns segundos avaliando o ambiente (linha de base/baseline) para não registrar as condições normais como anomalias.

## Configurações

Antes ou durante o uso, você pode ajustar as opções:

- **Método de Análise:**
  - *Hash de Frame:* Ideal para identificar mudanças bruscas nos pixels (como alguém ou algo passando rápido).
  - *Luminância:* Reage mais quando a iluminação geral do ambiente muda de repente.
  - *Espectro de Cores:* Foca em variações cromáticas (mudança brusca nas cores dominantes do cenário).
- **Intervalo:** Conforme a potência do seu computador, determine o intervalo entra cada checagem (100ms, 500ms, 1s, etc). Valores menores exigem mais processamento, experimente usar 500ms caso trave.
- **Limiar de Detecção:** Ajuste manual do nível de sensibilidade. Em ambientes instáveis, deixe o limiar maior. Em ambientes perfeitamente controlados, deixe o limiar menor.
- **Modos de Operação:**
  - *Livre:* Detecta e continua investigando e salvando os eventos continuamente.
  - *Alerta:* Pausa a checagem no primeiro desvio que cruzar o Limiar para análise manual; exige botão para confirmar e continuar.

## Exportando Dados

Quando finalizar, o histórico de eventos aparecerá numa tabela abaixo. Clicando em **Exportar ZIP** será baixado um arquivo contendo um sumário `.csv`, além das fotografias salvas apenas dos exatos momentos em que os picos estatísticos ("eventos") ocorreram.

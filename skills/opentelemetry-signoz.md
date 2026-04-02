---
description: Como instrumentar a aplicação para SigNoz usando OpenTelemetry (inclusive Ollama)
---

# OpenTelemetry e SigNoz no Contexto do Projeto

Este projeto envia métricas, logs e traces OTLP para um backend SigNoz local. 
As configurações essenciais estão no arquivo `.env` da raiz.

## 1. Variáveis de Ambiente (.env)

O `.env` atualmente configura o envio via porta **4318 usando HTTP/protobuf** (sobrescrevendo o 4317/grpc). Antes de inicializar serviços e agentes, garanta a carga das seguintes variáveis:

```env
# OpenTelemetry / SigNoz
OTEL_RESOURCE_ATTRIBUTES=service.name=agente-analise-livros
OTEL_TRACES_EXPORTER=otlp
OTEL_METRICS_EXPORTER=otlp
OTEL_LOGS_EXPORTER=otlp
OTEL_PYTHON_LOG_CORRELATION=true
OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION_ENABLED=true

# Endpoint ativo (OTLP sobre HTTP)
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
```

## 2. Instrumentando Requisições ao Ollama

O projeto **está utilizando a biblioteca oficial do Ollama** (`ollama`) para comunicação com o LLM. Para capturar atributos de Inteligência Artificial (Tokens, Modelo, Latência) no SigNoz, usamos a biblioteca de telemetria correspondente.

### Instalação de Dependências
Certifique-se de ter os pacotes corretos:
```bash
pip install opentelemetry-distro opentelemetry-exporter-otlp opentelemetry-instrumentation-ollama
```

### Inicialização no Código (Programática)
Diferente da instrumentação por CLI, o Ollama pode ser explicitamente instrumentado no próprio arquivo de acesso (por exemplo, `src/analisar_livro/ollama_client.py`) de forma declarativa e no início do módulo:

```python
from opentelemetry.instrumentation.ollama import OllamaInstrumentor
OllamaInstrumentor().instrument()

# Importações subsequentes do Ollama Client já estarão enviando spans ao OpenTelemetry
import ollama
```

### Inicialização por CLI
Alternativamente (ou de forma complementar para outras bibliotecas, como SQLite/requests padrão), pode-se invocar sua app usando o envelopamento de terminal:

```bash
opentelemetry-instrument python main_agente.py
```

## 3. Integração com Logging

Para enriquecer seus traces de OTLP com eventos de falhas:
```python
import logging

# Recomenda-se setar em nível apropriado. O OTel capturará os logs (por causa de OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION=true)
logging.getLogger().setLevel(logging.INFO)
```
Assim, se uma API falhar com erros 5xx (transientes) que seu código tenta repetir, todos esses avisos ficam amarrados ao fluxo do Span no painel do SigNoz.

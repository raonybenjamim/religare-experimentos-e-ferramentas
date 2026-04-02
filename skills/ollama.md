---
description: Como utilizar a Lib Oficial do Ollama para requisições no projeto
---

# Usando a Lib Oficial do Ollama

O projeto se comunica com o serviço de Language Model empregando o pacote oficial **Python Ollama SDK** (`pip install ollama`). Abaixo estão as práticas de engenharia de como utilizar eficientemente essa API, garantindo robustez a falhas (transientes e timeouts), como implementado em `src/analisar_livro/ollama_client.py`.

## 1. Inicializando o Cliente

Mesmo que exista o `ollama.chat` global, a recomendação para maior controle (lidando com múltiplos hosts ou controle fino de timeouts da requisição) é o uso da classe nativa local `ollama.Client`:

```python
import ollama

# Cria um cliente dedicado com host e timeout específico (em segundos)
client = ollama.Client(host="http://localhost:11434", timeout=300.0)
```

## 2. Estrutura de Retries e Opções Geração (Chat)

Chamadas na rede podem falhar. A API expõe `ollama.ResponseError` para nos auxiliar a interceptar códigos HTTP específicos (429, 50x) e aplicar _backoff_:

```python
import time
import logging
from ollama import ResponseError

log = logging.getLogger(__name__)

# Definimos as propriedades do modelo passadas via Options dict:
options = ollama.Options(
    temperature=0.0,
    top_p=0.9,
    num_ctx=8192
)

# Criando as mensagens (List de Messages ou Dict native do ollama)
messages = [
    ollama.Message(role="system", content="Você é útil."),
    ollama.Message(role="user", content="Realizar tarefas.")
]

max_retries = 3
for attempt in range(1, max_retries + 2):
    try:
        response = client.chat(
            model="llama3",
            messages=messages,
            stream=False,
            options=options
        )
        print("Finalizado:", response.message.content)
        break

    except ResponseError as exc:
        # Pega apenas erros HTTP elegíveis para nova tentativa (ex 429, 500, 503)
        if exc.status_code not in {429, 500, 502, 503, 504}:
            raise
        log.warning(f"Response Error transitório: {exc.status_code}")
        
    except Exception as exc:
        # Timeouts ou queda abrupta de conexão
        log.warning("Falha ao comunicar com Ollama (Timeout/Conn)")
    
    if attempt > max_retries:
        raise Exception("Esgotadas tentativas ao Ollama")
        
    # Exponention backoff
    time.sleep(2.0 ** attempt)
```

## 3. Retorno em JSON e Dicas de Prompts

O Ollama nem sempre respeita o formato JSON de forma implícita com base no comando de sistema nativo. Assim:
1. Recomenda-se adicionar um aviso claro de formatação no prompt ("retorne apenas um objeto JSON...").
2. Caso espere mapear num parser, passe a _string_ final por um `json.loads(response.message.content)`. Se ocorrer exceção aí, sua aplicação captura um _ParseError_.

**Nota**: O uso da classe `OllamaClient` já implementada no projeto abstrairá completamente loops, instrumentações e parsing para retornos fáceis em formato nativo da aplicação. Você deve preferir utilizá-la em vez de invocar `ollama.Client` diretamente na regra de negócios final.

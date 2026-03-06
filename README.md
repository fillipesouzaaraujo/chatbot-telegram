# Chatbot de Clima com Telegram + n8n

Este projeto implementa um **chatbot de consulta de clima** utilizando **n8n**, **Telegram Bot API**, **OpenWeather API** e um modelo de IA para interpretar mensagens do usuário.

O usuário pode enviar uma mensagem contendo uma cidade e o bot retornará a temperatura atual.

Exemplo:

```
Salvador, BA
```

Resposta:

```
A temperatura em Salvador é 28ºC
```

---

# Arquitetura do Workflow

Fluxo do processamento:

1. O usuário envia uma mensagem para o bot no Telegram
2. O **Telegram Trigger** inicia o workflow
3. Um modelo de IA interpreta a mensagem do usuário
4. A IA extrai a cidade e o estado no formato `"Cidade, UF"`
5. Se a cidade for identificada:

   * o workflow consulta a API do OpenWeather
   * formata a resposta
   * envia a temperatura para o usuário
6. Se a cidade **não for identificada**:

   * o bot pede que o usuário informe a cidade corretamente

---

# Pré-requisitos

Antes de executar o projeto é necessário possuir:

* n8n instalado
* Bot criado no Telegram
* Chave da API do OpenWeather
* Credencial da API da OpenAI configurada no n8n

Links úteis:

* https://n8n.io
* https://openweathermap.org/api
* https://core.telegram.org/bots

---

# Variáveis de ambiente

O workflow utiliza variáveis de ambiente para evitar expor credenciais.

Crie a seguinte variável:

```
OPENWEATHER_API_KEY
```

---

# Configuração das variáveis

## Linux / Mac

```
export OPENWEATHER_API_KEY=SUA_CHAVE
```

Depois reinicie o n8n.

---

## Docker

```
docker run -it --rm \
-p 5678:5678 \
-e OPENWEATHER_API_KEY=SUA_CHAVE \
n8nio/n8n
```

---

## Docker Compose

docker-compose.yml

```
version: "3"

services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - OPENWEATHER_API_KEY=${OPENWEATHER_API_KEY}
```

Arquivo `.env`

```
OPENWEATHER_API_KEY=SUA_CHAVE
```

---

# Importando o Workflow no n8n

1. Acesse o n8n
2. Vá em **Workflows**
3. Clique em **Import**
4. Selecione o arquivo:

```
workflow-chatbot-telegram.json
```

5. Salve o workflow

---

# Configuração das Credenciais

Após importar o workflow será necessário configurar duas credenciais.

## Telegram

1. Crie um bot no Telegram usando **BotFather**
2. Copie o **BOT TOKEN**
3. No n8n configure a credencial **Telegram API**
4. Insira o token do bot

---

## OpenAI

O workflow utiliza um modelo da OpenAI para interpretar a mensagem do usuário.

Configure a credencial **OpenAI API** no n8n com sua chave de API.

---

# Como executar o chatbot

1. Ative o workflow no n8n
2. Abra o Telegram
3. Envie uma mensagem para o bot

Exemplos válidos:

```
Salvador, BA
Curitiba, PR
Qual a temperatura em Recife?
Quero saber o clima em São Paulo
```

---

# Exemplos de resposta

Entrada do usuário:

```
Salvador, BA
```

Resposta do bot:

```
A temperatura em Salvador é 28ºC
```

---

# Exemplo de erro

Se a cidade não puder ser identificada:

Entrada:

```
qual o clima ai?
```

Resposta:

```
Envie a cidade e o estado para que eu possa consultar o clima.
```

---

# Estrutura do projeto

```
project
│
├── workflow-chatbot-telegram.json
└── README.md
```

---

# Observações técnicas

* A interpretação da mensagem é feita por um modelo de IA
* O modelo extrai a cidade no formato `"Cidade, UF"`
* A consulta de clima é realizada via **OpenWeather API**
* A chave da API é obtida via variável de ambiente:

```
{{$env.OPENWEATHER_API_KEY}}
```

Isso evita exposição de credenciais dentro do workflow.

---
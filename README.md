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
3. Um modelo de IA (Google Gemini) interpreta a mensagem do usuário
4. A IA extrai a cidade e o estado no formato `"Cidade, UF"`
5. Se a cidade for identificada:

   * o workflow consulta a API do OpenWeather
   * formata a resposta (com ícone conforme a faixa de temperatura)
   * envia a temperatura para o usuário
   * **grava a cidade pesquisada no PostgreSQL** (tabela `cidades_mais_pesquisadas`); se a tabela não existir, o workflow cria automaticamente
6. Se a cidade **não for identificada**:

   * o bot pede que o usuário informe a cidade corretamente

---

# Pré-requisitos

Antes de executar o projeto é necessário possuir:

* n8n instalado
* Bot criado no Telegram
* Chave da API do OpenWeather
* Credencial da API do Google Gemini configurada no n8n
* Banco **PostgreSQL** acessível (para salvar as cidades mais pesquisadas)

Links úteis:

* https://n8n.io
* https://openweathermap.org/api
* https://core.telegram.org/bots
* https://aistudio.google.com/apikey (API Key do Google Gemini)
* https://www.postgresql.org/docs/ (documentação PostgreSQL)

---

# Variáveis e Credenciais

Esta seção descreve **todas** as variáveis e credenciais necessárias para o projeto. Configure-as antes de importar e ativar o workflow.

## Resumo

| Item | Onde configurar | Obrigatório |
|------|-----------------|-------------|
| **OPENWEATHER_API_KEY** | Variável de ambiente | Sim |
| **Token do Bot Telegram** | Credencial no n8n | Sim |
| **Google Gemini (PaLM) API** | Credencial no n8n | Sim |
| **PostgreSQL** | Credencial no n8n | Sim |

---

## Variáveis de ambiente

O workflow usa variável de ambiente para a chave da OpenWeather, evitando expor a chave dentro do JSON do workflow.

| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `OPENWEATHER_API_KEY` | Chave da API OpenWeather Map | `a1b2c3d4e5f6...` |

**Como obter:** crie uma conta em [OpenWeather Map](https://openweathermap.org/api) e gere uma API key em "API keys".

---

## Credenciais no n8n

São configuradas na interface do n8n (**Settings** → **Credentials** ou ao editar um nó que exija credencial).

### Token do Bot Telegram

O **token do bot** é a credencial que permite ao n8n receber e enviar mensagens pelo Telegram.

1. **Obter o token**
   * Abra o Telegram e converse com [@BotFather](https://t.me/BotFather).
   * Envie `/newbot` e siga as instruções (nome e username do bot).
   * Ao final, o BotFather envia uma mensagem com o **token** (algo como `123456789:ABCdefGHIjkL MNOpqrsTUVwxyz`).

2. **Configurar no n8n**
   * No n8n: **Settings** (ou ícone de engrenagem) → **Credentials** → **Add Credential**.
   * Procure por **Telegram API** e selecione.
   * No campo **Access Token**, cole o token recebido do BotFather.
   * Dê um nome à credencial (ex.: `Telegram account`) e salve.

3. **Associar ao workflow**
   * Após importar o workflow, abra-o e clique no nó **Telegram Trigger** (e em qualquer nó **Telegram** que enviar mensagem).
   * Em **Credentials**, selecione a credencial **Telegram API** que você criou (ou crie uma nova na hora).
   * Salve o workflow.

**Exemplo do valor do token (apenas formato):**

```
123456789:AAHxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Nunca compartilhe esse token nem o coloque em repositórios públicos.

### Google Gemini (PaLM) API

O workflow usa o **Google Gemini** (modelo `gemini-2.5-flash`) para interpretar a mensagem do usuário e extrair cidade/estado.

1. **Obter a API Key**
   * Acesse [Google AI Studio](https://aistudio.google.com/apikey) (ou [Google Cloud Console](https://console.cloud.google.com/) para APIs Gemini/Vertex).
   * Crie ou copie uma **API Key** para usar com a API Gemini.

2. **Configurar no n8n**
   * No n8n: **Credentials** → **Add Credential** → procure por **Google Gemini** ou **Google PaLM API**.
   * Selecione a credencial **Google Gemini(PaLM) Api** (ou equivalente).
   * No campo **API Key**, cole a chave obtida no Google AI Studio.
   * Dê um nome à credencial (ex.: `Google Gemini(PaLM) Api account`) e salve.

3. **Associar ao workflow**
   * Abra o workflow e clique no nó **Google Gemini**.
   * Em **Credentials**, selecione a credencial que você criou.
   * Salve o workflow.

### PostgreSQL

O workflow usa **PostgreSQL** para registrar as cidades mais pesquisadas na tabela `cidades_mais_pesquisadas`. A tabela é criada automaticamente na primeira execução bem-sucedida, se ainda não existir.

1. **Ter um banco PostgreSQL**
   * Instale o PostgreSQL localmente, use um serviço em nuvem (ex.: Supabase, Neon, AWS RDS) ou rode via Docker.
   * Anote: **host**, **porta** (padrão `5432`), **nome do banco**, **usuário** e **senha**.

2. **Criar a credencial no n8n**
   * No n8n: **Credentials** → **Add Credential** → procure por **Postgres**.
   * Selecione **Postgres** (nó de banco de dados).
   * Preencha:
     * **Host:** endereço do servidor (ex.: `localhost`, `db.seudominio.com` ou o host do seu provedor).
     * **Database:** nome do banco (ex.: `n8n`, `chatbot`).
     * **User:** usuário do banco.
     * **Password:** senha do usuário.
     * **Port:** porta (geralmente `5432`).
     * **SSL:** ative se o seu provedor exigir conexão SSL (ex.: serviços em nuvem).
   * Dê um nome à credencial (ex.: `Postgres account`) e salve.

3. **Associar ao workflow**
   * Após importar o workflow, abra-o e nos nós que usam Postgres selecione essa credencial:
     * **Save Cidades - Verifica se a tabela existe**
     * **Cria tabela: cidades_mais_pesquisadas**
     * **Insert rows in a table**
   * Salve o workflow.

**Exemplo de configuração (valores fictícios):**

| Campo    | Exemplo        |
|----------|----------------|
| Host     | `localhost`    |
| Database | `chatbot`     |
| User     | `n8n_user`    |
| Password | `sua_senha`   |
| Port     | `5432`        |

**Estrutura da tabela** (criada automaticamente pelo workflow):

```sql
CREATE TABLE cidades_mais_pesquisadas (
    id SERIAL PRIMARY KEY,
    cidade VARCHAR(120) NOT NULL,
    uf CHAR(2) NOT NULL,
    data_pesquisa TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Exemplos práticos de configuração

Abaixo, exemplos de como definir a variável de ambiente (e, quando fizer sentido, arquivo `.env`) em cada tipo de execução. O **token do Telegram** continua sendo configurado **sempre nas credenciais do n8n**, conforme a subseção anterior.

### Linux / Mac (terminal)

Defina a variável e inicie o n8n no mesmo shell (ou coloque os `export` no seu `~/.bashrc` ou `~/.zshrc` e reinicie o terminal antes de subir o n8n):

```bash
export OPENWEATHER_API_KEY=sua_chave_openweather_aqui
# Reinicie o n8n após definir a variável (ou inicie o n8n neste mesmo terminal)
n8n start
```

Se o n8n roda como serviço (systemd), crie um override ou arquivo de ambiente, por exemplo `/etc/default/n8n`:

```bash
# /etc/default/n8n (ajuste o caminho conforme sua instalação)
OPENWEATHER_API_KEY=sua_chave_openweather_aqui
```

Depois: `sudo systemctl restart n8n`.

---

### Docker (run)

Passe a variável com `-e`:

```bash
docker run -it --rm \
  -p 5678:5678 \
  -e OPENWEATHER_API_KEY=sua_chave_openweather_aqui \
  n8nio/n8n
```

---

### Docker Compose

**Arquivo `docker-compose.yml`:**

```yaml
version: "3"

services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - OPENWEATHER_API_KEY=${OPENWEATHER_API_KEY}
```

**Arquivo `.env`** (na mesma pasta do `docker-compose.yml`):

```bash
# Variáveis para o chatbot de clima
OPENWEATHER_API_KEY=sua_chave_openweather_aqui
```

Subir o stack:

```bash
docker compose up -d
# ou: docker-compose up -d
```

Lembre-se: após o primeiro acesso ao n8n, configure as credenciais **Telegram API**, **Google Gemini (PaLM) API** e **PostgreSQL** na interface, conforme a seção [Credenciais no n8n](#credenciais-no-n8n).

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

# Configuração das Credenciais (após importar)

Após importar o workflow, associe as credenciais aos nós. O passo a passo completo (obter token, criar credencial no n8n, associar ao workflow) está na seção [Variáveis e Credenciais](#variáveis-e-credenciais).

* **Telegram:** use a credencial **Telegram API** com o token do BotFather nos nós "Telegram Trigger" e "Telegram" (envio de mensagens).
* **Google Gemini:** use a credencial **Google Gemini(PaLM) Api** no nó **Google Gemini**.
* **PostgreSQL:** use a credencial **Postgres** nos nós **Save Cidades - Verifica se a tabela existe**, **Cria tabela: cidades_mais_pesquisadas** e **Insert rows in a table**.

Se ainda não tiver criado as credenciais, siga a seção [Credenciais no n8n](#credenciais-no-n8n).

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

Resposta do bot (com ícone conforme a temperatura):

```
🌞 A temperatura em Salvador é de 28ºC
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

* A interpretação da mensagem é feita pelo **Google Gemini** (modelo `gemini-2.5-flash`)
* O modelo extrai a cidade no formato `"Cidade, UF"`
* A consulta de clima é realizada via **OpenWeather API**
* Após cada consulta bem-sucedida, a cidade é gravada no **PostgreSQL** (tabela `cidades_mais_pesquisadas`), permitindo análises de cidades mais pesquisadas; a tabela é criada automaticamente pelo workflow na primeira vez
* A mensagem de sucesso inclui um ícone conforme a faixa de temperatura (ex.: 🌞, 🧥, 🌤️)
* A chave da OpenWeather é lida no workflow via variável de ambiente (`OPENWEATHER_API_KEY`), referenciada no nó como:

```
{{ $env.OPENWEATHER_API_KEY }}
```

Isso evita expor a chave dentro do JSON do workflow. O **token do Telegram**, a **API Key do Google Gemini** e as **credenciais do PostgreSQL** ficam apenas nas credenciais do n8n (não são variáveis de ambiente neste projeto).

---
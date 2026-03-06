# Lista de testes – Chatbot Clima (Telegram + OpenWeatherMap)

Testes compatíveis com a [API Current Weather 2.5](https://openweathermap.org/current) do OpenWeatherMap.  
O parâmetro `q` aceita: `nome da cidade` ou `cidade,código do país` (ex.: `London,UK`). Códigos de país em ISO 3166 (ex.: BR, US, UK, JP).

---

## 1. Casos de sucesso (cidade reconhecida pela IA e pela API)

| # | Mensagem do usuário | Formato esperado da IA (`message`) | Resultado esperado |
|---|---------------------|------------------------------------|--------------------|
| 1.1 | `London, UK` | London, UK | Resposta com temperatura em Londres |
| 1.2 | `Qual a temperatura em Londres?` | London, UK | Resposta com temperatura em Londres |
| 1.3 | `New York, US` | New York, US | Resposta com temperatura em New York |
| 1.4 | `Tokyo, JP` | Tokyo, JP | Resposta com temperatura em Tóquio |
| 1.5 | `Paris, FR` | Paris, FR | Resposta com temperatura em Paris |
| 1.6 | `Salvador, BA` | Salvador, BR ou Salvador, BA | Resposta com temperatura em Salvador |
| 1.7 | `São Paulo, SP` | São Paulo, SP ou São Paulo, BR | Resposta com temperatura em São Paulo |
| 1.8 | `Curitiba, PR` | Curitiba, PR ou Curitiba, BR | Resposta com temperatura em Curitiba |
| 1.9 | `Recife, PE` | Recife, PE ou Recife, BR | Resposta com temperatura em Recife |
| 1.10 | `Rio de Janeiro, RJ` | Rio de Janeiro, RJ ou Rio de Janeiro, BR | Resposta com temperatura no Rio |
| 1.11 | `Sydney, AU` | Sydney, AU | Resposta com temperatura em Sydney |
| 1.12 | `Lisbon, PT` | Lisbon, PT | Resposta com temperatura em Lisboa |

---

## 2. Casos de falha de extração (IA não identifica cidade)

| # | Mensagem do usuário | Resultado esperado |
|---|---------------------|--------------------|
| 2.1 | `qual o clima ai?` | Bot envia recomendação pedindo cidade e estado |
| 2.2 | `oi` | Bot envia recomendação |
| 2.3 | `me fala a temperatura` | Bot envia recomendação |
| 2.4 | `como está o tempo?` | Bot envia recomendação |
| 2.5 | `(mensagem vazia ou só espaços)` | Bot envia recomendação |

---

## 3. Casos de cidade não encontrada (IA ok, API retorna erro)

Quando a IA extrai um `message` válido mas a OpenWeatherMap não encontra a cidade (404 ou 400), o fluxo deve enviar: *"Infelizmente a API não identificou a sua cidade, tente outra, por exemplo London, UK"*.

| # | Mensagem do usuário | Formato possível da IA | Resultado esperado |
|---|---------------------|-------------------------|--------------------|
| 3.1 | `Cidade Inexistente, XX` | Cidade Inexistente, XX | Mensagem de cidade não identificada pela API |
| 3.2 | `Abcdefgh, ZZ` | Abcdefgh, ZZ | Mensagem de cidade não identificada pela API |

---

## 4. Casos de borda e formato

| # | Mensagem do usuário | Objetivo do teste |
|---|---------------------|--------------------|
| 4.1 | `London,UK` (sem espaço) | Verificar se a API aceita formato sem espaço após a vírgula |
| 4.2 | `london, uk` (minúsculas) | Verificar se a API normaliza nome da cidade |
| 4.3 | `  São Paulo , SP  ` (espaços extras) | Verificar se a IA normaliza e a API aceita |
| 4.4 | `Cidade com nome composto, SP` (ex.: São José dos Campos, SP) | Verificar cidades com mais de uma palavra |

---

## 5. Resumo de critérios de aceite

- **Sucesso:** Bot responde no formato *"A temperatura em [Nome da Cidade] é [X]ºC"*.
- **Falha de extração:** Bot responde com texto de recomendação (atributo `recomendation` da IA), pedindo cidade/estado.
- **Cidade não encontrada pela API:** Bot responde com a mensagem fixa sugerindo tentar outra cidade (ex.: London, UK).
- **Formato da API:** Usar sempre `units=metric` e garantir que `OPENWEATHER_API_KEY` está definida.

---

## 6. Dados de referência (OpenWeatherMap)

- **Endpoint:** `GET https://api.openweathermap.org/data/2.5/weather`
- **Parâmetros:** `q={cidade}`, `APPID={chave}`, `units=metric`
- **Exemplo de `q` que funcionam:** `London,UK`, `New York,US`, `São Paulo,BR`, `Tokyo,JP`, `Salvador,BR`

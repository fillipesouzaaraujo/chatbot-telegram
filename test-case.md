# Casos de teste – Chatbot Clima (Telegram)

Use esta lista para validar o comportamento do bot.

---

## 1. Sucesso (bot retorna temperatura)

| Mensagem do usuário | O que deve acontecer |
|---------------------|----------------------|
| `Salvador, BA` | Bot responde com a temperatura em Salvador (ex.: "🌞 A temperatura em Salvador é de 28ºC") |
| `São Paulo, SP` | Bot responde com a temperatura em São Paulo |
| `Curitiba, PR` | Bot responde com a temperatura em Curitiba |
| `Qual a temperatura em Recife?` | Bot responde com a temperatura em Recife |

---

## 2. Falha (usuário não informa cidade/estado)

| Mensagem do usuário | O que deve acontecer |
|---------------------|----------------------|
| `oi` | Bot pede para enviar cidade e estado no formato Cidade,UF |
| `qual o clima ai?` | Bot pede para enviar cidade e estado |
| `me fala a temperatura` | Bot pede para enviar cidade e estado |
| Mensagem vazia ou só espaços | Bot pede para enviar cidade e estado |

---

## 3. Cidade não encontrada (formato certo, API não acha a cidade)

| Mensagem do usuário | O que deve acontecer |
|---------------------|----------------------|
| `Cidade Inexistente, XX` | Bot envia: "❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP)." |

---

## Resumo

- **Sucesso:** resposta com ícone + temperatura (ex.: "🌞 A temperatura em [Cidade] é de [X]ºC").
- **Sem cidade/estado:** bot envia a recomendação pedindo formato Cidade,UF.
- **Cidade inexistente na API:** bot envia mensagem de cidade não encontrada.

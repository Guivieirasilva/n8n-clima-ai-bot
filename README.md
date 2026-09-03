# n8n-clima-ai-bot

## Clima AI Bot — Chatbot de Temperatura com Telegram, n8n, OpenWeather e Gemini

Projeto desenvolvido para a **Faculdade de Tecnologia da Rocketseat (FTR)**.

O **Clima AI Bot** é um chatbot integrado ao Telegram que permite consultar informações meteorológicas de cidades brasileiras. O usuário envia o nome da cidade, o workflow processa e normaliza a entrada, consulta a API da OpenWeather e retorna uma resposta clara e amigável.

Como recurso adicional, o projeto utiliza o **Google Gemini** para transformar os dados meteorológicos em uma resposta mais natural e lúdica. Caso a IA esteja indisponível, o workflow possui um fallback determinístico para garantir que o chatbot continue funcionando.

---

# Objetivo

Desenvolver uma automação utilizando **n8n** capaz de:

* Receber mensagens pelo Telegram;
* Validar a entrada do usuário;
* Normalizar cidade, estado e país;
* Criar a variável `queue`;
* Consultar a API OpenWeather;
* Validar a resposta recebida;
* Extrair informações meteorológicas;
* Utilizar IA para melhorar a apresentação da resposta;
* Possuir fallback caso a IA esteja indisponível;
* Enviar a resposta final pelo Telegram;
* Tratar entradas inválidas e cidades inexistentes.

O projeto prioriza:

* Segurança;
* Legibilidade;
* Manutenibilidade;
* Tratamento de erros;
* Uso de variáveis de ambiente;
* Separação de responsabilidades entre os nós do workflow.

---

# Tecnologias Utilizadas

* **n8n** — Automação e orquestração do workflow;
* **Telegram Bot API** — Comunicação com o usuário;
* **OpenWeather API** — Consulta das informações meteorológicas;
* **Google Gemini** — Formatação inteligente da resposta;
* **Docker** — Execução da infraestrutura;
* **PostgreSQL** — Banco de dados do n8n;
* **Redis** — Suporte ao modo Queue do n8n;
* **ngrok** — Exposição temporária do webhook HTTPS.

---

# Arquitetura do Workflow

O fluxo principal da aplicação segue a seguinte estrutura:

```text
Telegram Trigger
       ↓
Capturar Mensagem
       ↓
Validar Entrada
       ↓
Normalizar Entrada
       ↓
Consultar OpenWeather
       ↓
Validar Resposta da API
       ↓
Preparar Dados para IA
       ↓
Gerar Resposta Lúdica (Gemini)
       ↓
Telegram - Enviar Sucesso
```

Em caso de erro:

```text
Entrada inválida
       ↓
Formatar Erro de Entrada
       ↓
Telegram - Enviar Erro
```

Ou:

```text
Resposta inválida da OpenWeather
       ↓
Formatar Erro da API
       ↓
Telegram - Enviar Erro
```

Caso o Gemini esteja indisponível:

```text
Erro no Gemini
       ↓
Fallback - Formatar Resposta Determinística
       ↓
Telegram - Enviar Sucesso
```

---

# Como Funciona

## 1. Telegram Trigger

O workflow inicia quando o bot recebe uma mensagem no Telegram.

O nó utilizado é:

```text
Telegram Trigger
```

---

## 2. Captura da Mensagem

A mensagem enviada pelo usuário é capturada e armazenada.

São extraídas informações importantes, como:

```text
chatId
rawMessage
```

O `chatId` é utilizado posteriormente para enviar a resposta ao usuário.

---

## 3. Validação da Entrada

Antes de consultar a API, o workflow verifica se a mensagem recebida não está vazia.

Caso a entrada seja inválida, o usuário recebe uma mensagem explicando o formato esperado.

Exemplo:

```text
❌ Cidade não encontrada ou resposta inválida.

Use o formato:
Cidade,UF,BR

Exemplo:
São Paulo,SP,BR
```

---

## 4. Normalização da Entrada

A mensagem é processada para remover espaços desnecessários e preparar os dados para a consulta.

O workflow trabalha com os campos:

```text
city
uf
country
queue
```

A variável `queue` é utilizada na consulta à OpenWeather.

Exemplo:

```text
São Paulo, SP, BR
```

Após a normalização:

```text
São Paulo,SP,BR
```

---

# Formato da Entrada

O formato recomendado é:

```text
Cidade,UF,BR
```

Exemplos:

```text
São Paulo,SP,BR
Belo Horizonte,MG,BR
Curitiba,PR,BR
Rio de Janeiro,RJ,BR
```

Também é possível enviar uma entrada com espaços extras, pois o workflow realiza o tratamento necessário.

Exemplo:

```text
  São Paulo, SP, BR
```

---

# Consulta à OpenWeather

A consulta é realizada utilizando um nó:

```text
HTTP Request
```

Endpoint:

```text
https://api.openweathermap.org/data/2.5/weather
```

Os parâmetros utilizados são:

| Parâmetro | Descrição                             |
| --------- | ------------------------------------- |
| `q`       | Cidade armazenada na variável `queue` |
| `units`   | `metric`                              |
| `lang`    | `pt_br`                               |
| `appid`   | API Key da OpenWeather                |

Com:

```text
units=metric
```

A temperatura é retornada em:

```text
Graus Celsius (°C)
```

---

# Variáveis de Ambiente

Nenhuma chave secreta deve ser armazenada diretamente no código ou no workflow.

As variáveis necessárias são:

```env
OPENWEATHER_API_KEY=sua_chave_aqui
```

Caso a integração com o Gemini seja utilizada através de configuração por ambiente:

```env
GOOGLE_GEMINI_API_KEY=sua_chave_aqui
```

O token do Telegram também deve ser tratado como segredo:

```env
TELEGRAM_BOT_TOKEN=seu_token_aqui
```

> Nunca envie ou publique valores reais dessas variáveis no GitHub.

---

# Segurança

Este projeto segue as seguintes práticas de segurança:

* API Keys não devem ser versionadas;
* Tokens do Telegram não devem ser publicados;
* Arquivos `.env` devem estar no `.gitignore`;
* Credenciais devem ser configuradas pelo n8n;
* Variáveis de ambiente devem ser utilizadas para segredos;
* Respostas externas devem ser validadas antes do uso;
* Mensagens técnicas internas não devem ser expostas ao usuário final.

Exemplo de `.gitignore`:

```gitignore
.env
n8n-data/
postgres-data/
redis-data/
ngrok.yml
```

---

# Google Gemini

O Google Gemini é utilizado como uma camada opcional de apresentação.

A IA **não é responsável pela lógica crítica do sistema**.

Sua responsabilidade é apenas transformar os dados já obtidos da OpenWeather em uma resposta:

* Curta;
* Natural;
* Amigável;
* Em português brasileiro;
* Sem inventar informações;
* Adequada para envio pelo Telegram.

Os dados enviados para a IA incluem:

```text
Cidade
Temperatura
Sensação térmica
Condição climática
Umidade
```

A configuração utiliza baixa criatividade:

```text
temperature: 0.2
```

Isso reduz a probabilidade de respostas inconsistentes ou invenção de informações.

---

# Fallback Determinístico

O projeto possui uma alternativa caso o Gemini esteja indisponível.

O fallback gera uma resposta sem depender de IA.

Exemplo:

```text
🌤️ A temperatura em São Paulo é de 25°C.
```

Essa abordagem garante maior confiabilidade, pois o funcionamento principal do chatbot não depende exclusivamente de um serviço externo de IA.

---

# Tratamento de Erros

O workflow possui caminhos específicos para situações previsíveis.

## Entrada inválida

Exemplo:

```text
Mensagem vazia
```

Resposta:

```text
❌ Cidade não encontrada ou resposta inválida.

Use o formato:
Cidade,UF,BR

Exemplo:
São Paulo,SP,BR
```

---

## Cidade inexistente

Exemplo:

```text
CidadeQueNaoExiste,XX,BR
```

O workflow valida a resposta da OpenWeather antes de tentar acessar informações meteorológicas.

Isso evita erros causados por propriedades inexistentes.

---

## Falha da IA

Caso o Gemini não consiga responder:

```text
Gemini indisponível
       ↓
Fallback determinístico
       ↓
Resposta enviada ao Telegram
```

---

# Pré-requisitos

Para executar o projeto, você precisará de:

* Docker;
* Docker Compose;
* Conta no Telegram;
* Bot criado no Telegram;
* Conta na OpenWeather;
* API Key da OpenWeather;
* n8n;
* Credenciais do Google Gemini, caso deseje utilizar a IA;
* ngrok ou outra solução HTTPS para receber webhooks externamente.

---

# Criando o Bot no Telegram

## 1. Abra o Telegram

Procure por:

```text
@BotFather
```

## 2. Crie um novo bot

Envie:

```text
/newbot
```

## 3. Configure o bot

Defina:

* Nome do bot;
* Username do bot.

O username deve terminar com:

```text
bot
```

Exemplo:

```text
Clima AI Bot
clima_ai_bot
```

## 4. Guarde o token com segurança

O BotFather fornecerá um token.

Esse token é uma credencial secreta.

**Nunca publique esse token no GitHub.**

---

# Configurando a OpenWeather

## 1. Crie uma conta

Crie uma conta na OpenWeather.

## 2. Gere uma API Key

Após criar a conta, obtenha uma chave de API.

## 3. Configure a variável de ambiente

```env
OPENWEATHER_API_KEY=sua_chave_aqui
```

O workflow utiliza:

```javascript
{{ $env.OPENWEATHER_API_KEY }}
```

Dessa forma, a chave não precisa ficar armazenada diretamente no workflow.

---

# Executando com Docker

O projeto pode ser executado utilizando Docker Compose.

A infraestrutura pode incluir:

```text
n8n
PostgreSQL
Redis
ngrok
```

## Estrutura esperada

```text
n8n-clima-ai-bot/
│
├── workflow-telegram-chatbot.json
├── docker-compose.yml
├── README.md
├── .env.example
└── .gitignore
```

---

## Criando o arquivo `.env`

Crie um arquivo chamado:

```text
.env
```

Exemplo:

```env
OPENWEATHER_API_KEY=sua_chave_openweather
N8N_ENCRYPTION_KEY=gere_uma_chave_segura
NGROK_AUTHTOKEN=seu_token_ngrok
```

Esse arquivo **não deve ser enviado para o GitHub**.

---

## Subindo os containers

Execute:

```bash
docker compose up -d
```

Para verificar os containers:

```bash
docker compose ps
```

Para acompanhar os logs:

```bash
docker compose logs -f
```

Para parar os containers:

```bash
docker compose down
```

---

# Acessando o n8n

Após iniciar os containers, acesse o n8n pela porta configurada no Docker Compose.

Exemplo local:

```text
http://localhost:5678
```

---

# Importando o Workflow

## 1. Abra o n8n

Acesse a interface do n8n.

## 2. Crie ou abra um workflow

No menu de workflows, escolha a opção de importação.

## 3. Importe o arquivo

Selecione:

```text
workflow-telegram-chatbot.json
```

## 4. Configure as credenciais

Configure:

### Telegram

Adicione as credenciais do seu bot.

### Google Gemini

Configure as credenciais da API do Gemini, caso utilize o recurso de IA.

### OpenWeather

A chave deve ser disponibilizada através da variável:

```text
OPENWEATHER_API_KEY
```

---

# Ativando o Workflow

Depois de:

* Importar o workflow;
* Configurar as credenciais;
* Configurar as variáveis de ambiente;
* Testar a comunicação;

ative o workflow no n8n.

O Telegram Trigger passará a receber mensagens enviadas ao bot.

---

# Exemplos de Uso

## São Paulo

Entrada:

```text
São Paulo,SP,BR
```

Possível resposta:

```text
🌤️ A temperatura em São Paulo é de 25°C.
```

---

## Belo Horizonte

Entrada:

```text
Belo Horizonte,MG,BR
```

Resultado esperado:

```text
Informações meteorológicas da cidade são retornadas.
```

---

## Curitiba

Entrada:

```text
Curitiba,PR,BR
```

Resultado esperado:

```text
Informações meteorológicas válidas são retornadas ao usuário.
```

---

# Testes

Antes da entrega do projeto, recomenda-se executar os seguintes testes.

## Teste 1 — Cidade válida

Entrada:

```text
São Paulo,SP,BR
```

Esperado:

* A mensagem é recebida;
* A entrada é normalizada;
* A OpenWeather retorna dados;
* A temperatura é processada;
* O Telegram recebe uma resposta.

---

## Teste 2 — Segunda cidade válida

Entrada:

```text
Belo Horizonte,MG,BR
```

Esperado:

* Resposta válida;
* Cidade identificada corretamente;
* Temperatura apresentada.

---

## Teste 3 — Terceira cidade válida

Entrada:

```text
Curitiba,PR,BR
```

Esperado:

* Consulta realizada com sucesso;
* Resposta enviada pelo Telegram.

---

## Teste 4 — Cidade inexistente

Entrada:

```text
CidadeQueNaoExiste,XX,BR
```

Esperado:

```text
❌ Cidade não encontrada ou resposta inválida.
```

O workflow não deve quebrar.

---

## Teste 5 — Entrada com espaços

Entrada:

```text
  São Paulo, SP, BR
```

Esperado:

* Espaços desnecessários removidos;
* Consulta realizada normalmente.

---

# Boas Práticas Aplicadas

## Segurança

* Segredos fora do código;
* Uso de variáveis de ambiente;
* Credenciais configuradas externamente;
* Nenhuma API Key deve ser publicada.

## Manutenibilidade

* Nós com responsabilidades específicas;
* Nomes claros;
* Fluxo visual organizado;
* Separação entre sucesso e erro.

## Confiabilidade

* Validação da entrada;
* Validação da resposta da API;
* Tratamento de erros;
* Fallback para a IA.

## Simplicidade

O workflow utiliza recursos nativos do n8n sempre que possível.

A IA é utilizada apenas como uma camada adicional de apresentação, sem controlar a lógica principal do sistema.

---

# Fluxo de Dados

```text
Usuário
   ↓
Telegram
   ↓
Telegram Trigger
   ↓
Capturar Mensagem
   ↓
Validar Entrada
   ↓
Normalizar Entrada
   ↓
queue
   ↓
OpenWeather API
   ↓
Validar Resposta
   ↓
Dados Meteorológicos
   ↓
Gemini
   ↓
Resposta Natural
   ↓
Telegram
   ↓
Usuário
```

---

# Checklist de Segurança Antes do GitHub

Antes de publicar o projeto, verifique:

* [ ] Nenhuma API Key está no repositório;
* [ ] Nenhum token do Telegram está publicado;
* [ ] O arquivo `.env` está no `.gitignore`;
* [ ] O Docker Compose não contém segredos reais;
* [ ] O workflow exportado não contém credenciais reais;
* [ ] IDs sensíveis foram removidos quando necessário;
* [ ] Credenciais do n8n foram configuradas localmente;
* [ ] O projeto pode ser reproduzido por outro desenvolvedor.

---

# Melhorias Futuras

Possíveis evoluções do projeto:

* Suporte a previsão para os próximos dias;
* Detecção automática de localização;
* Comandos do Telegram;
* Cache de consultas;
* Histórico de pesquisas;
* Monitoramento de falhas;
* Rate limiting;
* Health checks;
* Melhorias na validação da entrada;
* Deploy em ambiente de produção;
* Substituição do ngrok por um domínio próprio com HTTPS.

---

# Autor

**Guilherme Vieira**

Projeto desenvolvido como parte dos estudos e atividades da:

**Faculdade de Tecnologia da Rocketseat (FTR)**



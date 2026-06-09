# Telegram Weather Chatbot - N8N & Google Gemini (Thanos Persona)

Este projeto consiste em um chatbot automatizado para o Telegram desenvolvido na plataforma **N8N** (versão 2.12.1). O bot recebe o nome e a UF de uma cidade brasileira, consulta a API do **OpenWeather**, processa matematicamente a temperatura para um número inteiro e gera uma resposta altamente personalizada utilizando o **Google Gemini**.

O grande diferencial deste projeto é a sua **Persona**: o bot age como o vilão **Thanos (Marvel Comics)**, frustrado por seu destino inevitável neste universo ser apenas informar o clima de cidades brasileiras e dar sugestões de atividades físicas ou de estudos para tirar o interlocutor da inércia. O fluxo conta com tratamento defensivo de erros e um sistema de **Fallback Determinístico** caso a IA balance ou fique indisponível.

---

## ⚙️ Pré-requisitos

Antes de iniciar, certifique-se de ter instalado em sua máquina:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) rodando localmente.
- Um editor de código (como o [VS Code](https://code.visualstudio.com/)) ou acesso ao terminal.

---

## 🚀 Passo a Passo de Instalação e Configuração

### 1. Clonar o Repositório e Preparar os Arquivos

Abra o seu terminal ou VS Code e clone este repositório para a sua máquina local. Garanta que a estrutura de arquivos contenha o `docker-compose.yml`, o `workflow-chatbot-telegram.json` e o arquivo `.env` que configuraremos a seguir.

---

### 2. Criar Conta no Ngrok e Obter a URL Pública

Como o Telegram exige uma URL segura (`https`) para disparar os eventos de Webhook para o seu ambiente local, utilizamos o Ngrok encapsulado no Docker.

1. Acesse o site oficial do [Ngrok](https://ngrok.com/) e crie uma conta gratuita.
2. No painel do Ngrok, vá até a seção **Your Authtoken** e copie o token gerado.
3. Se você tiver um domínio estático gratuito configurado no Ngrok (ex: `seu-dominio.ngrok-free.dev`), copie este endereço.

### 3. Configurar o Arquivo `.env`

Na raiz do projeto, crie ou edite o arquivo `.env` preenchendo as variáveis com os seus dados obtidos no passo anterior. **Substitua os valores de exemplo pelos seus dados reais**:

```env
NGROK_COMMAND_URL=sua-url-do-ngrok.ngrok-free.dev
WEBHOOK_URL=https://sua-url-do-ngrok.ngrok-free.dev
NGROK_AUTHTOKEN=SEU_TOKEN_REAL_DO_NGROK

---

### 4. Inicializar o Ambiente Docker

Com o Docker Desktop aberto e rodando em segundo plano, execute o seguinte comando no terminal da pasta do projeto para subir os contêineres do N8N e do Ngrok em modo background (-d):

docker compose up -d

Acesse o painel do seu N8N local abrindo o navegador no endereço do ngrok: https://sua-url-do-ngrok.ngrok-free.dev

Crie uma conta no N8N.

---

### 5. Importar o Workflow no N8N

No painel do N8N, crie um novo workflow vazio.

Clique no ícone de três pontinhos (...) no canto superior direito da tela.

Selecione a opção Import from File e escolha o arquivo workflow-chatbot-telegram.json presente neste repositório. O fluxo visual completo será carregado na sua tela.

---

### 🔑 Configuração das Credenciais no N8N

Para que o robô funcione, você precisa plugar as suas próprias chaves de acesso diretamente na interface gráfica do N8N. Nenhuma credencial real fica salva no código do repositório.

A. Credencial do Telegram (BotFather)

No seu aplicativo do Telegram, procure pelo bot oficial @BotFather.

Envie o comando /newbot e siga as instruções para escolher o nome e o username do seu bot.

Ao finalizar, o BotFather gerará um token de API (Ex: 123456789:ABCdefGhIJK...). Copie este token.

Volte ao editor do N8N, dê um duplo clique no nó City Message Receiver (Telegram Trigger).

No campo Credential for Telegram API, selecione Create New Credential, cole o seu token no campo Access Token e clique em salvar.

---

B. Credencial do OpenWeather (Nó HTTP)

Acesse o site do OpenWeather e crie uma conta gratuita.

Vá até a aba API Keys e gere um token de acesso para a API 2.5 (Plano gratuito).

No editor do N8N, abra o nó Fetch Weather (HTTP Request). Ele já virá pré-configurado no modo Query Parameter Auth.

Clique em Create New Credential para a autenticação de query.

No campo Name, digite estritamente appid.

No campo Value, cole a sua API Key gerada no OpenWeather e clique em salvar.

---

C. Credencial do Google Gemini (Google AI Studio)

Acesse a plataforma do Google AI Studio com a sua conta Google.

Clique em Get API Key e crie uma nova chave de API em um projeto de sua preferência. Copie a chave gerada.

No editor do N8N, localize o nó do modelo de linguagem conectado ao Agente, chamado Gemini (lmChatGoogleGemini).

Abra o nó, clique em Create New Credential no campo de credenciais e cole a sua API Key do Google AI Studio. Salve a alteração.

---

### 🤖 Ativação e Execução do Chatbot

Com todas as credenciais configuradas e salvas na interface:

Clique no botão Publish para publicar o workflow e deixá-lo visível para o seu chat bot no Telegram.

No Telegram, clique no link do bot disponibilizado pelo @BotFather para abrir a janela de chat com o seu novo bot climático.

Clique em Começar ou envie /start para iniciar a interação.

---

### 🧪 Cenários de Teste Mapeados
O chatbot foi construído de forma defensiva para validar o formato de entrada e responder com precisão a diferentes contextos. Teste enviando os comandos abaixo:

---

Cenário 1: Entrada Válida (Caminho Feliz)

O que enviar: Brasília, DF ou São Paulo, SP ou Lavandeira, TO

O que esperar de retorno: O bot interceptará o formato via Regex, consultará a API, aplicará a regra matemática nativa do JavaScript (Math.round) para arredondar a temperatura e devolverá o JSON processado pela IA em formato de texto.

Exemplo de Resposta esperada:

"A realidade tende a ser decepcionante... Neste universo lamentável, eu sou inevitável apenas para informar que a temperatura em Brasília é de 22°C. Perfeitamente equilibrado, como todas as coisas deveriam ser. Sob este céu limpo, ordeno que saia da sua inércia e vá fazer uma caminhada estratégica para expandir sua mente. Escolhas difíceis requerem determinação forte!"

---

Cenário 2: Tratamento de Erro (Cidade Inexistente ou Incorreta)

O que enviar: Grifnória ou apenas São Paulo (sem a vírgula e a UF).

O que esperar de retorno: A validação em Regex ou o código de tratamento de erro do nó HTTP capturarão a falha (como o Erro 404 da API) e enviarão uma resposta amigável e instrutiva ao usuário, impedindo que o fluxo quebre:

"❌ A cidade não foi encontrada, meu caro! Use o formato do infinito "Cidade, UF" (Exemplo: São Paulo, SP)"

---

Cenário 3: Resiliência contra Indisponibilidade da IA (Sistema de Fallback)

O que acontece se a API do Gemini falhar (Erro 503 ou falta de chaves): O nó do Agente está configurado com a diretriz Continue Error Output. Se a IA falhar ou demorar a responder, o fluxo desviará instantaneamente para o nó Fallback Message Sender, garantindo o envio determinístico da informação climática em formato de texto puro sem deixar o usuário no vácuo:

"Aqui é Thanos! A temperatura de Brasília é de 22°C! Ah... Se eu tivesse as jóias..."
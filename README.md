whatsapp-bot/
├── src/
│   └── index.js
├── .env
├── package.json
└── README.md
{
  "name": "whatsapp-bot",
  "version": "1.0.0",
  "description": "Bot WhatsApp com Node.js e WhatsApp Cloud API",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "dev": "node --watch src/index.js"
  },
  "dependencies": {
    "dotenv": "^16.4.5",
    "express": "^4.21.2"
  }
}
PORT=3000

VERIFY_TOKEN=meu_token_de_verificacao_123

WHATSAPP_TOKEN=COLOQUE_SEU_TOKEN_DA_META_AQUI

PHONE_NUMBER_ID=COLOQUE_SEU_PHONE_NUMBER_ID_AQUI
require("dotenv").config();

const express = require("express");

const app = express();

app.use(express.json());

const PORT = process.env.PORT || 3000;

const VERIFY_TOKEN = process.env.VERIFY_TOKEN;
const WHATSAPP_TOKEN = process.env.WHATSAPP_TOKEN;
const PHONE_NUMBER_ID = process.env.PHONE_NUMBER_ID;

/*
  =========================================
  WEBHOOK - VERIFICAÇÃO DA META
  =========================================
*/

app.get("/webhook", (req, res) => {
  const mode = req.query["hub.mode"];
  const token = req.query["hub.verify_token"];
  const challenge = req.query["hub.challenge"];

  if (mode === "subscribe" && token === VERIFY_TOKEN) {
    console.log("Webhook verificado com sucesso.");
    return res.status(200).send(challenge);
  }

  return res.sendStatus(403);
});

/*
  =========================================
  RECEBER MENSAGENS
  =========================================
*/

app.post("/webhook", async (req, res) => {
  try {
    console.log("Mensagem recebida:");
    console.log(JSON.stringify(req.body, null, 2));

    const entry = req.body.entry?.[0];
    const changes = entry?.changes?.[0];
    const value = changes?.value;

    const message = value?.messages?.[0];

    if (!message) {
      return res.sendStatus(200);
    }

    const from = message.from;

    if (message.type !== "text") {
      return res.sendStatus(200);
    }

    const text = message.text?.body?.trim() || "";

    console.log(`Mensagem de ${from}: ${text}`);

    await processCommand(from, text);

    return res.sendStatus(200);

  } catch (error) {
    console.error("Erro no webhook:", error);
    return res.sendStatus(500);
  }
});

/*
  =========================================
  PROCESSAR COMANDOS
  =========================================
*/

async function processCommand(user, text) {

  const command = text.toLowerCase();

  if (command === "!menu") {
    await sendMessage(
      user,
      `🤖 MENU DO BOT

1️⃣ !menu
2️⃣ !ajuda
3️⃣ !info

Digite um dos comandos acima.`
    );

    return;
  }

  if (command === "!ajuda") {
    await sendMessage(
      user,
      `📚 AJUDA

!menu - Abrir menu
!ajuda - Mostrar ajuda
!info - Informações do bot`
    );

    return;
  }

  if (command === "!info") {
    await sendMessage(
      user,
      `🤖 BOT WHATSAPP

Sistema criado em Node.js.

O bot está funcionando corretamente.`
    );

    return;
  }

  /*
    Resposta automática
  */

  if (
    command.includes("olá") ||
    command.includes("ola") ||
    command.includes("oi")
  ) {
    await sendMessage(
      user,
      `👋 Olá!

Sou o bot automático.

Digite !menu para ver os comandos.`
    );

    return;
  }
}

/*
  =========================================
  ENVIAR MENSAGEM PELO WHATSAPP CLOUD API
  =========================================
*/

async function sendMessage(to, message) {

  const url =
    `https://graph.facebook.com/vXX.X/${PHONE_NUMBER_ID}/messages`;

  try {

    const response = await fetch(url, {
      method: "POST",

      headers: {
        "Authorization": `Bearer ${WHATSAPP_TOKEN}`,
        "Content-Type": "application/json"
      },

      body: JSON.stringify({
        messaging_product: "whatsapp",
        recipient_type: "individual",
        to: to,
        type: "text",
        text: {
          preview_url: false,
          body: message
        }
      })
    });

    const data = await response.json();

    console.log("Resposta da API:");
    console.log(data);

  } catch (error) {
    console.error("Erro ao enviar mensagem:", error);
  }
}

/*
  =========================================
  SERVIDOR
  =========================================
*/

app.get("/", (req, res) => {
  res.send("t🤖 WhatsApp Bot está online!");
});

app.listen(PORT, () => {
  console.log(`Bot rodando na porta ${PORT}`);
});
mkdir whatsapp-bot

cd whatsapp-bot

npm install

npm start
Bot rodando na porta 3000
WhatsApp
   ↓
Meta WhatsApp Cloud API
   ↓
/webhook
   ↓
Node.js
   ↓
processCommand()
   ↓
!menu / !ajuda / !info
   ↓
WhatsApp

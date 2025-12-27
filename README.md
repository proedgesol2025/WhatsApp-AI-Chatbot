# WhatsApp-AI-Chatbot
PROJECT: WhatsApp AI Chatbot (MPWA + Chatwoot + OpenRouter)

🧱 SYSTEM ARCHITECTURE
WhatsApp User
   ↓
MPWA (Webhook)
   ↓
Chatwoot Inbox
   ↓ (Webhook)
AI Bot Server (Node.js + OpenRouter)
   ↓
Chatwoot API
   ↓
MPWA → WhatsApp User

📁 PROJECT STRUCTURE
whatsapp-ai-bot/
│
├── src/
│   ├── server.js
│   ├── webhook.js
│   ├── ai.js
│   ├── chatwoot.js
│
├── .env
├── package.json
├── Dockerfile
└── README.md

🔧 1️⃣ INSTALLATION REQUIREMENTS
Server

Ubuntu 20+

Node.js 18+

PM2 (optional)

Docker (optional)

Accounts

✅ Chatwoot (Self-hosted or Cloud)

✅ MPWA (WhatsApp Multi-Device API)

✅ OpenRouter account

🔐 2️⃣ ENVIRONMENT VARIABLES (.env)
PORT=3000

# OpenRouter
OPENROUTER_API_KEY=sk-or-xxxxxxxx
OPENROUTER_MODEL=meta-llama/llama-3.1-8b-instruct

# Chatwoot
CHATWOOT_BASE_URL=https://chatwoot.yourdomain.com
CHATWOOT_API_TOKEN=your_chatwoot_api_token

📦 3️⃣ INSTALL DEPENDENCIES
npm init -y
npm install express axios dotenv openai body-parser

🚀 4️⃣ SERVER ENTRY POINT (src/server.js)
import express from "express";
import bodyParser from "body-parser";
import dotenv from "dotenv";
import { handleWebhook } from "./webhook.js";

dotenv.config();

const app = express();
app.use(bodyParser.json());

app.post("/chatwoot-webhook", handleWebhook);

app.listen(process.env.PORT, () => {
  console.log("✅ AI Bot running on port", process.env.PORT);
});

🧠 5️⃣ AI LOGIC (OpenRouter) (src/ai.js)
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.OPENROUTER_API_KEY,
  baseURL: "https://openrouter.ai/api/v1",
  defaultHeaders: {
    "HTTP-Referer": "https://yourdomain.com",
    "X-Title": "WhatsApp AI Bot"
  }
});

export async function getAIReply(userMessage) {
  const completion = await client.chat.completions.create({
    model: process.env.OPENROUTER_MODEL,
    messages: [
      {
        role: "system",
        content:
          "You are a professional WhatsApp customer support assistant. Keep replies short and clear."
      },
      { role: "user", content: userMessage }
    ],
    temperature: 0.4
  });

  return completion.choices[0].message.content;
}

🔄 6️⃣ CHATWOOT MESSAGE SENDER (src/chatwoot.js)
import axios from "axios";

export async function sendMessage(conversationId, text) {
  const url = `${process.env.CHATWOOT_BASE_URL}/api/v1/conversations/${conversationId}/messages`;

  await axios.post(
    url,
    {
      content: text,
      message_type: "outgoing"
    },
    {
      headers: {
        Authorization: `Bearer ${process.env.CHATWOOT_API_TOKEN}`,
        "Content-Type": "application/json"
      }
    }
  );
}

🔔 7️⃣ WEBHOOK HANDLER (src/webhook.js)
import { getAIReply } from "./ai.js";
import { sendMessage } from "./chatwoot.js";

export async function handleWebhook(req, res) {
  const data = req.body;

  if (data.event !== "message_created") return res.sendStatus(200);
  if (data.message_type !== "incoming") return res.sendStatus(200);

  const message = data.content;
  const conversationId = data.conversation.id;

  // Security filter
  if (/otp|password|cvv|card/i.test(message)) {
    await sendMessage(conversationId,
      "⚠️ For security reasons, I can't help with sensitive information."
    );
    return res.sendStatus(200);
  }

  // Human handover
  if (message.toLowerCase().includes("agent")) {
    await sendMessage(conversationId,
      "👨‍💻 Connecting you to a human agent."
    );
    return res.sendStatus(200);
  }

  const reply = await getAIReply(message);
  await sendMessage(conversationId, reply);

  res.sendStatus(200);
}

🔗 8️⃣ CHATWOOT CONFIGURATION
Create WhatsApp Inbox

Settings → Inbox → WhatsApp

Copy Inbox ID

Create API Token

Add Webhook
POST https://your-ai-server.com/chatwoot-webhook


Events:

✅ message_created

📲 9️⃣ MPWA → CHATWOOT SETUP

MPWA incoming webhook should call Chatwoot:

{
  "content": "Hello",
  "message_type": "incoming"
}


MPWA sends messages → Chatwoot inbox
Chatwoot sends events → AI server

🧑‍💻 🔄 HUMAN TAKEOVER (IMPORTANT)

In Chatwoot:

Create Automation Rule

Trigger:

Message contains agent

Action:

Assign to human

Add label human_needed

🐳 1️⃣0️⃣ DOCKER DEPLOY (OPTIONAL)
Dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "src/server.js"]

docker build -t whatsapp-ai-bot .
docker run -d -p 3000:3000 --env-file .env whatsapp-ai-bot

🔐 SECURITY & BEST PRACTICES

✅ Never auto-message users
✅ Only reply to incoming chats
✅ Mask sensitive keywords
✅ Add delay (1–2 sec)
✅ Log AI responses
✅ Rate-limit users

🎯 FINAL RESULT

✔ WhatsApp AI Bot
✔ Human + AI hybrid
✔ OpenRouter multi-model support
✔ Scalable & safe
✔ No WhatsApp ban risk

🚀 NEXT (Tell me number)

1️⃣ RAG (AI trained on your website / PDFs)
2️⃣ Multi-language auto detection
3️⃣ Sales lead bot (CRM)
4️⃣ Fintech-safe conversation rules
5️⃣ MPWA webhook code
6️⃣ Admin dashboard

I’ll build the next module for you step-by-step 👌

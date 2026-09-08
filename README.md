# Evolution API — Self-Hosted WhatsApp REST API (One-Click Railway Deploy)

Evolution API is an open-source, self-hosted **WhatsApp REST API** built on Baileys. It gives programmatic control of WhatsApp accounts through a RESTful interface without requiring Meta's Business API approval — send messages, manage groups, stream events, and connect AI agents from a backend you fully control. Deploy the full stack (Evolution API + PostgreSQL + Redis) on Railway in one click, or self-host it anywhere Docker runs.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/evolution-api-whatsapp)

## What Is Evolution API?

Evolution API is an open-source WhatsApp API that lets you send and receive WhatsApp messages programmatically over a REST interface. It connects to WhatsApp by QR scan through the Baileys library, so you skip Meta's Business API onboarding entirely. Because it is self-hosted, your messages, contacts and session state stay on infrastructure you own — with no per-message fees and no third-party messaging vendor in the path.

## 🚀 How to Install & Set Up Evolution API (Quick Start)

Follow these steps to install and set up Evolution API for WhatsApp on Railway:

### Step 1: Deploy on Railway
1. Click **Deploy on Railway** above
2. Wait for all three services — Evolution API, PostgreSQL, Redis — to finish building (~3–5 minutes)

### Step 2: Open the URL Railway generated
1. Open the Manager UI at `https://<your-domain>/manager`
2. Log in with your `AUTHENTICATION_API_KEY` — the template sets it automatically; copy it from the Evolution API service's **Variables** tab in Railway

### Step 3: Connect WhatsApp
1. In the Manager, create a new instance and give it a name — you will use this name in every API path
2. Open the instance to reveal its QR code
3. On your phone open WhatsApp → **Linked devices** → **Link a device**, and scan the QR code
4. The instance state changes to connected

### Step 4: Send your first message
1. Call `POST /message/sendText/<instance>` with the `apikey` header
2. Use the copy-paste curl command below

```bash
curl -X POST "https://<your-domain>/message/sendText/my-instance" \
  -H "apikey: $AUTHENTICATION_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "number": "5511999999999",
    "text": "Hello from Evolution API"
  }'
```

## About Hosting Evolution API

This Railway template deploys three pre-wired services so you don't have to configure them by hand:

- **Evolution API** (`evoapicloud/evolution-api:v2.3.7`) — the WhatsApp REST API, Manager UI, and messaging/webhook/event layer, served on port 8080
- **PostgreSQL 16** — persistent store for instances, messages, contacts, and session state
- **Redis** — cache/session layer, required if you plan to run more than one instance

On Railway these are wired together over the private network with credentials injected via reference variables, so there's no manual connection-string plumbing. The deployment gives you a fully self-hosted WhatsApp backend with no per-message fees and no Meta Business API onboarding — you scan a QR code via Baileys and you're live.

## Common Use Cases

- **Customer support automation** — connect a WhatsApp number to a bot or agent framework and answer support tickets without a live human on every message.
- **AI agent & automation integrations** — wire Evolution API into n8n, Chatwoot, Typebot, Dify, Flowise, Evo AI, or OpenAI to build conversational WhatsApp flows. The n8n + Evolution API pairing is the most common no-code path for WhatsApp automation.
- **Transactional notifications** — send order confirmations, appointment reminders, or alerts from your own backend without per-message vendor fees.
- **Multi-number operations** — run several WhatsApp instances from a single deployment for different teams, brands, or regions.
- **Event-driven pipelines** — stream WhatsApp events to RabbitMQ, SQS, NATS, Kafka, Pusher, or a WebSocket for downstream processing.
- **Media-heavy workflows** — attach S3/MinIO storage for handling images, audio, and documents sent through WhatsApp.

## How Evolution API Compares (vs Twilio & Meta Cloud API)

If you are weighing Evolution API against Twilio or Meta's official Cloud API, the trade-off is cost and control versus official approval:

- **Cost** — Evolution API is a flat self-hosting bill (roughly $5–10/month on Railway) regardless of message volume. Twilio and Meta Cloud API bill per message or per conversation window, which scales with usage.
- **Onboarding** — Evolution API connects by QR scan through Baileys with no Meta Business API approval. Twilio and Meta Cloud API require official onboarding and business verification.
- **Data ownership** — with Evolution API, messages, contacts and session state live in your own PostgreSQL instance; nothing routes through a third-party vendor.
- **Compliance** — this is the trade-off: Twilio and Meta Cloud API are officially sanctioned, while unofficial integrations like Evolution API carry inherent risk (see the ban-safety FAQ below). For high-volume regulated use, Meta's Cloud API is the fully compliant route.

## Dependencies for Evolution API Hosting

### Deployment Dependencies
- [Evolution API (upstream source)](https://github.com/evolution-foundation/evolution-api)
- [Evolution API documentation](https://docs.evolutionfoundation.com.br/)
- [Baileys — the WhatsApp library it is built on](https://github.com/WhiskeySockets/Baileys)
- [A WhatsApp account you can scan a QR code with](https://www.whatsapp.com/)

## ⚙️ Configuration

| Variable | Required | Description |
|---|---|---|
| `SERVER_URL` | Yes | Your full Railway public domain (e.g. `https://your-app.up.railway.app`) — used to construct webhooks and links |
| `AUTHENTICATION_API_KEY` | Yes | Strong secret key used to authenticate all API and Manager UI requests. Generate with `openssl rand -hex 32` |
| `DATABASE_PROVIDER` | Yes | Set to `postgresql` |
| `DATABASE_CONNECTION_URI` | Yes | Postgres connection string for persistent instance/message/contact state |
| `CACHE_REDIS_URI` | Yes | Redis connection string |
| `CACHE_REDIS_ENABLED` | Yes for multi-instance | Set to `true` to enable Redis caching; required when running more than one WhatsApp instance |

A Railway Volume mounted at `/evolution/instances` on the Evolution API service is required for session persistence across redeploys — it is not an environment variable but is essential configuration.

## 🐳 Self-Host Evolution API with Docker Compose

Prefer to run Evolution API on your own Docker host instead of Railway? Clone the repo and bring up the stack:

```bash
git clone https://github.com/sahilrupani/evolution-api-railway-template.git
cd evolution-api-railway-template
cp .env.example .env
```

Edit `.env` and set:
- `SERVER_URL` to the URL you'll access the API on
- `AUTHENTICATION_API_KEY` — generate one with `openssl rand -hex 32`

Then start the stack:

```bash
docker compose up -d
```

Open `http://localhost:8080/manager` to log in and create your first instance.

## ❓ Frequently Asked Questions (FAQ)

### Do I need Meta Business API approval?
No. Evolution API connects by QR scan through Baileys, which bypasses Meta's Business API onboarding entirely.

### How do I use Evolution API without getting my WhatsApp banned?
Unofficial integrations carry inherent risk since they aren't Meta-approved. To reduce the chance of a ban: use numbers with an established history, avoid bulk unsolicited messaging, rate-limit your sends, warm up new numbers gradually, and enable 2FA on the WhatsApp account. For high-volume commercial use, Meta's official Cloud API is the fully compliant route.

### Is my data private?
Yes — the deployment is entirely yours. Messages, contacts and session state live in your own PostgreSQL instance on Railway; nothing routes through a third-party messaging vendor.

### Can I run more than one WhatsApp number?
Yes. Create multiple instances in the Manager UI; each connects its own number. Redis must be enabled (`CACHE_REDIS_ENABLED=true`) for multi-instance operation.

### Can I migrate off Railway later?
Yes. The same stack runs anywhere Docker does — this repo ships the compose file. Move your Postgres data and the `/evolution/instances` volume and the instances come back.

### What phone number format does the API expect?
International format, digits only — country code, area code, then the number, with no `+`, spaces or dashes.

### Why did my WhatsApp session disappear after a redeploy?
This template provisions a persistent volume at `/evolution/instances` automatically, where Baileys session/auth state lives. If your session drops across a redeploy, the volume was detached or removed — re-attach a Railway Volume at `/evolution/instances` to keep the session and avoid re-scanning the QR code.

### How much does this cost to run?
Roughly $5–10/month on Railway's Hobby plan for all three services combined, flat regardless of message volume. Compare that to Twilio ($0.005–$0.085 per message) or Meta's Cloud API, which bills per 24-hour conversation window.

## 🛠️ Support & Issues

If you run into problems with this template, open an issue at [github.com/sahilrupani/evolution-api-railway-template/issues](https://github.com/sahilrupani/evolution-api-railway-template/issues). Please include a description of the problem, steps to reproduce it, and any relevant logs from the Evolution API service.

---

*This is a community-maintained Railway template for [Evolution API](https://github.com/evolution-foundation/evolution-api). It is not affiliated with, endorsed by, or supported by the Evolution API maintainers, WhatsApp, or Meta.*

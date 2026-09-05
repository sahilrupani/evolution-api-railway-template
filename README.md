# Evolution API — Self-Host WhatsApp REST API on Railway (One-Click Deploy)

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/evolution-api-whatsapp?referralCode=zxcgoT)

Evolution API is an open-source WhatsApp REST API built on [Baileys](https://github.com/WhiskeySockets/Baileys). It gives you programmatic control of WhatsApp accounts through a RESTful interface — send messages, manage groups, stream events, and connect AI agents — **without requiring Meta's Business API approval**. This template deploys the full stack (API + PostgreSQL + Redis) pre-wired and ready to use on Railway.

## What this deploys

| Service | Image | Purpose |
|---|---|---|
| Evolution API | `evoapicloud/evolution-api:v2.3.7` | WhatsApp REST API — instances, messaging, webhooks, Manager UI on port 8080 |
| PostgreSQL | `postgres:16-alpine` | Persistent store for instances, messages, contacts, session state |
| Redis | `redis:8-alpine` | Cache / session layer, required for multi-instance |

All three services are wired over Railway's private network with credentials injected via reference variables — no manual connection-string wiring required.

## Why self-host

- **No per-message fees.** Flat infrastructure cost of roughly **$5–10/month** on Railway's Hobby plan, regardless of message volume.
- **No Meta Business approval.** Connect via QR scan (Baileys), skip the Business API onboarding process entirely.
- **Compare the alternatives:** Twilio charges $0.005–$0.085 per message; Meta's Cloud API bills per 24-hour conversation window. Evolution API's cost stays flat whether you send 100 messages or 100,000.
- **Full control.** Your data, your instances, your integrations — nothing routes through a third-party SaaS layer.

## Deploy to Railway

1. Click the **Deploy on Railway** button above.
2. Railway provisions Evolution API, PostgreSQL, and Redis, and injects the connection variables automatically.
3. Mount a volume at `/evolution/instances` (recommended) so WhatsApp sessions survive redeploys.
4. Open the Manager UI on the generated Railway domain (port 8080) and scan the QR code to connect a WhatsApp number.
5. Grab your `AUTHENTICATION_API_KEY` from the service variables and start calling the REST API.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/evolution-api-whatsapp?referralCode=zxcgoT)

## Self-host with Docker Compose

Prefer to run it yourself instead of on Railway? This repo ships a ready-to-use compose file.

```bash
git clone https://github.com/sahilrupani/evolution-api-railway-template.git
cd evolution-api-railway-template
cp .env.example .env
docker compose up -d
```

Then open **http://localhost:8080** for the Manager UI and REST API.

> ⚠️ Make sure a volume is mounted at `/evolution/instances` (already configured in the compose file) — without it, QR auth and session state are lost on every restart.

## Configuration

| Variable | Description |
|---|---|
| `DATABASE_PROVIDER` | Set to `postgresql` |
| `DATABASE_CONNECTION_URI` | Postgres connection string |
| `CACHE_REDIS_URI` | Redis connection string |
| `CACHE_REDIS_ENABLED` | Set to `true` |
| `SERVER_URL` | Public URL of your Evolution API instance |
| `AUTHENTICATION_API_KEY` | API key required to authenticate REST calls |

Note: these self-host variable names differ from the ones Railway injects automatically via reference variables — if migrating between environments, map them accordingly. A volume must be mounted at `/evolution/instances`, or QR authentication is lost on every redeploy.

## Use cases

- Running **multiple WhatsApp instances** from a single deployment
- Powering WhatsApp integrations for **n8n, Chatwoot, Typebot, Dify, Flowise, Evo AI**
- Connecting WhatsApp to **OpenAI**-based conversational agents
- Streaming events to **RabbitMQ, SQS, NATS, Kafka, Pusher, or WebSocket**
- Storing media in **S3 or MinIO** instead of local disk

## FAQ

**Do I need Meta Business API approval?**
No. Evolution API connects via QR scan using Baileys, bypassing Meta's official approval process entirely.

**Is this compliant with WhatsApp's Terms of Service?**
Unofficial integrations carry inherent risk. Use numbers with established history, avoid bulk unsolicited messaging, rate-limit your traffic, and enable 2FA. For high-volume commercial use, Meta's official Cloud API is the compliant route.

**Will my WhatsApp sessions survive a redeploy?**
Only if a volume is mounted at `/evolution/instances`. Without it, you'll need to re-scan the QR code after every redeploy.

**How much does this cost to run?**
Roughly $5–10/month on Railway's Hobby plan for all three services combined, flat regardless of message volume — compared to per-message billing on Twilio or Meta's Cloud API.

**What license is this under?**
See the [upstream repository](https://github.com/evolution-foundation/evolution-api) for licensing terms.

**Where are the full docs?**
[docs.evolutionfoundation.com.br](https://docs.evolutionfoundation.com.br/)

---

*This template packages [Evolution API](https://github.com/evolution-foundation/evolution-api) for one-click deployment on Railway. It is community-maintained and not affiliated with the Evolution API project, Meta, or Railway.*
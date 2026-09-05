# Evolution API Railway Template — Self-Host the WhatsApp REST API (One-Click Deploy)

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/evolution-api-whatsapp?referralCode=zxcgoT)

**Deploy a self-hosted WhatsApp API on Railway in one click** — no Meta Business API approval, no per-message fees. This Railway template provisions Evolution API, PostgreSQL, and Redis, pre-wired over the private network. A production-ready `docker-compose.yml` is included if you'd rather self-host WhatsApp API on your own server.

[Evolution API](https://github.com/evolution-foundation/evolution-api) is an open-source WhatsApp REST API built on [Baileys](https://github.com/WhiskeySockets/Baileys). It gives you programmatic control of WhatsApp accounts through a RESTful interface — send messages, manage groups, stream webhook events, and connect AI agents — from a backend you fully control.

---

## Contents

- [What This Railway Template Deploys](#what-this-railway-template-deploys)
- [Why Self-Host Evolution API Instead of Twilio or Meta Cloud API](#why-self-host-evolution-api-instead-of-twilio-or-meta-cloud-api)
- [Deploy Evolution API to Railway (One-Click)](#deploy-evolution-api-to-railway-one-click)
- [Self-Host Evolution API with Docker Compose](#self-host-evolution-api-with-docker-compose)
- [Sending Your First WhatsApp Message](#sending-your-first-whatsapp-message)
- [Environment Variables Reference](#environment-variables-reference)
- [What You Can Build](#what-you-can-build)
- [Troubleshooting Evolution API](#troubleshooting-evolution-api)
- [FAQ](#faq)

---

## What This Railway Template Deploys

| Service | Image | Purpose |
|---|---|---|
| Evolution API | `evoapicloud/evolution-api:v2.3.7` | WhatsApp REST API — instances, messaging, webhooks, Manager UI on port 8080 |
| PostgreSQL | `postgres:16-alpine` | Persistent store for instances, messages, contacts, session state |
| Redis | `redis:8-alpine` | Cache / session layer, required for multi-instance |

All three services are wired over Railway's private network with credentials injected via reference variables — no manual connection-string wiring required.

**Prerequisites:** a Railway account (Hobby plan or above), a WhatsApp number you can scan a QR code with, and — for the Docker route — Docker Engine with the Compose plugin.

---

## Why Self-Host Evolution API Instead of Twilio or Meta Cloud API

| | Evolution API (self-hosted) | Twilio WhatsApp | Meta Cloud API |
|---|---|---|---|
| **Pricing model** | Flat infrastructure cost | Per message | Per 24-hour conversation window |
| **Typical cost** | ~$5–10/month on Railway Hobby, all three services | $0.005–$0.085 per message | Varies by conversation category |
| **Cost at 100k messages** | Unchanged | Scales linearly | Scales with conversations |
| **Meta Business approval** | Not required (QR scan via Baileys) | Required | Required |
| **Data control** | Fully self-hosted | Third-party SaaS | Meta-hosted |
| **Officially supported** | No — unofficial integration | Yes | Yes |

Self-hosting wins on cost and control; the official APIs win on compliance guarantees. See the [ToS question in the FAQ](#is-this-compliant-with-whatsapps-terms-of-service) before committing to a high-volume commercial use case.

---

## Deploy Evolution API to Railway (One-Click)

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/evolution-api-whatsapp?referralCode=zxcgoT)

1. Click **Deploy on Railway** above and wait for all three services to build (~3–5 minutes).
2. **Mount a volume at `/evolution/instances`** on the Evolution API service. Skip this and your WhatsApp session is lost on every redeploy.
3. Set `SERVER_URL` to your Railway public domain (`https://…`) — QR generation and webhook delivery depend on it.
4. Set `AUTHENTICATION_API_KEY` to a strong secret: `openssl rand -hex 32`.
5. Open `https://<your-domain>/manager`, create an instance, and scan the QR code with WhatsApp → **Linked devices**.

---

## Self-Host Evolution API with Docker Compose

Prefer your own server? The full stack ships in this repo.

```bash
git clone https://github.com/sahilrupani/evolution-api-railway-template.git
cd evolution-api-railway-template
cp .env.example .env
```

**Edit `.env` before starting** — it ships with a placeholder API key:

```bash
# generate a strong key and paste it into AUTHENTICATION_API_KEY
openssl rand -hex 32
```

Then bring the stack up:

```bash
docker compose up -d
```

Open **http://localhost:8080/manager**, create an instance, and scan the QR code.

The compose file declares healthchecks on Postgres and Redis and gates the API behind `condition: service_healthy`, so the API never starts before its dependencies are reachable — this is what prevents the common `P1001: Can't reach database server` restart loop. A named volume `evolution_instances` is mounted at `/evolution/instances`, so sessions survive `docker compose restart`.

---

## Sending Your First WhatsApp Message

Once an instance is connected, every request authenticates with the `apikey` header.

**Send a text message** (`instance` is the name you gave it in the Manager UI):

```bash
curl -X POST "http://localhost:8080/message/sendText/my-instance" \
  -H "apikey: $AUTHENTICATION_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "number": "5511999999999",
    "text": "Hello from Evolution API"
  }'
```

The `number` field takes the recipient in international format, digits only — country code, area code, then the number, with no `+` or spaces. `sendText` also accepts optional `delay`, `linkPreview`, `quoted`, `mentioned`, and `mentionsEveryOne` fields.

On Railway, swap `http://localhost:8080` for your public domain. Full endpoint reference: [docs.evolutionfoundation.com.br](https://docs.evolutionfoundation.com.br/).

---

## Environment Variables Reference

| Variable | Required | Description |
|---|---|---|
| `SERVER_URL` | ✅ | Public URL the API is reached at. Local Docker: `http://localhost:8080`. Production: your HTTPS domain — required for QR and webhooks |
| `AUTHENTICATION_API_KEY` | ✅ | Master API key for **all** requests. Generate with `openssl rand -hex 32` |
| `DATABASE_ENABLED` | ✅ | `true` |
| `DATABASE_PROVIDER` | ✅ | `postgresql` |
| `DATABASE_CONNECTION_URI` | ✅ | Postgres connection string. ⚠️ Self-host uses `DATABASE_CONNECTION_URI`, **not** `DATABASE_URL` |
| `DATABASE_CONNECTION_CLIENT_NAME` | | Client name recorded on the connection (default `evolution`) |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` | | Credentials for the bundled Postgres container. If you change the password, update `DATABASE_CONNECTION_URI` to match |
| `CACHE_REDIS_ENABLED` | ✅ | `true` — required for multi-instance |
| `CACHE_REDIS_URI` | ✅ | Redis connection string, e.g. `redis://redis:6379/6` |
| `CACHE_REDIS_PREFIX_KEY` | | Key namespace (default `evolution`) |
| `CACHE_LOCAL_ENABLED` | | `false` when Redis is in use |
| `LOG_LEVEL` | | e.g. `ERROR` |
| `LANGUAGE` | | `en`, `pt-BR`, or `es` |

Railway injects its own reference variables for the database and cache, so the names differ from this self-host set — map them accordingly when migrating between environments.

---

## What You Can Build

- Run **multiple WhatsApp instances** from a single deployment
- Power WhatsApp integrations for **n8n, Chatwoot, Typebot, Dify, Flowise, and Evo AI**
- Connect WhatsApp to **OpenAI**-based conversational agents and AI chatbots
- Stream events to **RabbitMQ, SQS, NATS, Kafka, Pusher, or WebSocket**
- Store media in **S3 or MinIO** instead of local disk
- Build WhatsApp notification, support-desk, and customer-messaging backends

---

## Troubleshooting Evolution API

**`P1001: Can't reach database server` on startup.**
The API booted before Postgres was accepting connections. The bundled compose file already prevents this with healthchecks plus `depends_on: condition: service_healthy`. On Railway, confirm the database service is running and that `DATABASE_CONNECTION_URI` points at the private-network host.

**QR code has to be re-scanned after every redeploy.**
No persistent volume at `/evolution/instances`. Mount one on the Railway service; the compose file already does this via the `evolution_instances` volume.

**`401 Unauthorized` on every request.**
The `apikey` header is missing or doesn't match `AUTHENTICATION_API_KEY`. Note the header is `apikey` — not `Authorization` or `X-API-Key`.

**QR code never renders, or webhooks never arrive.**
`SERVER_URL` is wrong. It must be the externally reachable URL, HTTPS in production — not `localhost`.

**Database connection string is ignored.**
Self-hosting reads `DATABASE_CONNECTION_URI`. `DATABASE_URL` is a common mix-up and is not the variable this deployment uses.

---

## FAQ

### Do I need Meta Business API approval?
No. Evolution API connects via QR scan using Baileys, bypassing Meta's Business API onboarding entirely.

### Is this compliant with WhatsApp's Terms of Service?
Unofficial integrations carry inherent risk, including number bans. Use numbers with established history, avoid bulk unsolicited messaging, rate-limit your traffic, and enable 2FA. For high-volume commercial messaging, Meta's official Cloud API is the compliant route.

### How much does it cost to run Evolution API on Railway?
Roughly **$5–10/month** on Railway's Hobby plan for all three services combined — flat, regardless of message volume, versus per-message billing on Twilio or per-conversation billing on Meta's Cloud API.

### Will my WhatsApp sessions survive a redeploy?
Only with a volume mounted at `/evolution/instances`. Without it you'll re-scan the QR code after every redeploy.

### Can I run multiple WhatsApp numbers from one deployment?
Yes. Create multiple instances in the Manager UI; each connects its own number. Redis (`CACHE_REDIS_ENABLED=true`) is required for multi-instance operation.

### Can I self-host this without Railway?
Yes — `docker-compose.yml` in this repo runs the identical stack on any Docker host. See [Self-Host Evolution API with Docker Compose](#self-host-evolution-api-with-docker-compose).

### What license is this under?
See the [upstream repository](https://github.com/evolution-foundation/evolution-api) for licensing terms.

### Where are the full API docs?
[docs.evolutionfoundation.com.br](https://docs.evolutionfoundation.com.br/)

---

*This repository packages [Evolution API](https://github.com/evolution-foundation/evolution-api) for one-click deployment on Railway. It is community-maintained and not affiliated with the Evolution API project, Meta, WhatsApp, or Railway.*

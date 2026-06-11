# Purpose of n8n in the JachAI Project

## 1. Short Summary

In **JachAI**, n8n is **not the brain of the fact-checking system**. Your **FastAPI backend is the brain**. n8n is the **automation and integration layer** that connects JachAI with external platforms like:

- Telegram
- WhatsApp
- Messenger
- Discord
- Email
- Scheduled jobs
- Admin alerts
- Daily reports
- System monitoring workflows

The simple architecture is:

```txt
External platform → n8n → JachAI FastAPI backend → n8n → external platform
```

Not:

```txt
External platform → n8n performs the fact-checking itself
```

The core AI verification should stay inside FastAPI. n8n should only handle automation, messaging, scheduling, and integrations.

---

## 2. Current JachAI Core System

Your current JachAI backend already handles the important verification logic:

- Claim submission
- Text, image, and URL handling
- OCR for image claims
- Input cleaning and PII masking
- Language detection
- Claim extraction
- Search query generation
- Tavily live search
- Article crawling when needed
- Evidence chunking
- Source credibility scoring
- pgvector retrieval
- NVIDIA reranking
- Stance classification
- Verdict generation
- Confidence scoring
- Guardrails
- Redis caching
- PostgreSQL persistence
- Async verification jobs
- SSE live status updates
- Admin dashboard
- Rumor clustering
- Source registry
- Audit logs
- System health checks

Because this is already working, n8n should not replace the backend pipeline.

The correct responsibility split is:

```txt
FastAPI = intelligence and verification layer
Next.js = public web product and admin UI
PostgreSQL + pgvector = evidence and claim storage
Redis = cache, status, and rate limit layer
Tavily = live evidence search
Gemini/NVIDIA/Kimi/bge-m3 = AI and retrieval stack
n8n = automation, messaging, alerts, scheduling, and external integrations
```

---

## 3. Why n8n Is Useful for JachAI

Your website already lets users verify claims. But misinformation usually spreads through communication channels like:

- Telegram groups
- WhatsApp groups
- Messenger chats
- Facebook posts
- News links
- Screenshots
- Community chats

n8n helps JachAI work in those places.

Instead of forcing every user to open the website, n8n lets users verify a claim directly from platforms like Telegram.

Example:

```txt
User sends claim to Telegram bot
        ↓
n8n receives the Telegram message
        ↓
n8n sends the claim to JachAI backend
        ↓
JachAI backend verifies the claim
        ↓
n8n receives/fetches the final result
        ↓
n8n replies to the Telegram user
```

This makes JachAI feel like a real-world verification assistant, not just a website.

---

## 4. What n8n Should NOT Do

n8n should **not** perform the actual fact-checking logic.

Do not put these inside n8n:

- Claim extraction
- Tavily evidence ranking logic
- Embedding generation
- pgvector retrieval
- Reranking
- Stance classification
- Verdict generation
- Confidence scoring
- Source trust logic
- AI prompt orchestration
- Database-level verification logic

These belong inside FastAPI.

Why?

Because FastAPI already gives you:

- Version-controlled backend code
- Structured services
- Database models
- Redis caching
- AI clients
- Error handling
- Tests
- Admin dashboard
- Audit logs
- Security
- Health checks
- Clear API contracts

If you move core verification logic into n8n, the system becomes harder to test, debug, scale, and maintain.

Rule:

```txt
FastAPI verifies facts.
n8n connects JachAI to the outside world.
```

---

## 5. Main Purpose of n8n for Telegram

For Telegram, n8n acts as a bridge between the Telegram bot and your backend.

Without n8n, you would need to manually build Telegram bot handling inside your backend:

- Telegram webhook setup
- Message parsing
- Command handling
- Reply formatting
- Retry logic
- Job polling
- Sending Telegram messages
- Error handling
- Telegram bot token management

With n8n, these become visual workflow steps:

- Telegram Trigger
- `/start` command handling
- `/help` command handling
- Message validation
- Backend webhook call
- Job polling
- Final result fetch
- Telegram reply formatting
- Sending verdict back to user

This keeps your backend focused on the hard part: fact-checking.

---

## 6. How n8n Fits Into JachAI Architecture

### Website flow

```txt
User submits text / image / URL
        ↓
Next.js frontend
        ↓
FastAPI backend
        ↓
Verification job
        ↓
Claim extraction + query generation
        ↓
Tavily search + crawling
        ↓
Evidence retrieval + reranking
        ↓
Verdict generation
        ↓
PostgreSQL + Redis
        ↓
Result page / admin dashboard
```

### Telegram flow with n8n

```txt
Telegram user
        ↓
n8n Telegram Trigger
        ↓
n8n sends message to FastAPI webhook
        ↓
FastAPI creates verification job
        ↓
FastAPI runs JachAI pipeline
        ↓
n8n polls job status
        ↓
n8n fetches final claim result
        ↓
n8n sends verdict back to Telegram
```

So n8n is an **external channel adapter**.

---

## 7. Why This Is Valuable for Demo and Hackathon

This makes JachAI look like a real product.

Instead of only saying:

```txt
Users can verify claims on our website.
```

You can say:

```txt
JachAI can be integrated into messaging platforms where misinformation actually spreads.
```

Demo story:

```txt
1. Someone forwards a suspicious claim in Telegram.
2. User sends it to the JachAI Telegram bot.
3. Bot replies: “JachAI is checking this claim.”
4. Backend runs live evidence search and verification.
5. Bot replies with verdict, confidence, summary, and sources.
6. Admin dashboard updates with the new claim.
7. If similar claims repeat, rumor cluster count increases.
```

This shows:

- Public user flow
- AI verification
- Real-time automation
- Admin intelligence
- Multi-platform readiness

---

## 8. n8n Workflow 1: Telegram Claim Verification

This is the most important workflow for now.

### Purpose

Allow users to verify claims directly from Telegram.

### Flow

```txt
Telegram message
  ↓
n8n Telegram Trigger
  ↓
Validate message
  ↓
Send “JachAI is checking this claim...”
  ↓
Call JachAI backend webhook
  ↓
Receive job_id
  ↓
Poll verification job
  ↓
Fetch final claim result
  ↓
Format Telegram response
  ↓
Send verdict back to user
```

### Backend endpoint

```http
POST /api/v1/webhooks/inbound-message
```

### Payload sent by n8n

```json
{
  "platform": "telegram",
  "sender_id": "telegram_chat_id",
  "message_id": "telegram_message_id",
  "text": "user claim here",
  "metadata": {
    "username": "telegram_username",
    "first_name": "telegram_first_name",
    "last_name": "telegram_last_name",
    "chat_type": "private"
  }
}
```

The backend now accepts this Telegram-friendly payload directly, auto-detects URLs inside the `text` field, and enqueues a verification job immediately.

### Expected backend response

```json
{
  "job": {
    "id": "uuid",
    "claim_id": null,
    "input_type": "text",
    "status": "queued"
  },
  "claim": null,
  "cached": false
}
```

### n8n then polls

```http
GET /api/v1/verification-jobs/{job_id}
```

### When completed, n8n fetches

```http
GET /api/v1/claims/{claim_id}
```

### n8n environment variables

```env
JACHAI_BACKEND_URL=https://api-jachai.rafidahad.me
JACHAI_INTERNAL_API_KEY=your_backend_internal_key
JACHAI_TELEGRAM_MAX_POLL_ATTEMPTS=75
JACHAI_TELEGRAM_POLL_DELAY_MS=5000
```

The internal API key belongs in n8n because n8n is a server-side integration layer.
It should not be exposed in the Next.js frontend.

### Final Telegram response example

```txt
❌ JachAI Verdict: Likely False

Confidence: High (91%)

Claim checked:
The government is giving free 50GB internet.

Summary:
Multiple sources indicate this is a fake free-data scam. No official government or telecom source confirms the offer.

Sources:
1. Source title
2. Source title
3. Source title
```

---

## 9. n8n Workflow 2: URL Claim Verification

Once text verification works, URL verification should be supported.

### Purpose

Allow users to send a suspicious news link or social media link to Telegram.

### Flow

```txt
Telegram user sends URL
  ↓
n8n receives message
  ↓
n8n sends text to backend webhook
  ↓
Backend detects URL
  ↓
Backend runs URL claim pipeline
  ↓
n8n polls job
  ↓
n8n replies with verdict
```

### Backend behavior

The backend should detect whether the Telegram message contains a URL and, if needed, split that into:

- `message_url`
- optional supporting text from the rest of the message

```txt
if message contains URL:
    route to URL claim pipeline
else:
    route to text claim pipeline
```

n8n does not need to deeply understand the URL. It can simply pass the text to the backend.

---

## 10. n8n Workflow 3: Telegram Image Verification

This should come after text and URL are stable.

### Purpose

Allow users to send screenshots to the Telegram bot.

### Flow

```txt
Telegram photo
  ↓
n8n receives photo message
  ↓
n8n downloads file from Telegram
  ↓
n8n sends multipart/form-data to backend image endpoint
  ↓
Backend runs OCR + claim pipeline
  ↓
n8n polls job
  ↓
n8n sends verdict
```

### Backend endpoint

```http
POST /api/v1/claims/image
```

### Notes

Image support is more complex because n8n needs to:

- Get Telegram file ID
- Download the image file
- Send it as multipart form data
- Preserve caption/note text if user added any

For demo, text-only Telegram is enough first.

---

## 11. n8n Workflow 4: Viral Rumor Cluster Alert

Your backend already has rumor clustering. n8n can check those clusters regularly.

### Purpose

Notify admins when a rumor starts spreading fast.

### Flow

```txt
Schedule Trigger, every 10 minutes
  ↓
GET /api/v1/rumor-clusters
  ↓
Filter high-risk or high-frequency clusters
  ↓
Send alert to Telegram/Discord/email
```

### Example alert

```txt
🚨 JachAI Viral Rumor Alert

Cluster: Free internet scam
Risk: High
Claims detected: 127
Dominant language: Banglish

Open admin dashboard to review.
```

### Why this matters

This turns JachAI from a passive fact-checking tool into a rumor intelligence system.

---

## 12. n8n Workflow 5: System Health Alert

Your backend already has system health checks.

### Purpose

Notify you if important services are down.

### Flow

```txt
Schedule Trigger, every 5 minutes
  ↓
GET /api/v1/health/system
  ↓
Check service statuses
  ↓
If unhealthy, send Telegram/Discord alert
```

### Services to monitor

- Database
- Redis
- Tavily API
- NVIDIA API
- Gemini API
- Kimi OCR path
- Evidence index readiness
- Backend availability

### Example alert

```txt
🚨 JachAI System Alert

Service: Tavily Search
Status: Unhealthy
Time: 2026-06-11 12:30 AM

Action needed: Check Tavily API key/quota/network.
```

### Why this matters

This gives you basic production monitoring without setting up Prometheus/Grafana immediately.

---

## 13. n8n Workflow 6: Daily Admin Digest

### Purpose

Send a daily summary of platform activity.

### Flow

```txt
Every day at 9 AM
  ↓
GET /api/v1/dashboard/summary
GET /api/v1/dashboard/verdict-distribution
GET /api/v1/dashboard/language-distribution
GET /api/v1/rumor-clusters
  ↓
Format report
  ↓
Send to Telegram/email/Discord
```

### Example daily digest

```txt
JachAI Daily Digest

Claims verified today: 312
Likely False: 48%
Misleading: 19%
Top language: Bangla
High-risk clusters: 5
Most repeated rumor: Free internet scheme
```

### Why this matters

Admins get insights automatically without opening the dashboard all the time.

---

## 14. n8n Workflow 7: Manual Review Notification

Some claims should be reviewed by humans, especially low-confidence or high-risk claims.

### Purpose

Notify admins when claims need manual review.

### Flow

```txt
Schedule Trigger, every 30 minutes
  ↓
GET /api/v1/claims?review_status=needs_review
  ↓
If new items exist
  ↓
Send reviewer alert
```

### Example alert

```txt
⚠️ JachAI Manual Review Queue

12 claims need review.
3 are high-risk.
5 are low-confidence.
Open admin dashboard to inspect.
```

### Why this matters

A real fact-checking platform should have a human-in-the-loop review path.

---

## 15. n8n Workflow 8: Scheduled Evidence Ingestion

### Purpose

Keep the internal evidence database fresh.

### Flow

```txt
Schedule Trigger
  ↓
Fetch trusted RSS/news/fact-check sources
  ↓
Send URLs to backend source ingestion endpoint
  ↓
Backend crawls, embeds, and stores evidence
```

### Backend endpoint

```http
POST /api/v1/sources/ingest
```

### Example sources

- Rumor Scanner
- Prothom Alo
- The Daily Star
- TBS News
- BBC Bangla
- Reuters
- AFP
- Government portals

### Why this matters

Fresh internal evidence makes later verification faster, cheaper, and more reliable.

---

## 16. n8n Workflow 9: Demo Reset / Seed Workflow

### Purpose

Prepare the system before a hackathon demo.

### Flow

```txt
Manual Trigger
  ↓
Call seed evidence endpoint/script
  ↓
Create demo claims
  ↓
Verify demo claims
  ↓
Confirm dashboard has data
```

### Why this matters

It prevents your demo from looking empty.

Before presentation, you can click one button and prepare:

- Demo evidence
- Demo claims
- Recent verdicts
- Dashboard metrics
- Cluster examples

---

## 17. Why Not Just Use Backend Cron Jobs?

You can use backend cron jobs. But n8n is easier for external automation.

### Backend cron jobs are better for:

- Internal technical tasks
- Database cleanup
- Embedding refresh
- Batch indexing
- Deep backend operations

### n8n is better for:

- Telegram bot integration
- Discord alerts
- Email reports
- Admin notifications
- Scheduled summaries
- Webhook automation
- No-code/low-code external integrations

n8n gives you:

- Visual workflow editor
- Easy Telegram integration
- Easy scheduling
- Easy webhook triggers
- Retry logs
- Execution history
- Manual trigger buttons
- Fast workflow modification without backend deployment

---

## 18. Recommended n8n Build Order

Since your core backend is already working, build n8n in this order:

```txt
1. Telegram text claim verification
2. Telegram URL claim verification
3. System health alert
4. Viral rumor cluster alert
5. Daily admin digest
6. Manual review notification
7. Scheduled evidence ingestion
8. Telegram image verification
9. Demo reset workflow
```

Start simple. Do not overcomplicate the Telegram workflow at first.

---

## 19. Required n8n Environment Variables

Use these variables in n8n:

```env
JACHAI_BACKEND_URL=https://your-backend-domain.com
JACHAI_INTERNAL_API_KEY=your_internal_api_key
JACHAI_TELEGRAM_MAX_POLL_ATTEMPTS=75
JACHAI_TELEGRAM_POLL_DELAY_MS=5000
```

### Meaning

`JACHAI_BACKEND_URL`

Your deployed FastAPI backend URL.

`JACHAI_INTERNAL_API_KEY`

The same key used in your backend `.env`:

```env
INTERNAL_API_KEY=your_internal_api_key
```

`JACHAI_TELEGRAM_MAX_POLL_ATTEMPTS`

Maximum number of times n8n will check whether the verification job is complete.

`JACHAI_TELEGRAM_POLL_DELAY_MS`

Delay between each poll request.

The default `75` attempts with `5000` ms delay gives the Telegram workflow about `375` seconds to wait for longer fact-check jobs.

Example:

```txt
10 attempts × 5000 ms = 50 seconds max wait time
```

---

## 20. Telegram Bot Setup

### Step 1: Create bot

Open Telegram and message:

```txt
@BotFather
```

Run:

```txt
/newbot
```

Then provide:

- Bot display name
- Bot username ending in `bot`

BotFather will give you a bot token.

### Step 2: Add Telegram credential in n8n

In n8n:

```txt
Credentials
→ New
→ Telegram API
→ Paste bot token
→ Save
```

### Step 3: Connect credential to workflow nodes

Connect the credential to:

- Telegram Trigger
- Send /start Reply
- Send /help Reply
- Send /about Reply
- Send Invalid Reply
- Send Checking Message
- Send Final Verdict

### Step 4: Activate workflow

In n8n:

```txt
Open workflow
→ Save
→ Toggle Active
```

### Step 5: Test bot

Send:

```txt
/start
```

Then send:

```txt
Is the government giving free 50GB internet?
```

---

## 21. Telegram Commands to Support

### `/start`

Response:

```txt
👋 Welcome to JachAI.

Send me any suspicious claim, viral message, news text, or URL. I’ll verify it using live evidence, retrieval, reranking, and AI reasoning.

Example:
Is the government giving free 50GB internet?
```

### `/help`

Response:

```txt
How to use JachAI:

1. Send a claim, viral post, news text, or URL
2. Wait while JachAI searches evidence
3. Get a verdict with confidence and sources

Supported:
Bangla, English, Hindi, Banglish, Hinglish, Mixed text, and URLs.
```

### `/about`

Response:

```txt
JachAI is a multilingual fact-checking assistant for South Asian misinformation. It checks claims using live search, evidence retrieval, source scoring, reranking, and AI-based verdict generation.
```

---

## 22. Best Demo Flow

For hackathon or product demo, show this:

```txt
1. User opens Telegram bot.
2. User sends a viral claim.
3. Bot replies: “JachAI is checking this claim...”
4. Backend performs Tavily search, retrieval, reranking, stance classification, and verdict generation.
5. Bot replies with verdict, confidence, summary, and sources.
6. Admin dashboard updates with the new claim.
7. Rumor cluster count updates if similar claims exist.
```

This makes JachAI look like a real AI product instead of just a backend API.

---

## 23. Final Recommendation

Your current AI providers are working well, so keep them.

Do not rebuild the core pipeline.

Now use n8n to add product-level automation:

```txt
1. Telegram claim verification
2. System health alert
3. Viral rumor cluster alert
4. Daily admin digest
5. Manual review notification
```

The most important first workflow is:

```txt
Telegram text claim verification
```

Then add URL support, health alerts, and rumor alerts.

---

## 24. Final One-Line Explanation

The purpose of n8n in JachAI is to act as the **automation and communication layer** that connects your working FastAPI fact-checking engine to real-world platforms like Telegram, alerts, schedules, and admin notifications, while keeping the actual AI verification logic safely inside the backend.

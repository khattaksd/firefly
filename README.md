# 🔥 Firefly — Agent Observability Platform

## Spec v0.1 (MVP)

---

## What This Is
Real-time observability for AI agents. Track LLM calls, errors, and costs across Python agents. One decorator to instrument, one dashboard to observe.

**Core value prop**: Instrument your agent in under 5 minutes. See everything in under 5 seconds.

---

## MVP Tech Stack
| Layer | Tech |
|-------|------|
| Python SDK | Python 3.11+ |
| Backend | FastAPI + Python |
| Storage | ClickHouse |
| Real-time | WebSocket (FastAPI) |
| Frontend | Next.js 15 + React |
| Hosting | Hetzner CX22 VPS ($6/mo) + Cloudflare Tunnel |
| Total cost | ~$6-8/mo |

---

## Python SDK

```bash
pip install firefly-sdk
```

### Option A — Decorator
```python
from firefly import observe_agent

@observe_agent("sales-outreach", workspace="acme")
def run(prompt: str):
    response = llm.invoke(prompt)
    return response
```

### Shared SDK Contract
- **Batch flush**: every 1 second or 100 events
- **Offline queue**: disk-backed, syncs on reconnect
- **Idempotent**: dedup by `trace_id`
- **Test mode**: captures traces in-memory for unit tests

---

## Trace Schema

```json
{
  "trace_id": "UUID",
  "agent_id": "string",
  "workspace": "string",
  "timestamp": "ISO 8601 UTC",
  "event_type": "llm_call | error",
  "llm_provider": "openai | anthropic | google",
  "model": "string",
  "prompt_tokens": 342,
  "completion_tokens": 128,
  "latency_ms": 840,
  "status": "success | error | timeout",
  "error_message": "string (optional)"
}
```

---

## Backend API

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/ingest/traces` | Batch trace ingestion |
| GET | `/api/v1/traces` | Query traces (filter by agent, time, status) |
| GET | `/api/v1/traces/{trace_id}` | Single trace details |
| WS | `/ws/traces` | Real-time trace stream |

### Ingestion Request
```json
[
  {
    "trace_id": "abc-123",
    "agent_id": "sales-outreach",
    "workspace": "acme",
    "timestamp": "2026-05-10T14:23:01Z",
    "event_type": "llm_call",
    "llm_provider": "openai",
    "model": "gpt-4o",
    "prompt_tokens": 342,
    "completion_tokens": 128,
    "latency_ms": 840,
    "status": "success"
  }
]
```

Response: `202 Accepted`

---

## Frontend

### MVP Screens
1. **Dashboard** — basic agent health cards (success rate, avg latency)
2. **Trace Explorer** — search/filter traces by agent/time/status
3. **Cost Analytics** — simple token count per agent

Real-time via WebSocket, no polling.

---

## Authentication & Security
- Simple API key-based auth
- Workspace isolation: API key → workspace mapping, all queries scoped by workspace
- ClickHouse encryption at rest (AES-256)
- TLS 1.3 for all traffic via Cloudflare
- No PII collection by default

---

## Onboarding Flow
1. Sign up → get workspace + API key (simple form)
2. `pip install firefly-sdk`
3. Add one decorator: `@observe_agent("my-agent")`
4. Run agent → traces appear live
5. Done. No config files. No YAML.

---

## Hosting

Hetzner CX22 VPS running Docker:
- `clickhouse/clickhouse-server` (port 8123)
- `firefly-backend` (FastAPI + WebSocket)
- Cloudflare Tunnel for ingress
- Cloudflare Pages for frontend

---

## Retention Policy
- Active traces: **30 days** in ClickHouse
- Full data deletion on workspace deletion (completed within 24 hours)

---

## Backlog (Aspirational — Not in MVP)

### SDKs & Integrations
- TypeScript SDK with `@observeAgent` decorator
- Go SDK with context-based tracer
- LangChain integration (`instrument_langchain()`)
- Framework integrations (crewAI, autogen, etc.)

### Enhanced Observability
- Tool invocation tracking
- Conversation tree visualization
- Retry/correlation tracking
- Stream chunk events
- `total_tokens` field in schema

### Backend & Storage
- PostgreSQL for user/auth management
- Multi-tenant admin dashboard
- Advanced cost analytics (real dollar cost from provider pricing)
- Configurable retention tiers (7/30/90/365 days)
- Cloudflare R2 archival

### Alerting
- Alert engine with Slack, email, webhook channels
- Metrics: `success_rate`, `error_rate`, `avg_latency_ms`, `cost_per_hour`, `token_spike`
- Evaluated every 30s via cron

### Frontend Enhancements
- Conversation replay visualization
- Trace correlation/dependency view
- Advanced search/filter
- Export (JSON/CSV)

### Infrastructure & Compliance
- Multi-region deployment
- Auto-scaling
- Rate limiting and DDoS protection
- SOC 2 compliance features
- Advanced audit logging

---

## Immediate Next Steps
1. Build Python SDK core with `@observe_agent` decorator
2. Implement ClickHouse ingestion pipeline
3. Create basic FastAPI backend endpoints
4. Build simple Next.js dashboard
5. Add basic API key authentication
6. Deploy on Hetzner VPS
7. Test with sample LLM agents

---
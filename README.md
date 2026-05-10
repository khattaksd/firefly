# 🔥 Firefly — Agent Observability Platform

## Spec v0.1

### Overview
Firefly is an agent observability platform. Real-time visibility into AI agent behavior:
LLM calls, tool invocations, errors, costs, conversation trees.

**Core value prop**: Instrument your agent in under 5 minutes. See everything in under 5 seconds.

### Tech Stack
| Component | Tech |
|-----------|------|
| Python SDK | Python 3.11+ |
| TypeScript SDK | Node 20+ |
| Go SDK | Go 1.22+ |
| Backend | FastAPI + Python |
| Storage | ClickHouse + PostgreSQL |
| Real-time | WebSocket (FastAPI) |
| Frontend | Next.js 15 + React |

---

### SDK Design

#### Python
```bash
pip install firefly-sdk
```

Option A — decorator:
```python
from firefly import observe_agent

@observe_agent("sales-outreach", workspace="acme")
def run(prompt: str):
    response = llm.invoke(prompt)
    return response
```

Option B — zero code change for LangChain:
```python
from firefly import instrument_langchain
instrument_langchain()
```

#### TypeScript
```bash
npm install @firefly/sdk
```

```typescript
import { observeAgent } from '@firefly/sdk';

@observeAgent('sales-outreach', { workspace: 'acme' })
async function run(prompt: string) {
  const response = await llm.chat(messages);
  return response;
}
```

#### Go
```bash
go get github.com/firefly/go-sdk
```

```go
tracer := firefly.NewTracer(firefly.Config{
  Workspace: "acme",
  APIKey:    os.Getenv("FIREFLY_API_KEY"),
})
ctx := tracer.Start(context.Background(), "sales-outreach")
defer tracer.End(ctx)
```

#### Shared SDK Contract
- Batch flush: every 1 second or 100 events
- Offline queue: disk-backed, syncs on reconnect
- Idempotent: dedup by trace_id
- Test mode: captures traces in-memory for unit tests

---

### Ingestion Pipeline

POST /api/v1/ingest/traces
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
    "status": "success",
    "parent_span_id": "abc-123.parent"
  }
]
```

Response: 202 Accepted

---

### Backend API

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/v1/ingest/traces | Batch trace ingestion |
| GET | /api/v1/traces | Query traces |
| GET | /api/v1/traces/{trace_id}/replay | Full conversation tree |
| POST | /api/v1/alerts | Create alert rule |
| GET | /api/v1/alerts | List alert rules |
| GET | /api/v1/costs | Cost aggregation |
| WS | /ws/traces | Real-time stream |

---

### Trace Data Model

```json
{
  "trace_id": "UUID",
  "agent_id": "string",
  "workspace": "string",
  "timestamp": "ISO 8601 UTC",
  "event_type": "llm_call | tool_call | error | retry | stream_chunk",
  "llm_provider": "openai | anthropic | google | azure | custom",
  "model": "string",
  "prompt_tokens": 342,
  "completion_tokens": 128,
  "total_tokens": 470,
  "latency_ms": 840,
  "status": "success | error | timeout | rate_limited",
  "error_message": "string (optional)",
  "parent_span_id": "string (optional)",
  "tool_name": "string (optional)",
  "tool_input": "object (optional)",
  "tool_output": "object (optional)",
  "span_metadata": "object (optional)"
}
```

---

### Frontend

Key screens:
1. Dashboard — per-agent health cards
2. Trace Explorer — search/filter traces
3. Conversation Replay — step-by-step visualization
4. Alerts — configure rules, view history
5. Cost Analytics — per-agent cost over time

Real-time via WebSocket, no polling.

---

### Alert Engine

```json
{
  "agent_id": "sales-outreach",
  "metric": "success_rate",
  "threshold": 0.90,
  "window_minutes": 60,
  "channel": "slack",
  "webhook_url": "https://hooks.slack.com/..."
}
```

Metrics: success_rate, error_rate, avg_latency_ms, cost_per_hour, token_spike
Channels: Slack, email, webhook
Evaluated every 30s via cron.

---

### Onboarding Flow

1. Sign up → get workspace + API key
2. pip install firefly-sdk
3. Add one line: instrument_langchain() or @observe_agent("my-agent")
4. Run agent → traces appear live
5. Done.

---

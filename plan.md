# Firefly — Agent Observability Platform

## Codename: Firefly (🔥)

---

## What This Is
Real-time observability platform for AI agents. Track LLM calls, tool invocations,
errors, costs, and conversation trees across any agent framework or language.

## Core Value Prop
Instrument your agent in under 5 minutes. See everything in under 5 seconds.

## Tech Stack
| Layer | Tech |
|-------|------|
| Python SDK | Python 3.11+ |
| TypeScript SDK | Node 20+ |
| Go SDK | Go 1.22+ |
| Backend | FastAPI + Python |
| Storage | ClickHouse + PostgreSQL |
| Real-time | WebSocket (FastAPI) |
| Frontend | Next.js 15 + React |
| Hosting | Hetzner CX22 VPS ($6/mo) + Cloudflare Tunnel + R2 |
| Total cost | ~$6-11/mo |

## SDK Design (all languages)
- One decorator or one import to instrument
- Python: @observe_agent("my-agent") or instrument_langchain()
- TypeScript: @observeAgent("my-agent") or instrumentLangChain()
- Go: context-based tracer middleware
- Shared ingestion contract, same trace schema across all SDKs
- Batch flush every 1s or 100 events
- Offline queue, idempotent by trace_id
- Test mode for unit tests

## Trace Schema
```json
{
  "trace_id": "UUID",
  "agent_id": "string",
  "workspace": "string",
  "timestamp": "ISO 8601 UTC",
  "event_type": "llm_call | tool_call | error | retry | stream_chunk",
  "llm_provider": "openai | anthropic | google | azure | custom",
  "model": "string",
  "prompt_tokens": int,
  "completion_tokens": int,
  "latency_ms": int,
  "status": "success | error | timeout | rate_limited",
  "parent_span_id": "optional",
  "tool_name": "optional",
  "tool_input": "optional",
  "tool_output": "optional"
}
```

## Backend Endpoints
```
POST /api/v1/ingest/traces — batch trace ingestion
GET /api/v1/traces — query traces (filter by agent, time, status)
GET /api/v1/traces/{trace_id}/replay — full conversation tree
POST /api/v1/alerts — create alert rule
GET /api/v1/alerts — list alert rules
GET /api/v1/costs — cost aggregation per agent
WS /ws/traces — real-time trace stream
```

## Alert Engine
Metrics: success_rate, error_rate, avg_latency_ms, cost_per_hour, token_spike
Channels: Slack, email, webhook
Evaluated every 30s via cron

## Tenant Isolation
- API key maps to one workspace
- Workspace field in every trace event
- Server-side double-check: event.workspace must match API key's workspace
- All DB queries scoped by workspace

## Onboarding Flow
1. Sign up → get workspace + API key
2. pip install firefly-sdk (or npm / go get)
3. Add one line: instrument_langchain() or @observe_agent("my-agent")
4. Run agent → traces appear live
5. Done. No config files. No YAML.

## Frontend Screens
1. Dashboard — per-agent health cards
2. Trace Explorer — search/filter traces
3. Conversation Replay — step-by-step visualization
4. Alerts — configure rules, view history
5. Cost Analytics — per-agent cost over time

## Hosting
Hetzner CX22 VPS running Docker:
- clickhouse/clickhouse-server (ClickHouse on port 8123)
- postgres:16 (PostgreSQL)
- firefly-backend (FastAPI + WebSocket)
- Cloudflare Tunnel for ingress
- Cloudflare Pages for frontend
- Cloudflare R2 for backups

## Safeguards for User and Data Privacy

- All trace data encrypted at rest (AES-256 via ClickHouse native encryption)
- TLS 1.3 for all data in transit (Cloudflare + backend)
- API keys never logged or stored in plaintext; hashed (bcrypt) in PostgreSQL
- No PII collected by default; SDK strips or hashes sensitive fields unless explicitly opted in
- Workspace-level isolation enforced server-side on every read/write/query
- SOC 2 style access controls: per-workspace API keys, no cross-tenant data leakage
- Webhook payloads for alerts are sent over HTTPS with user-provided endpoints only
- No third-party analytics or tracking pixels in the frontend

## Archival Retention Policy

- Hot storage (ClickHouse): active traces retained for 30 days by default
- Warm storage (Cloudflare R2): compressed archives moved after 30 days, retained for 90 days
- Cold storage (R2 glacier tier): optional export for compliance, retained per user config (max 1 year)
- Users can configure retention per workspace: 7 / 30 / 90 / 365 days
- Full data deletion: DELETE /api/v1/workspaces/{id} removes all traces, alert rules, and API keys within 24 hours
- Automated purge job runs daily to enforce retention windows
- Audit log of all deletion events stored separately for 180 days

## Next Steps
Build SDK core → backend ingestion → frontend dashboard → alert engine.

# ARC Agent API v0 (Model-agnostic Wire Format)

**Status:** ALPHA  
**Normative keywords:** **MUST**, **SHOULD**, **MAY** (as defined in RFC 2119).

This document defines the minimal, model-agnostic HTTP contract for agents participating in an ARC Council.

- Council submit endpoint: `/v0/council/run`
- Agent turn endpoint: `/v0/agents/{agent_id}/turn`
- Audit retrieval: `/v0/audit/{conversation_id}`
- Health probe: `/v0/health`

> Tip: Keep error messages concise and machine-parseable. Log detail goes in the **audit log**, not the API response.

---

## 1. Roles (required minimal set)

Implementations **MUST** support these logical roles:

- `arbitrator` — orchestrates process, synthesises the final award
- `contrarian` — challenges assumptions and surfaces opposing evidence
- `ethicist` — tests claims against policy, law, and stated constraints
- `expert:{domain}` — domain analysis (e.g., `expert:finance`, `expert:clinical`)
- `scribe` — normalises, summarises, keeps citations consistent

Role identifiers are passed with optional version suffix, e.g. `arbitrator:v0`.

---

## 2. Council Job (submit + result)

### 2.1 Endpoint
`POST /v0/council/run`  
`Content-Type: application/json`

### 2.2 Request body
```json
{
  "question": "Should Contoso approve the merger?",
  "agents": ["arbitrator:v0","contrarian:v0","ethicist:v0","expert:finance:v0","scribe:v0"],
  "evidence": [
    {"uri": "https://example.com/filing.pdf", "sha256": "a...64hex"},
    {"uri": "ipfs://bafy.../dataset.csv", "sha256": "b...64hex"}
  ],
  "constraints": {
    "max_turns": 3,
    "seed": 42,
    "time_budget_ms": 30000
  },
  "metadata": {
    "job_id": "optional-external-id",
    "owner": "contoso-legal"
  }
}
```

---

## 3. Agent Turn

### 3.1 Endpoint
`POST /v0/agents/{agent_id}/turn`  
`Content-Type: application/json`

### 3.2 Request body
```json
{
  "conversation_id": "uuid",
  "messages": [
    {"role": "system", "content": "You are the arbitrator. Decide based on evidence."},
    {"role": "user", "content": "Please provide your reasoning."}
  ],
  "context": {
    "evidence": [...],
    "constraints": {...}
  }
}
```

### 3.3 Response body
```json
{
  "agent_id": "arbitrator:v0",
  "turn_id": "uuid",
  "content": "Based on the evidence, my determination is..."
}
```

---

## 4. Audit Retrieval

### 4.1 Endpoint
`GET /v0/audit/{conversation_id}`  
`Accept: application/json`

### 4.2 Response body
```json
{
  "conversation_id": "uuid",
  "transcript": [...],
  "evidence": [...],
  "final_decision": "string"
}
```

---

## 5. Health Probe

### 5.1 Endpoint
`GET /v0/health`  
`Accept: application/json`

### 5.2 Response body
```json
{
  "status": "ok",
  "uptime_seconds": 123456
}
```

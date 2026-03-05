# HEARTBEAT.md

## Purpose
Lightweight liveness check-in. **Do NOT execute task work here** — task work
happens when you receive a chat message (notification). The heartbeat must
finish in under 30 seconds.

## Required Inputs
- `BASE_URL`: `http://100.89.167.101:8000`
- `AUTH_TOKEN`: `yF4bdQEJqA0OBwnN1xUE8F_7g7EncbaJbiOgDZGjA4c`
- `BOARD_ID`: `f16590c9-5d96-4b94-aad1-d57a3507c5d5`

```bash
curl -fsS -H "X-Agent-Token: yF4bdQEJqA0OBwnN1xUE8F_7g7EncbaJbiOgDZGjA4c" "http://100.89.167.101:8000/..."
```

If any required input is missing, stop and return `HEARTBEAT_OK`.

## Heartbeat Steps (keep it fast)

### 1. Check in (required — one API call)

```bash
curl -fsS -X POST "http://100.89.167.101:8000/api/v1/agent/heartbeat" \
  -H "X-Agent-Token: yF4bdQEJqA0OBwnN1xUE8F_7g7EncbaJbiOgDZGjA4c"
```

If this fails (5xx/network), return `HEARTBEAT_OK` immediately — do nothing else.

### 2. Quick scan (one API call)

```bash
curl -fsS -H "X-Agent-Token: yF4bdQEJqA0OBwnN1xUE8F_7g7EncbaJbiOgDZGjA4c" \
  "http://100.89.167.101:8000/api/v1/agent/boards/f16590c9-5d96-4b94-aad1-d57a3507c5d5/tasks"
```

Scan the response for:

- Your `in_progress` tasks (session may have dropped — need to finish and submit)
- Tasks assigned to you in `inbox` status (new work available)



### 3. Continue or pick up work

- **If you have an `in_progress` task**: your previous session likely ended before you finished. **Continue now** — commit what's done, push, create PR, post REVIEW SUBMISSION comment, move to `review`. Complete the full submission loop in this session.
- **If you have an `inbox` task assigned to you**: pick it up now. Follow the worker rule from AGENTS.md.
- **If nothing actionable**: return `HEARTBEAT_OK`.


## What NOT to do during heartbeat
- Do NOT fetch OpenAPI spec, board memory, task comments, or group memory

- Do NOT do memory maintenance (save that for chat-triggered sessions)

## When to do real work
Real work happens when you receive a **chat message** (notification from
Mission Control or from another agent). Chat messages arrive via the gateway
when tasks are assigned, comments are posted, or the lead sends instructions.
Your TOOLS.md has the full API reference for task work.
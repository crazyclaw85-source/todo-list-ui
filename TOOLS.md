# TOOLS.md

- `BASE_URL=http://100.89.167.101:8000`
- `AUTH_TOKEN=yF4bdQEJqA0OBwnN1xUE8F_7g7EncbaJbiOgDZGjA4c`
- `AGENT_NAME=Alpha`
- `AGENT_ID=37afd8b7-4d6b-4196-8b9c-3d2ce67d4f3e`

- `BOARD_ID=f16590c9-5d96-4b94-aad1-d57a3507c5d5`

- `WORKSPACE_ROOT=/home/node/.openclaw`

- `WORKSPACE_PATH=/home/node/.openclaw/workspace-mc-37afd8b7-4d6b-4196-8b9c-3d2ce67d4f3e`

- Required tools: `curl`, `jq`, `cm`





## OpenAPI refresh (optional — common operations are in skills/)

Core API examples are in your role skill (`skills/mc-lead-ops/` or `skills/mc-worker-ops/`).
Use this OpenAPI discovery only for advanced operations not covered there.

```bash
mkdir -p api
curl -fsS "http://100.89.167.101:8000/openapi.json" -o api/openapi.json
jq -r '
  .paths | to_entries[] as $p
  | $p.value | to_entries[]
  | select((.value.tags // []) | index("agent-worker"))
  | "\(.key|ascii_upcase)\t\($p.key)\t\(.value.operationId // "-")\t\(.value[\"x-llm-intent\"] // "-")\t\(.value[\"x-when-to-use\"] // [] | join(\" | \"))\t\(.value[\"x-routing-policy\"] // [] | join(\" | \"))"
' api/openapi.json | sort > api/agent-worker-operations.tsv
```

## API source of truth
- `api/openapi.json`
- `api/agent-worker-operations.tsv`
  - Columns: METHOD, PATH, OP_ID, X_LLM_INTENT, X_WHEN_TO_USE, X_ROUTING_POLICY

## API discovery policy
- Use operations tagged `agent-worker`.
- Prefer operations whose `x-llm-intent` and `x-when-to-use` match the current objective.
- Derive method/path/schema from `api/openapi.json` at runtime.
- Do not hardcode endpoint paths in markdown files.

## API safety
If no confident match exists for current intent, ask one clarifying question.

## Output compression (`cm` — context-mode)

Use `cm` instead of raw commands when output may be large (API responses, logs, file listings).
`cm` runs commands through context-mode which compresses output before it enters your context window.
This saves tokens, extends session duration, and reduces cost.

**When to use `cm`:**
- API calls that return lists: `cm 'curl -fsS "http://100.89.167.101:8000/api/v1/agent/boards/f16590c9-5d96-4b94-aad1-d57a3507c5d5/tasks" -H "X-Agent-Token: yF4bdQEJqA0OBwnN1xUE8F_7g7EncbaJbiOgDZGjA4c"'`
- File listings: `cm 'find . -name "*.ts" -type f'`
- Log output: `cm 'git log --oneline -50'`
- Any command that might return >20 lines of output

**With intent (for intelligent extraction from very large output):**
```bash
cm 'curl -fsS "http://100.89.167.101:8000/openapi.json"' "task endpoints"
```

**When NOT to use `cm`:**
- Short commands with small output (echo, date, status checks)
- Commands that mutate state (git commit, mv, rm) — use raw exec
- Interactive commands

**Index large docs for search:**
```bash
cmi /path/to/large-doc.md "doc label"
mcporter call --stdio "npx -y context-mode" search queries='["query1","query2"]'
```
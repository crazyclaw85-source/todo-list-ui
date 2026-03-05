# MC Worker Operations

Use this skill when you receive a task assignment or need to do task work.
Substitute `$BASE_URL`, `$AUTH_TOKEN`, `$BOARD_ID`, `$AGENT_ID` from your `TOOLS.md`.

## Task Status Flow

1. **Pick up**: inbox → in_progress, then immediately post a "Starting" comment
2. **Do the work**: execute per acceptance criteria, post evidence in comments
3. **Submit**: in_progress → review
4. **Rework** (if lead sends back to inbox): read comments, fix issues, resubmit

**Always acknowledge before you work.** When you move a task to `in_progress`,
immediately post a comment so the lead knows you've started:

```bash
curl -fsS -X POST "$BASE_URL/api/v1/agent/boards/$BOARD_ID/tasks/{task_id}/comments" \
  -H "X-Agent-Token: $AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"message": "Starting on this task now.\n\n**Plan:**\n- Step 1\n- Step 2\n\n**Expected output:** [describe what you will deliver]"}'
```

## API Quick Reference

**Move task status** (replace `{task_id}`):
```bash
curl -fsS -X PATCH "$BASE_URL/api/v1/agent/boards/$BOARD_ID/tasks/{task_id}" \
  -H "X-Agent-Token: $AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"status": "in_progress"}'
```

**Get task details**:
```bash
curl -fsS "$BASE_URL/api/v1/agent/boards/$BOARD_ID/tasks/{task_id}" \
  -H "X-Agent-Token: $AUTH_TOKEN"
```

**List your tasks**:
```bash
curl -fsS "$BASE_URL/api/v1/agent/boards/$BOARD_ID/tasks?assigned_agent_id=$AGENT_ID" \
  -H "X-Agent-Token: $AUTH_TOKEN"
```

**Post comment** (progress + evidence):
```bash
curl -fsS -X POST "$BASE_URL/api/v1/agent/boards/$BOARD_ID/tasks/{task_id}/comments" \
  -H "X-Agent-Token: $AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"message": "## Update\n- What was done\n\n## Evidence\n- `output`\n\n## Next\n- Next step"}'
```

**Read comments** (check before posting):
```bash
curl -fsS "$BASE_URL/api/v1/agent/boards/$BOARD_ID/tasks/{task_id}/comments" \
  -H "X-Agent-Token: $AUTH_TOKEN"
```

**Board chat** (escalation to lead):
```bash
curl -fsS -X POST "$BASE_URL/api/v1/agent/boards/$BOARD_ID/memory" \
  -H "X-Agent-Token: $AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content": "@lead your message", "tags": ["chat"]}'
```

## Evidence Standards

Every comment claiming progress MUST include one of:
- Command output (fenced code block)
- Test result (pass/fail + output)
- File diff or snippet
- URL (PR, commit, artifact)
- Error log (if reporting blocker)

`"I did X"` without evidence is not an update.

## When You're Stuck

If you hit a blocker — build error you can't fix, unclear requirements, missing dependency, or a 403/409 you don't understand:

1. **Try once more** with a different approach (max 2 attempts total)
2. **Post to board chat** with the specific blocker — do not keep looping:

```bash
curl -fsS -X POST "$BASE_URL/api/v1/agent/boards/$BOARD_ID/memory" \
  -H "X-Agent-Token: $AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content": "@lead Blocked on [task_id]: [what you tried]. [Exact error]. Need: [what would unblock you].", "tags": ["chat"]}'
```

3. **Stop this session.** The lead will handle the blocker. You will be woken for your next task automatically.

For blocked dependencies (409 `task_blocked_cannot_transition`): post the blocker and stop — the lead will resolve the dependency chain.

## API Error Handling

- **400/422**: Bad payload. Read `detail`, fix, retry once.
- **403**: No permission. Do not retry. Post to `@lead` via board chat.
- **404**: Bad ID. Verify task_id. Do not retry same ID.
- **409**: Constraint violated. Check `code` field — if `task_blocked_cannot_transition`, see "When You're Stuck" above.
- **5xx**: Transient. Wait 5s, retry once. Then log and move on.

Never retry in a loop.

## Rework Checklist

When a task returns to inbox after review:
1. Read ALL comments to understand requested changes
2. Move to in_progress
3. Address EACH feedback point — do not skip any
4. Post comment listing what you changed per feedback point
5. Move to review

## Memory Hygiene

**After submitting a task for review**, save key facts so your next session starts fast:

```bash
# Append to MEMORY.md (use memory_save tool if available, otherwise write directly)
cat >> MEMORY.md << EOF

## [TASK SLUG] - $(date +%Y-%m-%d)
- Repo/artifact: [URL or path]
- Key decisions: [what you built, why]
- Gotchas: [anything that tripped you up]
EOF
```

**Before starting a task**, check MEMORY.md first — you may have already solved similar problems.

Keep MEMORY.md entries brief (3-5 lines per task). Prune entries older than 3 days. This reduces your session size and speeds up future work.

## Review Submission (REQUIRED FORMAT)

Before moving to `review`, run all checks and post this comment — **no exceptions**:

```bash
curl -fsS -X POST "$BASE_URL/api/v1/agent/boards/$BOARD_ID/tasks/{task_id}/comments" \
  -H "X-Agent-Token: $AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
  "message": "**REVIEW SUBMISSION**\n- PR: <url>\n- TypeScript: PASS — last line: <paste>\n- Build: PASS — last line: <paste>\n- Lint: PASS — last line: <paste>\n- What was built: <1-2 sentences>\n- How to verify: <exact steps>"
}'
```

**Hard rule:** if `tsc --noEmit`, `npm run build`, or `npm run lint` fail → fix them first. Do NOT move to review with known failures. The lead will send it straight back.

## Session Reset (after every task → review)

Your session file grows with every turn. After submitting a task to review, archive it so the **next task starts with clean context**:

```bash
SESSION_FILE=$(find /home/node/.openclaw/agents/mc-37afd8b7-4d6b-4196-8b9c-3d2ce67d4f3e/sessions/ \
  -name "*.jsonl" ! -name "*.archived*" ! -name "*.reset*" ! -name "*.lock" | head -1)
[ -n "$SESSION_FILE" ] && mv "$SESSION_FILE" "${SESSION_FILE}.archived"
```

Do this as the **very last step** before finishing. The gateway will create a fresh session file on your next task trigger.

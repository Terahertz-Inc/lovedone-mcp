---
name: lovedone-family
description: Help a family coordinate messages and day-to-day logistics for an aging parent via LovedOne Family Inbox (senior tech support, not clinical advice).
---

# LovedOne Family Inbox

Every tool named below comes from `https://api.lovedone.app/mcp/v1`.

Positioning: **senior tech support / family coordination — not healthcare.** Do not give medical advice or frame tools as clinical.

Set `LOVEDONE_API_KEY` from inbox.lovedone.app → Settings → MCP keys (or complete OAuth).

## Steps

1. **Ping / profile** — `ping`, then `get_user_profile`.
2. **Household** — `list_loved_ones`, `list_inbox_messages` for the current state.
3. **Writes** — invite/assign/reply only when the user clearly asks; confirm before mutating.

## Rules

- No clinical, diagnosis, or "patient" language.
- Never paste production keys into chat.
- Docs: https://inbox.lovedone.app/agents

## Done when

- Reads cite tool results; writes were confirmed.


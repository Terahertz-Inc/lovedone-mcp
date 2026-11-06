---
name: lovedone-soft
description: Help home-care agency office staff with LovedOne Soft clients, visits, notes, invoices, or scheduling (senior tech support, not clinical advice).
---

# LovedOne Soft agency ops

Every tool named below comes from `https://api.lovedone.app/mcp/vendor/v1`.

Positioning: **senior tech support / agency ops — not healthcare.**

Set `LOVEDONE_SOFT_API_KEY` from soft.lovedone.app → Settings → MCP keys.

## Steps

1. **Ping / profile** — `ping`, `get_agency_profile`.
2. **Lists** — `list_clients`, `list_visits` for the current window.
3. **Writes** — notes, invoices, schedule changes only after an explicit ask; confirm before mutating.

## Rules

- No clinical advice. Operational logistics only.
- Never paste production keys into chat.
- Docs: https://soft.lovedone.app/agents

## Done when

- Reads cite tool results; writes were confirmed.


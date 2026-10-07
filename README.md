# LovedOne MCP servers

This repo lists LovedOne's Model Context Protocol servers in the official MCP
Registry. All three live under the `app.lovedone/*` namespace, which is
`lovedone.app` written backwards.

| Server | Registry name | Endpoint | Status |
|---|---|---|---|
| [LovedOne Family Inbox](#lovedone-family-inbox) | `app.lovedone/family-inbox` | `https://api.lovedone.app/mcp/v1` | Live |
| [LovedOne Soft MCP](#lovedone-soft-mcp) | `app.lovedone/soft` | `https://api.lovedone.app/mcp/vendor/v1` | Live |
| [LovedOne Pipe MCP](#lovedone-pipe-mcp-coming-later) | `app.lovedone/pipe` | `https://api.lovedone.app/mcp/pipe/v1` (planned) | **Not live yet** |

The public provider directory is a separate server, `com.joinsandwich/directory`
(`https://www.joinsandwich.com/api/mcp`, no key needed). It isn't part of this repo.

> This is a public listing repo (README, `server.json` files, LICENSE). The MCP
> servers are hosted at the endpoints above; their source is not in this repo.

## LovedOne Family Inbox

**The family inbox MCP for [LovedOne](https://lovedone.app),**
the family inbox for people caring for an aging parent. Each loved one gets a
private `@lovedone.app` address for doctors, pharmacies, insurers, and
attorneys, and every message lands in one inbox the whole family can see.
This MCP lets an AI assistant summarize your own family's inbox and care
records, invite family, mint provider codes, and reply to messages.

- 📚 **Docs:** [docs.lovedone.app/mcp-family](https://docs.lovedone.app/mcp-family) · [inbox.lovedone.app/agents](https://inbox.lovedone.app/agents)
- 🌐 **Endpoint:** `https://api.lovedone.app/mcp/v1` (Streamable HTTP, JSON-RPC 2.0)
- 🔑 **MCP key:** sign in, then go to [inbox.lovedone.app/settings/mcp-keys](https://inbox.lovedone.app/settings/mcp-keys)
- 🗂️ **Registry name:** `app.lovedone/family-inbox` (formerly `com.joinsandwich/care-circle`)
- 📄 **Listing file:** [`server.json`](server.json)

### ⚠️ Health information

LovedOne inboxes hold mail from doctors, pharmacies, and insurers, so
**responses can contain protected health information (PHI)**, including in
message subjects, previews, and bodies.

- Only connect LovedOne to an AI client you trust with that data, and check
  that client's data-retention and training settings first.
- A key can read your whole family workspace. Don't share it, and
  revoke it at [inbox.lovedone.app/settings/mcp-keys](https://inbox.lovedone.app/settings/mcp-keys) when you're done.
- Write tools (invite, provider codes, replies) need an owner/coordinator
  and, for OAuth, the `care_circle:write` scope. Viewers stay read-only.

### Authentication

- **API key (supported):** create a key at
  [inbox.lovedone.app/settings/mcp-keys](https://inbox.lovedone.app/settings/mcp-keys)
  and send it as `Authorization: Bearer <key>`.
- **OAuth 2.1:** LovedOne runs an OAuth authorization server at
  `https://inbox.lovedone.app` (authorization code + PKCE, dynamic client
  registration). Public scopes: `profile:read`, `care_circle:read`, and
  `care_circle:write`. Write tools require `care_circle:write` on OAuth
  tokens; API keys skip the scope gate and use workspace role.

### Connect

**Cursor** (`~/.cursor/mcp.json` or `.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "lovedone": {
      "url": "https://api.lovedone.app/mcp/v1",
      "headers": { "Authorization": "Bearer YOUR_LOVEDONE_KEY" }
    }
  }
}
```

**Claude Code:**

```bash
claude mcp add --transport http lovedone https://api.lovedone.app/mcp/v1 \
  --header "Authorization: Bearer YOUR_LOVEDONE_KEY"
```

**Claude Desktop** (`claude_desktop_config.json`, via the `mcp-remote` bridge):

```json
{
  "mcpServers": {
    "lovedone": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote", "https://api.lovedone.app/mcp/v1",
        "--header", "Authorization:${AUTH_HEADER}"
      ],
      "env": { "AUTH_HEADER": "Bearer YOUR_LOVEDONE_KEY" }
    }
  }
}
```

Then ask: *"Summarize this week's messages in Mom's inbox and who's handling each one."*

### Tools

Read tools are scoped to the key owner's family workspace. Write tools need
an owner/coordinator and OAuth `care_circle:write` when the caller is OAuth.

| Tool | What it does |
|---|---|
| `ping` | Smoke test: returns the caller's uid, tenant, and auth method. No PHI. |
| `get_user_profile` | The signed-in user's profile and linked family workspace. |
| `list_loved_ones` | Loved ones on the workspace: id, name, and Family Inbox address. |
| `list_family_members` | People on the workspace: role, relationship, invite status. |
| `list_inbox_messages` | Recent messages in one loved one's inbox (subject, sender, category, preview). May contain PHI. |
| `get_inbox_message` | One message as plain text. May contain PHI. |
| `list_care_events` | Recent care events, for one loved one or across the workspace. |
| `invite_family_member` | **Writes.** Invites a sibling or other family member by email (coordinator or viewer). |
| `create_provider_code` | **Writes.** Mints a short-lived Soft provider link code for one loved one. |
| `reply_to_inbox_message` | **Writes.** Sends a plaintext reply to a Family Inbox message. May contain PHI. |

## LovedOne Soft MCP

**The developer MCP for care agencies on LovedOne Soft.** One key covers one
agency. Agents and developers can read the agency's clients, visits, and
invoices, log visit notes, and schedule visits. A note on a client who is
linked to a LovedOne family inbox is copied into that family's inbox. Auth
is an agency API key or OAuth (`profile:read`, `care_circle:read`,
`care_circle:write`).

- 🌐 **Endpoint:** `https://api.lovedone.app/mcp/vendor/v1` (Streamable HTTP, JSON-RPC 2.0)
- ℹ️ **Path note:** the live Soft MCP is `/mcp/vendor/v1`, not `/mcp/soft/v1` (that path is not an MCP route today). Family inbox is `/mcp/v1`, not bare `/mcp`.
- 🔑 **MCP key:** sign in at [soft.lovedone.app](https://soft.lovedone.app), then go to Settings → MCP keys
- 📚 **Docs:** [docs.lovedone.app/mcp-vendor](https://docs.lovedone.app/mcp-vendor) · [soft.lovedone.app/agents](https://soft.lovedone.app/agents)
- 🗂️ **Registry name:** `app.lovedone/soft`
- 📄 **Listing file:** [`soft/server.json`](soft/server.json)

A family key on this endpoint (or an agency key on the family endpoint) is
rejected with HTTP 403 `WRONG_SURFACE`.

> ⚠️ Visit notes can contain health information. The same rules as the family
> inbox apply: use a trusted client and revoke keys you no longer need.

**Cursor:**

```json
{
  "mcpServers": {
    "lovedone-soft": {
      "url": "https://api.lovedone.app/mcp/vendor/v1",
      "headers": { "Authorization": "Bearer YOUR_SOFT_KEY" }
    }
  }
}
```

**Claude Code:**

```bash
claude mcp add --transport http lovedone-soft https://api.lovedone.app/mcp/vendor/v1 \
  --header "Authorization: Bearer YOUR_SOFT_KEY"
```

| Tool | What it does |
|---|---|
| `ping` | Smoke test: returns the caller's uid, tenant, and auth method. No PHI. |
| `get_agency_profile` | The agency: id, name, slug, plan, and the caller's role. |
| `list_clients` | The agency's clients, and whether each is linked to a family inbox. |
| `get_client` | One client plus recent visits and notes. May contain PHI. |
| `list_visits` | Visit and note events, for the agency or one client. May contain PHI. |
| `add_visit_note` | **Writes.** Logs a note on a client; copies it to a linked family inbox. OAuth needs `care_circle:write`. |
| `schedule_visit` | **Writes.** Creates a scheduled visit for a client. OAuth needs `care_circle:write`. |
| `list_invoices` | Invoices the agency issued to families (amounts, status). |

## LovedOne Pipe MCP (coming later)

**LovedOne Pipe MCP will connect LovedOne to healthcare systems,** bringing partner
events from EHR and EVV systems into a family's care record.

- 🚧 **Status: not live.** `https://api.lovedone.app/mcp/pipe/v1` is the
  planned endpoint and does not answer yet. Don't add it to a client.
- 🗂️ **Registry name (reserved):** `app.lovedone/pipe`
- 📄 **Listing file:** [`pipe/server.json`](pipe/server.json), which won't be
  published until the endpoint is live.

## Links

- [lovedone.app](https://lovedone.app) · [Privacy](https://lovedone.app/privacy) · [Terms](https://lovedone.app/terms)
- Contact: hello@lovedone.app

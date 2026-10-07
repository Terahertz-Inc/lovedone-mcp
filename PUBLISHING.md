# Publishing to the official MCP Registry

This folder is the content for the public listing repo
**`Terahertz-Inc/lovedone-mcp`**. The official registry (registry.modelcontextprotocol.io)
reads each `server.json`. Other directories (Glama, MCP Market, MCP.Directory,
PulseMCP, …) copy from the registry or want a public repo with a README and
LICENSE.

| File | Registry name | Endpoint | Publish? |
|---|---|---|---|
| `server.json` | `app.lovedone/family-inbox` | `https://api.lovedone.app/mcp/v1` | Yes |
| `soft/server.json` | `app.lovedone/soft` | `https://api.lovedone.app/mcp/vendor/v1` | Yes |
| `pipe/server.json` | `app.lovedone/pipe` | `https://api.lovedone.app/mcp/pipe/v1` | **No. Wait until the endpoint is live.** |

Nothing here has been published. Do these steps on your own machine.

> **Pipe:** the registry does not check that a remote URL answers, so
> `mcp-publisher publish pipe/server.json` would succeed today and create a
> listing that points at nothing. Don't publish it until
> `https://api.lovedone.app/mcp/pipe/v1` answers `initialize`. If the final
> URL is different, update `remotes[0].url` first.

## How the namespace works

The registry names a domain-verified server after the domain written
backwards: `lovedone.app` → `app.lovedone/*`.

- **HTTP proof** (`/.well-known/mcp-registry-auth` on `https://lovedone.app`)
  grants `app.lovedone/*` only. That covers all three names above, because
  they all sit directly under `app.lovedone/`. It does **not** grant
  subdomain namespaces like `app.lovedone.soft/*`.
- **DNS proof** (TXT record on the `lovedone.app` apex) grants
  `app.lovedone/*` **and** `app.lovedone.*/*` (all subdomains).
- Remote URLs don't have to match the namespace, so endpoints on
  `api.lovedone.app` are fine either way. They only have to be `https` and
  not localhost.

## 0. Before you start

- [ ] Confirm the license in `LICENSE` (MIT placeholder; it only covers the
      files in this listing repo, not the hosted services).
- [ ] Create the public repo `Terahertz-Inc/lovedone-mcp` and copy this folder's
      contents to its root. Every `server.json` → `repository.url` already
      points there (Soft and Pipe use `subfolder`).
- [ ] Install the CLI: `brew install mcp-publisher` (or download
      `mcp-publisher` from https://github.com/modelcontextprotocol/registry/releases).
- [ ] OpenSSL 3 (macOS LibreSSL can't do Ed25519): `brew install openssl@3`,
      then use `$(brew --prefix openssl@3)/bin/openssl` below.

## 1. Generate the domain key (once per domain)

```bash
OPENSSL="$(brew --prefix openssl@3)/bin/openssl"   # or plain `openssl` on Linux
DOMAIN="lovedone.app"
mkdir -p ~/.mcp-registry-keys && cd ~/.mcp-registry-keys
"$OPENSSL" genpkey -algorithm Ed25519 -out "$DOMAIN.pem"
chmod 600 "$DOMAIN.pem"

# Public key (safe to publish) → goes in the hosting env var
"$OPENSSL" pkey -in "$DOMAIN.pem" -pubout -outform DER | tail -c 32 | base64
```

Keep `~/.mcp-registry-keys/$DOMAIN.pem` somewhere secure and private (an
encrypted backup, a password manager, whatever you already use). You need it
again for every future publish. **Never commit the .pem or the private hex to
any repo, and don't paste it into chats or tickets.**

## 2. Serve the proof file

The registry fetches `https://lovedone.app/.well-known/mcp-registry-auth`
(apex domain, no redirects) and expects one line:

```
v=MCPv1; k=ed25519; p=<base64 public key from step 1>
```

`lovedone.app` is the Vercel project for `apps/lovedone` in
`Terahertz-Inc/sandwich-platform`. Its middleware serves this file from an
env var:

1. Vercel → **sandwich-platform-app-lovedone** → Settings → Environment Variables →
   add `MCP_REGISTRY_PUBLIC_KEY` = the base64 public key (Production).
2. Redeploy production.

**Already have a key?** `docs/MCP-REGISTRY-NEXT-STEPS.md` (Aug 12) says a
`lovedone.app` Ed25519 key was generated at `.mcp-registry-key/lovedone.app.pem`
in a local checkout, for DNS proof (public key
`xeYgtT+9LOWuSSaB/GjxZJg9NrQQY7cJ8Mgweqqn6us=`). If you still have that pem,
reuse it here instead of generating a new one. As of Oct 3 there is no
`MCPv1` TXT record on lovedone.app and the proof path returns 404.

Check it:

```bash
curl -s -H 'Accept: text/plain' https://lovedone.app/.well-known/mcp-registry-auth
# → v=MCPv1; k=ed25519; p=...
```

(Alternative: DNS proof. Put the same string in an apex TXT record on
`lovedone.app` and use `mcp-publisher login dns` instead of `login http`.)

## 3. Log in and publish

```bash
cd path/to/Terahertz-Inc/lovedone-mcp   # the repo root
PRIVATE_KEY="$("$OPENSSL" pkey -in ~/.mcp-registry-keys/lovedone.app.pem -noout -text | grep -A3 'priv:' | tail -n +2 | tr -d ' :\n')"
mcp-publisher login http --domain lovedone.app --private-key "$PRIVATE_KEY"
unset PRIVATE_KEY

mcp-publisher validate server.json && mcp-publisher publish server.json
mcp-publisher validate soft/server.json && mcp-publisher publish soft/server.json
# pipe/server.json: NOT YET. Only after https://api.lovedone.app/mcp/pipe/v1 is live.

mcp-publisher logout

curl -s "https://registry.modelcontextprotocol.io/v0/servers?search=app.lovedone" | jq '.servers[].server | {name, version}'
```

The login token is short-lived. Server names are permanent: they can be
deprecated or deleted later but never renamed. To ship a change, bump
`version` in that `server.json` and publish it again.

## 4. Point the old care-circle entry at the new name

After `app.lovedone/family-inbox` is live, run
`scripts/mcp-registry-retire-joinsandwich.sh` in `sandwich-platform`
(dry run by default). It deprecates `com.joinsandwich/care-circle` only.
`com.joinsandwich/directory` (the provider directory) stays active.

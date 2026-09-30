# Setup MCP Servers

Configure all MCP servers for the pure-tools development environment. Run once on a new machine or after a fresh Claude Code install.

## Current MCP stack

| Server | Scope | Purpose |
|--------|-------|---------|
| GitHub | user | Repos, issues, PRs, releases via API |
| Supabase | user | DB schema, SQL, migrations, Edge Functions |
| Vercel | plugin | Deployments, env vars, logs, domains |
| Anthropic Claude Docs | plugin | Anthropic API + Claude Code docs |
| Google Drive / Gmail / Calendar | plugin | Already wired via claude.ai |

---

## 1. GitHub MCP

Requires a GitHub Personal Access Token with repo + org read/write.

```bash
claude mcp add github -s user \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=$(gh auth token) \
  -- npx -y @modelcontextprotocol/server-github
```

If `gh` is not authenticated: `gh auth login` first.

---

## 2. Supabase MCP

Requires a Supabase Personal Access Token.

1. Go to **supabase.com/dashboard/account/tokens**
2. Generate token → select **All access**
3. Run:

```bash
claude mcp add supabase "npx --yes @supabase/mcp-server-supabase" -s user -e SUPABASE_ACCESS_TOKEN=<token>
```

---

## 3. Vercel MCP

Installed via the Claude Code plugin marketplace (already bundled). Just needs auth:

1. Run `/env` or start any Vercel MCP operation — Claude will prompt for auth
2. Approve at the Vercel OAuth URL
3. Paste the `localhost:47556/callback?...` URL back if the redirect page fails

Or trigger manually:
```
claude mcp authenticate plugin:vercel:vercel
```

---

## 4. Anthropic Claude Docs MCP

Already active via the `claude.ai` plugin marketplace. No setup needed.
URL: `https://api.anthropic.com/v1/pages/mcp`

---

## Verify all connected

```bash
claude mcp list
```

Expected output — all show `✔ Connected`:
```
claude.ai Claude Docs    ✔ Connected
plugin:vercel:vercel     ✔ Connected
github                   ✔ Connected
supabase                 ✔ Connected
```

---

## Notes

- MCP tokens are stored in `~/.claude.json` (user scope) — never commit this file.
- GitHub token auto-refreshes via `gh auth token` if you re-run the add command.
- Supabase token does not expire unless revoked manually.
- Vercel OAuth token refreshes automatically via the plugin.

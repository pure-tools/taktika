# Deploy

Check, promote, and troubleshoot Vercel deployments for pure-tools projects.

## Projects on Vercel

| Project | GitHub repo |
|---------|-------------|
| cuefade | pure-tools/cuefade |
| garden | pure-tools/garden |
| top-3-in-sports | pure-tools/top-3-in-sports |

---

## Check deployment status

Via Vercel MCP (no CLI needed):
> "What's the latest deployment status for cuefade?"

Or via CLI:
```bash
vercel ls <project-name>
vercel inspect <deployment-url>
```

---

## View build / runtime logs

Via Vercel MCP:
> "Show me the build logs for the last cuefade deployment"
> "Show runtime errors for cuefade in the last hour"

Or CLI:
```bash
vercel logs <deployment-url>
vercel logs <deployment-url> --since 1h
```

---

## Trigger a deployment

Push to `master`/`main` triggers auto-deploy via GitHub integration.

Manual deploy:
```bash
vercel --prod
```

---

## Promote preview → production

Via Vercel MCP:
> "Promote deployment <url> to production for cuefade"

Or CLI:
```bash
vercel promote <deployment-url>
```

---

## Rollback

Via Vercel MCP:
> "Roll back cuefade to the previous production deployment"

Or CLI:
```bash
vercel rollback
```

---

## Environment variables

Via `/env` skill or Vercel MCP:
> "List env vars for cuefade"
> "Add STRIPE_PRICE_ID to cuefade production"

See also: `/env` skill (vercel plugin).

---

## Troubleshooting checklist

1. **Build failed** → check build logs for TypeScript or ng-packagr errors. Run `npm run build` locally first.
2. **Runtime 500** → check runtime logs via Vercel MCP. Usually a missing env var or Supabase connection issue.
3. **SSR prerender crash** → `localStorage` accessed in Node context. Fix: wrap in try/catch (see `/fix-tests` pattern 3).
4. **Env var missing** → `vercel env ls production` — compare against what the app reads from `environment.ts`.
5. **Old deployment served** → check if CDN cache invalidation is needed: `vercel alias` or redeploy.

---

## Common env vars across projects

| Var | Projects |
|-----|---------|
| `SUPABASE_URL` | cuefade, garden |
| `SUPABASE_ANON_KEY` | cuefade, garden |
| `STRIPE_PRICE_ID` | cuefade |
| `STRIPE_PUBLIC_KEY` | cuefade |
| `STRIPE_SECRET_KEY` | cuefade (server) |
| `LS_STORE_SLUG` | garden |
| `LS_PRODUCT_ID` | garden |

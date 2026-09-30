# New Pure-Tools Project

Scaffold a new Angular app or library in the pure-tools ecosystem. This skill composes all other taktika skills.

## Decide: app or library?

| Type | When | Template |
|------|------|----------|
| `@pure-tools/xxxka` library | Reusable Angular service/guard/pipe | See `pure-tools/pure-tool-anatomy.md` |
| Consumer app | End-user product (cuefade, garden, top-3-in-sports) | Continue below |

For a **library** → follow `pure-tools/pure-tool-anatomy.md` then come back here for MCP + repo setup only.

---

## Consumer app checklist

### 1. Scaffold Angular app

```bash
ng new <app-name> --routing --style css --ssr false
cd <app-name>
```

Or with SSR (top-3-in-sports pattern):
```bash
ng new <app-name> --routing --style css --ssr true
```

### 2. Install pure-tools packages

```bash
npm install @pure-tools/paletka @pure-tools/mobilka @pure-tools/monetka @pure-tools/babetka @pure-tools/slushalka
```

### 3. Wire pure-tools in app.config.ts

```ts
import { provideTheme } from '@pure-tools/paletka';
import { provideResponsive } from '@pure-tools/mobilka';
import { providePayments } from '@pure-tools/monetka';
import { AUTH_PROVIDER, provideSecurka } from '@pure-tools/babetka';
import {
  provideSlushalka, providePageViewTracking, provideErrorTracking,
  umamiAdapter, consoleAdapter,
} from '@pure-tools/slushalka';

export const appConfig: ApplicationConfig = {
  providers: [
    // ...
    provideTheme(),
    provideResponsive({ strategy: 'combination' }),
    providePayments({ provider: 'stripe', publicKey: env.stripePublicKey, productId: env.stripePriceId }),
    { provide: AUTH_PROVIDER, useExisting: AuthService },   // or useFactory adapter if signal name differs
    provideSecurka(),
    provideSlushalka({
      adapters: env.production ? [umamiAdapter()] : [consoleAdapter()],
      superProps: { app: '<app-name>' },
    }),
    providePageViewTracking(),   // hash routes included
    provideErrorTracking(),      // browser errors → $exception (Vercel logs never see these)
  ],
};
```

### 4. Wire SessionTimeoutService in app.ts

```ts
import { Component, inject, effect } from '@angular/core';
import { SessionTimeoutService } from '@pure-tools/babetka';

export class App {
  private auth = inject(AuthService);
  private sessionTimeout = inject(SessionTimeoutService);
  constructor() {
    effect(() => {
      if (this.auth.isLoggedIn()) this.sessionTimeout.start();
      else this.sessionTimeout.stop();
    });
  }
}
```

### 4b. Analytics events

Track the funnel from day one — cheap now, impossible to backfill later:

- `upgrade_modal_shown` with `{ source }` = which gated feature triggered it
- `checkout_started` / `checkout_failed` / `checkout_completed` (Stripe `?upgraded=1` return)
- one event per core feature (e.g. cuefade: `crossfade_started`, `cue_point_added`)
- `identify(user.id)` on login (never email), `setSuperProps({ plan })`, `reset()` on logout only

Umami: load `https://cloud.umami.is/script.js` with `data-auto-track="false"` only when `umamiWebsiteId` is set (see cuefade `core/analytics/cuefade-analytics.ts`).
Specs: any TestBed that creates a class injecting `AnalyticsService` needs `{ provide: AnalyticsService, useValue: { track: vi.fn(), ... } }`.

### 5. Add Supabase (if auth + DB needed)

```bash
npm install @supabase/supabase-js
```

Create `src/app/core/services/supabase.client.ts` with env-driven URL + anon key.
AuthService pattern: `isLoggedIn = computed(() => user() !== null)` + `signOut()`.

### 6. Set up Vercel project

```bash
vercel link
vercel env add SUPABASE_URL
vercel env add SUPABASE_ANON_KEY
vercel env add STRIPE_PRICE_ID       # if payments
```

Or use Vercel MCP directly from chat.

### 7. Create GitHub repo

```bash
gh repo create pure-tools/<app-name> --public
git remote add origin https://github.com/pure-tools/<app-name>.git
git push -u origin main
```

### 8. MCP setup (new machine only)

→ Follow `setup-mcps.md`

---

## Post-setup checklist

- [ ] `npm run build` passes
- [ ] `npm test` passes
- [ ] `vercel env ls` shows all required vars
- [ ] `AUTH_PROVIDER` wired — `babetka` guards work
- [ ] Analytics events visible in console (dev) — `umamiWebsiteId` set for prod
- [ ] Repo pushed to `pure-tools/<name>`
- [ ] Added to consumer list in `pure-tools/update-pure-tools.md`

---

## Related skills

| Skill | When to use |
|-------|------------|
| `pure-tools/pure-tool-anatomy.md` | Building a library, not an app |
| `pure-tools/update-pure-tools.md` | After publishing a new pure-tools version |
| `setup-mcps.md` | New machine or fresh Claude Code install |
| `deploy.md` | Deploy status, logs, promote, rollback |
| `fix-logs.md` | Runtime errors → fix PRs |

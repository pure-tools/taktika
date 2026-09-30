# Update Pure Tools Dependencies

Update all pure-tools consumer projects to the latest versions of `@pure-tools/paletka`, `@pure-tools/monetka`, `@pure-tools/mobilka`, `@pure-tools/babetka`, and `@pure-tools/slushalka`. Run this after publishing a new version of any of these packages.

## Consumer projects

| Path | Remote | Uses |
|------|--------|------|
| `C:\Work\git\cuefade` | https://github.com/pure-tools/cuefade | paletka, mobilka, monetka, babetka, slushalka |
| `C:\Work\git\top-3-in-sports` | https://github.com/pure-tools/top-3-in-sports | paletka, mobilka, monetka, babetka |
| `C:\Work\git\garden` | https://github.com/pure-tools/garden | paletka, mobilka, monetka, babetka |

Only upgrade packages a project already depends on — don't add new ones here (that's an integration, not an upgrade).

## Steps

For each project:

1. **Check current versions** — run `npm list @pure-tools/paletka @pure-tools/monetka @pure-tools/mobilka @pure-tools/babetka @pure-tools/slushalka` to see what's installed.

2. **Get latest versions from npm** (`--prefer-online` — the local npm cache can report a stale version for minutes after a publish):
   ```
   npm view @pure-tools/paletka version --prefer-online
   npm view @pure-tools/monetka version --prefer-online
   npm view @pure-tools/mobilka version --prefer-online
   npm view @pure-tools/babetka version --prefer-online
   npm view @pure-tools/slushalka version --prefer-online
   ```

3. **Install latest** in each project directory:
   - cuefade: `npm install @pure-tools/paletka@latest @pure-tools/monetka@latest @pure-tools/mobilka@latest @pure-tools/babetka@latest @pure-tools/slushalka@latest`
   - garden: same minus slushalka
   - top-3-in-sports: same but with `--legacy-peer-deps` (TypeScript 6 / typescript-eslint conflict)

4. **Build each project** to verify no breaking changes:
   - `npm run build` and `npm test`
   - If build fails, diagnose and fix before committing.
   - Test failures in files with uncommitted local changes are WIP, not the upgrade — commit only `package.json` + `package-lock.json`, never stage the user's WIP.

5. **Commit and push** each project:
   ```
   git add package.json package-lock.json
   git commit -m "chore: upgrade pure-tools deps to latest"
   git push
   ```

## Notes

- top-3-in-sports always needs `--legacy-peer-deps` due to pre-existing TypeScript 6 vs typescript-eslint peer conflict.
- garden uses `isAuthenticated` (not `isLoggedIn`) on its `AuthService` — the `AUTH_PROVIDER` factory adapter in `app.config.ts` bridges this.
- 0.x minor bumps (e.g. mobilka 0.3 → 0.4) are outside `^` ranges and may be breaking — check the release commit. mobilka 0.4.0: Capacitor services import from `@pure-tools/mobilka/native`.
- If a new pure-tools package introduces breaking API changes, check each project's integration points before pushing.
- Run projects in parallel where possible to save time.

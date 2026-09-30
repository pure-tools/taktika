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

2. **Get latest versions from npm**:
   ```
   npm view @pure-tools/paletka version
   npm view @pure-tools/monetka version
   npm view @pure-tools/mobilka version
   npm view @pure-tools/babetka version
   npm view @pure-tools/slushalka version
   ```

3. **Install latest** in each project directory:
   - cuefade: `npm install @pure-tools/paletka@latest @pure-tools/monetka@latest @pure-tools/mobilka@latest @pure-tools/babetka@latest @pure-tools/slushalka@latest`
   - garden: same minus slushalka
   - top-3-in-sports: same but with `--legacy-peer-deps` (TypeScript 6 / typescript-eslint conflict)

4. **Build each project** to verify no breaking changes:
   - `npm run build` and `npm test`
   - If build fails, diagnose and fix before committing.
   - cuefade: `ng build` currently fails on master with 3 `@capacitor/*` resolve errors from mobilka's dynamic imports — pre-existing; compare against master before blaming the upgrade.

5. **Commit and push** each project:
   ```
   git add package.json package-lock.json
   git commit -m "chore: upgrade pure-tools deps to latest"
   git push
   ```

## Notes

- top-3-in-sports always needs `--legacy-peer-deps` due to pre-existing TypeScript 6 vs typescript-eslint peer conflict.
- garden uses `isAuthenticated` (not `isLoggedIn`) on its `AuthService` — the `AUTH_PROVIDER` factory adapter in `app.config.ts` bridges this.
- If a new pure-tools package introduces breaking API changes, check each project's integration points before pushing.
- Run projects in parallel where possible to save time.

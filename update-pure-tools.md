# Update Pure Tools Dependencies

Update all pure-tools consumer projects to the latest versions of `@pure-tools/paletka`, `@pure-tools/monetka`, `@pure-tools/mobilka`, and `@pure-tools/babetka`. Run this after publishing a new version of any of these packages.

## Consumer projects

| Path | Remote |
|------|--------|
| `C:\Work\git\cuefade` | https://github.com/pure-tools/cuefade |
| `C:\Work\git\top-3-in-sports` | https://github.com/pure-tools/top-3-in-sports |
| `C:\Work\git\garden` | https://github.com/pure-tools/garden |

## Steps

For each project:

1. **Check current versions** — run `npm list @pure-tools/paletka @pure-tools/monetka @pure-tools/mobilka @pure-tools/babetka` to see what's installed.

2. **Get latest versions from npm**:
   ```
   npm view @pure-tools/paletka version
   npm view @pure-tools/monetka version
   npm view @pure-tools/mobilka version
   npm view @pure-tools/babetka version
   ```

3. **Install latest** in each project directory:
   - cuefade and garden: `npm install @pure-tools/paletka@latest @pure-tools/monetka@latest @pure-tools/mobilka@latest @pure-tools/babetka@latest`
   - top-3-in-sports: same but with `--legacy-peer-deps` (TypeScript 6 / typescript-eslint conflict)

4. **Build each project** to verify no breaking changes:
   - `npm run build`
   - If build fails, diagnose and fix before committing.

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

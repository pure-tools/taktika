# Publish a Pure-Tools Library

Full release workflow: version bump → build → publish → update all consumers.

## Steps

### 1. Confirm tests pass

```bash
npm test
```

Fix any failures before proceeding.

### 2. Bump version in package.json

Semantic versioning:
- `patch` (0.1.0 → 0.1.1) — bug fix, no API change
- `minor` (0.1.0 → 0.2.0) — new feature, backwards compatible
- `major` (0.1.0 → 1.0.0) — breaking API change

```bash
npm version patch   # or minor / major
```

Or edit `package.json` manually.

### 3. Build

```bash
npm run build
```

Verify no errors. Output goes to `dist/`.

### 4. Publish to npm

```bash
npm publish dist/ --access public
```

If 2FA/token error: ensure automation token is set in `~/.npmrc`:
```
//registry.npmjs.org/:_authToken=npm_xxxxx
```
Token must be **Classic → Automation** type from npmjs.com/settings → Access Tokens.

### 5. Verify on npm

```bash
npm view @pure-tools/<name> version
```

Wait up to 2 minutes for registry propagation.

### 6. Update all consumer projects

→ Run `/update-pure-tools`

### 7. Commit and push the version bump

```bash
git add package.json package-lock.json
git commit -m "chore: release v<version>"
git push
```

## Consumer projects

| Project | Path | Notes |
|---------|------|-------|
| cuefade | `C:\Work\git\cuefade` | standard npm install |
| top-3-in-sports | `C:\Work\git\top-3-in-sports` | needs `--legacy-peer-deps` |
| garden | `C:\Work\git\garden` | standard npm install |

## Breaking change checklist

If bumping major version, before publishing:
- [ ] Document what changed in the commit message
- [ ] Check each consumer's integration point for the changed API
- [ ] Update consumer code before or alongside the publish

# Scaffold a New Pure-Tools Library

Creates all files for a new `@pure-tools/<name>ka` Angular library from scratch.

## Inputs needed

- `NAME` — short name, e.g. `validka`, `chartka`, `formka`
- `DESCRIPTION` — one-line npm description
- `PEER_DEPS` — additional Angular peer deps needed (e.g. `@angular/forms`, `@angular/router`)

---

## Step 1 — Create directory

```bash
mkdir C:\Work\git\<NAME>
cd C:\Work\git\<NAME>
```

## Step 2 — package.json

```json
{
  "name": "@pure-tools/<NAME>",
  "version": "0.1.0",
  "description": "<DESCRIPTION>",
  "scripts": {
    "build": "ng-packagr -p ng-package.json",
    "publish:lib": "npm run build && npm publish dist/ --access public",
    "test": "vitest run",
    "test:watch": "vitest"
  },
  "peerDependencies": {
    "@angular/core": ">=22.0.0"
  },
  "devDependencies": {
    "@angular/compiler": "^22.0.0",
    "@angular/compiler-cli": "^22.0.0",
    "@angular/core": "^22.0.0",
    "@angular/platform-browser-dynamic": "^22.0.0",
    "ng-packagr": "^22.0.0",
    "typescript": "~6.0.0",
    "vitest": "^4.0.0",
    "jsdom": "^27.0.0"
  },
  "keywords": ["angular", "<NAME>"],
  "license": "MIT"
}
```

## Step 3 — ng-package.json

```json
{
  "$schema": "./node_modules/ng-packagr/ng-package.schema.json",
  "lib": { "entryFile": "src/index.ts" }
}
```

## Step 4 — tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "lib": ["ES2022", "dom"],
    "strict": true,
    "experimentalDecorators": true,
    "useDefineForClassFields": false
  }
}
```

## Step 5 — tsconfig.lib.json

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "declaration": true,
    "inlineSources": true,
    "outDir": "../../out-tsc/lib"
  },
  "exclude": ["**/*.spec.ts"]
}
```

## Step 6 — vitest.config.ts

```ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    include: ['src/**/*.spec.ts'],
    environment: 'jsdom',
    environmentOptions: { jsdom: { url: 'http://localhost' } },
    globals: false,
    setupFiles: ['src/test-setup.ts'],
  },
});
```

## Step 7 — src/test-setup.ts

```ts
import { TestBed } from '@angular/core/testing';
import { BrowserDynamicTestingModule, platformBrowserDynamicTesting } from '@angular/platform-browser-dynamic/testing';

TestBed.initTestEnvironment(
  BrowserDynamicTestingModule,
  platformBrowserDynamicTesting(),
  { teardown: { destroyAfterEach: true } },
);
```

## Step 8 — src/provide-\<name\>.ts

```ts
import { InjectionToken, makeEnvironmentProviders, type EnvironmentProviders } from '@angular/core';

export interface <Name>Config {
  // add config options
}

export const <NAME>_CONFIG = new InjectionToken<<Name>Config>('<NAME>_CONFIG');

export const <NAME>_DEFAULTS: Required<<Name>Config> = {
  // defaults
};

export function provide<Name>(config: <Name>Config = {}): EnvironmentProviders {
  return makeEnvironmentProviders([
    { provide: <NAME>_CONFIG, useValue: { ...<NAME>_DEFAULTS, ...config } },
  ]);
}
```

## Step 9 — src/index.ts

```ts
export type { <Name>Config } from './provide-<name>';
export { <NAME>_CONFIG, <NAME>_DEFAULTS, provide<Name> } from './provide-<name>';
// export services, guards, interfaces as added
```

## Step 10 — Install and verify

```bash
npm install
npm test       # should pass (0 tests initially — that's fine)
npm run build  # must succeed before publishing
```

## Step 11 — Init git + GitHub repo

```bash
git init
git add .
git commit -m "feat: initial release of @pure-tools/<NAME> v0.1.0"
gh repo create pure-tools/<NAME> --public
git remote add origin https://github.com/pure-tools/<NAME>.git
git push -u origin master
```

## Step 12 — Publish

→ Follow `/publish-pure-tool`

## Step 13 — Add to consumer list

Update `pure-tools/update-pure-tools.md` consumer projects table with the new package.

---

## Reference

→ `pure-tools/pure-tool-anatomy.md` for service patterns, testability rules, SSR-safe localStorage.

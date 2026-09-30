# Pure Tools Library Anatomy

Reference for building and maintaining a `@pure-tools/xxxka` Angular library.

## Naming convention

`@pure-tools/<name>ka` — short Bulgarian/Slavic diminutive. Examples: `mobilka`, `monetka`, `paletka`, `babetka`.

## Repository structure

```
xxxka/
  src/
    index.ts                  ← public API — only export from here
    provide-xxxka.ts          ← InjectionToken + provideXxxka() factory
    interfaces/
      xxxka.ts                ← public interfaces / types
    services/
      xxx.service.ts
      xxx.service.spec.ts
    guards/                   ← optional
      xxx.guard.ts
  ng-package.json             ← ng-packagr entrypoint
  package.json
  tsconfig.json
  tsconfig.lib.json
  vitest.config.ts
  src/test-setup.ts
```

## package.json shape

```json
{
  "name": "@pure-tools/xxxka",
  "version": "0.1.0",
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
    "ng-packagr": "^22.0.0",
    "typescript": "~6.0.0",
    "vitest": "^4.0.0",
    "jsdom": "^27.0.0"
  },
  "license": "MIT"
}
```

## ng-package.json

```json
{
  "$schema": "./node_modules/ng-packagr/ng-package.schema.json",
  "lib": { "entryFile": "src/index.ts" }
}
```

## provide-xxxka.ts pattern

```ts
import { InjectionToken, makeEnvironmentProviders, type EnvironmentProviders } from '@angular/core';

export interface XxxkaConfig { /* options */ }

export const XXXKA_CONFIG = new InjectionToken<Required<XxxkaConfig>>('XXXKA_CONFIG');

export const XXXKA_DEFAULTS: Required<XxxkaConfig> = { /* defaults */ };

export function provideXxxka(config: XxxkaConfig = {}): EnvironmentProviders {
  return makeEnvironmentProviders([
    { provide: XXXKA_CONFIG, useValue: { ...XXXKA_DEFAULTS, ...config } },
  ]);
}
```

## Service testability rule

**Always use constructor injection (`@Inject()`)**, not field `inject()`, so services can be instantiated with `new MyService(mockDep)` in tests — no TestBed needed.

```ts
// ✓ testable
constructor(
  @Inject(XXXKA_CONFIG) private readonly config: Required<XxxkaConfig>,
) {}

// ✗ requires TestBed
private config = inject(XXXKA_CONFIG);
```

## vitest.config.ts

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

## src/test-setup.ts

```ts
import { TestBed } from '@angular/core/testing';
import { BrowserDynamicTestingModule, platformBrowserDynamicTesting } from '@angular/platform-browser-dynamic/testing';

TestBed.initTestEnvironment(
  BrowserDynamicTestingModule,
  platformBrowserDynamicTesting(),
  { teardown: { destroyAfterEach: true } },
);
```

## localStorage in SSR

Never use `typeof localStorage !== 'undefined'` — passes in Angular prerender when localStorage is a stub without methods. Use try/catch instead:

```ts
// ✗ breaks in prerender
if (typeof localStorage !== 'undefined') localStorage.getItem(key);

// ✓ SSR-safe
try { return localStorage.getItem(key); } catch { return null; }
```

## Consumer integration pattern

In `app.config.ts`:
```ts
import { provideXxxka } from '@pure-tools/xxxka';
// ...
provideXxxka({ /* config */ })
```

## Publish workflow

```bash
# 1. bump version in package.json
# 2. build + publish
npm run publish:lib

# 3. update all consumers
# → run /update-pure-tools skill
```

## Current pure-tools ecosystem

| Package | Purpose |
|---------|---------|
| `@pure-tools/paletka` | CSS var themes, random/persist theme |
| `@pure-tools/mobilka` | Responsive breakpoint signals |
| `@pure-tools/monetka` | Stripe / Lemon Squeezy payments |
| `@pure-tools/babetka` | Auth guards, session timeout, rate limiting, sanitization, AI security |

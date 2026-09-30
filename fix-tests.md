# Fix Failing Angular / Vitest Tests

Playbook for the 6 most common test failures in pure-tools Angular projects.

---

## 1. `Need to call TestBed.initTestEnvironment() first`

**Cause:** TestBed used without Angular test env setup.

**Fix:** Add `src/test-setup.ts` and reference it in `vitest.config.ts`:

```ts
// src/test-setup.ts
import { TestBed } from '@angular/core/testing';
import { BrowserDynamicTestingModule, platformBrowserDynamicTesting } from '@angular/platform-browser-dynamic/testing';
TestBed.initTestEnvironment(BrowserDynamicTestingModule, platformBrowserDynamicTesting(), { teardown: { destroyAfterEach: true } });
```

```ts
// vitest.config.ts
setupFiles: ['src/test-setup.ts']
```

---

## 2. `Cannot configure test module when already instantiated`

**Cause:** Service uses `inject()` field declarations — Angular instantiates it during DI before TestBed is configured.

**Fix:** Convert service to constructor injection (`@Inject()`), then test with `new MyService(mockDep)` — no TestBed at all:

```ts
// ✗ triggers early instantiation
private config = inject(MY_CONFIG);

// ✓ testable without TestBed
constructor(@Inject(MY_CONFIG) private config: MyConfig) {}
```

```ts
// spec — no TestBed needed
service = new MyService(mockDep);
```

---

## 3. `localStorage.clear is not a function` / `localStorage is not defined`

**Cause A:** jsdom missing URL — localStorage requires http/https origin.

**Fix A:** Add to `vitest.config.ts`:
```ts
environmentOptions: { jsdom: { url: 'http://localhost' } }
```

**Cause B:** jsdom stub in SSR/prerender has localStorage object without methods (`typeof` check passes but `.getItem()` fails).

**Fix B:** Wrap all localStorage in try/catch, not `typeof` checks:
```ts
try { return localStorage.getItem(key); } catch { return null; }
```

**Cause C:** Spec calls `localStorage.clear()` directly — mock it instead:
```ts
const store: Record<string, string> = {};
vi.stubGlobal('localStorage', {
  getItem: (k: string) => store[k] ?? null,
  setItem: (k: string, v: string) => { store[k] = v; },
  removeItem: (k: string) => { delete store[k]; },
  clear: () => { for (const k of Object.keys(store)) delete store[k]; },
  get length() { return Object.keys(store).length; },
  key: (i: number) => Object.keys(store)[i] ?? null,
});
```

---

## 4. `No provider found for InjectionToken X`

**Cause:** Test module missing a provider that the component/service under test requires.

**Fix:** Add to `TestBed.configureTestingModule({ providers: [...] })`:
```ts
// babetka tokens
{ provide: AUTH_PROVIDER, useValue: { isLoggedIn: signal(false), signOut: async () => {} } },
{ provide: SECURKA_CONFIG, useValue: SECURKA_DEFAULTS },

// generic token
{ provide: MY_TOKEN, useValue: mockValue },
```

---

## 5. Fake timers: `setTimeout` not advancing

**Cause:** `vi.useFakeTimers()` not called before the code that sets the timer, or `vi.advanceTimersByTime()` called on wrong instance.

**Fix pattern:**
```ts
beforeEach(() => {
  vi.useFakeTimers();
  service = new MyService(...); // create AFTER fake timers
});
afterEach(() => {
  service.stop();         // clear any pending timers
  vi.useRealTimers();
});

it('fires after timeout', () => {
  service.start();
  vi.advanceTimersByTime(5_001);
  expect(mockFn).toHaveBeenCalledOnce();
});
```

---

## 6. `NG0201: No provider found` for Router / ActivatedRoute

**Cause:** Component injects Router but test module has no router.

**Fix:**
```ts
providers: [provideRouter([])]
```

Use `provideRouter([])` not `RouterTestingModule` (deprecated in Angular 22+).

---

## Diagnostic checklist

1. Run `npm test 2>&1` — read the FIRST error only, fix it, re-run.
2. `tsc --noEmit` — catches type errors that vitest swallows.
3. Check if error is in source or spec — source errors mean real bugs, spec errors are usually missing providers or stale types.
4. Pre-existing spec failures? Check git log — if they predate your change, note them and don't fix unless asked.

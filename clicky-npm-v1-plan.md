# Clicky npm v1 API and 4-Week Plan

## v1 npm API Surface

### Package layout

- `@clicky/core`: state machine, event protocol, agent orchestration
- `@clicky/browser`: DOM overlay, voice input, action runner, capture helpers
- `@clicky/react` (optional v1.1): thin hooks/provider wrappers

For a true v1 in 4 weeks, ship as one package first: `@clicky/browser`.

### Primary API

```ts
import { createClickyAgent } from "@clicky/browser";

const clicky = createClickyAgent({
  apiBaseUrl: "https://your-worker.example.com",
  productId: "acme-dashboard",
  environment: "production",
  context: {
    productName: "Acme Dashboard",
    audience: "first-time users",
    goals: ["create first report", "invite teammate"],
    constraints: ["never click Delete", "never submit payment forms"],
  },
  capabilities: {
    voice: true,
    pageVision: true,
    suggestActions: true,
    autoAct: false, // default false for safety
  },
  ui: {
    theme: "dark",
    icon: "clicky",
    position: "bottom-right",
    followMouse: true,
    zIndex: 2147483000,
  },
});
```

### Config types (v1)

```ts
type ClickyConfig = {
  apiBaseUrl: string;
  productId: string;
  environment?: "development" | "staging" | "production";
  user?: {
    id?: string;
    email?: string;
    name?: string;
    traits?: Record<string, string | number | boolean>;
  };
  context?: {
    productName?: string;
    audience?: string;
    goals?: string[];
    glossary?: Record<string, string>;
    constraints?: string[]; // safety/business limits
  };
  capabilities?: {
    voice?: boolean;
    pageVision?: boolean; // DOM snapshot + optional screenshot prompt
    suggestActions?: boolean; // returns actionable suggestions
    autoAct?: boolean; // execute actions directly (guardrailed)
  };
  ui?: {
    theme?: "light" | "dark" | "system";
    icon?: "clicky" | "dot";
    position?: "bottom-right" | "bottom-left";
    followMouse?: boolean;
    zIndex?: number;
  };
  hotkey?: {
    mode?: "hold" | "toggle";
    key?: "Control" | "Alt";
    requireFocus?: boolean; // tab must be focused
  };
};
```

### Lifecycle and controls

```ts
await clicky.init(); // mount UI, wire listeners
clicky.enable(); // user-facing on
clicky.disable(); // user-facing off
clicky.destroy(); // unmount and cleanup

clicky.setContext({
  goals: ["connect data source", "build first chart"],
});

clicky.startListening(); // push-to-talk start
clicky.stopListening(); // push-to-talk stop
clicky.ask("Walk me through creating my first report");
```

### Action model (safe by default)

```ts
type ClickyAction =
  | { type: "highlight"; selector: string; label?: string }
  | { type: "scrollIntoView"; selector: string }
  | { type: "click"; selector: string; requiresConfirm?: boolean }
  | { type: "type"; selector: string; text: string; submit?: boolean };

clicky.executeAction(action); // only if allowed by policy
clicky.setPolicy({
  allowAutoActions: false,
  allowedActionTypes: ["highlight", "scrollIntoView", "click"],
  blockedSelectors: ["[data-danger='true']", "button.delete", "#billing-submit"],
});
```

### Events and observability

```ts
clicky.on("state.change", (state) => {});
clicky.on("transcript.final", (text) => {});
clicky.on("response.chunk", (chunk) => {});
clicky.on("response.final", (message) => {});
clicky.on("action.proposed", (action) => {});
clicky.on("action.executed", (result) => {});
clicky.on("error", (err) => {});
```

Core state:

- `idle | listening | processing | guiding | acting | disabled | error`

### Framework adapters

- Vanilla: single script/init path
- React helper (optional early):
  - `ClickyProvider`
  - `useClicky()`
  - `useClickyState()`

## 4-week plan (MVP -> Beta -> hardening)

## Week 1 - MVP foundation (single-app reliable demo)

Goal: usable guided onboarding in one production app.

- Build `createClickyAgent()` core with state machine + event emitter
- Implement overlay widget + follow-mouse icon + enable/disable toggle
- Add push-to-talk (tab-focused key hold + button fallback)
- Add transcript pipeline -> `/chat` (your worker) -> streaming text response
- Implement basic DOM context capture:
  - route, page title, visible headings/buttons, key `data-*` attributes
- Parse agent response for action suggestions (`highlight`, `scrollIntoView`)
- Ship safety defaults: no auto-click/type yet
- Deliverable: "guide me through feature X" working end-to-end

Exit criteria:

- Works on Chrome + one framework app (for example, Next.js)
- 5 happy-path onboarding prompts complete without crashes

## Week 2 - Beta features (multi-app and actionability)

Goal: move from demo to beta SDK.

- Add policy engine (`allowedActionTypes`, blocked selectors, confirmations)
- Add optional click execution with explicit confirmation UI
- Add `setContext()` runtime updates for app-specific guidance
- Improve element targeting strategy:
  - selector generation hierarchy (`id` -> `data-testid` -> role/name -> fallback)
- Add retries/fallback when element not found
- Add analytics hooks/events (`action.proposed`, `action.executed`, errors)
- Publish pre-release package (`0.8.x-beta`) + quickstart docs + sample app

Exit criteria:

- Works across 2-3 internal test apps
- >=80% proposed highlights/clicks resolve correct targets in scripted scenarios

## Week 3 - Production hardening I (reliability + UX)

Goal: reduce breakage in real user sessions.

- Harden hotkey + mic permission flows and user messaging
- Handle SPA route changes, modals, lazy-rendered UI, stale selectors
- Add "agent stuck" recovery: timeout, cancel, retry, graceful fallback text
- Add accessibility-aware targeting improvements (ARIA labels/roles)
- Add robust error taxonomy + developer debug mode
- Add integration tests (Playwright) for major action flows
- Add performance budget checks (overlay + listeners minimal overhead)

Exit criteria:

- Crash-free in long sessions (30+ min test runs)
- Deterministic tests for top onboarding journeys

## Week 4 - Production hardening II (packaging + launch readiness)

Goal: merge-ready v1.0 package.

- Finalize API shape, typings, and migration-safe defaults
- Finalize docs:
  - quickstart, safety model, policy config, events, troubleshooting
- Add framework examples (vanilla + React)
- Add semantic versioning, changelog, release CI
- Security/privacy review:
  - data sent to backend documented clearly
  - opt-in controls and visible agent activity indicator
- Beta feedback fixes + polish release candidate
- Publish `1.0.0`

Exit criteria:

- External developer can integrate in <30 min from docs
- Stable across Chrome/Edge/Safari baseline target
- Clear guardrails for action execution

## Scope cuts to stay inside 4 weeks

- No full autonomous computer-use browsing outside current site
- No cross-tab/system-wide control
- No visual screenshot CV dependency for v1 (DOM-first is faster/safer)
- No heavy React abstraction in initial release (core browser package first)

## Suggested v1 defaults

- `autoAct = false`
- `suggestActions = true`
- confirmation required for `click` and always for `type`/`submit`
- explicit blocked selector list shipped by default for dangerous UI

---
name: scan-web
description: Scan a live web page with Playwright MCP and generate test cases into a reviewable test-plan spec (test case table + Gherkin + verified locators + page-object plan + test data), then after user approval apply it into the project's test layers (features / steps / pages / locators / data). Triggers on "scan web", "quét web", "scan-web", "generate test cases from URL", "tạo test case từ trang web", "apply test plan".
---

# scan-web — Scan → Spec → Approve → Apply

Two-phase, human-in-the-loop workflow. **Never generate test code directly from a scan** — always produce the spec MD first and wait for user approval.

```
/scan-web <URL> [feature-name]        → Phase 1 (SCAN) + Phase 2 (SPEC), then STOP
/scan-web --apply <spec-file.md>      → Phase 3 (APPLY to layers)
```

Write the spec and all user-facing output in the user's language (Vietnamese for VN teams); keep generated code identifiers in English.

## Phase 1 — SCAN (Playwright MCP)

Requires the `playwright-test` MCP server (bundled with this plugin; the project must have `playwright` installed since the server runs via `npx playwright run-test-mcp-server`).

1. `browser_navigate` to the URL. If the page requires login, read credentials from the project's `.env*` files or ask the user — never hardcode them anywhere.
2. `browser_snapshot` to map the page: forms, inputs, buttons, tables, navigation, dialogs.
3. Explore each interactive flow relevant to the requested feature:
   - Fill forms with obviously-fake data to observe validation messages (empty submit, invalid format, boundary lengths).
   - Follow navigation to discover post-action states (success redirect, error banner).
   - **Never perform destructive or irreversible actions** during scan: no deletes, payments, sends, or submissions that create real records on non-test environments. When unsure, note the flow in the spec as "not exercised during scan".
4. For every element a test will touch, call `browser_generate_locator` and record the result. Prefer role/label-based locators over CSS.
5. Collect observed behaviors: validation messages (exact text), redirects, toasts, table columns, default states.

## Phase 2 — SPEC (output: `specs/<feature>-testplan.md`)

Write one MD file per feature into the project's `specs/` directory (create it if missing). Structure:

```markdown
# Test Plan — <Feature> (<URL>)
status: draft   <!-- user flips to approved -->

## 1. Overview
Scope, environment, preconditions, out-of-scope.

## 2. Test Cases
| ID | Title | Priority | Technique | Steps (summary) | Expected |
|----|-------|----------|-----------|-----------------|----------|
TC IDs: <FEAT>-NNN. Techniques: EP / BVA / Decision Table / State Transition.
Cover happy path + each validation rule observed in Phase 1 + edge cases.

## 3. Gherkin (draft)
One ```gherkin block per scenario. Natural language only —
NO credentials, NO URLs, NO selectors. Data-driven cases use
Scenario Outline with Examples referencing test-data files.

## 4. Locators (verified from scan)
| Element | Description | Locator | Verified |
Locators exactly as returned by browser_generate_locator.

## 5. Page Object plan
Class name, file path, method list (name, params, what it does).
Methods wrap UI interaction only — no assertions about business data.

## 6. Steps mapping
| Gherkin step | Page Object call |

## 7. Test Data
Tables of valid/invalid datasets → target CSV/JSON file paths.
Fake data only; real credentials stay in .env.

## 8. Open questions
Anything not verifiable during scan.
```

End Phase 2 by telling the user: review/edit the spec, then run `/scan-web --apply specs/<file>.md`. **Do not proceed to Phase 3 in the same run.**

## Phase 3 — APPLY (after approval)

1. Read the approved spec. If `status: draft`, warn the user and ask for confirmation before continuing.
2. **Detect the project's test convention — never impose one:**
   - `playwright-bdd` or `@cucumber/cucumber` in package.json → 4-layer BDD layout (see below).
   - Plain `@playwright/test` → `tests/pages/*.ts` + `tests/*.spec.ts` (POM, no Gherkin — convert scenarios to `test()` blocks).
   - Empty project → scaffold the 4-layer BDD layout.
3. **Read one existing file of each layer first** (feature, steps, page, locator) and copy its style: import paths, fixture usage, naming, locator registry pattern.
4. Generate, honoring the layer contract:

| Layer | Path (adapt to project) | Contract |
|---|---|---|
| Feature | `tests/features/<N>.<Feature>/<feature>.feature` | Natural language only. No credentials/URLs/selectors. |
| Steps | `tests/steps/<Feature>/<feature>.steps.ts` | Bridge Gherkin → Page Object. No UI logic, no selectors. |
| Page Object | `tests/pages/<Feature>/<Feature>Page.ts` | All UI interaction. Selectors only via the locator layer. |
| Locators | `tests/Locators/AppLocator.ts` (append) or project's registry | Single source of selectors, from spec §4. |
| Data | `tests/data/<feature>/*.csv` | From spec §7. Fake data only. |

5. Verify: run the project's generation step if any (`npm run bddgen`), then run the new tests (`npx playwright test --grep <tag>`). If a locator fails, re-verify it with `browser_generate_locator` and fix — update both the code and the spec so they stay in sync.
6. Report: files created, test results, any spec items skipped and why.

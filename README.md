# qassist-skills

A [Claude Code](https://claude.com/claude-code) plugin marketplace for QA automation on Playwright projects.

## Install

Inside any Claude Code session:

```
/plugin marketplace add kyodanh/qassist-skills
/plugin install scan-web@qassist-skills
```

Then open Claude Code in your Playwright test project and use the commands below.

## Plugins

### scan-web — Scan → Spec → Approve → Apply

Scans a live web page with Playwright MCP, generates a **reviewable test-plan spec** (test case table + Gherkin + verified locators + page-object plan + test data), and only after you approve the spec does it generate code into your project's test layers.

```
/scan-web https://your-app.example.com/login login     # Phase 1+2: scan + write specs/login-testplan.md
# ... review & edit the spec, set status: approved ...
/scan-web --apply specs/login-testplan.md              # Phase 3: generate feature/steps/pages/locators/data
```

Human-in-the-loop by design: it never generates test code directly from a scan, never performs destructive actions (deletes, payments, real submissions) while scanning, and never hardcodes credentials — logins come from your `.env` files.

Layers produced (adapts to the target project's existing conventions instead of imposing its own):

```
tests/features/   Gherkin — BA/QA readable, no selectors/credentials
tests/steps/      Gherkin → Page Object bridge
tests/pages/      Page Objects — all UI interaction
tests/Locators/   Central selector registry
tests/data/       CSV test data (fake data only)
```

**Requirements** in the target project: `playwright` installed — the bundled MCP server runs via `npx playwright run-test-mcp-server`. Works best with `playwright-bdd`; falls back to plain `@playwright/test` POM style (scenarios become `test()` blocks).

## Local development

Try a plugin without installing it:

```bash
claude --plugin-dir ./plugins/scan-web
```

Edit `plugins/scan-web/skills/scan-web/SKILL.md`, restart the session, and the changes take effect.

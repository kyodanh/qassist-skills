# qassist-skills

Claude Code plugin marketplace for QA automation.

## Install

```
/plugin marketplace add <github-owner>/qassist-skills
/plugin install scan-web@qassist-skills
```

## Plugins

### scan-web

Scan a live web page → generate a reviewable test plan → apply into test layers after approval.

```
/scan-web https://your-app.example.com/login login     # scan + write specs/login-testplan.md
# ... review & edit the spec, set status: approved ...
/scan-web --apply specs/login-testplan.md              # generate feature/steps/pages/locators/data
```

Layers produced (adapts to the target project's conventions):

```
tests/features/   Gherkin — BA/QA readable, no selectors/credentials
tests/steps/      Gherkin → Page Object bridge
tests/pages/      Page Objects — all UI interaction
tests/Locators/   Central selector registry
tests/data/       CSV test data
```

Requirements in the target project: `playwright` installed (the bundled MCP server runs `npx playwright run-test-mcp-server`). Works best with `playwright-bdd`; falls back to plain `@playwright/test` POM style.

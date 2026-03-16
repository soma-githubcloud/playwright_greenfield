# SQE Kit — Synthetic Quality Engineer

An AI-powered quality engineering system built on **Claude Code**. It covers the full quality
lifecycle — from raw requirement documents all the way to runnable, production-ready test
automation artifacts — across three self-contained capability modules.

---

## Three Capabilities

```
Capability 1 — req-to-story/         Requirement Doc  →  User Stories
Capability 2 — story-to-testcase/    User Story       →  Manual Test Cases
Capability 3 — tc-to-automate/       Test Cases       →  Automation Artifacts
```

**Start at any point** in the pipeline — or chain all three together:

```
Req Doc
  → /gen-user-stories
      → User Stories
          → /gen-manual-tests
              → Manual TCs (BDD .feature or non-BDD .md)
                  → /generate-automation-tests
                      → POMs + Spec Files + BDD Artifacts + CI Pipeline
```

Or run the full chain in one command:
```
/e2e --req inputs/req-docs/prd.pdf --url https://staging.myapp.com
```

---

## What Gets Generated

| Artifact | Location |
|----------|----------|
| User story files (`US-NNN.md`) | `outputs/<project>/user-stories/` |
| Manual TC documents (BDD `.feature` or non-BDD `.md`) | `outputs/<project>/manual-tests/manual-tcs/` |
| Page Object Model classes | `outputs/<project>/src/pages/` |
| Playwright spec files (`*.spec.ts`) | `outputs/<project>/tests/nonbdd/` |
| Gherkin feature files + step definitions | `outputs/<project>/tests/bdd/` |
| Test data fixtures | `outputs/<project>/src/fixtures/` |
| `playwright.config.ts` + `package.json` + `tsconfig.json` | `outputs/<project>/` |
| GitHub Actions CI pipeline | `outputs/<project>/configs/.github/workflows/ui.yml` |
| Allure + Playwright HTML reports | `outputs/<project>/allure-report/` |

---

## Quick Start

### Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Node.js | 18.x LTS+ | [nodejs.org](https://nodejs.org) |
| Git | 2.x+ | [git-scm.com](https://git-scm.com) |
| Claude Code CLI | Latest | `npm install -g @anthropic/claude-code` |
| Java | 8+ | Required for Allure CLI only |

### 1 — Clone and open

```bash
git clone <repo-url> sqe-kit
cd sqe-kit
claude        # or open in VS Code with the Claude Code extension
```

### 2 — Scaffold the project (first time only)

```
/setup-kit
```

Scaffolds the output project structure, installs Playwright dependencies, and validates the kit.

### 3 — Generate your first test

```
# From a requirement document (full chain)
/e2e --req inputs/req-docs/your-spec.pdf --url https://staging.myapp.com

# From a user story
/e2e --story inputs/user-stories/your-story.md --url https://staging.myapp.com

# From ready-made test cases
/generate-automation-tests --url https://practice.expandtesting.com/login --cases "TC-01: Successful Login
Steps:
  1. Navigate to the login page
  2. Enter username: practice
  3. Enter password: SuperSecretPassword!
  4. Click Login
Assertions:
  - URL changes to /secure
  - Flash message contains 'You logged into a secure area!'"
```

### 4 — Run tests

```bash
# Via kit commands (runs both Playwright specs + Cucumber BDD)
/run-smoke          # @smoke tagged tests only

# Directly
cd outputs/sqe-Functional-TestDesignkit-playwright-gf
npx playwright test                  # Playwright specs
npx cucumber-js                      # BDD feature files
npx allure open allure-report        # View Allure report
```

---

## Project Structure

```
sqe-kit/
├── kit.config.json               ← Active kit configuration (kit-u1: Playwright TypeScript)
├── CLAUDE.md                     ← Orchestrator — all routing + quality rules
│
├── req-to-story/                 ← Capability 1 (kit-independent)
│   ├── agents/                   ← req-ingestor, req-analyzer, feature-splitter, user-story-writer
│   ├── skills/                   ← req-text-extractor
│   └── rules/                    ← user-story-quality
│
├── story-to-testcase/            ← Capability 2 (kit-independent)
│   ├── agents/                   ← quality-master-orchestrator, 8 specialist agents, gap-filler
│   ├── skills/                   ← user-story-parser, tc-formatter-bdd, tc-formatter-nonbdd
│   └── rules/                    ← manual-tc-quality
│
├── tc-to-automate/               ← Capability 3 (kit-aware)
│   ├── kits/
│   │   ├── _base/agents/         ← intake, locator, pom, testgen, bdd, datagen, ci, reporting
│   │   └── kit-u1/               ← Playwright TypeScript (KIT.md + templates)
│   ├── configs/integrations.yml  ← Plugins: Jira, GitHub, Allure Server
│   └── rules/                    ← playwright-ts, page-objects, bdd-gherkin
│
├── inputs/
│   ├── req-docs/                 ← Place requirement documents here
│   ├── user-stories/             ← Place user story files here
│   ├── snapshots/                ← Place HTML DOM snapshots here (Tier 2 locators)
│   └── test-cases/               ← Place test case files here
│
├── plugins/                      ← github.md, jira.md, allure-server.md
│
├── docs/
│   ├── TECHNICAL-REFERENCE.md   ← Full technical reference for kit developers
│   └── USER-GUIDE.md            ← Step-by-step user guide
│
└── outputs/
    └── sqe-Functional-TestDesignkit-playwright-gf/   ← All generated artifacts
```

---

## Command Reference

| Command | Capability | What it does |
|---------|-----------|--------------|
| `/gen-user-stories` | 1 | Requirement doc → `US-NNN.md` user story files |
| `/gen-manual-tests` | 2 | User story → manual TCs (BDD `.feature` or non-BDD `.md`) |
| `/e2e` | 1 + 2 + 3 | Req doc OR user story → manual TCs → automation artifacts |
| `/generate-automation-tests` | 3 | Test cases → full automation artifact set |
| `/generate-locators` | 3 | Extract locators → `selectors.json` only |
| `/generate-page-objects` | 3 | Build POM files from existing `selectors.json` |
| `/run-smoke` | — | Run `@smoke` tests (Playwright + Cucumber) + generate reports |
| `/setup-kit` | — | Scaffold project structure + install + validate |

---

## Locator Strategy Tiers

The kit selects locators using three tiers in priority order:

| Tier | Strategy | Trigger | Score cap |
|------|----------|---------|-----------|
| 1 | Playwright MCP (live browser) | `--url` provided | None — up to 1.0 |
| 2 | DOM snapshot parsing | `--dom` provided | None — up to 1.0 |
| 3 | AI-guided inference | Neither `--url` nor `--dom` | 0.75 — always verify |

Always use Tier 1 or 2 when possible. Tier 3 locators are flagged in `selectors-lint.html`
and must be verified against the real application before running in CI.

---

## Active Kit

This kit uses **kit-u1** — Playwright + TypeScript with Playwright MCP and optional Cucumber BDD.

The configuration is in `kit.config.json` at the project root. To reconfigure (test style,
project name, output directory), run `/setup-kit`.

---

## Supported Input Formats

### Requirement Documents (Capability 1)
`.pdf`, `.md`, `.txt`, `.docx`, `.xlsx`, `.yaml`, `.json`, `.xml`, `.bpmn`, `.csv`, `.eml`, `.png`, `.jpg`

### User Stories (Capability 2)
`.md`, `.txt`, `.pdf`, `.yml` — or inline text in any of these formats:
- Agile: `As a <role>, I want <goal>, so that <benefit>`
- Plain English: `# Feature: X` + description paragraphs
- Structured: `## Feature` + `**Acceptance Criteria**` sections

### Test Cases (Capability 3)
Plain-English numbered steps, Gherkin-like syntax, OpenAPI spec (JSON), Postman collection (JSON), or `.md` files

---

## Quality Rules (Non-Negotiable)

- Locators are never inline in spec files — always in Page Object classes
- BDD artifacts are always paired (`.feature` + `.steps.ts` in same run)
- `waitForTimeout()` is forbidden — use `expect(locator).toBeVisible()`
- Every BDD scenario must have `@TC-NNN`, `@ui`/`@api`, and `@smoke`/`@regression` tags
- Selector file is always `selectors.json` — never `locators.json`
- Generation is idempotent — existing files are never overwritten without confirmation

---

## Documentation

| Document | Contents |
|----------|----------|
| [docs/USER-GUIDE.md](docs/USER-GUIDE.md) | Step-by-step guide for first-time users |
| [docs/TECHNICAL-REFERENCE.md](docs/TECHNICAL-REFERENCE.md) | Architecture, agents, skills, rules, config reference |

---

*Kit version: 2.1 | Active kit: kit-u1 (Playwright TypeScript) | Last updated: 2026-03-16*

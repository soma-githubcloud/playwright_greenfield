# SQE (Synthetic Quality Engineer) — User Guide

> **Who this guide is for**: Anyone cloning this repository and using the kit for the first time,
> including users who have never installed Node.js, Playwright, or Claude Code.

---

## Table of Contents

1. [What Is This Kit?](#1-what-is-this-kit)
2. [Prerequisites](#2-prerequisites)
3. [Clone & First-Time Setup](#3-clone--first-time-setup)
4. [Configure Claude Code](#4-configure-claude-code)
5. [Project Structure at a Glance](#5-project-structure-at-a-glance)
6. [Quick-Start: Your First Test in 5 Minutes](#6-quick-start-your-first-test-in-5-minutes)
6b. [Quick-Start: From Requirement Doc to Tests](#6b-quick-start-from-requirement-doc-to-tests)
7. [Command Reference](#7-command-reference)
8. [Working with DOM Snapshots](#8-working-with-dom-snapshots)
9. [Working with Test Cases](#9-working-with-test-cases)
10. [Locator Strategy — Which to Choose](#10-locator-strategy--which-to-choose)
11. [Generating BDD Tests](#11-generating-bdd-tests)
12. [Running Tests & Viewing Reports](#12-running-tests--viewing-reports)
13. [Configuring Plugins & Pushing to GitHub](#13-configuring-plugins)
14. [Troubleshooting & FAQ](#14-troubleshooting--faq)

---

## 1. What Is This Kit?

The SQE Kit is an AI-powered quality engineering scaffold that accepts requirement documents,
user stories, or ready-made test cases and generates production-ready Playwright TypeScript
tests — including manual TC documents, Page Object Models, spec files, BDD feature files,
and CI pipelines.

You describe **what** to test (or provide a requirement document). The kit figures out **how** to test it.

**Three capabilities — start from wherever you are in the quality lifecycle:**

| Capability | Input | Command | Output |
|-----------|-------|---------|--------|
| **1 — Req-to-Story** | Requirement doc (BRD, PRD, Feature Spec, Excel, Word, image, meeting notes) | `/gen-user-stories` | `US-NNN.md` user story files |
| **2 — Story-to-Testcase** | User story (`.md`, `.txt`, `.pdf`, `.yml`, inline text) | `/gen-manual-tests` or `/e2e --story` | Manual TCs (BDD `.feature` or non-BDD `.md`) → optionally automation |
| **3 — TC-to-Automate** | Test cases or scenarios (inline text, `.md` file) | `/generate-automation-tests` | POMs + spec files + BDD artifacts |

**You can start at any capability** — or chain them together:
```
Req Doc → /gen-user-stories → User Stories → /gen-manual-tests → Manual TCs → /generate-automation-tests → Automation
```
Or run the full chain in one command:
```
/e2e --req inputs/req-docs/prd.pdf --url https://staging.myapp.com
```

**What gets generated for you:**
- `manual-tests/manual-tcs/` — Manual test case documents (BDD or non-BDD format)
- `src/pages/` — Page Object Model (POM) classes with correct locators
- `tests/nonbdd/` — Playwright `test()` spec files
- `tests/bdd/` — Gherkin `.feature` files + step definitions (optional)
- `playwright.config.ts` — Ready-to-run config with Allure reporting
- `.github/workflows/ui.yml` — GitHub Actions CI pipeline (optional)

---

## 2. Prerequisites

Install these tools **before** cloning. The table below includes direct download links and the
minimum versions required.

| Tool | Minimum Version | Why It's Needed | Install |
|---|---|---|---|
| **Node.js** | 18.x LTS | Runs Playwright and all TypeScript tooling | [nodejs.org](https://nodejs.org) — download "LTS" |
| **npm** | 9.x (bundled with Node.js) | Package manager | Comes with Node.js |
| **Git** | 2.x | Clone the repository | [git-scm.com](https://git-scm.com) |
| **Claude Code CLI** | Latest | AI-powered generation engine | See §4 below |
| **VS Code** (recommended) | Any recent | Editor with Claude Code extension | [code.visualstudio.com](https://code.visualstudio.com) |

### Verify your installs

Open a terminal and run:

```bash
node --version    # should print v18.x or higher
npm --version     # should print 9.x or higher
git --version     # should print 2.x or higher
```

If any command fails, install the missing tool before continuing.

---

## 3. Clone & First-Time Setup

### Step 1 — Clone the repository

```bash
git clone <repository-url> sqe-kit
cd sqe-kit
```

> Replace `<repository-url>` with the actual URL provided by your team.

### Step 2 — Install root-level dependencies (if any)

The kit itself has no npm dependencies at the root level. Dependencies are installed **inside**
each generated project. Skip this step unless a `package.json` exists at the root.

### Step 3 — Verify the kit configuration

```bash
cat kit.config.json
```

You should see `"id": "kit-u1"` and `"projectName": "sqe-Functional-TestDesignkit-playwright-gf"`. This is the
active kit configuration.

### Step 4 — Run `/setup-kit` ⬅ REQUIRED before first use

> **This step is mandatory.** `/setup-kit` scaffolds the output project folder, installs
> Playwright and all dependencies, and validates the kit configuration. Nothing else works
> until this has been run at least once on a new machine or a fresh clone.

Open Claude Code (see §4) and type in the chat:

```
/setup-kit
```

What it does:
- Creates `outputs/sqe-Functional-TestDesignkit-playwright-gf/` with the full folder structure
- Writes `playwright.config.ts`, `package.json`, `tsconfig.json`
- Runs `npm install` and `npx playwright install chromium`
- Runs validation checks (TypeScript compile + test discovery)
- Confirms: `"Kit kit-u1 scaffolded and validated successfully"`

You only need to run `/setup-kit` **once per machine / per fresh clone**. After that, go
straight to `/generate-automation-tests` or `/e2e` for all subsequent work.

### Step 5 — Check the snapshots directory (optional)

If your team has provided HTML snapshots of the application:

```bash
ls inputs/snapshots/
```

You should see `.html` files named after the pages they represent (e.g., `loginPage.html`,
`homePage.html`). If this directory is empty or does not exist, you will use AI-guided mode
instead (see §10).

---

## 4. Configure Claude Code

Claude Code is the AI engine that powers all generation commands.

### Install Claude Code CLI

```bash
npm install -g @anthropic/claude-code
```

### Authenticate

```bash
claude login
```

Follow the browser prompt to authenticate with your Anthropic account. A valid API key is
required. If your team uses a shared API key, set it as an environment variable instead:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

### Open the project in Claude Code

```bash
# Option A — VS Code extension (recommended)
code .
# Then open the Claude Code panel from the sidebar or press Ctrl+Shift+P → "Claude Code"

# Option B — CLI mode
claude
```

### Confirm the kit is active

At the start of any Claude Code session, type anything and look for the startup confirmation line:

```
Active kit: kit-u1 | playwright | both
```

If you do not see this line, ask Claude: _"What is the active kit?"_

---

## 5. Project Structure at a Glance

```
sqe-kit/
├── kit.config.json               ← Active kit configuration (kit-u1: Playwright TypeScript)
├── CLAUDE.md                     ← Orchestrator rules (do not edit unless extending the kit)
│
├── req-to-story/                 ← Capability 1: Req Doc → User Stories (kit-independent)
│   ├── agents/                   ← req-ingestor, req-analyzer, feature-splitter, user-story-writer
│   ├── skills/                   ← req-text-extractor (format detection + extraction)
│   └── rules/                    ← user-story-quality rules
│
├── story-to-testcase/            ← Capability 2: User Story → Manual TCs (kit-independent)
│   ├── agents/                   ← quality-master-orchestrator, 8 specialist agents, gap-filler
│   ├── skills/                   ← user-story-parser, tc-formatter-bdd, tc-formatter-nonbdd
│   └── rules/                    ← manual-tc-quality rules
│
├── tc-to-automate/               ← Capability 3: Test Cases → Automation (kit-aware)
│   ├── kits/
│   │   ├── _base/agents/         ← Shared agents: intake, locator, pom, testgen, bdd, etc.
│   │   └── kit-u1/               ← Playwright TypeScript kit (KIT.md + templates)
│   ├── configs/
│   │   └── integrations.yml      ← Plugin configuration (Jira, GitHub, Allure Server)
│   └── rules/                    ← Code generation rules (playwright-ts, page-objects, bdd-gherkin)
│
├── inputs/
│   ├── req-docs/                 ← PUT REQUIREMENT DOCUMENTS HERE (for /gen-user-stories, /e2e --req)
│   ├── user-stories/             ← PUT USER STORY FILES HERE (for /gen-manual-tests, /e2e --story)
│   ├── snapshots/                ← PUT HTML SNAPSHOTS HERE (for Tier 2 locator extraction)
│   └── test-cases/               ← PUT TEST CASE FILES HERE (for /generate-automation-tests)
│
└── outputs/
    └── sqe-Functional-TestDesignkit-playwright-gf/   ← All generated files go here
        ├── user-stories/         ← Capability 1 output (US-NNN.md + US-INDEX.md)
        ├── manual-tests/         ← Capability 2 output
        │   ├── manual-tcs/bdd/   ← Final BDD .feature manual TCs
        │   └── manual-tcs/nonbdd/ ← Final non-BDD TC documents
        ├── intake.summary.json   ← Capability 3 input contract
        ├── selectors.json        ← Selector map
        ├── src/pages/            ← Page Object Model classes
        ├── tests/nonbdd/         ← Playwright spec files
        ├── tests/bdd/            ← Feature files + step definitions
        └── playwright.config.ts  ← Test runner config
```

---

## 6. Quick-Start: Your First Test in 5 Minutes

This walkthrough generates a login test for `practice.expandtesting.com` — no DOM snapshot
required (AI-guided mode).

> **Before you begin**: If this is a fresh clone or a new machine, you must run `/setup-kit`
> first (§3 Step 4). If you have already done that, skip straight to Step 2 below.

### Step 1 — Run `/setup-kit` (first time only)

In the Claude Code chat:

```
/setup-kit
```

Wait for: `"Kit kit-u1 scaffolded and validated successfully"` — then continue.

### Step 2 — Open Claude Code and confirm the kit

```
Active kit: kit-u1 | playwright | both
```

### Step 3 — Run the generate command

In the Claude Code chat, type:

```
/generate-automation-tests --cases "TC-01: Successful Login (Happy Path)
Area: Authentication
Preconditions: User account is available at practice.expandtesting.com
Steps:
  1. Navigate to the login page
  2. Enter valid username: practice
  3. Enter valid password: SuperSecretPassword!
  4. Click Login
  Assertions:
  - URL changes to /secure
  - A message containing 'Logged in as' is visible
  - No error messages are displayed"
```

### Step 4 — Watch the pipeline run

Claude Code will:
1. Parse your test case (intake-agent)
2. Infer locators from the step text (AI-guided, scores ≤ 0.75)
3. Generate `LoginPage.ts` and `SecureAreaPage.ts`
4. Generate `TC-01-successful-login.spec.ts`
5. Configure reporting

You will see a summary table at the end listing all created files.

### Step 5 — Install generated project dependencies

```bash
cd outputs/sqe-Functional-TestDesignkit-playwright-gf
npm install
npx playwright install chromium
```

### Step 6 — Run the tests

```bash
npx playwright test
```

### Step 7 — View the report

```bash
npx playwright show-report
```

A browser window opens showing the Playwright HTML report.

---

## 6b. Quick-Start: From Requirement Doc to Tests

This walkthrough shows the full pipeline: requirement document → user stories → manual TCs → automation.

### Step 1 — Place your requirement document in `inputs/req-docs/`

Supported formats: `.pdf`, `.docx`, `.xlsx`, `.md`, `.txt`, `.yaml`, `.json`, `.xml`, `.png`, `.jpg`

```bash
cp ~/Downloads/product-requirements.pdf inputs/req-docs/
```

### Step 2 — Run the full pipeline with `/e2e --req`

```
/e2e --req inputs/req-docs/product-requirements.pdf --url https://staging.myapp.com
```

This runs the complete chain in one command:
1. `req-ingestor` → detects format, extracts text
2. `req-analyzer` → classifies doc, identifies actors, NFRs, scope
3. `feature-splitter` → splits into discrete features
4. `user-story-writer` → generates `US-NNN.md` per feature
5. For each user story: full manual TC generation (5-phase pipeline)
6. For each user story: automation artifacts (POMs + specs + BDD)

### Step 2 (alternative) — Generate user stories only first

```
/gen-user-stories --req inputs/req-docs/product-requirements.pdf
```

Then review the generated stories:

```
outputs/sqe-Functional-TestDesignkit-playwright-gf/user-stories/
├── US-001-user-registration.md
├── US-002-profile-management.md
├── US-003-payment-flow.md
└── US-INDEX.md
```

Then feed stories into the next capability:

```bash
# Generate manual TCs for one story
/gen-manual-tests --story outputs/sqe-Functional-TestDesignkit-playwright-gf/user-stories/US-001-user-registration.md

# Or full pipeline for all stories (manual TCs + automation)
/e2e --req inputs/req-docs/product-requirements.pdf --url https://staging.myapp.com

# Or manual TCs only for all stories
/gen-user-stories --req inputs/req-docs/product-requirements.pdf --then-manual-tests
```

---

## 7. Command Reference

All commands are typed in the Claude Code chat panel. Commands starting with `/` are **slash
commands** that trigger automated generation pipelines.

### Command Overview

| Command | Capability | Input | Output |
|---------|-----------|-------|--------|
| `/gen-user-stories` | 1 — Req-to-Story | Requirement doc | `US-NNN.md` user story files |
| `/gen-manual-tests` | 2 — Story-to-Testcase | User story | Manual TCs only (BDD `.feature` or non-BDD `.md`) |
| `/e2e` | 1 + 2 + 3 | Req doc (`--req`) OR user story (`--story`) | User stories → manual TCs → automation artifacts |
| `/generate-automation-tests` | 3 — TC-to-Automate | Test cases or user story | Automation artifacts (POMs + specs) |
| `/generate-locators` | 3 | URL or DOM | `selectors.json` only |
| `/generate-page-objects` | 3 | `selectors.json` | POM files only |
| `/run-smoke` | — | — | Runs `@smoke` tests (Playwright + Cucumber) + report |
| `/setup-kit` | — | — | Scaffolds project structure + validates |

---

### `/gen-user-stories` — Generate user stories from requirement documents

The leftmost step in the quality pipeline. Accepts any requirement document and produces
structured `US-NNN.md` user story files ready for `/gen-manual-tests` or `/e2e --story`.

```bash
# Single PDF (BRD or PRD)
/gen-user-stories --req docs/requirements/prd-v2.pdf

# Feature specification in markdown
/gen-user-stories --req docs/features/checkout-feature.md --type feature

# Multiple files combined into one analysis
/gen-user-stories --req "docs/brd.pdf,docs/api-contract.yaml"

# Entire requirements directory
/gen-user-stories --req docs/requirements/

# Word document (auto-converts via pandoc)
/gen-user-stories --req docs/feature-spec.docx

# Whiteboard photo or UX wireframe screenshot
/gen-user-stories --req whiteboard-session.png --type meeting

# Preview features without writing any files
/gen-user-stories --req docs/prd.pdf --dry-run

# Generate stories then immediately generate manual TCs for each
/gen-user-stories --req docs/prd.pdf --then-manual-tests --style both
```

**Supported file formats**:

| Format | How it's read |
|--------|--------------|
| `.pdf`, `.md`, `.txt`, `.eml` | Native — no conversion |
| `.yaml`, `.json`, `.xml`, `.bpmn`, `.csv` | Native — parsed as structured text |
| `.png`, `.jpg` | Native — Claude analyses image visually (whiteboard, wireframe, scanned form) |
| `.docx` | Converted via `pandoc` (auto-installed if missing) |
| `.xlsx` | Converted via `pandas` (auto-installed if missing) |
| `.fig` | Not supported — export as PDF or PNG from Figma first |

**Output location**: `outputs/<project>/user-stories/`

Each generated `US-NNN-<slug>.md` can be passed directly to:
```bash
/gen-manual-tests --story outputs/<project>/user-stories/US-001-<slug>.md
/e2e --story outputs/<project>/user-stories/US-001-<slug>.md --url https://staging.myapp.com
```

---

### `/gen-manual-tests` — Generate manual test cases from a user story

Runs the full 5-phase pipeline: analyze → generate → evaluate → gap-fill → format.
**Does NOT generate automation artifacts.** Use `/e2e --story` if you also need automation.

```bash
# Full workflow — BDD + non-BDD (default)
/gen-manual-tests --story user-stories/checkout.md

# Non-BDD format (TC-ID / Steps / Expected Results)
/gen-manual-tests --story user-stories/login.md --style nonbdd

# Both BDD and non-BDD
/gen-manual-tests --story user-stories/payment.md --style both

# Quick mode (skip quality evaluation)
/gen-manual-tests --story user-stories/auth.md --mode 2

# Specific test types only
/gen-manual-tests --story user-stories/auth.md --types positive,negative,security

# Preview what would be generated
/gen-manual-tests --story user-stories/checkout.md --dry-run

# Custom project name
/gen-manual-tests --story user-stories/checkout.md --project ecommerce-app
```

**Supported input formats for `--story`**: `.md`, `.txt`, `.pdf`, `.yml`/`.yaml`, or inline text.

**Output location**: `outputs/<project>/manual-tests/manual-tcs/`

---

### `/e2e` — Full pipeline: req doc OR user story → manual TCs → automation

The most powerful command. Runs the complete quality pipeline from input to runnable tests.

```bash
# ── From a requirement document ─────────────────────────────────────────────
# Full pipeline (req → stories → manual TCs → automation)
/e2e --req inputs/req-docs/prd.pdf --url https://staging.myapp.com

# Req doc → stories → manual TCs only (no automation)
/e2e --req inputs/req-docs/prd.pdf --skip-automation

# Req doc with doc-type hint
/e2e --req inputs/req-docs/feature-spec.docx --type feature --skip-automation

# ── From a user story ───────────────────────────────────────────────────────
# Full pipeline (user story → manual TCs → automation)
/e2e --story user-stories/checkout.md --url https://app.example.com

# User story → manual TCs only
/e2e --story user-stories/checkout.md --skip-automation

# Both BDD and non-BDD formats
/e2e --story user-stories/login.md --style both --url https://app.example.com

# Inline user story
/e2e --story "As a shopper I want to add items to cart so that I can purchase them" --skip-automation

# Quick mode
/e2e --story user-stories/checkout.md --mode 2 --skip-automation
```

**Supported input formats for `--req`**: all requirement doc formats (`.pdf`, `.docx`, `.xlsx`, `.md`, etc.)
**Supported input formats for `--story`**: `.md`, `.txt`, `.pdf`, `.yml`/`.yaml`, or inline text.

---

### `/generate-automation-tests` — Generate automation artifacts

Generates the full artifact set: locators → POMs → specs → (optionally) BDD.
Accepts either **test cases** (`--cases`) or a **user story** (`--story`).

```bash
# Input Type 2 — test cases directly
/generate-automation-tests --cases "TC-01: ..."
/generate-automation-tests --dom inputs/snapshots/ --cases "TC-01: ..."
/generate-automation-tests --url https://myapp.com/login --cases "TC-01: ..."
/generate-automation-tests --cases path/to/testcases.md --tags smoke

# Input Type 1 — user story first, then automation
/generate-automation-tests --story user-stories/checkout.md --url https://app.example.com
/generate-automation-tests --story user-stories/login.md --style both --dom inputs/snapshots/

# Include BDD feature + step files
/generate-automation-tests --dom inputs/snapshots/ --cases "TC-01: ..." --bdd

# Preview files without writing anything
/generate-automation-tests --dom inputs/snapshots/ --cases "TC-01: ..." --dry-run
```

**What gets generated:**

| Input | Locator Quality | Output |
|---|---|---|
| `--url` only | Highest (live DOM, scores up to 1.0) | POMs + specs + BDD |
| `--dom` + `--cases` | High (real HTML, scores up to 1.0) | POMs + specs only |
| `--dom` + `--cases` + `--bdd` | High | POMs + specs + BDD |
| `--cases` only | Inferred (AI, scores ≤ 0.75) | POMs + specs only |

---

### `/generate-locators` — Extract locators only

Produces or updates `selectors.json` without touching POMs or specs.

```bash
/generate-locators --dom inputs/snapshots/
/generate-locators --url https://myapp.com/login
```

---

### `/generate-page-objects` — Regenerate POMs from existing selectors

Use after manually editing `selectors.json`.

```bash
/generate-page-objects
```

---

### `/run-smoke` — Run smoke tests

```bash
/run-smoke
/run-smoke --headed          # Run with browser visible
/run-smoke --browser firefox # Use Firefox instead of Chromium
/run-smoke --report-only     # Regenerate reports without re-running tests
```

---

### `/setup-kit` — Initialize or re-initialize the kit

Scaffolds the project directory, installs dependencies, and runs validation. Also run this if
you change `kit.config.json` settings.

```bash
/setup-kit
```

Validation is built into setup — no separate validate command is needed.

---

## 8. Working with DOM Snapshots

DOM snapshots are `.html` files saved from the browser. They give the AI real element IDs,
aria labels, and data attributes — producing much more reliable locators than AI inference.

### How to save a DOM snapshot

**Option A — Chrome DevTools:**
1. Open the page in Chrome
2. Press `F12` → Elements tab
3. Right-click `<html>` → Copy → Copy outerHTML
4. Paste into a file in `inputs/snapshots/` (e.g., `inputs/snapshots/loginPage.html`)

**Option B — Command line:**
```bash
curl -s "https://myapp.com/login" -o inputs/snapshots/loginPage.html
```

**Option C — Playwright script:**
```javascript
const { chromium } = require('@playwright/test');
const fs = require('fs');
const browser = await chromium.launch();
const page = await browser.newPage();
await page.goto('https://myapp.com/login');
fs.writeFileSync('inputs/snapshots/loginPage.html', await page.content());
await browser.close();
```

### File naming convention

Name files so the kit can infer page names automatically:

| File name | Inferred page name |
|---|---|
| `loginPage.html` | `LoginPage` |
| `login-page.html` | `LoginPage` |
| `amazon-home-page.html` | `AmazonHomePage` |
| `search_results.html` | `SearchResults` |
| `checkout.html` | `Checkout` |

The rule: strip the extension, convert kebab/snake to PascalCase.

### Multi-page flows

Put all snapshot files in `inputs/snapshots/` and pass the directory:

```bash
/generate-automation-tests --dom inputs/snapshots/ --cases "TC-01: ..."
```

The kit discovers all `.html` files, generates one POM per page, and wires page transitions
between them based on your test case steps.

Example directory structure for a login → dashboard flow:

```
inputs/snapshots/
├── loginPage.html         → LoginPage.ts
└── dashboardPage.html     → DashboardPage.ts
```

In your test case, mention both pages in the steps:

```
Steps:
1. Navigate to the login page
2. Enter username and password
3. Click Login                     ← transition happens here
4. Assert dashboard URL is /dashboard
5. Assert welcome message is visible
```

The kit detects the transition at step 3 and generates `waitForURL()` at that point.

---

## 9. Working with Test Cases

### Inline test cases

Pass test cases directly in the command:

```bash
/generate-automation-tests --cases "TC-01: Login with valid credentials
Steps:
  1. Navigate to /login
  2. Enter valid username
  3. Enter valid password
  4. Click Submit
  Assertions:
  - URL is /dashboard
  - Welcome message is visible"
```

### File-based test cases

Save test cases in a markdown file and pass the path:

```bash
/generate-automation-tests --cases tests/test-cases.md
```

**File format (`tests/test-cases.md`):**

```markdown
# TC-01: Successful Login (Happy Path)
Tags: smoke
Preconditions: User account exists at https://myapp.com

## Steps
1. Navigate to the login page at /login
2. Fill username with valid credentials
3. Fill password with valid credentials
4. Click the Login button

## Expected Results
- URL changes to /dashboard
- A success banner displays "Welcome back"
- No error messages are shown

---

# TC-02: Login with Wrong Password (Negative)
Tags: regression, negative
Preconditions: User account exists

## Steps
1. Navigate to the login page
2. Fill username with valid credentials
3. Fill password with an incorrect value
4. Click Login

## Expected Results
- URL remains /login
- An error message "Invalid credentials" is displayed
```

### Multiple test cases in one run

List all test cases in the same markdown file — the kit generates one spec per test case.

```bash
/generate-automation-tests --dom inputs/snapshots/ --cases tests/test-cases.md --tags smoke
```

---

## 10. Locator Strategy — Which to Choose

The kit uses three strategies in priority order. Choose the highest tier available.

### Tier 1 — Live URL (most accurate)

**When to use**: You have a running application accessible via URL.

```bash
/generate-automation-tests --url https://staging.myapp.com/login --cases "TC-01: ..."
```

- Playwright MCP opens a real browser, extracts actual DOM elements
- Locator scores up to 1.0
- Generates both spec and BDD by default

**Requirement**: Browser MCP must be enabled (it is by default in `.claude/settings.json`).

### Tier 2 — DOM Snapshot (high accuracy)

**When to use**: You have `.html` files from the application but no live URL.

```bash
/generate-automation-tests --dom inputs/snapshots/ --cases "TC-01: ..."
```

- Kit parses real HTML to extract `data-testid`, `aria-label`, `id`, `name`, `role`
- Locator scores up to 1.0 for definitive matches
- Generates spec files only by default (add `--bdd` for BDD)

### Tier 3 — AI-Guided (inferred, lowest accuracy)

**When to use**: No URL and no snapshots — test cases only.

```bash
/generate-automation-tests --cases "TC-01: ..."
```

- AI infers selectors from step wording (e.g., "Enter username" → `getByLabel('Username')`)
- Locator scores capped at 0.75 — **always verify before running in CI**
- A warning is added to `selectors.json` and shown in the summary
- Run `npx playwright test --headed` to visually confirm locators work

**Tip**: After running tests once in AI-guided mode, save the real HTML from the browser and
regenerate with `--dom inputs/snapshots/` to get confirmed locators.

---

## 11. Generating BDD Tests

BDD (Behavior-Driven Development) artifacts consist of:
- A **Gherkin `.feature` file** with `Given/When/Then` scenarios
- A **step definitions file** (`.steps.ts`) that maps steps to POM actions

### When BDD is generated automatically

| Strategy | Default output | Override |
|---|---|---|
| `--url` (live) | spec + BDD | n/a |
| `--dom` (snapshot) | spec only | add `--bdd` flag |
| `--cases` only (AI) | spec only | add `--bdd` flag |

### Force BDD generation

```bash
/generate-automation-tests --dom inputs/snapshots/ --cases "TC-01: ..." --bdd
```

### BDD file output locations

```
outputs/sqe-Functional-TestDesignkit-playwright-gf/
├── tests/bdd/
│   ├── login.feature           ← Gherkin scenarios
│   └── login.steps.ts          ← Step definitions (imports from POMs)
```

### Running BDD tests

```bash
cd outputs/sqe-Functional-TestDesignkit-playwright-gf

# Run all BDD scenarios
npx cucumber-js

# Run only @smoke-tagged scenarios
npx cucumber-js --tags @smoke

# Run only @regression scenarios
npx cucumber-js --tags @regression

# Dry-run (validate step definitions without executing)
npx cucumber-js --dry-run
```

> **Note**: Cucumber requires a `cucumber.js` config file (generated by `/setup-kit`). If you see
> "No step definitions found", check that `tests/bdd/*.steps.ts` files exist and the config
> points to the correct paths.

### Running both Playwright and BDD via kit commands

```bash
# Smoke pack — runs Playwright specs + Cucumber scenarios tagged @smoke
/run-smoke

# BDD only
/run-smoke --bdd-only

# Playwright only
/run-smoke --nonbdd-only
```

---

## 12. Running Tests & Viewing Reports

### Install dependencies (first time only)

```bash
cd outputs/sqe-Functional-TestDesignkit-playwright-gf
npm install
npx playwright install chromium    # installs Chromium browser
```

### Run all tests

```bash
npx playwright test
```

### Run smoke tests only

```bash
npx playwright test --grep "@smoke"
# or use the kit command:
/run-smoke
```

### Run BDD feature files (Cucumber)

```bash
cd outputs/sqe-Functional-TestDesignkit-playwright-gf

# Run all BDD scenarios
npx cucumber-js

# Run @smoke scenarios only
npx cucumber-js --tags @smoke
```

### Run both Playwright + BDD via kit commands

```bash
# Runs Playwright specs AND Cucumber features, generates combined Allure report
/run-smoke       # @smoke tagged only
```

### Run a specific spec file

```bash
npx playwright test tests/nonbdd/TC-01-successful-login.spec.ts
```

### Run in headed mode (browser visible)

```bash
npx playwright test --headed
```

### Run in a specific browser

```bash
npx playwright test --project=firefox
npx playwright test --project=webkit
```

### View Playwright HTML report

```bash
npx playwright show-report
```

Opens a browser at `http://localhost:9323` showing pass/fail results with screenshots and traces.

### View Allure report

```bash
# Generate the report from results
npx allure generate --output allure-report allure-results

# Open in browser
npx allure open allure-report
```

> **Note**: You need Java 8+ installed to run Allure CLI. Install from
> [allure.qatools.ru](https://allure.qatools.ru/docs/getting-started/installation/).
> Alternatively, install the npm wrapper: `npm install -g allure-commandline`

### Re-run failed tests only

```bash
npx playwright test --last-failed
```

---

## 13. Configuring Plugins

Plugins extend the kit with integrations for Jira, GitHub, and Allure Server.

### Enable a plugin

Edit `tc-to-automate/configs/integrations.yml`:

```yaml
plugins:
  github:
    enabled: true
    repo: "org/repo-name"
    branch: "main"

  jira:
    enabled: true
    host: "https://yourorg.atlassian.net"
    project: "QA"

  allureServer:
    enabled: false
    host: "http://localhost:5050"
```

### What each plugin does

| Plugin | Trigger | What it does |
|---|---|---|
| `github` | After any generation command | Commits generated files + optionally opens a PR |
| `jira` | Before generation + after `/run-smoke` | Fetches test cases from Jira; pushes results back |
| `allureServer` | After `/run-smoke` | Publishes Allure results to a shared Allure server |

### GitHub plugin — push generated project to a repo

Use this to push the generated `outputs/<project>/` folder to a GitHub repository without
running any git commands manually.

#### Step 1 — Generate a GitHub Personal Access Token (PAT)

1. Go to GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. Click **Generate new token**
3. Select scope: **repo** (full control of private repositories)
4. Copy the token — you will not see it again

#### Step 2 — Set environment variables

Open a **Command Prompt or PowerShell** window and set these before launching Claude Code:

**Windows Command Prompt:**
```cmd
set GITHUB_OWNER=your-github-username
set GITHUB_REPO=your-repo-name
set GITHUB_TOKEN=ghp_your_token_here
```

**Windows PowerShell:**
```powershell
$env:GITHUB_OWNER = "your-github-username"
$env:GITHUB_REPO  = "your-repo-name"
$env:GITHUB_TOKEN = "ghp_your_token_here"
```

**macOS / Linux:**
```bash
export GITHUB_OWNER="your-github-username"
export GITHUB_REPO="your-repo-name"
export GITHUB_TOKEN="ghp_your_token_here"
```

> Set these in the **same terminal session** you will use to launch Claude Code.

#### Step 3 — Enable the plugin in `tc-to-automate/configs/integrations.yml`

```yaml
github:
  enabled: true
  repoOwner: ${GITHUB_OWNER}
  repoName:  ${GITHUB_REPO}
  token:     ${GITHUB_TOKEN}
  createPr:  true
  prBaseBranch: main
  commitMessage: "feat: generated kit-u1 test artifacts"
```

#### Step 4 — Launch Claude Code from the same terminal

```cmd
cd c:\Users\<you>\sqe-kit
claude
```

#### Step 5 — Trigger the push in the Claude Code chat

```
Please follow plugins/github.md to push the sqe-Functional-TestDesignkit-playwright-gf to GitHub.
```

#### What gets pushed / excluded

| Pushed | Excluded (add to `.gitignore`) |
|---|---|
| `src/pages/*.ts` | `node_modules/` |
| `tests/nonbdd/*.spec.ts` | `allure-results/`, `allure-report/` |
| `tests/bdd/` (if generated) | `playwright-report/`, `test-results/` |
| `playwright.config.ts` | `cucumber.json` |
| `package.json`, `tsconfig.json` | |
| `selectors.json`, `intake.summary.json` | |

#### After cloning the pushed repo

Anyone who clones the repo runs:

```bash
npm install
npx playwright install chromium
npx playwright test
```

---

## 14. Troubleshooting & FAQ

### "Active kit" line does not appear at startup

**Cause**: Claude Code did not read `kit.config.json` on startup.
**Fix**: Ask Claude: _"Please read kit.config.json and confirm the active kit."_

---

### `npx playwright test` finds 0 tests

**Cause**: The generated project directory has no tests, or `playwright.config.ts` points to
the wrong `testDir`.

**Fix**:
```bash
cd outputs/sqe-Functional-TestDesignkit-playwright-gf
npx playwright test --list    # lists all discovered tests
```

If the list is empty, check `testDir` in `playwright.config.ts` and ensure spec files exist in
`tests/nonbdd/`.

---

### Tests fail with "locator not found" or timeout errors

**Cause**: AI-guided selectors were inferred and may not match the real DOM.

**Fix**:
1. Run in headed mode: `npx playwright test --headed`
2. Take DOM snapshots of the failing pages (see §8)
3. Re-run: `/generate-automation-tests --dom inputs/snapshots/ --cases "TC-01: ..."`
4. The kit will regenerate POMs with confirmed selectors

---

### `npx playwright install` fails behind a corporate proxy

**Fix**:
```bash
HTTPS_PROXY=http://proxy.corp.com:8080 npx playwright install chromium
```

---

### Allure command not found

**Fix** — install the npm allure wrapper:
```bash
npm install -g allure-commandline
```

Or use Java allure CLI:
```bash
# macOS
brew install allure

# Windows (scoop)
scoop install allure
```

---

### Generated selectors have score 0.75 — what does this mean?

Score 0.75 means the locator was **AI-inferred** from step text, not extracted from a real DOM.
It will work in many cases but is not guaranteed. Always:
1. Run tests in `--headed` mode first to visually confirm
2. Replace with DOM-based selectors when possible (see §8 and §10)

The score is visible in `selectors.json` and flagged in `selectors-lint.html`.

---

### I edited a POM file manually — how do I regenerate without losing my changes?

The kit asks before overwriting existing files. If you want to regenerate cleanly:
1. Delete the specific `.ts` file
2. Re-run `/generate-page-objects`

If you want to keep manual changes: **do not delete** the file. Run `/generate-automation-tests --dry-run`
first to preview what would change.

---

### Can I use the kit without VS Code?

Yes. Run Claude Code in terminal mode:

```bash
cd sqe-kit
claude
```

All slash commands work identically in CLI mode.

---

### Where are the generated files?

All output goes to:
```
outputs/<projectName>/
```

The project name is set in `kit.config.json` under `"projectName"`. Default is
`sqe-Functional-TestDesignkit-playwright-gf`.

---

### How do I reset the project and start fresh?

```bash
# Delete all generated files for the current project
rm -rf outputs/sqe-Functional-TestDesignkit-playwright-gf/

# Then re-run any generation command
```

The kit will regenerate everything from scratch.

---

*Last updated: 2026-03-16 | Kit version: 2.1 | Three capabilities: req-to-story / story-to-testcase / tc-to-automate*

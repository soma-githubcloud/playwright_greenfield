# /setup-kit — Initialize Automation Kit

Initialize the Playwright TypeScript automation project by scaffolding the full project
structure, installing dependencies, and validating the setup.

This kit uses **kit-u1** (Playwright TypeScript). No kit selection is needed.

## Steps

1. **Read current state**
   - Check if `kit.config.json` already exists. If it does, show current configuration and ask:
     "Scaffold already exists. Re-scaffold will overwrite generated config files. Continue? (yes/no)"
   - If no config exists, proceed.

2. **Confirm project settings** — Ask the user:
   - Test style? (`nonbdd` | `bdd` | `both` — default: `both`)
   - Output directory? (default: `./outputs`)
   - Project name? (default: `sqe-Functional-TestDesignkit-playwright-gf`)

3. **Write kit.config.json** at project root
   - Set `id: "kit-u1"`, `mode: "greenfield"`, `tech`, `testStyle`, `outputDir`, `projectName`
   - Set `templates` path: `tc-to-automate/kits/kit-u1/templates`
   - Set `agents` paths pointing to `tc-to-automate/kits/_base/agents/` and `tc-to-automate/kits/kit-u1/agents/`

4. **Load the kit scaffolder agent**
   - Read and execute `tc-to-automate/kits/kit-u1/agents/framework-setup-agent.md`
   - Follow the agent instructions precisely to create the output project structure

5. **Confirm completion**
   - List all created files and directories
   - Show the user the next suggested commands:
     - `/generate-automation-tests` — if you have test cases ready
     - `/e2e --story <path>` — if you have a user story
     - `/e2e --req <path>` — if you have a requirement document

6. **Validate scaffold**
   - `cd <outputDir>/<projectName>`
   - Run TypeScript compile check:
     ```bash
     npx tsc --noEmit
     ```
   - Run linter (if `.eslintrc` or `eslint.config.*` exists):
     ```bash
     npx eslint "**/*.ts" --max-warnings 0
     ```
   - Run Playwright test discovery:
     ```bash
     npx playwright test --list
     ```
   - If `testStyle` includes `"bdd"`, run Cucumber dry-run:
     ```bash
     npx cucumber-js --dry-run
     ```
   - **PASS**: Report `"Kit kit-u1 scaffolded and validated successfully."`
   - **FAIL**: Show exact error output. Attempt to fix automatically if the fix is unambiguous
     (e.g., missing import, wrong relative path, missing config key). Re-run validation after fix.
     Never leave failing validation without explaining what's wrong and how to fix it.

## Notes
- Never overwrite existing source files without explicit user confirmation
- Always create `.gitignore` with `node_modules/`, `allure-results/`, `playwright-report/`, `test-results/`
- Pin dependency versions; do not use `latest` tags in `package.json`
- Validation (Step 6) runs automatically — no need to run `/validate-kit` separately

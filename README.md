# AI Test Agent

An Electron desktop app powered by Claude AI that automatically generates test cases, scaffolds a Cypress or Playwright framework, writes automation scripts, self-heals fragile locators, and analyzes test failures with AI root cause analysis — all from a requirements document, organized by module for any scale of web application.

---

## Features

| # | Feature | Description |
|---|---------|-------------|
| 1 | **Choose Your Framework** | Switch between Cypress and Playwright from the sidebar — every generated file, prompt, and script adapts automatically |
| 2 | **Generate Test Cases** | Reads your requirements and produces structured TC-N test cases (UI + API) per module |
| 3 | **Create Test Framework** | Scaffolds a full Cypress 13 or Playwright Test project with page objects, custom commands/helpers, fixtures, and a self-heal utility |
| 4 | **Generate Test Scripts** | Converts test cases into runnable spec files (`.cy.js` for Cypress, `.spec.js` for Playwright) organized by module |
| 5 | **Self-Heal Locators** | Audits selectors that are genuinely broken or likely to break, and proposes a fallback chain (`data-cy`/`data-testid` → `aria` → `text` → `CSS`) — stable selectors (including IDs) are left untouched |
| 6 | **Re-run Self-Heal** | Re-runs only the self-heal phase after agent completion or when loading a saved session, without re-generating test cases or scripts |
| 7 | **Add New Requirements** | Append new modules to an already-completed run — upload or paste new requirements and the agent generates test cases, framework files, and scripts for only the new modules, without touching existing output |
| 8 | **AI Failure Analyzer** | Injected into every generated framework — reads test results (Cypress or Playwright JSON, auto-detected), calls the AI API, and writes a root-cause analysis report to disk |
| 9 | **GitHub Actions Integration** | Every generated framework includes a workflow tailored to its framework that runs tests on push/PR, triggers the failure analyzer automatically, and posts the analysis as a PR comment |
| 10 | **Module-based Pipeline** | Runs the full pipeline per module — handles enterprise apps with thousands of test cases |
| 11 | **Select & Regenerate TCs** | Pick specific test cases to regenerate scripts for, with AI-assisted selection |
| 12 | **Upload Requirements** | Upload `.txt`, `.md`, `.pdf`, or `.docx` requirement files directly |
| 13 | **Export to Excel** | Download all generated test cases as a `.xlsx` file that opens natively in Excel |
| 14 | **Bug Fix Chat** | Built-in AI chat with full code context to fix errors in generated scripts |
| 15 | **Session Save / Load** | Save your entire run (results + chat history) to disk and reload it later |
| 16 | **Extract Files** | Writes all generated framework files directly to a folder on your disk |
| 17 | **Multi-provider Support** | Switch between Claude (Anthropic) and OpenAI models from the sidebar |

---

## Prerequisites

- [Node.js](https://nodejs.org/) v18 or later
- [Git](https://git-scm.com/)
- An [Anthropic API key](https://console.anthropic.com/) — or an OpenAI API key if using the OpenAI provider

---

## Project structure

```
ai-test-agent/
├── main.js                  # Electron main process + IPC handlers
├── preload.js               # Secure context bridge (IPC)
├── index.html               # UI markup
├── styles.css               # Dark theme design system
├── renderer.js              # All UI logic, agent pipeline, chat
├── agent.js                 # Legacy agent helpers (kept for reference)
├── package.json
├── electron-builder.yml     # Cross-platform build config
├── .env.example             # Environment variable template
├── .gitignore
└── README.md
```

---

## Getting started

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/ai-test-agent.git
cd ai-test-agent
```

### 2. Install dependencies

```bash
npm install
```

### 3. Set up environment (optional)

```bash
cp .env.example .env
```

Edit `.env` and add your API key if you want it pre-filled:

```
ANTHROPIC_API_KEY=sk-ant-...
BASE_URL=https://your-app.com
```

### 4. Run in development mode

```bash
npm start
```

---

## How to use

### Step 1 — Configure

| Field | What to enter |
|-------|--------------|
| AI Provider | Choose Claude (Anthropic) or OpenAI |
| API Key | Your key from [console.anthropic.com](https://console.anthropic.com) or [platform.openai.com](https://platform.openai.com) |
| Model | Select the model — Sonnet 4.6 is recommended for the best speed/quality balance |
| Test Framework | Choose **Cypress** or **Playwright** — this determines every file path, prompt, and config generated for the rest of the run |
| Base URL | The URL of the web app you are testing, e.g. `https://myapp.com` |
| Modules | One module name per line, e.g. `Authentication`, `Shopping Cart`, `Checkout` |
| Requirements | Upload a `.txt` / `.md` / `.pdf` / `.docx` file, or paste text directly |

> **Switching frameworks mid-project:** the Test Framework picker only affects what gets generated going forward. If you change it after a run is already in progress, finish or restart the run first — switching frameworks between an initial run and an **Add New Requirements** append on the same project is not supported, since the two frameworks have incompatible file layouts.

### Step 2 — Run the agent

Click **Run Agent**. You will be prompted to pick an output folder, then the pipeline runs:

```
Once (shared):
  └── Generate shared config files for the selected framework:
      Cypress    → package.json, cypress.config.js, cypress/support/commands.js,
                   cypress/utils/selfHeal.js, .env.example, README.md,
                   AI Failure Analyzer scripts, GitHub Actions workflow
      Playwright → package.json, playwright.config.js, support/globalSetup.js,
                   support/commands.js, utils/selfHeal.js, .env.example, README.md,
                   AI Failure Analyzer scripts, GitHub Actions workflow

For each module:
  ├── Phase 1 — Generate up to 50 test cases
  ├── Phase 2 — Generate page objects, fixtures, and custom commands/helpers
  ├── Phase 3 — Generate spec files (happy-path + edge-cases)
  ├── Phase 4 — Self-heal locators (broken/fragile selectors only)
  └── Write all files to disk immediately
```

### Step 3 — Review output

Use the four tabs to review generated content:

- **Test Cases** — collapsible cards, tagged UI or API, with module headers
- **Framework** — full project scaffold with all config files for the selected framework
- **Test Scripts** — runnable spec files per module (`.cy.js` or `.spec.js`)
- **Self-Heal** — colour-coded locator audit (STALE / REASON / HEALED / STRATEGY)

### Step 4 — Re-run self-heal (optional)

After the agent completes, a **Re-run Self-Heal** button appears. Click it to re-analyze the existing scripts and regenerate the self-heal report without touching test cases or scripts. This also appears when loading a saved session that contains scripts.

> **Self-heal policy:** Only selectors that are genuinely broken or highly likely to break are flagged (e.g. auto-generated IDs, `nth-child` indexes, volatile class names, deeply nested chains). Stable selectors — including meaningful IDs — are left untouched. The healing chain target differs by framework: Cypress prefers `data-cy`, Playwright prefers `data-testid`.

### Step 5 — Add new requirements (optional)

Once a run is complete, an **Add New Requirements** button appears next to the Excel export option. Use it to extend an existing run with new functionality discovered after the fact:

1. Click **Add New Requirements**
2. Enter the new module name(s), one per line (must not match an existing module name)
3. Upload a new requirements document, or paste the new requirements text
4. Click **Generate & Append**

The agent runs the full per-module pipeline (test cases → framework → scripts → self-heal) for only the new modules, using the same framework as the original run, and appends the results to all four panels. Existing test cases, framework files, scripts, and self-heal reports are never overwritten. If an output folder was already chosen for the run, new files are written there automatically. New module names are also added to the sidebar **Modules** field so they're included in any future Extract or Re-run Self-Heal actions.

### Step 6 — Export

| Button | What it does |
|--------|-------------|
| **Extract Framework** | Opens a folder picker and writes all framework + script files to disk |
| **Download Test Cases as Excel** | Saves a `.xlsx` with columns: ID, Type, Title, Preconditions, Steps, Expected Result, Priority |
| **Copy** | Copies the current tab's raw output to clipboard |
| **Save** | Saves the current tab's raw output as a file (named `cypress-framework.md` or `playwright-framework.md` depending on the active framework) |

### Step 7 — Fix bugs with AI chat

Click **Fix Bugs** (floating button, bottom-right). The chat has full context of your generated framework and scripts. Describe any error or paste a stack trace and the AI will return fixed files in `===FILE===` format. Click **Save fixed files to disk** under any response to write the fixes directly.

### Step 8 — Save and reload sessions

Use **Save** / **Load** in the session bar (bottom of the sidebar) to persist your entire run — all four panels of results plus chat history — to a folder on disk. Reload it later to pick up exactly where you left off, including the Re-run Self-Heal and Add New Requirements buttons.

---

## Generated project structure

### Cypress

After running the agent for a 3-module site with Cypress selected, your output folder looks like:

```
output-folder/
├── package.json
├── cypress.config.js                  ← JSON reporter pre-configured
├── .env.example
├── README.md
├── scripts/
│   ├── analyze-failures.js            ← AI root cause analyzer (Node.js)
│   └── analyze-failures-local.sh      ← One-command local runner
├── .github/
│   └── workflows/
│       └── ai-failure-analysis.yml    ← GitHub Actions workflow
├── cypress/
│   ├── support/
│   │   ├── commands.js
│   │   └── e2e.js
│   ├── utils/
│   │   └── selfHeal.js
│   ├── pages/
│   │   ├── Authentication/index.js
│   │   ├── ShoppingCart/index.js
│   │   └── Checkout/index.js
│   ├── fixtures/
│   │   ├── authentication.json
│   │   ├── shopping-cart.json
│   │   └── checkout.json
│   ├── e2e/
│   │   ├── authentication/
│   │   │   ├── happy-path.cy.js
│   │   │   └── edge-cases.cy.js
│   │   ├── shopping-cart/
│   │   │   ├── happy-path.cy.js
│   │   │   └── edge-cases.cy.js
│   │   └── checkout/
│   │       ├── happy-path.cy.js
│   │       └── edge-cases.cy.js
│   └── reports/
│       ├── test-cases-authentication.md
│       ├── test-cases-shopping-cart.md
│       ├── test-cases-checkout.md
│       ├── results.json               ← Written by Cypress after a run
│       └── failure-analysis.md        ← Written by AI analyzer after a run
```

### Playwright

The same 3-module site with Playwright selected produces:

```
output-folder/
├── package.json
├── playwright.config.js               ← JSON reporter pre-configured
├── .env.example
├── README.md
├── scripts/
│   ├── analyze-failures.js            ← AI root cause analyzer (Node.js, framework-agnostic)
│   └── analyze-failures-local.sh      ← One-command local runner
├── .github/
│   └── workflows/
│       └── ai-failure-analysis.yml    ← GitHub Actions workflow (Playwright variant)
├── support/
│   ├── globalSetup.js                 ← Auth/login state setup
│   └── commands.js
├── utils/
│   └── selfHeal.js
├── pages/
│   ├── Authentication/index.js
│   ├── ShoppingCart/index.js
│   └── Checkout/index.js
├── fixtures/
│   ├── authentication.json
│   ├── shopping-cart.json
│   └── checkout.json
├── tests/
│   ├── authentication/
│   │   ├── happy-path.spec.js
│   │   └── edge-cases.spec.js
│   ├── shopping-cart/
│   │   ├── happy-path.spec.js
│   │   └── edge-cases.spec.js
│   └── checkout/
│       ├── happy-path.spec.js
│       └── edge-cases.spec.js
└── playwright-report/
    ├── results.json                   ← Written by Playwright after a run
    └── failure-analysis.md            ← Written by AI analyzer after a run
```

### Appending new modules

After using **Add New Requirements** to add e.g. a `Wishlist` module, new files are written alongside the existing ones for whichever framework the project was created with, without disturbing anything already generated. Example for a Playwright project:

```
output-folder/
├── pages/
│   ├── Authentication/ ...  ← unchanged
│   ├── ShoppingCart/   ...  ← unchanged
│   ├── Checkout/       ...  ← unchanged
│   └── Wishlist/             ← new
│       └── index.js
├── tests/
│   ├── authentication/ ...  ← unchanged
│   ├── shopping-cart/  ...  ← unchanged
│   ├── checkout/       ...  ← unchanged
│   └── wishlist/             ← new
│       ├── happy-path.spec.js
│       └── edge-cases.spec.js
└── fixtures/
    └── wishlist.json         ← new
```

---

## Run the generated tests

### Cypress

```bash
cd output-folder
npm install

# Open Cypress Test Runner (interactive)
npx cypress open

# Run all tests headlessly (also writes results.json for the analyzer)
npx cypress run

# Run a specific module
npx cypress run --spec "cypress/e2e/authentication/**"
```

### Playwright

```bash
cd output-folder
npm install
npx playwright install --with-deps   # download browser binaries (first time only)

# Open Playwright's interactive UI mode
npx playwright test --ui

# Run all tests headlessly (also writes results.json for the analyzer)
npx playwright test

# Run a specific module
npx playwright test tests/authentication
```

---

## AI Failure Analyzer

Every generated framework — Cypress or Playwright — includes a root-cause analyzer that reads test results and uses AI to explain each failure, categorize it, and suggest a concrete fix. The analyzer auto-detects which JSON shape it's reading (Cypress/Mocha vs Playwright) so the same script works for either framework.

### Run locally

```bash
cd output-folder

# Set your Anthropic API key
export ANTHROPIC_API_KEY=sk-ant-your-key-here

# Run the analyzer (runs the test suite first if no results.json exists yet)
bash scripts/analyze-failures-local.sh
```

The report is saved to `cypress/reports/failure-analysis.md` (Cypress) or `playwright-report/failure-analysis.md` (Playwright). A preview is also printed to the terminal.

### What the report contains

For every failing test, the analyzer outputs:

| Section | Content |
|---------|---------|
| **ROOT CAUSE** | The exact technical reason the test failed |
| **CATEGORY** | One of: `Selector` / `Timing` / `Network` / `Assertion` / `Config` / `Data` / `Environment` / `Unknown` |
| **FIX** | A concrete code fix or actionable step, with corrected code where applicable |
| **PREVENTION** | How to prevent this class of failure in future tests |

### Run via GitHub Actions

The included `.github/workflows/ai-failure-analysis.yml` workflow is generated to match the framework you chose. Both variants:

1. Run `npm ci`, and for Playwright also `npx playwright install --with-deps`
2. Run the full test suite on every push to `main`/`develop` and on every pull request
3. Automatically run the AI failure analyzer after the test run (even if tests fail)
4. Upload the results JSON, the failure analysis report, and any screenshots/videos/traces as downloadable artifacts (retained for 14 days)
5. Post the full failure analysis as a comment on pull requests

**Setup — add these secrets/variables in your GitHub repository settings:**

| Name | Where | Value |
|------|-------|-------|
| `ANTHROPIC_API_KEY` | Repository secret | Your Anthropic API key |
| `CYPRESS_USERNAME` | Repository secret | Test user email |
| `CYPRESS_PASSWORD` | Repository secret | Test user password |
| `BASE_URL` | Repository variable | Your app's URL (e.g. `https://myapp.com`) |

Then push your framework to GitHub — the workflow triggers automatically.

**Manual trigger:** Go to Actions → "Cypress + AI Failure Analysis" (or "Playwright + AI Failure Analysis") → Run workflow.

---

## Module capacity guide

| App scale | Modules | Claude calls | Files generated |
|-----------|---------|-------------|-----------------|
| Small (5–10 pages) | 2–3 | ~10 | ~25–35 |
| Medium (20–30 pages) | 4–6 | ~20 | ~45–65 |
| Large eCommerce | 8–12 | ~40 | ~85–125 |
| Enterprise (100+ pages) | 15–25 | ~80 | ~155–260 |

Each module generates up to 50 test cases and 5–7 files within the 8,000 token output limit, regardless of which framework is selected. Modules added later via **Add New Requirements** follow the same per-module cost.

---

## Build standalone executables

```bash
# Windows — produces dist/AI Test Agent Setup.exe
npm run build:win

# macOS — produces dist/AI Test Agent.dmg
npm run build:mac

# Linux — produces dist/AI Test Agent.AppImage
npm run build:linux

# All platforms at once
npm run build
```

Built files are output to the `dist/` folder.

> **macOS note:** Right-click the `.app` and choose Open for local testing without a developer certificate.

> **Windows note:** The `.exe` installer is unsigned. Windows Defender may show a SmartScreen warning — click "More info → Run anyway" for local builds.

---

## Environment variables

### AI Test Agent app

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | Claude API key — can also be entered in the UI at runtime |
| `BASE_URL` | Default base URL for the app under test |
| `USERNAME` | Default test user email |
| `PASSWORD` | Default test user password |
| `API_KEY` | API key for the app under test (used in API test specs) |

### Generated framework (`.env.example`)

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | Used by the AI failure analyzer script |
| `BASE_URL` | The app under test (set as `CYPRESS_BASE_URL` for Cypress, `BASE_URL` for Playwright) |
| `USERNAME` | Test user email |
| `PASSWORD` | Test user password |
| `API_KEY` | API key for the app under test |
| `ANALYZER_MODEL` | AI model for failure analysis (default: `claude-sonnet-4-6`) |

---

## Tech stack

| Technology | Version | Purpose |
|-----------|---------|---------|
| [Electron](https://www.electronjs.org/) | v28 | Desktop shell, file system access, IPC |
| [Claude API](https://docs.anthropic.com/) | claude-sonnet-4-6 | AI backbone for all pipeline phases and failure analysis |
| [Cypress](https://www.cypress.io/) | v13 | Generated test framework option #1 |
| [Playwright](https://playwright.dev/) | latest (`@playwright/test`) | Generated test framework option #2 |
| [electron-builder](https://www.electron.build/) | v24 | Cross-platform packaging (.exe, .dmg, .AppImage) |
| [mammoth](https://github.com/mwilliamson/mammoth.js) | v1.6 | DOCX text extraction |
| Vanilla HTML/CSS/JS | — | Zero frontend framework, zero bundle step |

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Run Agent button does nothing | Open DevTools (`Ctrl+Shift+I`) → Console and check for red errors |
| `401 Unauthorized` from API | API key is wrong — Claude keys start with `sk-ant-`, OpenAI keys with `sk-proj-` |
| `403 Forbidden` from API | Missing `anthropic-dangerous-direct-browser-access` header — update `renderer.js` |
| Files not created after Extract | Check DevTools console for `✓ Written:` log lines from `main.js` |
| PDF text not extracted | Install poppler-utils (`brew install poppler` on Mac, `apt install poppler-utils` on Linux) |
| DOCX not extracting | Run `npm install mammoth` and restart |
| Test cases CSV is empty | Open DevTools console, run `copy(results[0])`, paste into a text editor to check parser output |
| Phase 1 stuck running | A previous `startAgent` call is still running — wait or restart the app |
| Excel file garbled characters | Open with Excel → Data → From Text/CSV → select UTF-8 encoding |
| Re-run Self-Heal button missing | It only appears after agent completes or when a session with scripts is loaded — check that Panel 2 (Test Scripts) has content |
| Add New Requirements button missing | Same gating as Re-run Self-Heal — requires an existing run or a loaded session with scripts |
| "Module already exists" toast | The new module name matches one already in the sidebar Modules field — rename it or remove the old one first |
| Clicking Playwright/Cypress hides the API key field | Make sure the AI Provider tab click handler is scoped to `#providerTabs .provider-tab`, not the unscoped `.provider-tab` selector — otherwise clicking a framework tab also fires the provider-switch logic and hides the wrong section |
| Failure analyzer: `ANTHROPIC_API_KEY is not set` | Run `export ANTHROPIC_API_KEY=sk-ant-...` before running the analyzer script |
| Failure analyzer: `No results file found` | Run the test suite first (`npx cypress run` or `npx playwright test`), or use `bash scripts/analyze-failures-local.sh` which runs it automatically |
| GitHub Actions: analyzer step skipped | Ensure `ANTHROPIC_API_KEY` is added as a repository secret (Settings → Secrets → Actions) |
| GitHub Actions: PR comment not posted | The workflow needs `issues: write` and `pull-requests: write` permissions — check your repository Actions permissions settings |
| Playwright JSON results look corrupted | Prefer the `reporter` config option (`['json', { outputFile: ... }]`) over shell-redirecting stdout — some Playwright versions print extra non-JSON output to stdout that breaks a naive `> results.json` redirect |

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/your-feature`
3. Commit your changes: `git commit -m "feat: describe what you added"`
4. Push to the branch: `git push origin feat/your-feature`
5. Open a Pull Request

---

## License

MIT — see [LICENSE](LICENSE) for details.
# DeepGuard — AI Dependency Guardian

A VS Code extension that detects, verifies, and **secures** your project's
dependencies *before* anything is installed — turning the "paste → run → error →
google → install → repeat" loop into one safe click.

> **Detect → Verify → Secure → Install**

**Languages:** Python (pip/PyPI) and JavaScript/TypeScript (npm). DeepGuard
auto-detects the project's ecosystem and runs the **same pipeline and the same
GUI** for both.

## What it does

| Area | Feature |
|------|---------|
| **Detect** | Comment/docstring-aware import parsing (not naive regex). Resolves the import-name → install-name gap that breaks other tools (`langchain_openai`→`langchain-openai`, scoped/subpath npm imports). Infers **hidden** deps (`os.system("playwright install")`). **Excludes your own code** — local Python modules and JS monorepo/workspace packages + tsconfig path aliases are never treated as registry packages. |
| **Verify** | Diffs detected packages against the active environment (`pip list` / `node_modules`), showing installed vs. missing with a per-dependency risk column. |
| **Secure** | **Typosquat protection** against the top-8,000 packages per registry, gated on registry existence and tuned to avoid false positives. **Supply-chain malware scan** of each package's install code (`setup.py` / npm lifecycle scripts) before install. **CVE scan** via [OSV.dev](https://osv.dev). **Vulnerability audit** via `npm audit` / `pip-audit` (with a built-in OSV fallback). **Project-wide secret detection**. **License surfacing**. Every finding carries a **severity**. |
| **Install** | One-click install of the *safe, missing* packages only. Detects missing wheels and swaps to maintained drop-ins; corrects deprecated stub packages; recognizes `abi3` wheels (clean installs on new Python versions). Failures explain *why* in plain English. Nothing installs automatically. |
| **Artifacts** | Generate `requirements.txt`, a CycloneDX **SBOM**, scaffold a project (**Bootstrap**), and create a `.venv`. |

Analysis is **automatic**: a fast offline pass runs as you open/type (inline
squiggles + status bar), and the full networked pass runs on save or on demand.

## The report panel

Open it via **DeepGuard: Analyze Whole Project** (or the status-bar shield). It shows:

- **Summary tiles** (dependencies / missing / typosquats / CVEs / secrets / code
  risks) that double as **filters** — click one to isolate that section.
- A card per finding type with a **severity column**; CVEs are grouped by severity.
- **Click any finding** to jump to the exact `file:line` in your code.
- A **vulnerability audit card** (`npm audit` / `pip-audit`) with a non-breaking
  one-click fix.
- A **live install progress card** (package chips, streaming output, dismissable).
- Typosquats also surface as **quick-fix lightbulbs** in the editor.

## Usage

1. Open a Python or JavaScript/TypeScript project.
2. It scans automatically on activation; review the panel and click **Install** or
   **Set up project**.
3. For a single file/paste flow: **DeepGuard: Analyze Dependencies in File** /
   **Analyze Pasted Code (Selection)**.

### Commands
- `DeepGuard: Analyze Dependencies in File`
- `DeepGuard: Analyze Pasted Code (Selection)`
- `DeepGuard: Analyze Whole Project`
- `DeepGuard: Set Up Project (scan all + venv + install)`
- `DeepGuard: Generate requirements.txt`
- `DeepGuard: Generate SBOM (CycloneDX)`
- `DeepGuard: Create Virtual Environment`
- `DeepGuard: Bootstrap Project (scaffold files)`
- `DeepGuard: Scan a Package for Malicious Install Code`

### Settings
- `deepguard.analyzeOnSave` (default `true`)
- `deepguard.enableCveScan` (default `true`) — OSV.dev + registry lookups
- `deepguard.enableSecretScan` (default `true`)
- `deepguard.enableMalwareScan` (default `true`) — scan install code before installing
- `deepguard.useVirtualEnv` (`ask` | `always` | `never`, default `ask`) — how Python
  installs handle a `.venv`. JavaScript projects never use one.
- `deepguard.pythonPath` — override interpreter (else uses the Python extension's
  selection, a workspace `.venv`, or PATH)

## How the security data stays current

DeepGuard ships **no vulnerability database**. CVE, audit, and registry data are
queried **live** at scan time from the canonical sources — [OSV.dev](https://osv.dev)
(aggregates the PyPA Advisory DB, GitHub Security Advisories, etc.), the GitHub
Advisory Database (via `npm audit`), `pip-audit`, and the PyPI/npm registries — so
results are as fresh as those upstreams. The only bundled datasets are the
typosquat popularity baselines and the curated drop-in/alias tables, which refresh
when the extension is rebuilt.

## Develop & run

```bash
npm install
npm run compile        # or: npm run watch
npm test               # unit test suite (Node's built-in runner)
```

Press **F5** to launch the Extension Development Host and open a project.

```bash
npm run package                 # type-check + build into dist/
npx @vscode/vsce package        # produce deepguard-0.5.0.vsix
code --install-extension deepguard-0.5.0.vsix
```

## Architecture

```
src/
  core/
    detect.ts        # Python import parsing + name resolution + hidden deps
    verify.ts        # installed-vs-missing via pip list
    typosquat.ts     # edit-distance + existence + false-positive filters
    secrets.ts       # credential pattern scan (per-file + whole-project)
    codeScan.ts      # code-risk (SAST-lite) rules for both languages
    registry.ts      # PyPI/OSV lookups + wheel-compatibility (incl. abi3)
    audit.ts         # shared vulnerability-audit model
    installer.ts     # smart resolve: drop-in swaps, stub aliases, failure hints
    concurrency.ts   # bounded-concurrency network fan-out
    sdistScan.ts     # Python sdist setup.py malware scan
    analyze.ts       # orchestrates detect -> verify -> secure
    artifacts.ts     # requirements.txt + CycloneDX SBOM
    scaffold.ts      # project bootstrap
    data.ts          # name maps, popular baseline, WHEEL_FALLBACKS, PACKAGE_ALIASES
    project.ts       # whole-project scanning + local-module discovery
    pythonEnv.ts     # interpreter discovery
  lang/javascript/   # detect, analyze, registry, verify, typosquat, malware, audit
  lang/python/       # audit (pip-audit + OSV fallback)
  ui/
    panel.ts         # webview report: tiles/filters, severity columns, install card
    diagnostics.ts   # inline typosquat + secret + code-risk squiggles
    quickFix.ts      # typosquat quick-fix code actions
  extension.ts       # activation, commands, install flow
```

Security is the moat: the `typosquat` + `registry` + `audit` + `sdistScan` layer
turns import detection into a real supply-chain guard. See `SECURITY.md` for the
trust-boundary model and the extension's own hardening.

## License

MIT

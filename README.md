# DepGuard — AI Dependency Guardian

A VS Code extension that turns the slow "paste → run → error → google → install → repeat"
loop into one safe click — and screens every dependency for danger **before** anything is installed.

> **Detect → Verify → Secure → Install**

**Languages:** Python (pip/PyPI) and JavaScript/TypeScript (npm). DepGuard
auto-detects the project's ecosystem and runs the same pipeline for both —
typosquat screening (top-8,000 packages per registry), OSV CVE scanning,
registry license/existence checks, and a code-risk scan of your own source.

## What it does

| Area | Feature |
|------|---------|
| **Detect** | Parses imports from the active Python file (comment/docstring-aware, not naive regex). Resolves the import-name → install-name gap that breaks other tools: `bs4`→`beautifulsoup4`, `sklearn`→`scikit-learn`, `langchain_openai`→`langchain-openai`. Infers **hidden** deps such as `os.system("playwright install")`. **Excludes local modules** — `import button` next to a `button.py` is your code, not a PyPI package, so it's never installed. |
| **Verify** | Diffs detected packages against the active interpreter (`pip list`), showing installed versions vs. what's missing. |
| **Secure** | **Typosquat protection** against the real top-8,000 PyPI packages by download count *plus* a PyPI-existence check. **Supply-chain malware scan** — downloads each package's source distribution and inspects `setup.py` for install-time network calls, process spawning, and obfuscated `exec`, blocking high-risk packages before install. **CVE scan** via [OSV.dev](https://osv.dev), pinned to the version you'd actually run. **Secret detection** for API keys/tokens. **License surfacing** from PyPI metadata. |
| **Install** | One-click install of the *safe, missing* packages only — typosquats, local modules, and unknown packages are excluded. Nothing is ever installed automatically. |
| **Artifacts** | Generate `requirements.txt` (pinned or names-only) and a CycloneDX **SBOM** (`sbom.json`). Create a `.venv` with one command. |
| **Bootstrap** | Scaffold a complete project from the detected deps + project type: entrypoint (FastAPI/Flask/generic), `requirements.txt`, `.env`/`.env.example`, `README.md`, `.gitignore`, `Dockerfile`, and a `tests/` folder. Never overwrites existing files. |

Analysis is **automatic**: a fast offline pass runs as you open/type/paste (inline squiggles + a status-bar indicator), and the full networked pass runs on save or when you click the status bar.

Typosquats and secrets also appear as **inline squiggles** in the editor.

## Usage

1. Open a Python file.
2. Run **DepGuard: Analyze Dependencies in File** (editor title-bar shield icon, right-click menu, or Command Palette). Select code first and use **Analyze Pasted Code (Selection)** for the paste flow.
3. Review the panel — detected deps, missing packages, typosquat/CVE/secret warnings — and click **Install**.

Analysis also runs automatically on save (toggle with `depguard.analyzeOnSave`).

### Commands
- `DepGuard: Analyze Dependencies in File`
- `DepGuard: Analyze Pasted Code (Selection)`
- `DepGuard: Generate requirements.txt`
- `DepGuard: Generate SBOM (CycloneDX)`
- `DepGuard: Create Virtual Environment`
- `DepGuard: Bootstrap Project (scaffold files)`
- `DepGuard: Set Up Project (scan all + venv + install)`
- `DepGuard: Scan a Package for Malicious Install Code`

### Settings
- `depguard.analyzeOnSave` (default `true`)
- `depguard.enableCveScan` (default `true`) — OSV.dev + PyPI lookups
- `depguard.enableSecretScan` (default `true`)
- `depguard.pythonPath` — override interpreter (else uses the Python extension's selection, a workspace `.venv`, or PATH)

## Develop & run

```bash
npm install
npm run compile        # or: npm run watch
npm test               # run the unit test suite (Node's built-in runner)
```

Press **F5** in VS Code to launch the Extension Development Host, open a Python file, and run the analyze command.

```bash
npm run package        # type-check + build into dist/
npx @vscode/vsce package   # produce depguard-0.1.0.vsix
code --install-extension depguard-0.1.0.vsix
```

## Architecture

```
src/
  core/
    detect.ts      # import parsing + name resolution + hidden-dep inference
    verify.ts      # installed-vs-missing via pip list
    typosquat.ts   # edit-distance + PyPI-existence screen   <- the moat
    secrets.ts     # credential pattern scan
    registry.ts    # PyPI (license/existence) + OSV.dev (CVEs)
    analyze.ts     # orchestrates detect -> verify -> secure
    artifacts.ts   # requirements.txt + CycloneDX SBOM
    data.ts        # name maps, popular-package baseline, project signals
    pythonEnv.ts   # interpreter discovery
  ui/
    panel.ts       # webview install-preview / health dashboard
    diagnostics.ts # inline typosquat + secret squiggles
  extension.ts     # activation, commands, install flow
```

The detect/verify code is the easy part. The defensibility is the **security data layer**
(`typosquat.ts` + `registry.ts`) — refresh `POPULAR_PACKAGES` from live PyPI download stats
and the typosquat baseline becomes production-grade.

## License

MIT

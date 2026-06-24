# Changelog

## 0.4.0

- Transitive dependency scanning: a new panel section resolves and scans the
  FULL dependency tree (direct + indirect) for known vulnerabilities, not just
  your direct imports. Prefers the actually-installed environment (venv /
  node_modules) for ground-truth versions, falling back to deps.dev resolution
  when nothing is installed. Vulnerable indirect packages are listed with their
  relation, version, and CVE ids. Works for both Python and npm.

## 0.3.2

- Added an automated test suite (`npm test`, Node's built-in runner, no extra
  deps): 18 tests covering import detection, name resolution, hidden deps, local
  -module exclusion, typosquat screening, the Python/JS code scan, setup.py and
  npm install-script malware rules, and the typo-corrected install set.

## 0.3.1

- npm supply-chain malware gate: before `npm install`, DepGuard scans each
  package's lifecycle install scripts (preinstall/install/postinstall) for the
  patterns real npm attacks use — piping a download into a shell, inline
  `node -e`, base64-decoded payloads, process spawning — and blocks high-risk
  packages. Legit native builds (e.g. `node-gyp rebuild`) are not flagged.

## 0.3.0

- Multi-language: added JavaScript / TypeScript (npm) support alongside Python.
  DepGuard auto-detects the project's ecosystem and runs the full pipeline —
  import detection (`import`/`require`, scoped packages, subpaths), missing-vs-
  installed via node_modules, typosquat screening against the top-8,000 npm
  packages, CVE scanning via OSV (npm ecosystem), npm registry license/existence,
  JS code-risk scan (eval, child_process, innerHTML, etc.), and `npm install`.

## 0.2.5

- The panel "malware" section is now a Code security scan of YOUR project's own
  source (eval/exec, os.system, subprocess shell=True, pickle/yaml load, network
  calls) — runs automatically (local, no downloads). Findings also appear as
  inline editor squiggles. The dependency sdist malware scan remains available
  at install time and via the "Scan a Package" command.

## 0.2.4

- The report panel now has a malware tile and a "Supply-chain malware scan"
  section with an on-demand "Scan for malware" button; results (risk + findings
  per package) are shown right in the panel.

## 0.2.3

- Typosquat warnings are still shown, but the CORRECTED package now appears in
  the install list and is installed by "Install" / "Set up project" (the wrong
  name is replaced, never installed). The Missing section shows the correction.

## 0.2.2

- Prefers an existing project `.venv` automatically when picking the interpreter.
- Fixed name resolution for packages whose PyPI name has no separator
  (`speech_recognition` → `SpeechRecognition`, `discord` → `discord.py`, etc.) —
  they no longer show up as false typosquats.
- Typosquat / wrong-package warnings now include an explanation, a "Verify on
  PyPI" link, and a one-click button to install the correct package.

## 0.2.1

- Activates automatically on startup and scans the WHOLE project (every .py
  file) the moment a folder opens — no need to open a file and run a command.
- Status bar now reflects the whole project; clicking it opens the full project
  report. New `DepGuard: Analyze Whole Project` command.
- First automatic scan offers to open the report or run Set Up Project.

## 0.2.0

- Typosquat detection now screens against the real top-8,000 PyPI packages by
  download count (was a small curated list).
- New supply-chain malware scan: downloads each package's source distribution
  and inspects `setup.py` for install-time network calls, process spawning, and
  obfuscated `exec`. High-risk packages are blocked before install. New
  `DepGuard: Scan a Package for Malicious Install Code` command and
  `depguard.enableMalwareScan` setting.

## 0.1.0

Initial release.

- **Detect**: comment/docstring-aware Python import parsing, import→PyPI name
  resolution, and hidden-dependency inference.
- **Verify**: installed-vs-missing diff against the active interpreter.
- **Secure**: typosquat protection (edit-distance + PyPI existence), CVE scan
  via OSV.dev, secret detection, and license surfacing.
- **Install**: one-click install of safe, missing packages only; install
  preview with explicit confirmation.
- Inline editor diagnostics for typosquats and secrets.
- `requirements.txt` and CycloneDX SBOM generation; `.venv` creation.
- Analyze-on-save.

# Security policy

DeepGuard is a security tool that activates automatically when a project folder
is opened, runs subprocesses (`pip`/`npm`), fetches data from public registries,
and renders that data in a webview. That makes it a target: **opening a malicious
repository must never let it execute code or exfiltrate data.** Much of this
codebase is AI-authored, so we do not trust any change until automated gates and
(for sensitive surfaces) human review have verified it.

## Trust boundaries ("danger zones")

These are the surfaces where a bug becomes a vulnerability. Changes to the files
backing them require Code Owner review (see `.github/CODEOWNERS`) and the PR
security checklist.

| Boundary | Risk | Primary control |
|----------|------|-----------------|
| Process execution (`src/core/proc.ts`) | Command injection / RCE | `run()` rejects shell metacharacters in args; fails closed. Frozen by `src/test/proc.test.ts`. |
| Webview rendering (`src/ui/panel.ts`) | XSS (renders remote registry/CVE data) | Escape every interpolated value; Content-Security-Policy. |
| Network parsing (`registry.ts`, `audit.ts`) | Malformed/oversized untrusted input | Defensive parsing; tolerate missing/typed fields. |
| Filesystem writes (`scaffold.ts`, `artifacts.ts`) | Path traversal | Writes constrained to the workspace root. |
| External links (`extension.ts`) | Phishing / SSRF | Host+scheme allowlist before `openExternal`. |

## Invariants encoded as tests

Security guarantees are captured as tests so neither a human nor the AI can
silently regress them. Today:

- `run()` never executes an argument containing shell metacharacters
  (`src/test/proc.test.ts`).

When you add a trust boundary, add the matching invariant test.

## Reporting a vulnerability

Open a private security advisory on the repository (Security → Advisories), or
email the maintainer. Please do not file a public issue for an unpatched
vulnerability.

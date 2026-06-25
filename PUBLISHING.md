# Publishing DeepGuard to the VS Code Marketplace

A one-time setup, then `npm run publish:patch` for every release. ~20 minutes
the first time.

---

## 0. Before you start — edit two things in `package.json`

1. **`publisher`** (currently `"vulncraft"`). This MUST match a publisher ID you
   own on the Marketplace (you create it in step 2). Pick a unique id — e.g.
   your name or company. Lowercase, no spaces.
2. **`repository` / `bugs` / `homepage`** — replace `YOUR_GITHUB_USERNAME` with
   your GitHub user (or remove these fields if you won't host the code publicly).
   The Marketplace shows a "Repository" link from this.

> The extension ID becomes `<publisher>.deepguard`. Once published you cannot
> rename it, so choose the publisher id deliberately.

---

## 1. Install the publishing tool

```bash
npm install -g @vscode/vsce
```

## 2. Create a publisher

A publisher is your Marketplace identity. Two ways:

**Web (easiest):**
1. Go to https://marketplace.visualstudio.com/manage
2. Sign in with a Microsoft account.
3. Click **Create publisher**, choose an ID (must equal `publisher` in
   `package.json`) and a display name.

## 3. Get a Personal Access Token (PAT)

vsce authenticates with an Azure DevOps PAT.

1. Go to https://dev.azure.com and sign in with the **same** Microsoft account.
2. If you have no organization, create one (any name).
3. Click your profile icon → **Personal access tokens** → **New Token**.
4. Settings:
   - **Organization:** **All accessible organizations** (important).
   - **Expiration:** up to 1 year.
   - **Scopes:** click **Show all scopes**, find **Marketplace**, check
     **Manage** (read + publish).
5. **Create** and copy the token now (you won't see it again).

## 4. Log in

```bash
vsce login <your-publisher-id>
# paste the PAT when prompted
```

## 5. Publish

From the `deepguard-vscode/` folder:

```bash
npm test            # make sure the suite passes first
npm run publish:patch   # bumps 0.5.0 -> 0.5.1, packages, publishes
```

Or publish the current version as-is without bumping:

```bash
vsce publish
```

It appears at `https://marketplace.visualstudio.com/items?itemName=<publisher>.deepguard`
within a few minutes. Users then install with:

```bash
code --install-extension <publisher>.deepguard
```

---

## Release checklist (every time)

1. Update `CHANGELOG.md` with the new version's notes.
2. `npm test` — all green.
3. `npm run publish:patch` (or `:minor` / `:major`).

`vsce` reads the version from `package.json`; the `publish:*` scripts bump it
for you and `vscode:prepublish` recompiles automatically.

## Notes / gotchas

- **Icon:** must be a real PNG (already provided at `media/icon.png`). ✓
- **README:** becomes the Marketplace page — already written. ✓
- **LICENSE:** present (MIT). ✓
- **No telemetry:** DeepGuard sends nothing to its own servers; it only calls
  PyPI, npm, and OSV.dev. Worth stating on the Marketplace page for
  security-conscious users.
- If `vsce` complains about activation/size, run `vsce ls` to see exactly what
  goes into the package (`.vscodeignore` already excludes `src/`, tests, and
  `node_modules`).

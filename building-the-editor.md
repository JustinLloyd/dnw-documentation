# DNW Editor — VSCode Web Build & Run Guide

This document covers everything needed to build, run, and ship the DNW Editor, which is a customised VSCode Web instance that hosts the DNW workspace client extension.

---

## Prerequisites

- Node.js 22.x (`nvm` recommended)
- Python 3.10+ (for the workspace engine)
- `git`
- Linux or WSL2 (WSL2 on Windows is supported; native NTFS paths are slow — see rsync note below)

```bash
node --version   # should be v24.x
npm --version    # should be 10.x or 11.x
```

---

## 1. Fresh Build of VSCode Web

This is a one-time setup. It clones the VSCode repository and compiles it from source.

```bash
# Clone VSCode at the pinned version
git clone https://github.com/microsoft/vscode ~/vscode-clean
cd ~/vscode-clean
git checkout 1.127.0

# Install dependencies
# Skip Electron and Playwright — we only need the web build
ELECTRON_SKIP_BINARY_DOWNLOAD=1 PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1 npm install

# Apply DNW patches (see Patches section below)
# These must be applied before compilation
git apply patches/reporter-patch.diff

# Compile
# Uses tsgo (the fast TypeScript compiler) — takes ~47 seconds, zero errors
npm run compile
```

After `npm run compile`, the built output lives in `out/`. This is what `@vscode/test-web` serves.

**Why not `npm run gulp vscode-web`?**

The gulp build uses the older TypeScript compiler and hits 34 upstream errors in Microsoft's own code. It's only needed for a production-minified bundle. For development, `npm run compile` is sufficient and much faster. The reporter patch (applied above) makes the gulp build non-fatal if you do need it.

---

## 2. Incremental Build of VSCode Web

After the initial fresh build, subsequent rebuilds only recompile changed files:

```bash
cd ~/vscode-clean
npm run compile
```

That's it. `tsgo` handles incremental compilation automatically. On a warm build with minor changes, this takes a few seconds.

If you're actively editing VSCode source files, you can run the watch mode instead:

```bash
npm run watch
```

This stays running and recompiles on save. You still need to restart the dev server (step 3) to pick up changes.

---

## 3. Running VSCode Web for Development

The dev server is `@vscode/test-web`, Microsoft's own tool for serving VSCode Web from sources. It serves from `out/`, not the gulp output directory.

```bash
cd ~/vscode-clean

node node_modules/@vscode/test-web/out/server/index.js \
    --sourcesPath . \
    --port 9001 \
    --browserType none \
    --extensionPath /path/to/dnw-workspace-client \
    --folder-uri dnw:/
```

**Parameters:**

| Flag | Purpose |
|------|---------|
| `--sourcesPath .` | Serve from compiled `out/` in this directory |
| `--port 9001` | Port to listen on |
| `--browserType none` | Don't auto-launch a browser (you open it yourself) |
| `--extensionPath` | Path to a directory containing your sideloaded extension(s) |
| `--folder-uri dnw:/` | Open VSCode with the DNW virtual filesystem as the workspace |

After starting, open `http://localhost:9001` in your browser.

**Note on `--folder-uri`:** The `dnw:/` URI tells VSCode to open the workspace via the dnw-workspace-client FileSystemProvider. The workspace engine must be running at `localhost:2040` for files to appear.

---

## 4. Building the DNW Extensions

### dnw-workspace-client

This extension provides the `dnw://` FileSystemProvider that connects VSCode to the workspace engine.

```bash
cd /path/to/dnw-workspace-client

npx esbuild src/extension.ts \
    --bundle \
    --outfile=dist/extension.js \
    --format=cjs \
    --platform=browser \
    --external:vscode \
    --sourcemap
```

**Why `--format=cjs`?** VSCode Web's extension host uses `_loadCommonJSModule`. ESM bundles will fail to load silently. Always use CJS.

**Output:** `dist/extension.js` — this is the file VSCode loads.

**Verify the build worked:**
```bash
ls -lh dist/extension.js   # should be non-trivial size, e.g. 15-50KB
```

---

## 5. Standard Build Script for Extensions (`package.json`)

Add these scripts to `package.json` in `dnw-workspace-client`. This becomes the canonical way to build, watch, and launch the extension for development.

```json
{
  "scripts": {
    "build": "esbuild src/extension.ts --bundle --outfile=dist/extension.js --format=cjs --platform=browser --external:vscode --sourcemap",
    "build:prod": "esbuild src/extension.ts --bundle --outfile=dist/extension.js --format=cjs --platform=browser --external:vscode --minify",
    "watch": "esbuild src/extension.ts --bundle --outfile=dist/extension.js --format=cjs --platform=browser --external:vscode --sourcemap --watch",
    "launch": "npm run build && node $VSCODE_WEB_PATH/node_modules/@vscode/test-web/out/server/index.js --sourcesPath $VSCODE_WEB_PATH --port 9001 --browserType none --extensionPath $(pwd) --folder-uri dnw:/",
    "launch:watch": "concurrently \"npm run watch\" \"npm run serve\"",
    "serve": "node $VSCODE_WEB_PATH/node_modules/@vscode/test-web/out/server/index.js --sourcesPath $VSCODE_WEB_PATH --port 9001 --browserType none --extensionPath $(pwd) --folder-uri dnw:/"
  }
}
```

Set the environment variable once in your shell profile:

```bash
export VSCODE_WEB_PATH=~/vscode-clean
```

**Workflow:**

```bash
# Build and launch in one step
npm run launch

# Or: watch for TypeScript changes, serve separately
npm run launch:watch
```

For `launch:watch`, install `concurrently` first:

```bash
npm install --save-dev concurrently
```

---

## 6. Incremental Build + Run Script Patched into VSCode Web

Add this script to `~/vscode-clean/package.json` so it becomes part of the VSCode project's own scripts. This is the canonical way to build-and-run from within the VSCode source tree.

In `~/vscode-clean/package.json`, under `"scripts"`, add:

```json
"dnw:serve": "node node_modules/@vscode/test-web/out/server/index.js --sourcesPath . --port 9001 --browserType none --extensionPath $DNW_EXTENSION_PATH --folder-uri dnw:/",
"dnw:build-serve": "npm run compile && npm run dnw:serve",
"dnw:watch-serve": "concurrently \"npm run watch\" \"npm run dnw:serve\""
```

Set the environment variable:

```bash
export DNW_EXTENSION_PATH=/path/to/dnw-workspace-client
```

**Usage:**

```bash
cd ~/vscode-clean

# Full incremental recompile, then serve
npm run dnw:build-serve

# Watch mode: recompile on file changes, serve stays running
# (you reload the browser after changes appear)
npm run dnw:watch-serve

# Serve only (if you know nothing changed in VSCode itself)
npm run dnw:serve
```

Or as a standalone shell script `scripts/dnw-dev.sh` if you prefer not to touch `package.json`:

```bash
#!/usr/bin/env bash
set -e

VSCODE_ROOT="${VSCODE_ROOT:-$HOME/vscode-clean}"
DNW_EXT="${DNW_EXTENSION_PATH:-/mnt/e/dnw/dnw-workspace-client}"
PORT="${DNW_PORT:-9001}"

echo "==> Compiling VSCode..."
cd "$VSCODE_ROOT"
npm run compile

echo "==> Starting dev server on port $PORT..."
node node_modules/@vscode/test-web/out/server/index.js \
    --sourcesPath . \
    --port "$PORT" \
    --browserType none \
    --extensionPath "$DNW_EXT" \
    --folder-uri dnw:/
```

Make it executable:

```bash
chmod +x scripts/dnw-dev.sh
./scripts/dnw-dev.sh
```

---

## 7. Shipping Extensions as Built-In Extensions

VSCode Web distinguishes between three ways to ship an extension:

- **Sideloaded** (`--extensionPath`): loaded at runtime by the dev server, not compiled in. Best for development.
- **Bundled** (`out/extensions/`): compiled into VSCode Web's output, shipped as part of the build. Ships with the product.
- **Marketplace**: not applicable for DNW's private use.

To ship `dnw-workspace-client` as a built-in extension baked into the VSCode Web build:

### Step 1: Copy the extension into VSCode's extensions directory

```bash
cp -r /path/to/dnw-workspace-client ~/vscode-clean/extensions/dnw-workspace-client
```

The directory name becomes the extension's internal ID. The `package.json` inside must have a matching `name` field.

### Step 2: Register it in the extensions list

Open `~/vscode-clean/build/builtInExtensions.json`. Add an entry:

```json
{
  "name": "dnw-workspace-client",
  "version": "0.0.1",
  "repo": ""
}
```

For a purely local extension (no remote repo), the `repo` field can be empty or omitted.

### Step 3: Ensure the extension has a compiled entry point

The extension directory must contain its compiled output at the path specified in `package.json`'s `browser` (for web) or `main` (for desktop) field.

For `dnw-workspace-client`, build it first:

```bash
cd ~/vscode-clean/extensions/dnw-workspace-client
npx esbuild src/extension.ts \
    --bundle \
    --outfile=dist/extension.js \
    --format=cjs \
    --platform=browser \
    --external:vscode
```

### Step 4: Compile VSCode Web

```bash
cd ~/vscode-clean
npm run compile
```

The extension in `extensions/dnw-workspace-client/` will be included automatically.

### Step 5: Serve (no `--extensionPath` needed)

```bash
node node_modules/@vscode/test-web/out/server/index.js \
    --sourcesPath . \
    --port 9001 \
    --browserType none \
    --folder-uri dnw:/
```

The extension is now part of the build — no external path required.

### Built-in extension considerations

- The extension's `package.json` `activationEvents` should include `"*"` to ensure activation on startup, or the specific events your extension listens for.
- Built-in extensions cannot be disabled by the user the same way marketplace extensions can.
- You must re-run `npm run compile` in `~/vscode-clean` after modifying the extension source — the build does not watch the `extensions/` subdirectory automatically.
- For active development, still use `--extensionPath` sideloading (step 3 above). Switch to built-in only for packaging a release.

---

## Patches

Patches live in `~/vscode-clean/patches/` and are applied with `git apply` before compilation.

| Patch file | What it does | Required for |
|---|---|---|
| `reporter-patch.diff` | Makes TypeScript errors non-fatal in the gulp build | `npm run gulp vscode-web` only |

Future patches (Step 2 — not yet applied):
- Remove Copilot extension from build
- Remove GitHub auth extension
- Disable telemetry
- Remove unneeded extensions

Apply all patches at once:

```bash
cd ~/vscode-clean
for p in patches/*.diff; do git apply "$p"; done
```

---

## WSL2 / NTFS Performance Note

If your code lives on an NTFS mount (e.g. `/mnt/e/...`), `npm run compile` and the dev server will be significantly slower due to NTFS-over-WSL2 I/O overhead.

**Fix:** rsync the code to native ext4 before building.

```bash
rsync -av --delete /mnt/e/dnw/dnw-workspace-client/ ~/dnw-workspace-client/
```

Then point `--extensionPath` at `~/dnw-workspace-client` instead of the NTFS path. Run the rsync before launching the dev server whenever you've made changes on the Windows side.

---

## Quick Reference


# Fresh build (once)
```bash
cd ~/vscode-clean
ELECTRON_SKIP_BINARY_DOWNLOAD=1 PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1 npm install
git apply patches/reporter-patch.diff
npm run compile
```

# Incremental build
```bash
cd ~/vscode-clean && npm run compile
```

# Build extension
```bash
cd /path/to/dnw-workspace-client && npm run build
```

# Run dev server
```bash
cd ~/vscode-clean && npm run dnw:serve
```

# Build extension + run dev server together
```bash
cd ~/vscode-clean && npm run dnw:build-serve
```
# DNW -- Developer Environment Setup

Target reader: senior developer who knows Python, TypeScript, C#, Linux, Windows/macOS, VSCode extensions, REST APIs. This document gets you from zero to a running local mesh in roughly two hours, assuming dependencies install without drama.

## Platform

Target and officially supported development platform: Ubuntu 26.04 LTS (Resolute Raccoon).

WSL2 on Windows is Linux from the toolchain's perspective and is fully supported. Run Ubuntu 26.04 as your WSL2 distribution.

macOS is best-effort -- most things work without modification, nothing is guaranteed.

Windows native is single-service debugging only. Run one service in your IDE of choice (Rider, PyCharm, WebStorm) pointed at a mesh running in WSL2. Do not attempt to run the full process-compose mesh on Windows native. You can try, it's not guaranteed to work or continue working.

---

## Prerequisites

Install these. Versions matter.

| Tool            | Version    | Platform | Notes                                                          |
|-----------------|------------|----------|----------------------------------------------------------------|
| Python          | 3.14       | All      | pyenv recommended                                             |
| Node.js         | 24.x       | All      | Use nvm. `nvm install 24 && nvm use 24`                       |
| .NET SDK        | 10.x       | All      | See below. Required for PBX and The Clacks only.              |
| Git             | any recent | All      |                                                               |
| process-compose | latest     | All      | Binaries included — see below                                 |

### Python

```bash
# Ubuntu 26.04 / WSL2 — pyenv recommended
curl https://pyenv.run | bash
pyenv install 3.14
pyenv global 3.14
python --version   # should print 3.14.x
```

### Node.js

```bash
# All platforms — nvm
nvm install 24
nvm use 24
node --version   # should print v24.x.x
```

### .NET SDK 10

Required only if working on PBX or The Clacks (C# / YARP services) or performing a full build of the mesh.

On Ubuntu 26.04, .NET 10 is in the main archive — no PPAs or Microsoft feeds required:

```bash
sudo apt update
sudo apt install dotnet-sdk-10.0
dotnet --version   # should print 10.x.x
```

### process-compose

Binaries live in `dnw-container/bin/` — you do not need to install it globally.

| File                    | Platform |
|-------------------------|----------|
| `process-compose-linux` | Linux    |
| `process-compose-macos` | macOS    |
| `process-compose.exe`   | Windows  |

If you need a fresh binary: https://github.com/F1bonacc1/process-compose/releases. Check with your lead before committing a new version — the mesh is sensitive to changes in process-compose.
---

## Repository Layout

Everything lives under a single root directory. On Windows this is typically `%USERHOME%\dnw`. On Linux/macOS, wherever you put it -- `~/dnw` is fine. This path is `DNW_ROOT` throughout all scripts and documentation.

```
dnw/                                            ← DNW_ROOT
├── .venv-linux/                                ← Python venv for Linux/macOS
├── .venv-windows/                              ← Python venv for Windows (binary wheels differ)
├── .env                                        ← shared config -- safe to commit, contains no secrets
├── requirements.txt                            ← shared Python dependencies
├── dnw-build/                                  ← build scripts for everything
├── dnw-container/                              ← process-compose configs, binaries, launch scripts
│   ├── bin/
│   │   ├── process-compose-linux
│   │   ├── process-compose-macos
│   │   └── process-compose.exe
│   ├── compose.yaml                            ← single compose file for all services
│   ├── run.sh                                  ← Linux/macOS launch script
│   └── run.ps1                                 ← Windows launch script
├── dnw-workspace-engine/                       ← REST API backend for the editor filesystem
│   ├── backend/src/
│   ├── file-service-provider-extension/src/    ← VSCode Web extension (FileSystemProvider)
│   ├── manuscript-view-extension/src/          ← VSCode Web extension (manuscript view))
│   ├── lore-view-extension/src/                ← VSCode Web extension (lore view))
│   └── lore-view-extension/src/                ← VSCode Web extension (lore view))
├── dnw-heartbeat/                              ← service health / heartbeat
│   └── backend/src/
├── dnw-library/                                ← shared Python library
│   └── backend/src/
├── dnw-documentation/                          ← ports.md, hosts.md, architecture notes
└── soap-a-grimdark-farce/                      ← a manuscript workspace
```

Service repos follow the pattern `dnw-{service-name}/`. Each service has `backend/src/` and/or `client/src/` depending on what components it has. Not every service has both. The convention is still settling -- consistency within the codebase matters more than any particular choice.

---

## Minimum Repos to Clone

For a working editor stack:

```bash
git clone <remote>/dnw-container
git clone <remote>/dnw-workspace-engine
git clone <remote>/dnw-workspace-client
git clone <remote>/dnw-library
git clone <remote>/dnw-editor
git clone <remote>/dnw-documentation
```

Clone all of them into the same parent directory. That parent directory becomes `DNW_ROOT`. Cloning all dnw-* repos is recommended -- the editor stack is only a small part of the overall mesh, and you will likely need to debug other services at some point.

The entire stack can run on a single machine, but the architecture is designed to scale across multiple hosts. The mesh is a collection of loosely coupled services that communicate over HTTP. Each service has its own repo, and each repo has its own build and deployment process, though generally follows a consistent pattern. The mesh is designed to be resilient to service failures -- if one service goes down, the others continue to operate.

`process-compose` is used for development and deployment, so what you run locally is what will be running in the production mesh.

---

## First-Time Setup

### 1. Create the virtual environment

Binary wheels are platform-specific -- Linux and Windows venvs cannot share packages. Create the venv for your platform from `DNW_ROOT`:

**Linux/macOS:**

```bash
python3.14 -m venv .venv-linux
source .venv-linux/bin/activate
pip install -r requirements.txt
```

**Windows:**

```powershell
py -3.14 -m venv .venv-windows
.\.venv-windows\Scripts\Activate.ps1
pip install -r requirements.txt
```

Both can coexist in the same `DNW_ROOT` -- the `.venv-linux` and `.venv-windows` directories are independent.

### 2. The `.env` file

The `.env` lives at `DNW_ROOT/.env` and is safe to commit -- it contains no secrets and there is no direct ingress to the mesh from outside localhost. All services point at this file.

```env
PYTHONUNBUFFERED=1
LLM_BASE_URL=http://localhost:1234/v1
LLM_API_KEY=sk-whatever
LLM_MODEL=google/gemma-4-26b-a4b
DNW_USER=yourname_<uuid>
DNW_PROJECT=your-project-name_<uuid>
DNW_RUN_MODE=localhost
MAPTILER_API_KEY=<your key>
```

`DNW_ROOT` is **not** in `.env` -- it is injected at runtime by the launch script.

When running a service directly outside of process-compose (e.g. debugging in PyCharm), point the run configuration at `E:\dnw\.env` (or the equivalent path on your platform) as the env file. See the IDE Setup section below.

.env is committed to a private repository. 

.env is committed to our private repositories. It contains topology -- hosts, ports, modes, model names, identifiers. It never contains credentials. If you find yourself wanting to put a secret in .env, the answer is .env.local which is gitignored.

The architecture has no external ingress, so most things that look like secrets (e.g. the LLM API key) are localhost placeholders and belong in .env.

The one current exception is MAPTILER_API_KEY which lives in .env until the mapping API replaces it entirely.

The threat model for credential exposure requires a GitHub compromise, at which point the .env is the least of our problems. Credentials with real blast radius, e.g. production database access, signing keys and auth secrets belong in .env.local which is gitignored. A temporary third-party API key with a thirty-second revocation path does not.

---

## Host File Configuration

The service mesh uses named hosts. In local development all resolve to `127.0.0.1`. You can generate a new development hosts file by running `dnw-container/bin/generate-hosts.py` from `DNW_ROOT`. It will output a list of hostnames to add to your hosts file.

### Windows -- `C:\Windows\System32\drivers\etc\hosts`

Open Notepad as Administrator:

```
127.0.0.1   workspace-engine
127.0.0.1   heartbeat-editor
127.0.0.1   mechanics-proxy
127.0.0.1   commitment
127.0.0.1   in-scene-terminal
127.0.0.1   thesaurus
127.0.0.1   lexicon-check
127.0.0.1   lexicon-watch
127.0.0.1   dependency-walker
```

### Linux / macOS -- `/etc/hosts`

```bash
sudo nano /etc/hosts
```

Add the same entries.

**WSL2 note:** editing the Windows hosts file is sufficient -- WSL2 inherits Windows DNS resolution for `127.0.0.1` mappings. You do not need to edit `/etc/hosts` inside the WSL2 distro separately.

For the authoritative list, see `dnw-documentation/hosts.md`.

---

## Launch the Mesh

Single entry point in `dnw-container/`. The scripts resolve `DNW_ROOT` from their own location -- no hardcoded paths.

### Linux / macOS

```bash
cd $DNW_ROOT/dnw-container
./run.sh
```

`run.sh`:
```bash
#!/usr/bin/env bash
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
export DNW_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"
export PYTHONPATH="${DNW_ROOT}/:${DNW_ROOT}/dnw-library/src:./src:./"

if [[ "$(uname)" == "Darwin" ]]; then
    PC_BIN="./bin/process-compose-macos"
else
    PC_BIN="./bin/process-compose-linux"
fi

cd "${SCRIPT_DIR}"
$PC_BIN -f compose.yaml -L mesh.log
```

### Windows

```powershell
cd $env:DNW_ROOT\dnw-container
.\run.ps1
```

`run.ps1`:
```powershell
$ScriptDir = Split-Path -Parent $MyInvocation.MyCommand.Path
$env:DNW_ROOT = (Resolve-Path "$ScriptDir\..").Path
$env:PYTHONPATH = "$env:DNW_ROOT\;$env:DNW_ROOT\dnw-library\src;.\src;.\"
Set-Location $ScriptDir
.\bin\process-compose.exe -f compose.yaml -L mesh.log
```

### What healthy looks like

process-compose opens a TUI. All services should reach `Running` or `Healthy` within 60–90 seconds. The workspace engine is the dependency anchor -- if it fails, most other services will not start.

Key log line:
```
[INFO]-[workspace-engine] - Workspace Engine Service ready -- version 1.0.0 on port 2040
```

Logs are also written to `dnw-container/mesh.log`. To follow without the TUI:
```bash
tail -f dnw-container/mesh.log
```

For port assignments, see `dnw-documentation/ports.md`.

---

## Open the Editor

Once the mesh is healthy:

```bash
cd ~/vscode-clean    # wherever you cloned vscode
npm run dnw:build-serve
```

Open: **http://localhost:9001**

The file explorer should show the manuscript workspace within a few seconds. If it's empty, check the workspace engine first:

```
http://localhost:2040/monitoring/health
```

For full VSCode Web build instructions, see `DNW-EDITOR-BUILD.md`.

---

## IDE Setup (PyCharm)

When debugging a service directly in PyCharm rather than through process-compose, match this run configuration:

| Field | Value |
|---|---|
| Python interpreter | `E:\dnw\.venv-windows\Scripts\python.exe` (or `.venv-linux` equivalent) |
| Script | `dnw-{service}/backend/src/main.py` |
| Working directory | The manuscript workspace root, e.g. `E:\dnw\soap-a-grimdark-farce` |
| Environment variables | `PYTHONUNBUFFERED=1` |
| Paths to `.env` files | `E:\dnw\.env` |

PyCharm's "Add content roots to PYTHONPATH" and "Add source roots to PYTHONPATH" checkboxes can remain enabled -- they don't conflict with the `.env` setup.

The `.env` file path needs to be the absolute path to `DNW_ROOT/.env` on your machine. PyCharm does not resolve this from a variable.

Bring up the mesh with the `process-compose` TUI first, then stop the service `F7` that you want to debug, and run it in PyCharm. The service will connect to the rest of the mesh as long as the other services are still running. Almost all services can be run standalone with no other parts of the mesh present. If a service needs to talk to another service, it will fail gracefully if the other service is not running and announce itself in the logs. 

---

## Stopping the Mesh

In the TUI: `q` or `Ctrl+C` or `F10`.

From the command line:
```bash
# Linux/macOS
./bin/process-compose-linux down

# Windows
.\bin\process-compose.exe down
```

---

## Quick Reference

```bash
# Start mesh (Linux/macOS)
cd $DNW_ROOT/dnw-container && ./run.sh

# Start mesh (Windows)
cd $env:DNW_ROOT\dnw-container; .\run.ps1

# Check workspace engine health
curl http://localhost:2040/monitoring/health

# Open editor
http://localhost:9001

# Tail logs
tail -f dnw-container/mesh.log

# Build extension only
cd $DNW_ROOT/dnw-workspace-client && npm run build

# Build and serve VSCode Web
cd ~/vscode-clean && npm run dnw:build-serve

# Activate venv (Linux/macOS)
source $DNW_ROOT/.venv-linux/bin/activate

# Activate venv (Windows)
$env:DNW_ROOT\.venv-windows\Scripts\Activate.ps1
```

Every service in the mesh has an accompanying `.http` file in the backend directory. These can be run in VSCode or Pycharm with the REST Client plugin. They are useful for testing service endpoints and debugging. All services also have a `/monitoring/health` endpoint that returns a JSON object with the service status. This is useful for checking if a service is running and healthy.

Note that until CSP is fixed in the VSCode Web deployment, the Workspace Engine and Workspace Client must be run on the same host, and the Workspace Client must use localhost for lookup.


## 8. Launch Script — `run-editor.sh` / `run-editor.ps1`

These scripts replace the arcane `@vscode/test-web` incantation with a single command that builds the extension first and then starts the dev server.

**What they do:**

1. Resolve `VSCODE_WEB_PATH` from a known location relative to `DNW_ROOT`, or from an explicit env var.
2. Build the extension (`npm run build` in `dnw-workspace-client`).
3. Launch the dev server with the correct flags.
4. Print the URL to open when ready.

**Usage:**

```bash
# Linux / macOS / WSL2
./dnw-container/run-editor.sh

# Windows (PowerShell)
.\dnw-container\run-editor.ps1
```

**Environment variables (all optional):**

| Variable | Default | Purpose |
|---|---|---|
| `DNW_ROOT` | Parent of `dnw-container/` | Root of the DNW monorepo |
| `VSCODE_WEB_PATH` | `../vscode-clean` or `~/vscode-clean` | Path to the compiled VSCode Web checkout |
| `EXTENSION_DIR` | `$DNW_ROOT/dnw-workspace-client` | Path to the workspace client extension |
| `DNW_EDITOR_PORT` | `9001` | Port for the dev server |
| `DNW_FOLDER_URI` | `dnw:/` | Folder URI to open in the editor |

**Resolution order for `VSCODE_WEB_PATH`:**

1. Explicit `VSCODE_WEB_PATH` env var.
2. `vscode-clean` as a sibling of `DNW_ROOT` (i.e. `E:/vscode-clean` if `DNW_ROOT` is `E:/dnw`).
3. `~/vscode-clean`.
4. Error with a clear message.

---

## 9. `dnw:launch` script in `vscode-clean/package.json`

Add this to the `scripts` block in `vscode-clean/package.json` (alongside the existing `dnw:serve` and `dnw:build-serve` entries):

```json
"dnw:launch": "node node_modules/@vscode/test-web/out/server/index.js --sourcesPath . --port ${DNW_EDITOR_PORT:-9001} --browserType none --extensionPath ${EXTENSION_DIR} --folder-uri ${DNW_FOLDER_URI:-dnw:/}"
```

> **Note:** This assumes `EXTENSION_DIR` is set. In practice, use `run-editor.sh` / `run-editor.ps1` rather than calling this directly — they handle the build step and env resolution. The `dnw:launch` entry exists so that `run-editor.sh` can delegate to it if preferred, and so the incantation is documented in one place.

---

## 10. Build All Extensions — `build-extensions.sh` / `build-extensions.ps1`

These scripts iterate the known extension directories under `DNW_ROOT` and run `npm run build` in each. Place them in `dnw-container/`.

**Usage:**

```bash
# Linux / macOS / WSL2
./dnw-container/build-extensions.sh

# Windows (PowerShell)
.\dnw-container\build-extensions.ps1
```

**Behaviour:**

- Skips directories that don't exist or have no `package.json` (graceful, not fatal).
- Reports pass/fail per extension.
- Exits non-zero if any extension fails.
- Prints a summary at the end.

**Adding a new extension:**

Open the script and append the directory name to the `EXTENSIONS` array near the top:

```bash
# build-extensions.sh
EXTENSIONS=(
    "dnw-workspace-client"
    "dnw-your-new-extension"   # ← add here
)
```

```powershell
# build-extensions.ps1
$Extensions = @(
    "dnw-workspace-client",
    "dnw-your-new-extension"   # ← add here
)
```

**Canonical extension build pattern:**

Each extension's `package.json` must have a `build` script that produces a deployable `dist/extension.js`. The `dnw-workspace-client` pattern is the reference:

```json
"scripts": {
    "build": "esbuild src/extension.ts --bundle --outfile=dist/extension.js --format=cjs --platform=browser --external:vscode --sourcemap",
    "build:prod": "esbuild src/extension.ts --bundle --outfile=dist/extension.js --format=cjs --platform=browser --external:vscode --minify",
    "watch": "npm run build -- --watch"
}
```

Key constraints:
- `--format=cjs` — required. VSCode Web extension host uses `_loadCommonJSModule`.
- `--platform=browser` — required. Node built-ins must not be bundled.
- `--external:vscode` — required. The `vscode` module is provided by the host.

---

## Updated Quick Reference

```bash
# Fresh build (once)
cd ~/vscode-clean
ELECTRON_SKIP_BINARY_DOWNLOAD=1 PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1 npm install
git apply patches/reporter-patch.diff
npm run compile

# Incremental build
cd ~/vscode-clean && npm run compile

# Build single extension
cd /path/to/dnw-workspace-client && npm run build

# Build all extensions
./dnw-container/build-extensions.sh

# Run dev server (build extension + launch)
./dnw-container/run-editor.sh

# Run dev server only (extension already built)
cd ~/vscode-clean && npm run dnw:launch
```

---

## Still Needs Writing

- `run-editor.sh` and `run-editor.ps1` committed to `dnw-container/` ✓ *(scripts produced, need committing)*
- `build-extensions.sh` and `build-extensions.ps1` committed to `dnw-container/` ✓ *(scripts produced, need committing)*
- `dnw:launch` added to `vscode-clean/package.json` scripts block
- rsync script: NTFS → ext4 before launching, for WSL2 developers working from Windows paths
- Step 2 quilt patches: remove Copilot, GitHub auth, telemetry from VSCode build
- CSP fix documentation: `workspace-engine:2040` blocked in dev; use `localhost:2040` as workaround
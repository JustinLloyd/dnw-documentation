# DNW -- Developer Environment Setup

Target reader: senior developer who knows Python, TypeScript, C#, Linux, Windows/macOS, VSCode extensions, REST APIs. This document gets you from zero to a running local mesh in roughly two hours, assuming dependencies install without drama.

---

## Platform

Target and officially supported development platform: Ubuntu 26.04 LTS (Resolute Raccoon).

WSL2 on Windows is Linux from the toolchain's perspective and is fully supported. Run Ubuntu 26.04 as your WSL2 distribution.

macOS is best-effort -- most things work without modification, nothing is guaranteed.

Windows native is single-service debugging only. Run one service in your IDE of choice (Rider, PyCharm, WebStorm) pointed at a mesh running in WSL2. Do not attempt to run the full process-compose mesh on Windows native. You can try, it's not guaranteed to work or continue working.

---

## Prerequisites

Install these. Versions matter.

| Tool            | Version    | Platform       | Notes                                                         |
|-----------------|------------|----------------|---------------------------------------------------------------|
| Python          | 3.14       | Linux / WSL2   | pyenv recommended                                            |
| Python          | 3.14       | Windows native | python.org installer. Single-service debugging only.         |
| Node.js         | 24.x       | All            | Use nvm. `nvm install 24 && nvm use 24`                      |
| .NET SDK        | 10.x       | All            | See below. Required for PBX and The Clacks only.             |
| Git             | any recent | All            |                                                              |
| process-compose | latest     | Linux / WSL2   | Binaries included — see below                                |

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

The binary is committed to the `dnw-container` repo. If it is missing, check your clone — do not download a new version without checking with your lead first. The mesh is sensitive to changes in process-compose.

---

## Environment Variables

Three environment variables must be set in your shell before anything will work. Add these to your `~/.bashrc` (Linux/WSL2) or `~/.zshrc` (macOS):

```bash
export DNW_ROOT=/mnt/e/dnw                          # parent of all dnw-* repos
export WORKSPACE_ROOT=/mnt/e/dnw/soap-a-grimdark-farce  # the project you are working on
export SYSTEM_ROOT=/mnt/e/dnw/system                # system data directory
```

Adjust paths for your machine. On WSL2 accessing a Windows drive, `/mnt/e/` is the typical prefix.

**These values are machine-specific and should not be listed in `.env`, nor committed anywhere. Every developer needs to set their own values as `run.py` checks for all three at startup and will tell you clearly if any are missing.

Restart your terminal or `source ~/.bashrc` after adding them.

---

## Repository Layout

Everything lives under a single root directory at `DNW_ROOT`.

Cloning repos into arbitrary locations is not supported as the build scripts, run scripts, and service discovery all assume a flat layout.

```
DNW_ROOT/                                       ← e.g. /mnt/e/dnw or ~/dnw
├── .venv-linux/                                ← Python venv for Linux/macOS
├── .venv-windows/                              ← Python venv for Windows (binary wheels differ)
├── .env                                        ← shared config -- safe to commit and contains no secrets
├── .env.local                                  ← secrets, gitignored and never committed
├── requirements.txt                            ← shared Python dependencies
├── logs/                                       ← all service logs land here (gitignored)
├── dnw-build/                                  ← build and run scripts for everything
│   ├── run.py                                  ← start the full mesh
│   ├── run-editor.py                           ← launch editor in extension debug mode
│   ├── build.py                                ← orchestrator -- runs all build steps, e.g. extensions, dotnet, python
│   ├── build-extensions.py                     ← compile and install VSCode extensions
│   ├── build-dotnet.py                         ← build C# / .NET services
│   ├── build-python.py                         ← setup Python services (pip install)
│   ├── lib_build.py                            ← shared build library
│   ├── lib_extensions_list.py                  ← canonical list of all VSCode extensions
│   └── lib_services_list.py                    ← canonical list of all services
├── dnw-container/                              ← process-compose config and binaries
│   ├── bin/
│   │   ├── process-compose-linux
│   │   ├── process-compose-macos
│   │   └── process-compose.exe
│   └── compose.yaml                            ← generated — do not edit directly
├── dnw-editor/                                 ← VSCode Web patch repo (quilt patches + pin)
├── dnw-workspace-engine/                       ← workspace filesystem service + extensions
│   ├── backend/src/
│   ├── file-system-provider-extension/src/     ← VSCode Web extension (FileSystemProvider)
│   ├── manuscript-extension/src/          ← VSCode Web extension (manuscript view)
│   ├── lore-extension/src/                ← VSCode Web extension (lore view)
│   ├── of-note-extension/
│   ├── build-extension/
│   ├── project-extension/
│   └── assets-extension/
├── dnw-library/                                ← shared Python library
├── dnw-heartbeat/                              ← service health monitoring
│   └── backend/src/
├── dnw-documentation/                          ← ports.md, hosts.md, architecture notes
└── soap-a-grimdark-farce/                      ← a manuscript workspace (example)
```

Service repos follow `dnw-{service-name}/`.
Services with both a backend and VSCode extension(s) are co-located in the same repo.

Not every service has both.

`compose.yaml` is generated — do not edit it directly, edit the source topology and regenerate using generate-*.py scripts in dnw-build.

---

## Minimum Repos to Clone

For a working editor stack:

```bash
cd $DNW_ROOT
git clone <remote>/dnw-build
git clone <remote>/dnw-container
git clone <remote>/dnw-workspace-engine
git clone <remote>/dnw-library
git clone <remote>/dnw-editor
git clone <remote>/dnw-documentation
```

Clone all of them into the same parent directory. That parent directory becomes `DNW_ROOT`. Cloning all dnw-* repos is recommended -- the editor stack is only a small part of the overall mesh, and you will likely need to debug other services at some point.

The entire stack can run on a single machine, but the architecture is designed to scale across multiple hosts. The mesh is a collection of loosely coupled services that communicate over HTTP. Each service has its own repo, and generally follows a consistent pattern. The mesh is designed to be resilient to service failures -- if one service goes down, the others continue to operate.

`process-compose` is used for development and deployment, so what you run locally is what will be running in the production mesh.

---

## First-Time Setup

### 1. Set environment variables

Add to `~/.bashrc` and restart your terminal:

```bash
export DNW_ROOT=/path/to/your/dnw
export WORKSPACE_ROOT=/path/to/your/project
export SYSTEM_ROOT=/path/to/your/system
```

### 2. Create the virtual environment
Binary wheels are platform-specific -- Linux and Windows venvs cannot share packages.  Create the venv for your platform from `DNW_ROOT`:

**Linux / WSL2:**

```bash
python3.14 -m venv .venv-linux
source .venv-linux/bin/activate
pip install -r requirements.txt
```

**Windows native (single-service debugging only):**

```powershell
py -3.14 -m venv .venv-windows
.\.venv-windows\Scripts\Activate.ps1
pip install -r requirements.txt
```

Both can coexist in the same `DNW_ROOT` -- the `.venv-linux` and `.venv-windows` directories are independent.

### 3. Set up Node.js dependencies

```bash
cd $DNW_ROOT
./dnw-build/build-extensions.py setup    # npm install in every extension
```

### 4. Compile extensions

```bash
./dnw-build/build-extensions.py compile  # npm run compile in every extension
```

### 5. Install extensions into dnw-editor

```bash
./dnw-build/build-extensions.py install  # compile + copy into dnw-editor/extensions/
```

### 6. The `.env` file

`.env` lives at `DNW_ROOT/.env` and is committed to the main dnw-root repo. All the services read from it via process-compose or when debugging.

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

`DNW_ROOT`, `WORKSPACE_ROOT`, and `SYSTEM_ROOT` are **not** in `.env` — they are machine-specific and live in `~/.bashrc`.

.env is committed to our private repositories. It contains topology -- hosts, ports, modes, model names, identifiers.

The threat model for this codebase has no external ingress, so most things that look like secrets (e.g. the LLM API key) are localhost placeholders and belong in .env.

The one current exception is MAPTILER_API_KEY which lives in .env until the mapping API replaces it entirely.

Credential exposure requires a GitHub compromise, at which point the .env is the least of our problems. Credentials with real blast radius, e.g. production database access, signing keys and auth secrets belong in .env.local which is gitignored. A temporary third-party API key with a thirty-second revocation path does not.

---

## Host File Configuration

The service mesh uses named hosts. In local development they should all resolve to `127.0.0.1`. You can generate a fresh hosts file:

```bash
python $DNW_ROOT/dnw-build/generate-hosts.py
```

Which will place a new hosts file in the current directory. 

### Linux / macOS / WSL2 — `/etc/hosts`

```bash
sudo nano /etc/hosts
```

**WSL2 note:** editing the Windows hosts file at `C:\Windows\System32\drivers\etc\hosts` is sufficient — WSL2 inherits Windows DNS resolution. You do not need to edit `/etc/hosts` inside the WSL2 distro separately.

For the authoritative list, see `dnw-documentation/hosts.md`.

---

## Launch the Mesh

The entry point is `dnw-build/run.py`. It performs pre-flight checks, sets the environment, and hands off to process-compose.

```bash
cd $DNW_ROOT
./dnw-build/run.py
```

Pre-flight checks (all must pass before process-compose starts):

- `DNW_ROOT` is set
- `WORKSPACE_ROOT` is set and exists on disk
- `SYSTEM_ROOT` is set
- `DNW_ROOT/.env` exists
- process-compose binary is present
- `dnw-container/compose.yaml` exists

If any check fails, `run.py` reports all failures at once with specific instructions for each.

### What healthy looks like

process-compose opens a TUI. All services should reach `Running` or `Healthy` within 60–90 seconds.


The workspace engine is the dependency anchor — if it fails, most other services will not start.

Key log line:
```
[INFO]-[workspace-engine] - Workspace Engine Service ready — version 1.0.0 on port 2040
```

Logs are written to `DNW_ROOT/logs/mesh.log`:
```bash
tail -f $DNW_ROOT/logs/mesh.log
```

For port assignments, see `dnw-documentation/ports.md`.

---

## Open the Editor

Once the mesh is healthy, open:

**http://localhost:2600**

The file explorer should show the manuscript workspace within a few seconds. If it is empty, check the workspace engine:

```
http://localhost:2040/monitoring/health
```

For full VSCode Web build instructions, see `building-the-editor.md`.

---

## Debugging a Single Service

The standard workflow:

1. Start the full mesh: `./dnw-build/run.py`
2. In the process-compose TUI, stop the service you want to debug (`F7`)
3. Start that service in your IDE — it connects to the rest of the running mesh
4. When done, restart it in the TUI or restart the mesh

Almost all services fail gracefully if dependencies are unavailable and announce the failure in their logs.

### PyCharm run configuration

| Field                 | Value                                    |
|-----------------------|------------------------------------------|
| Python interpreter    | `$DNW_ROOT/.venv-{platform}}/bin/python` |
| Script                | `dnw-{service}/backend/src/main.py`      |
| Working directory     | `$DNW_ROOT/dnw-{service}/backend`        |
| Environment variables | `PYTHONUNBUFFERED=1`                     |
| Paths to `.env` files | `$DNW_ROOT/.env`                         |

PyCharm's "Add content roots to PYTHONPATH" and "Add source roots to PYTHONPATH" can remain enabled.

The `.env` file path needs to be the absolute path to `DNW_ROOT/.env` on your machine. PyCharm does not resolve this from a variable in this field.


## Stopping the Mesh

In the TUI: `q` or `Ctrl+C` or `F10`.

From the command line:
```bash
# Linux/macOS
./bin/process-compose-linux down

---

## Debugging a VSCode Extension

To debug an extension using the VSCode Extension Development Host:

1. Start the full mesh: `./dnw-build/run.py`
2. Stop the editor process in the TUI (`F7` on `vscode-editor`)
3. Run the editor in debug mode:

```bash
./dnw-build/run-editor.py --extension dnw-workspace-client
```

This launches VSCode with `--extensionDevelopmentPath` pointing at the named extension. Set breakpoints in the extension source — the Extension Development Host connects automatically.

---

## dnw-build — Build Scripts for Developers

`dnw-build/` is the build and run tooling for the entire mesh. All scripts are Python with a shebang — run them directly.

### Entry points

| Script | Purpose |
|---|---|
| `run.py` | Start the full mesh via process-compose |
| `run-editor.py` | Launch editor in extension debug mode |
| `build.py` | Orchestrator — runs all build steps in order |
| `build-extensions.py` | Compile and install VSCode extensions |
| `build-dotnet.py` | Build C# / .NET services (PBX, The Clacks) |
| `build-python.py` | Set up Python services (`pip install`) |

### `build-extensions.py` commands

```bash
./dnw-build/build-extensions.py setup    # npm install in every extension (run once after clone)
./dnw-build/build-extensions.py compile  # npm run compile in every extension
./dnw-build/build-extensions.py install  # compile + copy into dnw-editor/extensions/

# Target a single extension
./dnw-build/build-extensions.py compile --only dnw-workspace-client
./dnw-build/build-extensions.py compile --dry-run
```

### Shared libraries for custom scripts

If you need to write a one-off build or automation script, import from the shared libraries:

```python
from lib_build import resolve_dnw_root, get_log_path
from lib_extensions_list import EXTENSIONS
from lib_services_list import PYTHON_SERVICES, DOTNET_SERVICES, ALL_SERVICES

dnw_root = resolve_dnw_root()   # exits with clear error if DNW_ROOT not set
```

`lib_build.py` — environment resolution, log path helper. Not a script, import only.
`lib_extensions_list.py` — canonical list of all VSCode extensions. Add new extensions here.
`lib_services_list.py` — canonical lists of Python and .NET services. Add new services here.

These lists are the single source of truth. The build scripts all read from them — you do not need to update scripts individually when adding a new extension or service.

---

## Canonical Extension Build Pattern

Each extension's `package.json` must have a `compile` script producing `dist/extension.js`:

```json
"scripts": {
    "compile": "node esbuild.mjs",
    "watch": "node esbuild.mjs --watch"
}
```

`esbuild.mjs` must use `format: 'cjs'` and `platform: 'browser'`:

```javascript
await esbuild.build({
    entryPoints: ['src/extension.ts'],
    bundle: true,
    outfile: 'dist/extension.js',
    format: 'cjs',           // required — VSCode Web extension host uses _loadCommonJSModule
    platform: 'browser',     // required — no Node built-ins
    external: ['vscode'],    // required — provided by the host
    sourcemap: true,
});
```

Common mistakes:
- `format: 'esm'` — produces `export {}` at the top level, extension host rejects it
- Missing `--external:vscode` — bundles the vscode stub, causes runtime errors
- `platform: 'node'` — bundles Node built-ins that do not exist in the browser host

---

## Stopping the Mesh

In the TUI: `q`, `Ctrl+C`, or `F10`.

From the command line:
```bash
$DNW_ROOT/dnw-container/bin/process-compose-linux down
```

---

## Quick Reference

```bash
# Set required environment variables (add to ~/.bashrc)
export DNW_ROOT=/mnt/e/dnw
export WORKSPACE_ROOT=/mnt/e/dnw/soap-a-grimdark-farce
export SYSTEM_ROOT=/mnt/e/dnw/system

# First-time setup
source .venv-linux/bin/activate
pip install -r requirements.txt
./dnw-build/build-extensions.py setup
./dnw-build/build-extensions.py install

# Start mesh
./dnw-build/run.py

# Check workspace engine health
curl http://localhost:2040/monitoring/health

# Open editor
http://localhost:2600

# Tail logs
tail -f $DNW_ROOT/logs/mesh.log

# Compile a single extension
./dnw-build/build-extensions.py compile --only dnw-workspace-client

# Debug an extension
./dnw-build/run-editor.py --extension dnw-workspace-client

# Activate venv
source $DNW_ROOT/.venv-linux/bin/activate
```

Every service has a `/monitoring/health` endpoint and an accompanying `.http` file in its backend directory for testing endpoints directly in VSCode or PyCharm with the REST Client plugin.

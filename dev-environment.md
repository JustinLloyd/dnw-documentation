# DNW -- Developer Environment Setup

Target reader: senior developer who knows Python, TypeScript, C#, Linux, Windows/macOS, VSCode extensions, REST APIs. This document gets you from zero to a running local mesh in roughly two hours, assuming dependencies install without drama.

---

## Prerequisites

Install these. Versions matter.

| Tool            | Version    | Notes                                                         |
|-----------------|------------|---------------------------------------------------------------|
| Python          | 3.14       | Windows: python.org installer. Linux/macOS: pyenv recommended |
| Node.js         | 24.x       | Use nvm. `nvm install 24 && nvm use 24`                       |
| Git             | any recent |                                                               |
| process-compose | latest     | Binaries included -- see below                                |

**process-compose** binaries live in `dnw-container/bin/` -- you do not need to install it globally.

| File                    | Platform |
|-------------------------|----------|
| `process-compose-linux` | Linux    |
| `process-compose-macos` | macOS    |
| `process-compose.exe`   | Windows  |

If you need a fresh binary: https://github.com/F1bonacc1/process-compose/releases. Check with your lead before committing a new version -- the mesh is sensitive to changes in process-compose.

---

## Repository Layout

Everything lives under a single root directory. On Windows this is typically `%USERHOME%\dnw`. On Linux/macOS, wherever you put it -- `~/dnw` is fine. This path is `DNW_ROOT` throughout all scripts and documentation.

```
dnw/                            ← DNW_ROOT
├── .venv-linux/                ← Python venv for Linux/macOS
├── .venv-windows/              ← Python venv for Windows (binary wheels differ)
├── .env                        ← shared config -- safe to commit, contains no secrets
├── requirements.txt            ← shared Python dependencies
├── dnw-container/              ← process-compose configs, binaries, launch scripts
│   ├── bin/
│   │   ├── process-compose-linux
│   │   ├── process-compose-macos
│   │   └── process-compose.exe
│   ├── compose.yaml            ← single compose file for all services
│   ├── run.sh                  ← Linux/macOS launch script
│   └── run.ps1                 ← Windows launch script
├── dnw-workspace-engine/       ← REST API backend for the editor filesystem
│   └── backend/src/
├── dnw-workspace-client/       ← VSCode Web extension (FileSystemProvider)
│   └── client/src/
├── dnw-heartbeat/              ← service health / heartbeat
│   └── backend/src/
├── dnw-library/                ← shared Python library
│   └── backend/src/
├── dnw-documentation/          ← ports.md, hosts.md, architecture notes
└── soap-a-grimdark-farce/      ← a manuscript workspace
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

.env is committed. It contains topology -- hosts, ports, modes, model names, identifiers. It never contains credentials. If you find yourself wanting to put a secret in .env, the answer is .env.local which is gitignored. The architecture has no external ingress, so most things that look like secrets (e.g. the LLM API key) are localhost placeholders and belong in .env. The one current exception is MAPTILER_API_KEY which lives in .env.local until the mapping API replaces it entirely.

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
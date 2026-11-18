# Project Cheat Sheet & User Guide (`using_project.md`)

A complete, dual-platform cheat sheet for running **mystatic** on **Linux** and **Windows**. This guide walks through installing all dependencies (Python, Ollama, Git, OpenSSL), launching the AI server, and using the CLI client, Web UI, modules, and security scripts.

---

## Table of Contents
1. [System Architecture Overview](#system-architecture-overview)
2. [Linux Setup Guide](#linux-setup-guide)
   - [1. System Packages & Python](#1-system-packages--python)
   - [2. Install & Start Ollama](#2-install--start-ollama)
   - [3. Project Setup & Virtual Environment](#3-project-setup--virtual-environment)
   - [4. Generate TLS/SSL Certificates](#4-generate-tlsssl-certificates)
   - [5. Run the Project on Linux](#5-run-the-project-on-linux)
3. [Windows Setup Guide](#windows-setup-guide)
   - [1. Install Python](#1-install-python)
   - [2. Install & Start Ollama](#2-install--start-ollama-on-windows)
   - [3. Install Git & Clone](#3-install-git--clone)
   - [4. Project Setup & Virtual Environment (PowerShell / CMD)](#4-project-setup--virtual-environment-powershell--cmd)
   - [5. SSL Certificates on Windows](#5-ssl-certificates-on-windows)
   - [6. Run the Project on Windows](#6-run-the-project-on-windows)
4. [Cheat Sheet: How to Use the Project Files](#cheat-sheet-how-to-use-the-project-files)
   - [Using `client.py` (CLI Client)](#using-clientpy-cli-client)
   - [Using the Web Chat UI (`serve_https.py` / `www/`)](#using-the-web-chat-ui-serve_httpspy--www)
   - [Using Pluggable Modules (`modules/`)](#using-pluggable-modules-modules)
   - [Using AI Skills (`skills/`)](#using-ai-skills-skills)
   - [Network Defense & Diagnostic Scripts (`scripts/`)](#network-defense--diagnostic-scripts-scripts)
5. [Configuration Reference (`config.json`)](#configuration-reference-configjson)
6. [Troubleshooting & Common Fixes](#troubleshooting--common-fixes)

---

## System Architecture Overview

```
                      ┌───────────────────────────────────────────────┐
                      │              User Interfaces                  │
                      │   [Web UI Browser]       [Terminal CLI]       │
                      │    (HTTPS :8443)          (client.py)         │
                      └───────────────┬──────────────────────┬────────┘
                                      │                      │
                                      ▼                      ▼
                       ┌─────────────────────────┐           │
                       │ Static & Proxy Server   │           │
                       │ (serve_https.py :8443)  │           │
                       └──────────────┬──────────┘           │
                                      │                      │
                                      ▼                      ▼
    ┌────────────────────────────────────────────────────────────────────────┐
    │          FastAPI Local AI Server (ai_server.py :8000 / :8444)          │
    │  - Tool-calling agent (run_command, read_file, write_file, edit_file)  │
    │  - SQLite session & message storage (history.db)                       │
    │  - Modular pre-processors (modules/) & tactical skills (skills/)       │
    └───────────────────────────────────┬────────────────────────────────────┘
                                        │
                                        ▼
                        ┌───────────────────────────────┐
                        │      Local LLM Engine         │
                        │  Ollama Daemon (:11434)       │
                        │  (llama3.2:3b / mistral / etc)│
                        └───────────────────────────────┘
```

---

## Linux Setup Guide

Tested on Ubuntu, Debian, Kali Linux, Linux Mint, Fedora, Arch.

### 1. System Packages & Python
Install Python 3, pip, venv, curl, git, and OpenSSL:

```bash
# Debian / Ubuntu / Kali
sudo apt update
sudo apt install -y python3 python3-pip python3-venv curl git openssl

# Fedora / RHEL
sudo dnf install -y python3 python3-pip curl git openssl

# Arch Linux
sudo pacman -S python python-pip curl git openssl
```

### 2. Install & Start Ollama
Ollama runs the local AI model without high memory overhead:

```bash
# 1. Install Ollama via official script
curl -fsSL https://ollama.com/install.sh | sh

# 2. Verify or start the Ollama service
sudo systemctl enable --now ollama
# (If running without systemd, launch it manually in a terminal):
# ollama serve >/dev/null 2>&1 &

# 3. Pull the default fast model (llama3.2:3b recommended)
ollama pull llama3.2:3b

# 4. Verify Ollama is answering
curl http://localhost:11434/api/tags
```

### 3. Project Setup & Virtual Environment
Navigate to the project directory and install Python dependencies:

```bash
cd /var/www/html/python/mystatic   # or your cloned directory

# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Upgrade pip and install server packages
pip install --upgrade pip
pip install fastapi uvicorn pydantic requests
```

### 4. Generate TLS/SSL Certificates
For secure HTTPS web UI and API encryption:

```bash
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout server.key \
  -out server.crt \
  -subj "/CN=localhost"

chmod 600 server.key
chmod 644 server.crt
```

### 5. Run the Project on Linux

#### Terminal 1 — Start the AI Backend:
```bash
source venv/bin/activate
python3 ai_server.py
```
*(Runs FastAPI on `http://0.0.0.0:8000` or HTTPS `https://0.0.0.0:8444`)*

#### Terminal 2 — Start the Web Server & Reverse Proxy:
```bash
source venv/bin/activate
python3 serve_https.py --port 8443
```
*(Serves `www/` and proxies API requests on `https://localhost:8443`)*

#### Terminal 3 — Use the CLI Client:
```bash
source venv/bin/activate
python3 client.py -i
```

---

## Windows Setup Guide

Tested on Windows 10 & Windows 11 (PowerShell & Command Prompt).

### 1. Install Python
1. Download Python 3.10+ from [python.org](https://www.python.org/downloads/) or run in PowerShell:
   ```powershell
   winget install Python.Python.3.11
   ```
2. **Important**: When running the installer, check the box:  
   ☑ **"Add python.exe to PATH"**

### 2. Install & Start Ollama on Windows
1. Download the Windows installer from [ollama.com/download/windows](https://ollama.com/download/windows) or run:
   ```powershell
   winget install Ollama.Ollama
   ```
2. Once installed, Ollama runs automatically in your system tray.
3. Open PowerShell or Command Prompt and download the model:
   ```powershell
   ollama pull llama3.2:3b
   ```
4. Test Ollama:
   ```powershell
   ollama run llama3.2:3b "Hello, are you working?"
   ```

### 3. Install Git & Clone
```powershell
winget install Git.Git
```
Clone the repository:
```powershell
git clone https://github.com/Ok4U-5alid/mystatic.git
cd mystatic
```

### 4. Project Setup & Virtual Environment (PowerShell / CMD)

#### In PowerShell:
```powershell
# If scripts are disabled, temporarily enable them for current session:
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

# Create virtual environment
python -m venv venv

# Activate virtual environment
.\venv\Scripts\Activate.ps1

# Install required packages
python -m pip install --upgrade pip
pip install fastapi uvicorn pydantic requests
```

#### In Command Prompt (CMD):
```cmd
python -m venv venv
venv\Scripts\activate.bat
python -m pip install --upgrade pip
pip install fastapi uvicorn pydantic requests
```

### 5. SSL Certificates on Windows
You can generate `server.crt` and `server.key` using Git Bash (which includes `openssl`):

```bash
# In Git Bash inside the mystatic folder:
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt -subj "//CN=localhost"
```

> [!TIP]
> **HTTP Only Mode**: If you prefer not to use SSL on Windows for local testing, edit `config.json` and set `"ssl": { "enabled": false }`, then connect to `http://localhost:8000`.

### 6. Run the Project on Windows

#### Window 1: Start AI Server
```powershell
.\venv\Scripts\Activate.ps1
python ai_server.py
```

#### Window 2: Start Web Server
```powershell
.\venv\Scripts\Activate.ps1
python serve_https.py --port 8443
```

#### Window 3: Run Interactive CLI Client
```powershell
.\venv\Scripts\Activate.ps1
python client.py -i
```

Open your browser to: **`https://localhost:8443`**  
*(Accept the browser self-signed security prompt to proceed).*

---

## Cheat Sheet: How to Use the Project Files

### Using `client.py` (CLI Client)

`client.py` is the primary terminal interface. It automatically detects whether your server is running on plain HTTP (`8000`) or HTTPS (`8444`).

| Goal | Command |
|---|---|
| **Interactive Chat Session** | `python client.py -i` |
| **Ask One Question (with Tools)** | `python client.py "Check listening ports on this machine"` |
| **Pure LLM Mode (No Tools)** | `python client.py --no-tools "Explain how TCP handshake works"` |
| **Run Input Through a Module** | `python client.py --module math_utils "15 * 48"` |
| **Show Server Status & Tools** | `python client.py --list` |
| **Show Chat History** | `python client.py --history` |
| **List Saved Chat Sessions** | `python client.py --sessions` |
| **Query History with Limit** | `python client.py --history 50` |
| **Override Server URL** | `AI_API_URL="http://localhost:8000" python client.py -i` |

---

### Using the Web Chat UI (`serve_https.py` / `www/`)

1. Start `serve_https.py`:
   ```bash
   python serve_https.py --port 8443 --dir www
   ```
2. Navigate to `https://localhost:8443` in Chrome, Firefox, Safari, or Edge.
3. Features:
   - **Voice Input**: Click the microphone icon for speech-to-text.
   - **TTS Output**: Enable audio responses in settings.
   - **Module Dropdown**: Select pre-processors dynamically (e.g., `math_utils`, `web_fetch`, `net_check`).
   - **Session Manager**: View, switch, search, and export previous conversations from `history.db`.
   - **Autonomous Tool Execution**: The AI will write files, execute scripts, and report back automatically when tool mode is active.

---

### Using Pluggable Modules (`modules/`)

Modules are custom Python pre-processors located in `modules/`. Each module must implement:
```python
def process(text: str) -> str:
    # Transform, inspect, or enrich the prompt
    return f"[Enriched] {text}"
```

#### Available Built-in Modules:
- **`modules/math_utils.py`**: Solves math expressions locally before sending to AI.
- **`modules/web_fetch.py`**: Fetches URLs found in messages and appends content for the AI.
- **`modules/net_check.py` & `net_check1.py`**: Diagnostic helpers checking local network interfaces.
- **`modules/buffering.py` & `error.py`**: Socket and stream troubleshooting tools.
- **`modules/firewall_nftables.py`**: Inspects and controls firewall rules.

#### How to use a module from the command line:
```bash
python client.py --module math_utils "Calculate 2 ** 16"
```

---

### Using AI Skills (`skills/`)

Skills are structured instructional playbooks located in `skills/<skill_name>/SKILL.md`. The AI server indexes these skills and injects tactical instructions into the agent's prompt when needed.

#### Built-in Skills:
- `skills/network_defense/`: Handling DDoS, SYN floods, port scans, and malicious IPs.
- `skills/firewall_control/`: Managing nftables sets, tables, and rules.
- `skills/packet_inspection/`: Raw packet decoding and sniffing recipes.

#### Refreshing Skills Index:
```bash
python sync_skills.py
```

---

### Network Defense & Diagnostic Scripts (`scripts/`)

*(Note: Network packet sniffing and firewall scripts require `sudo` or administrator privileges on Linux).*

| Script | Purpose | Example Usage |
|---|---|---|
| `firewall-nftables.py` | Configure Linux nftables firewall | `sudo python3 firewall-nftables.py --status` |
| `scripts/snifF.py` | Raw network packet sniffer & inspector | `sudo python3 scripts/snifF.py -i eth0` |
| `scripts/packet.py` | Custom packet construction & analysis | `python3 scripts/packet.py` |
| `scripts/router.py` | Local routing table inspection | `python3 scripts/router.py` |
| `check_ports.py` | Scan open local ports | `python3 check_ports.py` |
| `process_check.py` | Monitor background AI & server processes | `python3 process_check.py` |
| `save_conversations.py`| Export conversations from `history.db` | `python3 save_conversations.py --export json` |

---

## Configuration Reference (`config.json`)

All runtime options are managed in `config.json`. Key settings:

```json
{
  "server": {
    "host": "0.0.0.0",
    "api_port": 8000,
    "api_ssl_port": 8444,
    "web_port": 8443,
    "ssl": {
      "enabled": true,
      "cert_file": "server.crt",
      "key_file": "server.key"
    }
  },
  "ai": {
    "backend": "auto",           // "auto" | "ollama" | "transformers"
    "ollama": {
      "url": "http://localhost:11434",
      "model": "llama3.2:3b",     // Model pulled in Ollama
      "options": {
        "temperature": 0.7,
        "num_ctx": 4096
      }
    }
  },
  "agent": {
    "max_tool_steps": 8,         // Maximum agent self-correction steps
    "tool_timeout_seconds": 60,  // Execution timeout per tool
    "enable_tools_default": true
  }
}
```

---

## Troubleshooting & Common Fixes

| Issue | Cause | Solution |
|---|---|---|
| **`Connection refused` on :11434** | Ollama is not running | Run `ollama serve` (Linux) or start Ollama app (Windows). |
| **`model 'llama3.2:3b' not found`** | Model hasn't been pulled | Run `ollama pull llama3.2:3b`. |
| **`NET::ERR_CERT_AUTHORITY_INVALID`** | Self-signed certificate in browser | Click "Advanced" → "Proceed to localhost (unsafe)", or add `server.crt` to trusted root authorities. |
| **`Address already in use` (8000/8443)** | A previous server instance is still running | Find and kill it: <br>Linux: `fuser -k 8000/tcp 8443/tcp`<br>Windows: `netstat -ano \| findstr 8000` then `taskkill /PID <PID> /F`. |
| **PowerShell script execution disabled** | Windows security policy blocks `.ps1` | Run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` in PowerShell. |
| **Slow responses / High RAM usage** | Default model too large for GPU/CPU | Switch to a smaller model in `config.json` such as `llama3.2:1b` or `qwen2.5:1.5b`. |
| **Agent tool execution denied** | Security restrictions | Pass `--no-tools` in `client.py` or disable tools in `config.json` (`"enable_tools_default": false`). |
# browser-chat-terminal-clien
# browser-chat-terminal-clien

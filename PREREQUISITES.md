# Prerequisites

> Everything you need installed **before** starting Phase 1 of the Kitchen App guide.

---

## Hardware Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| RAM | 8 GB | 16 GB |
| Free disk space | 10 GB | 20 GB |
| CPU cores | 2 | 4+ |
| Internet | Required for downloads and `npm install` | Broadband |

Docker Desktop with Kubernetes enabled consumes ~2-4 GB of RAM at idle. Close unnecessary apps if you are on 8 GB.

---

## 1. Git — Version Control

**Required version:** 2.40+

### Install

**Windows:**

1. Download from [git-scm.com/download/win](https://git-scm.com/download/win)
2. Run the `.exe` as Administrator
3. Click Next on every screen (leave all defaults)
4. Click **Install**, then **Finish**

**macOS:**

```bash
brew install git
```

If Homebrew is not installed:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install git
```

**Linux (Ubuntu/Debian):**

```bash
sudo apt update && sudo apt install git -y
```

**Linux (Fedora/Red Hat):**

```bash
sudo dnf install git -y
```

### Verify

```bash
git --version
# Expected: git version 2.43.0 (or higher)
```

### Configure

```bash
git config --global user.name "Your Full Name"
git config --global user.email "you@example.com"
git config --global core.editor "code --wait"
```

Verify with:

```bash
git config --global --list
# Should show user.name and user.email
```

---

## 2. Node.js & NVM — JavaScript Runtime

**Required versions:** Node.js 18 **and** Node.js 20 (CI/CD tests on both)

### Install NVM

**Windows:**

1. Download from [github.com/coreybutler/nvm-windows](https://github.com/coreybutler/nvm-windows) → Releases → `nvm-setup.exe`
2. Run as Administrator
3. Open a **new** PowerShell as Administrator

**macOS / Linux:**

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

Close and reopen your terminal after install.

### Install Node.js via NVM

```bash
nvm install 18
nvm install 20
nvm use 20
```

### Verify

```bash
node --version    # Expected: v20.x.x
npm --version     # Expected: 10.x.x
nvm list          # Should show both v18.x.x and v20.x.x
```

---

## 3. Docker Desktop — Container Engine

**Required version:** Docker 25+ with Compose V2

### Install

**Windows:**

1. Visit [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Click **Download for Windows** → run installer
3. Enable **WSL 2 integration** when prompted
4. Restart your computer if asked
5. Start Docker Desktop → wait for the whale icon in the taskbar to stop animating

**macOS:**

1. Visit [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Download for Mac (choose Apple Silicon or Intel)
3. Open `.dmg` → drag Docker to Applications
4. Launch Docker Desktop → wait for the whale icon in the menu bar

**Linux (Ubuntu/Debian):**

```bash
sudo apt update && sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

### Verify

```bash
docker --version         # Expected: Docker version 25.x.x
docker compose version   # Expected: Docker Compose version v2.x.x
docker run hello-world   # Expected: "Hello from Docker!"
```

> If you see `Cannot connect to Docker daemon`, Docker Desktop is not running. Open the app and wait for the whale icon to stop animating.

---

## 4. Kubernetes (via Docker Desktop)

Docker Desktop includes a built-in single-node Kubernetes cluster. This project uses it instead of Minikube.

### Enable

1. Open Docker Desktop
2. Go to **Settings** (gear icon) → **Kubernetes**
3. Check **Enable Kubernetes**
4. Click **Apply & Restart**
5. Wait ~2 minutes for the green "Kubernetes is running" indicator

### Verify

```bash
kubectl config use-context docker-desktop
kubectl get nodes
# Expected: docker-desktop   Ready   control-plane   ...   v1.28.x
```

---

## 5. kubectl — Kubernetes CLI

Docker Desktop usually bundles `kubectl` automatically. If not:

**Windows:**

```powershell
winget install Kubernetes.kubectl
```

**macOS:**

```bash
brew install kubectl
```

**Linux:**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

### Verify

```bash
kubectl version --client
# Expected: Client Version: v1.28.x (or higher)

kubectl config current-context
# Expected: docker-desktop
```

---

## 6. VS Code — Code Editor

**Required version:** Latest stable

### Install

Download from [code.visualstudio.com](https://code.visualstudio.com) and follow the installer.

### Recommended Extensions

Open VS Code, press `Ctrl+Shift+X` (Windows/Linux) or `Cmd+Shift+X` (macOS), and install:

| Extension | Publisher | Why You Need It |
|-----------|-----------|-----------------|
| Docker | Microsoft | Dockerfile syntax highlighting, container commands |
| Kubernetes | Microsoft | kubectl integration, YAML editing, cluster explorer |
| YAML | Red Hat | Validates YAML structure for K8s manifests |
| GitLens | GitKraken | Inline blame annotations — who changed each line and when |
| ESLint | Microsoft | JavaScript linting errors as you type |
| REST Client | Huachao Mao | Test API endpoints directly from VS Code |

---

## 7. GitHub Account

A free GitHub account is required for:

- Hosting the repository
- GitHub Actions CI/CD pipeline (free for public repos)

### Sign Up

1. Go to [github.com](https://github.com) → **Sign up**
2. Follow the prompts (email, username, password)
3. Verify your email address

---

## 8. curl (or PowerShell alternatives)

Used to test API endpoints from the terminal.

**macOS / Linux:** Pre-installed.

**Windows:** `curl.exe` is pre-installed on Windows 10+. In PowerShell, use `curl.exe` (not `curl`, which is aliased to `Invoke-WebRequest`):

```powershell
curl.exe http://localhost:3000/health
```

Or use PowerShell's native command:

```powershell
Invoke-RestMethod http://localhost:3000/health
```

---

## 9. Shell and Terminal Requirements (Critical for execution)

This project uses both:
- **PowerShell commands** (Windows-friendly HTTP and kubectl usage)
- **Bash scripts** (`scripts/local-setup.sh`, `scripts/teardown.sh`)

### Windows requirement

Install at least one bash-capable shell:
- Git Bash (quickest), or
- WSL Ubuntu.

If you use PowerShell only, run Kubernetes and HTTP commands there, and run `.sh` scripts from Git Bash/WSL.

### Multi-terminal requirement

You must be able to keep **multiple terminals open simultaneously**.  
During Kubernetes validation on personal laptops (especially Windows), keep port-forward commands running in separate terminals:

```bash
kubectl port-forward -n kitchen-app svc/api-nodeport 30000:3000
kubectl port-forward -n kitchen-app svc/web-ui-service 30080:80
kubectl port-forward -n kitchen-app svc/prometheus-service 30090:9090
kubectl port-forward -n kitchen-app svc/grafana-service 30300:3000
```

If any terminal is closed, the mapped endpoint stops immediately.

---

## 10. Ports That Must Be Available

The following ports are used by the application. Make sure they are not occupied by other programs before starting.

| Port | Used By | Context |
|------|---------|---------|
| 3000 | Kitchen API | Docker Compose |
| 8080 | Web UI (Nginx) | Docker Compose |
| 27017 | MongoDB | Docker Compose |
| 30000 | API NodePort | Kubernetes |
| 30080 | Web UI NodePort | Kubernetes |
| 30090 | Prometheus NodePort | Kubernetes |
| 30300 | Grafana NodePort | Kubernetes |

Check if a port is in use:

**Windows (PowerShell):**

```powershell
netstat -ano | findstr :3000
```

**macOS / Linux:**

```bash
lsof -i :3000
```

---

## 11. Container Images Used

These are pulled automatically — no manual download needed. Listed here for reference and firewall/proxy allowlisting.

| Image | Version | Purpose |
|-------|---------|---------|
| `node` | `20-alpine` | API build & runtime |
| `mongo` | `7.0` | Database |
| `nginx` | `alpine` | Web UI & reverse proxy |
| `prom/prometheus` | `v2.52.0` | Metrics collection |
| `grafana/grafana` | `11.2.0` | Dashboards & visualization |

---

## 12. npm Packages (auto-installed)

These are installed automatically by `npm ci`. Listed for awareness.

**Runtime dependencies:**

| Package | Version | Purpose |
|---------|---------|---------|
| `express` | ^4.21.0 | HTTP server framework |
| `mongodb` | ^6.10.0 | MongoDB driver |
| `prom-client` | ^15.1.0 | Prometheus metrics library |

**Dev dependencies:**

| Package | Version | Purpose |
|---------|---------|---------|
| `eslint` | ^8.57.0 | JavaScript linter (runs in Dockerfile) |
| `jest` | ^29.7.0 | Unit test runner |
| `supertest` | ^7.0.0 | HTTP assertion library for tests |

---

## 13. Kubernetes Readiness Guardrails (before Phase 4)

Before Kubernetes deployment, confirm these commands work:

```bash
kubectl get nodes
kubectl get ns
kubectl config current-context
```

Before applying API deployment, these dependencies must exist in namespace `kitchen-app`:

```bash
kubectl get configmap api-config -n kitchen-app
kubectl get configmap api-server-override -n kitchen-app
kubectl get secret mongodb-secret -n kitchen-app
```

If any of these are missing, API pods commonly stay at `0/1 READY`.

---

## Verification Checklist

Run each command and confirm the expected output before proceeding to Phase 1.

```bash
git --version                    # git version 2.x.x
node --version                   # v20.x.x
npm --version                    # 10.x.x
nvm list                         # Shows v18.x.x and v20.x.x
docker --version                 # Docker version 25.x.x
docker compose version           # Docker Compose version v2.x.x
docker run hello-world           # "Hello from Docker!"
kubectl version --client         # Client Version: v1.28.x
kubectl get nodes                # docker-desktop   Ready
code --version                   # 1.x.x (any recent version)
```

If all commands succeed, you are ready to start **Phase 1: GitHub Setup**.

---

## Troubleshooting

### `nvm` command not found (macOS/Linux)

Close and reopen your terminal. If it still doesn't work, add this to `~/.bashrc` or `~/.zshrc`:

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

### Docker: `Cannot connect to the Docker daemon`

Docker Desktop is not running. Open the application and wait for the whale icon to indicate readiness.

### Docker: WSL 2 not installed (Windows)

If the Docker Desktop installer asks for WSL 2:

```powershell
wsl --install
```

Restart your computer, then re-run the Docker Desktop installer.

### `kubectl get nodes` shows `NotReady`

Kubernetes is still starting. Wait 1-2 minutes and try again. If it persists, restart Docker Desktop.

### Pods show `0/1 READY` after deployment

Use the dedicated **Kubernetes 0/1 READY Troubleshooting Playbook** in:

- `docs/STUDENT_GUIDE.html`

Follow symptom-to-fix blocks in order (quick triage, dependency fixes, rollout recovery).

### Port already in use

Stop the process occupying the port, or change the port mapping in `docker-compose.yml` / K8s Service manifests.

**Windows:**

```powershell
# Find process on port 3000
netstat -ano | findstr :3000
# Kill by PID
taskkill /PID <pid> /F
```

**macOS / Linux:**

```bash
lsof -ti :3000 | xargs kill -9
```

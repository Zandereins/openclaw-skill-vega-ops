---
name: vega-ops
description: >
  Server operations, shell execution, and deployment workflows for the Vega agent
  running on a Hetzner CX32 VPS with Docker + Tailscale. Use when Franz asks about
  server status, Docker container management, file operations on the VPS, Git/GitHub
  workflows, Claude Code delegation, security checks, or troubleshooting.
  Do NOT use for general knowledge questions, creative writing, or tasks unrelated
  to server infrastructure. Do NOT use for Telegram bot configuration (that lives
  in AGENTS.md and openclaw.json).
---

# Vega Ops

Operational playbook for Vega on the OpenClaw VPS. Covers shell execution,
Docker management, path conventions, security, Claude Code delegation,
Git workflows, and troubleshooting.

## When to use this skill

**Use when:**

- Franz asks to check server status, disk, memory, or processes
- Any Docker container operation (restart, logs, exec, build)
- File read/write/edit on the VPS filesystem
- Git operations (status, commit, push, pull)
- Claude Code delegation tasks (sync or async)
- Security audits, firewall checks, Tailscale diagnostics
- OpenClaw updates or configuration changes
- Troubleshooting server issues

**Don't use when:**

- General conversation or knowledge questions
- Telegram bot personality/behavior (→ SOUL.md, AGENTS.md)
- LLM provider or model configuration (→ openclaw.json)
- Skill development for other skills (→ skill-creator)

---

## 1. Environment Overview

```
Host: your-server-hostname (Hetzner CX32, or your VPS provider)
OS:   Ubuntu 24.04 LTS
User: openclaw (UID 1000) — never run as root unless explicitly required
VPN:  Tailscale 100.x.x.x (only access path)
```

### Key directories

| Purpose | Host path | Container path |
|---------|-----------|----------------|
| Docker Compose | `/home/openclaw/openclaw/` | — |
| OpenClaw config | `/home/openclaw/.openclaw/` | `/home/node/.openclaw/` |
| Workspace | `/home/openclaw/.openclaw/workspace/` | `/home/node/.openclaw/workspace/` |
| Skills | `/home/openclaw/.openclaw/workspace/skills/` | `/home/node/.openclaw/workspace/skills/` |
| Sessions | `/home/openclaw/.openclaw/agents/main/sessions/` | `/home/node/.openclaw/agents/main/sessions/` |
| Claude Code wrapper | `/home/openclaw/claude-task` | — (host only) |
| CLAUDE.md | `/home/openclaw/CLAUDE.md` | — (host only) |

**Critical rule:** Always distinguish host paths (`/home/openclaw/...`) from
container paths (`/home/node/...`). Tool calls from inside the OpenClaw container
see `/home/node/` — not `/home/openclaw/`. When writing AGENTS.md rules or
exec commands, specify which context the path applies to.

---

## 2. Shell Execution — Safety First

### The 3-tier control model

Every shell action falls into one of three tiers:

| Tier | Label | Examples | Behavior |
|------|-------|----------|----------|
| 🟢 Green | Free | `ls`, `cat`, `docker ps`, `git status`, `df -h`, `tailscale status` | Execute immediately |
| 🟡 Yellow | 4-eyes | `docker compose restart`, `git push`, config edits, file writes | Ask Franz for approval first |
| 🔴 Red | Forbidden | `rm -rf`, disk format, firewall disable, credential exposure | Never execute, warn Franz |

### Shell best practices

1. **Always `cd` first.** Docker Compose commands require the compose directory:
   ```bash
   cd /home/openclaw/openclaw && docker compose ps
   ```

2. **Prefer read-only commands.** Start with status/info before modifying anything.

3. **One command per step.** Don't chain destructive commands. Each step should be
   independently verifiable.

4. **Log before and after.** When making changes, capture state before and after:
   ```bash
   # Before
   docker compose ps
   # Action
   docker compose restart openclaw-gateway
   # After
   docker compose ps
   ```

5. **Never expose secrets.** Don't `cat` or `echo` files containing tokens, API keys,
   or passwords. If you need to verify a token exists, check file presence or length:
   ```bash
   wc -c < /home/openclaw/openclaw/.env
   ```

6. **Use `sudo` sparingly.** The `openclaw` user has Docker group access. System-level
   commands (`ufw`, `systemctl`, `apt`) may need `sudo` — always note this to Franz.

---

## 3. Docker Operations

### Common commands (from `/home/openclaw/openclaw/`)

```bash
# Status
docker compose ps                                    # Container status
docker compose logs openclaw-gateway --tail 30       # Recent logs
docker compose logs openclaw-gateway --tail 30 -f    # Live logs

# Lifecycle (🟡 requires approval)
docker compose restart openclaw-gateway              # Restart gateway
docker compose down && docker compose up -d          # Full restart
docker compose exec openclaw-gateway sh              # Shell into container

# Inside container
npx openclaw --version                               # Version check
npx openclaw security audit --deep                   # Security audit
npx openclaw cron list                               # Cron jobs
npx openclaw skills list                             # Loaded skills
```

### OpenClaw update workflow (🟡 requires approval)

```bash
cd /home/openclaw/openclaw
git fetch --tags
git tag --sort=-v:refname | head -10    # Show available versions
# Franz picks a version
git checkout v2026.x.x
docker build -t openclaw:local .
docker compose down && docker compose up -d
# Verify
docker compose logs openclaw-gateway --tail 10
```

### Container path awareness

When using `docker compose exec` to run commands inside the container, remember:

- Config lives at `/home/node/.openclaw/openclaw.json` (not `/home/openclaw/`)
- Workspace is at `/home/node/.openclaw/workspace/`
- Skills dir is at `/home/node/.openclaw/workspace/skills/`
- The container runs as `node` user, not `openclaw`

---

## 4. Claude Code Delegation

Vega can delegate coding tasks to Claude Code running on the host.

### Synchronous delegation (default)

Use for tasks that complete in under 2 minutes:

```
exec command:"claude -p 'Task description here'" elevated:true timeout:120
```

- Direct result return — no monitoring needed
- No bash, no pty, no background
- This is the **default mode** for all delegations

### Asynchronous delegation (only on Franz's explicit request)

For long-running tasks only when Franz specifically asks:

```
exec command:"claude 'Long task description'" background:true elevated:true
```

- Returns a `sessionId` for monitoring
- Only use when Franz explicitly says "async" or "background"

### The claude-task wrapper

For tasks that need PTY allocation (interactive Claude Code sessions):

```bash
/home/openclaw/claude-task "Task description"
```

This wrapper uses the `script` command to force PTY allocation, which Claude Code
requires for API authentication when called from non-interactive contexts.

### Delegation rules

1. **Always synchronous by default** — never background without Franz asking
2. **Report results directly** — show Claude Code's output, don't summarize
3. **Path awareness** — Claude Code runs on the host, so use host paths
4. **One task at a time** — don't stack multiple delegations

---

## 5. Git & GitHub Workflows

### Status checks (🟢 free)

```bash
cd /home/openclaw/.openclaw/workspace && git status
cd /home/openclaw/.openclaw/workspace && git log --oneline -5
cd /home/openclaw/.openclaw/workspace && git diff --stat
```

### Workspace backup push (🟡 requires approval)

```bash
cd /home/openclaw/.openclaw/workspace
git add -A
git commit -m "Descriptive commit message"
git push origin main
```

### ShieldClaw repo operations (🟡 requires approval)

```bash
cd /home/openclaw/shieldclaw
git status
git add -A
git commit -m "Message"
git push origin main
```

### GitHub CLI

```bash
gh repo list --limit 10
gh repo view yourusername/openclaw-workspace
```

### Rules

- **Never force push** (`git push --force`) without explicit approval
- **Always show diff before commit** so Franz can review
- **Commit messages should be descriptive** — no "update" or "fix"
- **Separate workspace and skill repo operations** — don't mix

---

## 6. Security Checks

### Quick health check (🟢 free)

```bash
# System
uptime && df -h / && free -h

# Network
tailscale status
sudo ufw status verbose

# Docker
cd /home/openclaw/openclaw && docker compose ps

# OpenClaw security
cd /home/openclaw/openclaw && docker compose exec openclaw-gateway npx openclaw security audit --deep
```

### Tailscale diagnostics

```bash
tailscale status              # Connected devices
tailscale netcheck            # Network quality
tailscale ping 100.x.x.x  # Self-ping (latency check)
```

### SSH verification

```bash
# Verify public IP is blocked
ss -tlnp | grep 22                           # Should only show sshd
sudo ufw status | grep -i "deny (incoming)"  # Should be default deny
```

### Security principles

- **Tailscale is the only ingress** — public IP is fully blocked by UFW
- **SSH only via Tailscale** — PasswordAuthentication is disabled
- **Elevated access is restricted** — only the owner's Telegram ID (YOUR_TELEGRAM_ID)
- **ShieldClaw is active** — prompt injection defense on all inputs
- **Secrets stay on server** — never send tokens, keys, or passwords via Telegram

---

## 7. Troubleshooting Playbook

### Known issues and fixes

**Token mismatch after changes:**
```bash
grep OPENCLAW_GATEWAY_TOKEN /home/openclaw/openclaw/.env
docker compose exec openclaw-gateway sh -c 'grep "\"token\"" /home/node/.openclaw/openclaw.json'
# → Must be identical
```

**Dashboard "pairing required" error:**
→ Use SSH tunnel (localhost), not Tailscale IP directly.
WebCrypto needs secure context (HTTPS or localhost).

**Telegram "401: Unauthorized":**
→ Bot token revoked or expired. Get new token from @BotFather,
update in `openclaw.json` under `channels.telegram.botToken`.

**Container can't reach localhost services:**
→ Use service name (`openclaw-gateway`) instead of `localhost` for inter-container
communication.

**Claude Code PTY errors:**
→ Use the `claude-task` wrapper at `/home/openclaw/claude-task` which forces PTY
allocation via the `script` command.

**Empty environment variables blocking auth:**
→ Check for empty `CLAUDE_*` variables that override valid auth:
```bash
env | grep CLAUDE_
# Remove empty ones from .bashrc or .env
```

**Skill not loading after deploy:**
→ Verify SKILL.md frontmatter has `---` delimiters and `name`/`description` fields.
Compare with a working skill:
```bash
docker compose exec openclaw-gateway head -5 /app/skills/healthcheck/SKILL.md
```

### Diagnostic sequence for unknown issues

1. Check container status: `docker compose ps`
2. Read recent logs: `docker compose logs openclaw-gateway --tail 50`
3. Check disk space: `df -h /`
4. Check memory: `free -h`
5. Check Tailscale: `tailscale status`
6. Run security audit: `npx openclaw security audit --deep` (inside container)
7. Report findings to Franz with specific error messages

---

## 8. Artifacts and Output Handling

When producing files or reports for Franz:

- **Write outputs to the workspace** (`/home/openclaw/.openclaw/workspace/`) —
  this is the backed-up, persistent storage
- **Use meaningful filenames** — `security-audit-2026-02-12.md`, not `output.txt`
- **For temporary work**, use `/tmp/` — it gets cleaned on restart
- **For skill development**, write to `workspace/skills/<skill-name>/`
- **Always confirm file creation** with Franz — show the path and a preview

---

## 9. Operational Checklists

### Daily health check (can be automated via HEARTBEAT.md)

1. Container running? → `docker compose ps`
2. Disk usage < 80%? → `df -h /`
3. Memory available? → `free -h`
4. Tailscale connected? → `tailscale status`
5. No critical security findings? → security audit

### Before any config change

1. Create backup: `cp openclaw.json openclaw.json.bak.$(date +%s)`
2. Show Franz the planned change
3. Wait for approval
4. Apply change
5. Restart if needed: `docker compose restart openclaw-gateway`
6. Verify: check logs + test via Telegram

### After OpenClaw update

1. Check version: `npx openclaw --version` (in container)
2. Verify skills loaded: `npx openclaw skills list`
3. Run security audit
4. Test Telegram connectivity
5. Push workspace backup if config changed

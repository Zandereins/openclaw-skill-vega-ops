# 🛰️ Vega Ops

**Server operations, shell execution, and deployment workflows for OpenClaw agents.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-FF6B35)](https://openclaw.ai)

## Why Vega Ops?

Running an OpenClaw agent on a production VPS requires operational knowledge beyond general coding skills. Path confusion (host vs container), unsafe shell execution, and deployment mishaps are common failure modes.

Vega Ops provides battle-tested operational patterns:

- **3-tier safety model** for shell execution (free / approval / forbidden)
- **Path mapping** between host and container contexts
- **Claude Code delegation** patterns (sync/async, PTY handling)
- **Security checks** for Tailscale, UFW, SSH hardening
- **Troubleshooting playbook** with proven fixes

## Architecture

```
┌─────────────────────────────────────────┐
│ Layer 1: SKILL.md (Agent Awareness)     │
│ ~300 tokens, always loaded              │
│ • Environment overview (paths, users)   │
│ • Shell execution safety rules          │
│ • Docker operation templates            │
│ • Claude Code delegation patterns       │
│ • Git/GitHub workflow checklists        │
├─────────────────────────────────────────┤
│ Layer 2: Reference Docs (On-Demand)     │
│ 0 tokens, loaded only when needed       │
│ • path-mapping.md — Host ↔ Container    │
│ • security-checklist.md — Quick audit   │
└─────────────────────────────────────────┘
```

## Installation

Copy the skill folder into your OpenClaw workspace:

```bash
cp -r vega-ops/ ~/.openclaw/workspace/skills/vega-ops/
```

The SKILL.md is automatically discovered and loaded by OpenClaw.

Restart the gateway to activate:
```bash
cd ~/openclaw && docker compose restart openclaw-gateway
```

Verify:
```bash
docker compose exec openclaw-gateway npx openclaw skills list | grep vega-ops
```

## Usage

### Passive Awareness (automatic)

Once installed, the SKILL.md rules are active in every conversation. Your agent will:
- Execute shell commands with safety guardrails (ask before destructive ops)
- Distinguish host paths from container paths
- Follow security best practices for server operations

### Quick Health Check

Ask your agent:
```
Run a health check on the VPS
```

Your agent will execute:
```bash
uptime && df -h / && free -h
tailscale status
sudo ufw status verbose
cd ~/openclaw && docker compose ps
```

### Claude Code Delegation

Ask your agent:
```
Delegate to Claude Code: Review AGENTS.md and suggest improvements
```

Your agent will use the synchronous delegation pattern from SKILL.md.

### Git Backup

Ask your agent:
```
Backup the workspace to GitHub
```

Your agent will follow the safe push workflow with diff preview.

## Token Efficiency

| Component | Token Cost | When Loaded |
|-----------|-----------|-------------|
| SKILL.md | ~300 tokens | Every message |
| path-mapping.md | 0 tokens | On-demand only |
| security-checklist.md | 0 tokens | On-demand only |

## Customization

This skill is tailored for a specific setup:
- **OS:** Ubuntu 24.04 LTS
- **VPN:** Tailscale mesh network
- **Container:** Docker (OpenClaw Gateway)
- **Delegation:** Claude Code

To adapt for your environment:

1. Edit `SKILL.md` section 1 (Environment Overview) with your paths
2. Update `references/path-mapping.md` with your volume mounts
3. Adjust the 3-tier control model for your approval workflow
4. Customize security checks for your network topology

## Requirements

- OpenClaw Gateway (Docker)
- Linux VPS (Ubuntu/Debian recommended)
- Tailscale (optional, but recommended for security)
- Claude Code (optional, for delegation features)

## Roadmap

- [x] **v1.0** — Initial release (shell safety, Docker ops, path mapping)
- [ ] **v1.1** — Automated health check skill (cron integration)
- [ ] **v1.2** — Interactive troubleshooting mode (guided diagnostics)
- [ ] **v1.3** — Multi-server support (workspace sync across nodes)

## Contributing

Contributions welcome! Areas of interest:
- Additional troubleshooting playbook entries
- Security check automations
- Platform-specific adaptations (Arch, Fedora, macOS)

## License

MIT — see [LICENSE](LICENSE)

---

*Built by [zaneins](https://github.com/Zandereins) — operational excellence for AI agents, one command at a time.* 🛰️

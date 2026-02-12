# vega-ops 🛰️

An [OpenClaw](https://openclaw.ai) skill for server operations, shell execution, and deployment workflows.

Built for **Vega**, an AI assistant running on a hardened Hetzner VPS with Docker, Tailscale mesh VPN, and Claude Code integration.

## What it does

`vega-ops` gives an OpenClaw agent the operational knowledge to safely manage a Linux VPS:

- **Shell execution** with a 3-tier safety model (free / approval-required / forbidden)
- **Docker container management** (lifecycle, logs, exec, updates)
- **Host ↔ Container path mapping** (avoids the #1 source of bugs in containerized setups)
- **Claude Code delegation** (sync and async patterns with PTY handling)
- **Git & GitHub workflows** (workspace backup, skill repos, safe push patterns)
- **Security checks** (Tailscale, UFW, SSH hardening, OpenClaw audit)
- **Troubleshooting playbook** (known issues with proven fixes)

## Design principles

Inspired by [OpenAI's Shell + Skills tips](https://platform.openai.com/docs/guides/skills-shell-tips) and adapted for the OpenClaw ecosystem:

1. **Skill descriptions are routing logic** — clear "use when / don't use when" boundaries
2. **Negative examples reduce misfires** — explicit "don't do this" cases
3. **Templates inside the skill** — zero token cost when not invoked
4. **Security by default** — containment-first approach for shell + network access
5. **Path mapping is a first-class concern** — host vs. container paths documented exhaustively

## Installation

### As a workspace skill (recommended)

```bash
# Copy to your OpenClaw workspace
cp -r vega-ops/ ~/.openclaw/workspace/skills/vega-ops/

# Restart gateway to pick up the skill
cd ~/openclaw && docker compose restart openclaw-gateway

# Verify
docker compose exec openclaw-gateway npx openclaw skills list
```

### Customization

This skill is tailored for a specific setup (Hetzner CX32, Ubuntu 24.04, Tailscale, Docker).
To adapt it for your environment:

1. Update paths in `SKILL.md` section 1 and `references/path-mapping.md`
2. Adjust the 3-tier control model in section 2 to match your approval workflow
3. Modify security checks in section 6 for your network topology
4. Update troubleshooting entries with your own known issues

## Structure

```
vega-ops/
├── SKILL.md                          # Main skill (loaded by OpenClaw)
├── references/
│   ├── path-mapping.md               # Host ↔ Container path translation table
│   └── security-checklist.md         # Quick security verification steps
├── LICENSE
└── README.md
```

## Requirements

- OpenClaw Gateway (Docker)
- Tailscale mesh VPN
- Ubuntu 24.04 (or compatible Linux)
- Claude Code (optional, for delegation features)

## License

MIT

## Author

Franz & Vega — Built in Dresden 🇩🇪

# Path Mapping Reference

## Host ↔ Container Path Translation

The OpenClaw gateway runs inside Docker. Paths differ between host and container.
Always specify which context you mean.

## Volume Mounts (docker-compose.yml)

```yaml
volumes:
  - ~/.openclaw:/home/node/.openclaw          # Config + state
  - ~/.openclaw/workspace:/home/node/.openclaw/workspace  # Workspace
```

## Full Path Map

| What | Host path | Container path |
|------|-----------|----------------|
| OpenClaw config | `/home/openclaw/.openclaw/openclaw.json` | `/home/node/.openclaw/openclaw.json` |
| Workspace root | `/home/openclaw/.openclaw/workspace/` | `/home/node/.openclaw/workspace/` |
| AGENTS.md | `/home/openclaw/.openclaw/workspace/AGENTS.md` | `/home/node/.openclaw/workspace/AGENTS.md` |
| SOUL.md | `/home/openclaw/.openclaw/workspace/SOUL.md` | `/home/node/.openclaw/workspace/SOUL.md` |
| USER.md | `/home/openclaw/.openclaw/workspace/USER.md` | `/home/node/.openclaw/workspace/USER.md` |
| IDENTITY.md | `/home/openclaw/.openclaw/workspace/IDENTITY.md` | `/home/node/.openclaw/workspace/IDENTITY.md` |
| TOOLS.md | `/home/openclaw/.openclaw/workspace/TOOLS.md` | `/home/node/.openclaw/workspace/TOOLS.md` |
| Skills dir | `/home/openclaw/.openclaw/workspace/skills/` | `/home/node/.openclaw/workspace/skills/` |
| Memory dir | `/home/openclaw/.openclaw/workspace/memory/` | `/home/node/.openclaw/workspace/memory/` |
| Credentials | `/home/openclaw/.openclaw/credentials/` | `/home/node/.openclaw/credentials/` |
| Sessions | `/home/openclaw/.openclaw/agents/main/sessions/` | `/home/node/.openclaw/agents/main/sessions/` |
| Bundled skills | — | `/app/skills/` |
| Docker Compose | `/home/openclaw/openclaw/` | — |
| .env file | `/home/openclaw/openclaw/.env` | — |
| Claude Code | `/home/openclaw/claude-task` | — |
| CLAUDE.md | `/home/openclaw/CLAUDE.md` | — |

## Users

| Context | User | UID | Home |
|---------|------|-----|------|
| Host | `openclaw` | 1000 | `/home/openclaw` |
| Container | `node` | 1000 | `/home/node` |

UID match (both 1000) ensures volume-mounted files have correct permissions.

## When to use which

- **Exec tool calls (from Vega inside container):** Use `/home/node/...` paths
- **Claude Code delegation (runs on host):** Use `/home/openclaw/...` paths
- **SSH commands (Franz connects to host):** Use `/home/openclaw/...` paths
- **Docker compose commands:** Always run from `/home/openclaw/openclaw/`
- **AGENTS.md / SOUL.md references:** Specify both paths when relevant

## Common mistakes

1. Writing `/home/openclaw/` in an exec command that runs inside the container
   → Fix: Use `/home/node/.openclaw/...`

2. Writing `/home/node/` in a Claude Code delegation command
   → Fix: Use `/home/openclaw/...` (Claude Code runs on host)

3. Referencing `/app/skills/` for workspace skills
   → Fix: Workspace skills are at `/home/node/.openclaw/workspace/skills/`
   → `/app/skills/` is for bundled skills only

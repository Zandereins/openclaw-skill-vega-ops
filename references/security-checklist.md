# Security Checklist

Quick-reference security verification for the Vega VPS.

## Layer 1: Network (Tailscale + UFW)

```bash
# Tailscale is up and connected
tailscale status
# Expected: Shows all devices (server, macbook, iphone)

# UFW denies all incoming except Tailscale
sudo ufw status verbose
# Expected: Default deny (incoming), allow tailscale0

# Public IP unreachable
# From external machine: ssh root@YOUR_PUBLIC_IP → should timeout
```

## Layer 2: Authentication

```bash
# SSH password auth disabled
grep -i "PasswordAuthentication" /etc/ssh/sshd_config
# Expected: PasswordAuthentication no

# Ed25519 key only
ls -la ~/.ssh/authorized_keys

# OpenClaw elevated access restricted
docker compose exec openclaw-gateway sh -c \
  'cat /home/node/.openclaw/openclaw.json | grep -A3 elevated'
# Expected: only your Telegram ID
```

## Layer 3: Container Security

```bash
# Container running as non-root (node user)
docker compose exec openclaw-gateway whoami
# Expected: node

# State dir permissions
ls -la /home/openclaw/.openclaw/
# Expected: drwx------ (700) for .openclaw

# No exposed ports beyond Tailscale
ss -tlnp | grep -v tailscale
# Only sshd:22 and docker-proxy:18789/18790 expected
```

## Layer 4: Application Security

```bash
# OpenClaw security audit
cd /home/openclaw/openclaw
docker compose exec openclaw-gateway npx openclaw security audit --deep

# ShieldClaw skill loaded
docker compose exec openclaw-gateway npx openclaw skills list
# Expected: ShieldClaw in list

# Gateway token not exposed in logs
docker compose logs openclaw-gateway --tail 100 | grep -i token
# Expected: no token values in output
```

## Layer 5: Operational Security

```bash
# Automatic security updates active
systemctl status unattended-upgrades
# Expected: active

# Cron security audit running
docker compose exec openclaw-gateway npx openclaw cron list
# Expected: security-audit job (daily 02:00 UTC)

# Workspace backup is current
cd /home/openclaw/.openclaw/workspace && git status
# Expected: clean or only memory/ changes
```

## Red flags (investigate immediately)

- UFW status shows ports open beyond tailscale0
- Unknown devices in `tailscale status`
- PasswordAuthentication set to `yes`
- Container running as root
- Security audit shows new CRITICAL findings
- Unknown files in workspace or skills directory

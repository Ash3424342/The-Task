---
title: "Compass MCP Setup Guide"
status: done
type: reference
sources:
  - "[[raw/compass-mcp-docs/Dataplatform · Setup Guide - Coda.pdf]]"
author: Saurabh Dubey
date: 2026-04-04
---

# Compass MCP Setup Guide

Source: Dataplatform Coda page (5 pages).

## Prerequisites
- Node.js 18+ (`brew install node` if missing)
- VPN connected (Compass runs on internal network)
- Groww email & password (for authentication)

## Quick Start
```bash
# One-command setup (recommended):
./scripts/compass-mcp-setup.sh

# This will:
# 1. Check Node.js and VPN
# 2. Ask for Groww email/password
# 3. Test connectivity to Compass MCP server
# 4. Install a local proxy that auto-starts on login
# 5. Let you choose which IDEs to configure
```

## How It Works
```
Your IDE (Cursor, etc.) → Local Proxy 127.0.0.1:3100 → Compass MCP (VPN endpoint)
   No VPN needed              Has VPN access              Internal network
```
Desktop apps can't route to VPN endpoints directly. The proxy runs as a macOS LaunchAgent that auto-starts on login.

## IDE Configuration

### Cursor — `~/.cursor/mcp.json`:
```json
{"mcpServers": {"compass": {"url": "http://127.0.0.1:3100/mcp"}}}
```

### Claude Code — CLI:
```bash
claude mcp add compass --transport http http://127.0.0.1:3100/mcp
```

### VS Code — `~/.vscode/mcp.json`:
```json
{"servers": {"compass": {"url": "http://127.0.0.1:3100/mcp"}}}
```

## Manual Setup (if script doesn't work)
```bash
export MCP_SERVER_URL="https://prod-dp-ai-datahub-gms.growwinfra.in/mcp"
export MCP_AUTH_HEADER="Basic $(echo -n 'your.email@groww.in:yourpassword' | base64)"
export MCP_PROXY_PORT=3100
export NODE_TLS_REJECT_UNAUTHORIZED=0
node ~/.compass-mcp/proxy.mjs
```

## Proxy Management
```bash
tail -f ~/.compass-mcp/proxy.log           # View logs
launchctl kickstart -k gui/$(id -u)/com.groww.compass-mcp-proxy  # Restart
launchctl bootout gui/$(id -u)/com.groww.compass-mcp-proxy       # Stop
./scripts/compass-mcp-setup.sh --uninstall  # Full uninstall
./scripts/compass-mcp-setup.sh --check      # Diagnostics
```

## Troubleshooting
| Issue | Fix |
|---|---|
| "fetch failed" / "Connect Timeout Error" | Run setup script, ensure proxy running |
| TLS / Certificate errors | Script sets `NODE_TLS_REJECT_UNAUTHORIZED=0` (safe on VPN) |
| "Authentication failed (HTTP 401)" | Re-run setup with correct credentials |
| Proxy not starting | Check port 3100 (`lsof -i :3100`), check Node.js path |
| IDE still won't connect | Restart IDE, check `curl http://127.0.0.1:3100/mcp` |

## Support
- Run `./scripts/compass-mcp-setup.sh --check` for diagnostics
- Check logs: `tail -f ~/.compass-mcp/proxy.log`
- Slack: **#dataplatform-helpdesk**

## Full document: [[raw/compass-mcp-docs/Dataplatform · Setup Guide - Coda.pdf]]

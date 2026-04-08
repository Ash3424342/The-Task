# One-Time Obsidian Setup Checklist

Complete these steps in the Obsidian app (Settings gear icon) after opening this vault.

## Required: Enable Core Plugins
Go to **Settings → Core plugins** and enable:
- [x] Backlinks
- [x] Graph View ✅ 2026-04-07
- [x] Canvas ✅ 2026-04-07
- [x] Templates ✅ 2026-04-07
- [x] Properties view ✅ 2026-04-07
- [x] Bases (database views — essential for Manager Dashboard) ✅ 2026-04-07
- [x] Daily Notes ✅ 2026-04-07
- [x] Bookmarks ✅ 2026-04-07
- [x] Outline ✅ 2026-04-07

## Recommended: Community Plugins
Go to **Settings → Community plugins → Turn off Restricted mode → Browse**:
- [x] **Local REST API** — Enables MCP server for semantic vault search from Claude Code ✅ 2026-04-07
- [x] **Templater** — Advanced templates for auto-generated notes ✅ 2026-04-07
- [x] **Tasks** — Enhanced checkbox/task support inside notes ✅ 2026-04-07

## Recommended: File Settings
Go to **Settings → Files & Links**:
- [x] Set **"Default location for new attachments"** → `In subfolder under vault root` → `raw/assets` ✅ 2026-04-07
- [x] Enable **"Automatically update internal links"** ✅ 2026-04-07

## Optional: Obsidian Web Clipper
- [ ] Install the official **Obsidian Web Clipper** browser extension (Chrome/Edge/Firefox)
- [ ] Configure it to save clipped pages into `raw/`
- [ ] After clipping, use hotkey to download attachments locally

## Optional: MCP Server for Claude Code
If you installed **Local REST API** above:
1. In the plugin settings, enable **Non-encrypted (HTTP) Server**
2. Copy the API URL and generate an API key
3. In terminal, run:
   ```
   npx @modelcontextprotocol/server-obsidian --api-key 20ecc0764441bcd7622891492bacf7aec3c68a9754d9d79fbec31d3a9cada345 --url http://localhost:27123
   ```
4. Add the MCP server to `~/.claude/settings.json`

## Optional: Hooks for Auto-Trigger
Claude Code supports hooks that can auto-trigger the Manager Skill on session start.
Check your Claude Code version's hook format in `~/.claude/settings.json` and add a SessionStart hook if desired.

---

Once complete, drop a test file in `raw/` and type `Run Manager Skill` in Claude Code to verify everything works.

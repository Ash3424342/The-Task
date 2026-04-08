---
title: "Build Content Engine MCP Server"
status: todo
priority: medium
owner: me
due: 2026-04-21
deliverable: "MCP server exposing email generation tools callable from Claude Code"
sources: "[[raw/claude.ai-Scoping a context engine and compass MCP integration.pdf]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
task-id: "CE-4"
project: "[[Projects/content-engine]]"
execution-log: "2026-04-07: Task extracted from scoping session"
---

# Build Content Engine MCP Server

## What
Wrap the entire Content Engine (analyzer, blocks, recipes, chart renderer, HTML renderer) as an MCP server with tools callable from Claude Code.

## Tools to Expose
- `list_recipes` — Show available email recipes
- `generate_email` / `assemble_email` — Build email from recipe JSON
- `generate_chart_image` — Render chart PNG from config
- `preview_email` — Preview generated email
- `analyze_email` — Parse/classify existing HTML email
- `validate_email` — Check against email best practices
- `get_design_system` — Return brand design tokens

## Tech
- Python FastMCP
- Parallel to [[wiki/Compass-MCP|Compass MCP]] (both connected simultaneously in Claude Code)

## Blocked By
- [[Tasks/CE-1-email-analyzer]]
- [[Tasks/CE-2-block-library]]
- [[Tasks/CE-3-chart-renderer]]

## Success Criteria
- [ ] All tools callable from Claude Code
- [ ] CM can go from prompt → production HTML in one session
- [ ] Generates valid, Gmail-safe HTML (<102KB)
- [ ] Chart images render at retina quality

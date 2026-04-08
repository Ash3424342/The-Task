---
title: "Implementation Plan v2 — Technical Blueprint"
status: done
priority: high
owner: me
deliverable: "Full code architecture with Python implementations for both tasks"
sources:
  - "[[raw/implementation_plan_v2.md]]"
  - "[[raw/implementation_plan.md]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
type: deliverable
version: "2.0"
---

# Implementation Plan v2

## Content Engine Implementation
- Project structure (src/{analyzer,blocks,recipes,charts,renderer,mcp,generator,utils})
- HTML parser + block detector (pattern matching for 18 blocks)
- Email classifier (5 types)
- Pydantic block schemas
- Chart renderer (Playwright HTML→PNG, 5 templates)
- Jinja2 email renderer (table-based, inline CSS, 600px width)
- MCP server (7 tools via FastMCP)
- LLM copy generator (Anthropic SDK, prompt templates)
- CLI entry point (Typer + Rich)

## Compass Skills Implementation
- Compass setup & connection (proxy port 3100, VPN)
- Using Compass tools: `search`, `get_database_tables`, `list_schema_fields`, `run_trino_query`
- Phase-by-phase discovery workflow with Claude Code examples
- Query testing framework with real guardrails (partition requirements, stale tables, F&O future dates, Pinot sampling)
- Per-skill enhancement guide with specific tables and SQL

## Tech Stack
Python 3.11+, uv, BeautifulSoup, Pydantic, Jinja2, Playwright, MCP SDK, Anthropic SDK, Typer, Rich, pytest

## Source Files
- Current: [[raw/implementation_plan_v2.md]] (54KB)
- Previous: [[raw/implementation_plan.md]] (99KB)

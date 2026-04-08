---
title: "Content Engine — AI-Powered Email Generator"
status: in-progress
priority: medium
owner: me
due: 2026-04-21
deliverable: "MCP server that generates production HTML emails from recipe + prompts"
sources:
  - "[[raw/claude.ai-Scoping a context engine and compass MCP integration.pdf]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
---

# Project: Content Engine

## Overview
Build [[wiki/Groww|Groww]]'s AI-powered HTML email generator from scratch. Campaign Managers select a recipe (JSON template of blocks), provide prompts, and the system generates production-ready HTML emails with chart images — output goes to [[wiki/NARAD|NARAD]] for sending.

## Scope (Immediate)
1. **Email Analyzer Pipeline** — Ingest ~450 HTML emails from [[wiki/WebEngage|WebEngage]] (90 days), parse, classify (5 types), decompose into 18 block types, output structured JSON
2. **Block Library** — 18 block types as Pydantic schemas: header_logo, hero_text, hero_image, chart_image, article_section, factor_card, cta_button, social_proof_banner, footer_standard, divider, card_wrapper, contract_row, filter_pills, listing_header, step_instruction, app_screenshot_pair, product_category_grid, social_showcase
3. **Recipe System** — 8 recipe templates: commodity_educational, onboarding_welcome, product_discovery, trading_alert, feature_announcement, educational_how_to, weekly_digest, promotional
4. **Chart Image Renderer** — 5 templates (price_card, listing_card, comparison, portfolio, metric), retina PNGs via Playwright
5. **HTML Email Renderer** — Jinja2 templates, table-based layout, inline CSS only (no modern CSS — email clients strip it), 600px max width, SEBI footer, unsubscribe link, 4 brand themes
6. **Content Engine MCP Server** — 7 tools: list_recipes, get_recipe, recommend_recipe, list_blocks, generate_email, generate_chart, export_email

## Reference Email (Gold Standard)
Commodity educational about crude oil:
header_logo → hero_text ("What moved Crude Oil prices this month?") → price_card chart (₹9,044, +48.29%) → article sections → 3 factor cards (Geopolitics, OPEC+, Inventory) → listing_card chart → CTA ("EXPLORE NOW") → social proof → SEBI footer

## Architecture
```
CM prompt → Claude Code → Content Engine MCP
  → picks recipe → fills blocks with generated text/images
  → chart PNGs → uploaded to Groww cloud → get URLs
  → production HTML with embedded image URLs
  → CM pastes into NARAD → send
```

## Tech Stack
- Python 3.11+ (all components)
- MCP SDK (FastMCP)
- BeautifulSoup + lxml (HTML parsing)
- Pydantic (block schemas)
- Playwright (HTML→PNG chart rendering)
- Jinja2 (email HTML rendering)
- Anthropic SDK (Claude Sonnet for copy generation)
- Typer + Rich (CLI)
- uv (package management)
- Claude Code workflow (no web app)

## Sub-brands (with design tokens)
- [[wiki/Groww|Groww]] (main) — green accent #00D09C, Inter font, light theme
- [[wiki/915-Brand|915]] (trading sub-brand) — dark theme
- AMC
- W

## Deliverables (Completed Planning Docs)
- [[Deliverables/task-description-v2]] — Full project spec
- [[Deliverables/project-plan-v2]] — Phased plan
- [[Deliverables/implementation-plan-v2]] — Technical blueprint with code
- [[Deliverables/bootstrap-prompt]] — Claude Code session starter

## Timeline
- Side project, 1-2 weeks
- Week 1: Foundation (analyzer, blocks, recipes, HTML renderer)
- Week 2: AI layer + MCP server

## Future Scope (NOT immediate)
- Full loop integration with [[wiki/Compass-MCP|Compass MCP]]: automated analysis → segmentation → content generation → campaign orchestration → [[wiki/NARAD|NARAD]] → delivery
- This is months away

## Related
- [[Projects/compass-mcp-skills]] (parallel system, indirect connection)
- [[wiki/NARAD|NARAD]] (email sender)
- [[wiki/WebEngage|WebEngage]] (email storage/source)

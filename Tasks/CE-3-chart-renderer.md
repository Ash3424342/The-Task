---
title: "Build Chart Image Renderer"
status: todo
priority: medium
owner: me
due: 2026-04-18
deliverable: "5 chart image templates generating retina PNGs programmatically"
sources: "[[raw/claude.ai-Scoping a context engine and compass MCP integration.pdf]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
task-id: "CE-3"
project: "[[Projects/content-engine]]"
execution-log: "2026-04-07: Task extracted from scoping session"
---

# Build Chart Image Renderer

## What
Build 5 chart image templates that generate email-ready PNG images at 3x retina resolution. These get embedded in emails via hosted URLs.

## Templates
1. **price_card** — Single asset price with change indicator and sparkline
2. **listing_card** — Multiple assets listed with prices
3. **comparison** — Side-by-side asset comparison
4. **portfolio** — Portfolio allocation/performance view
5. **metric** — Single large metric with context

## Tech
- Playwright-based HTML→PNG rendering
- 3x retina resolution
- Dark theme variant support

## Success Criteria
- [ ] All 5 templates render correctly
- [ ] Output is email-safe PNG at proper resolution
- [ ] Accepts JSON config for dynamic data
- [ ] Supports light and dark themes

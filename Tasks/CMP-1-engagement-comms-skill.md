---
title: "Enhance Engagement & Communications Compass Skill"
status: todo
priority: high
owner: me
due: 2026-04-14
deliverable: "Enhanced skill document for Engagement & Comms domain in Compass MCP"
sources: "[[raw/claude.ai-Scoping a context engine and compass MCP integration.pdf]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
task-id: "CMP-1"
project: "[[Projects/compass-mcp-skills]]"
execution-log: "2026-04-07: Task extracted from scoping session — Tier 1 priority, directly feeds Content Engine"
---

# Enhance Engagement & Communications Compass Skill

## What
The Engagement & Comms skill is the most important for the [[Projects/content-engine|Content Engine]] integration. It covers CTR/conversion data channels, campaigns, email backend data.

## Key Tables
- `engagement_pn_narad_master` — 250M rows/day (push notifications via [[wiki/NARAD|NARAD]])
- `engagement_email_backend_master` — 3-6M rows (email delivery/engagement data)

## What to Document
- [ ] All table schemas with column descriptions
- [ ] Join keys and relationships between tables
- [ ] Partition requirements (these are huge tables)
- [ ] Must-filter columns
- [ ] Query guardrails (avoid full scans)
- [ ] Example queries for common use cases
- [ ] Stale table indicators

## Priority
Tier 1 — foundational, directly feeds Content Engine's email analytics

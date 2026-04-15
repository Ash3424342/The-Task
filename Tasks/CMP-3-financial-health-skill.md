---
title: "Build User Financial Health Analytics Skill"
status: todo
priority: high
owner: me
due: 2026-04-22
deliverable: "Full skill document for User Financial Health domain (ground-up build from minimal)"
sources: "[[raw/compass-mcp-docs/Dataplatform · Analytics Skills Deep Dive - Coda.pdf]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
task-id: "CMP-3"
project: "[[Projects/compass-mcp-skills]]"
execution-log: "2026-04-07: Task created from audit — Tier 1 priority, currently minimal/summary only"
---

# Build User Financial Health Analytics Skill

## What
User Financial Health Analytics is currently a **minimal** skill (summary only). Needs ground-up build to full skill.

## Domain
User financial wellness metrics — AUM health, investment diversification, and financial behavior patterns.

## Workflow
1. Use Compass tools to discover all Financial Health tables
2. Document table schemas, row counts, partition columns
3. Identify join keys, must-filter columns, stale tables
4. Write and test 10+ example SQL queries
5. Document guardrails and data quality notes
6. Assemble into full skill document

## Success Criteria
- [ ] All relevant tables discovered and documented
- [ ] Partition columns and must-filter requirements identified
- [ ] 10+ tested SQL queries
- [ ] Guardrails section complete
- [ ] Follows Analytics Skills Deep Dive format

## Estimated Time
3-4 days (ground-up build)

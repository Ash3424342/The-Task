---
title: "Build MF Analytics Extended Skill"
status: todo
priority: high
owner: me
due: 2026-04-25
deliverable: "Full skill document for MF Extended domain (ground-up build from minimal)"
sources: "[[raw/compass-mcp-docs/Dataplatform · Analytics Skills Deep Dive - Coda.pdf]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
task-id: "CMP-4"
project: "[[Projects/compass-mcp-skills]]"
execution-log: "2026-04-07: Task created from audit — Tier 1 priority, currently minimal/summary only"
---

# Build MF Analytics Extended Skill

## What
MF Analytics Extended is currently a **minimal** skill (summary only). Needs ground-up build to cover extended MF tables beyond the base Mutual Funds skill.

## Domain
Extended mutual fund analytics — scheme-level NAV performance, deeper AUM breakdowns, advanced engagement funnels.

## Workflow
1. Use Compass tools to discover all Extended MF tables (beyond the 10+ already in the base MF skill)
2. Document table schemas, row counts, partition columns
3. Identify join keys, must-filter columns, stale tables
4. Write and test 10+ example SQL queries
5. Document guardrails and data quality notes
6. Assemble into full skill document

## Success Criteria
- [ ] All relevant extended MF tables discovered
- [ ] Clear distinction from base MF skill (no duplication)
- [ ] 10+ tested SQL queries
- [ ] Guardrails section complete
- [ ] Follows Analytics Skills Deep Dive format

## Estimated Time
3-4 days (ground-up build)

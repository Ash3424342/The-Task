---
title: "Build Context Push Analytics Skill"
status: in-progress
priority: high
owner: me
due: 2026-04-18
deliverable: "Full skill document for Context Push domain (ground-up build from minimal)"
sources: "[[raw/compass-mcp-docs/Dataplatform · Analytics Skills Deep Dive - Coda.pdf]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
task-id: "CMP-2"
project: "[[Projects/compass-mcp-skills]]"
execution-log: "2026-04-07: Task created — Tier 1 priority, minimal stub. | 2026-04-19: Prompt 1 (discovery) complete — found 4 tables across cep/dashboards_bgv/ds_user_data schemas. | 2026-04-19: Prompt 2 (validation) blocked — ALL 4 tables are Trino-access-denied. MCP token has core_bgv access only. Shipping catalog-quality spec; Trino access request filed."
blocked-by: "Trino access to platform_iceberg.cep, platform_iceberg.dashboards_bgv, platform_iceberg.ds_user_data schemas"
---

# Build Context Push Analytics Skill

## What
Context Push Analytics is currently a **minimal** skill (summary only). Needs ground-up build to full skill with tables, schemas, guardrails, tested SQL, and example queries.

## Domain
Contextual push notification analytics — trigger patterns, delivery, and user response for context-aware notifications.

## Workflow
1. Use Compass tools (`search`, `get_database_tables`, `list_schema_fields`) to discover all Context Push tables
2. Document table schemas, row counts, partition columns
3. Identify join keys, must-filter columns, stale tables
4. Write and test 10+ example SQL queries
5. Document guardrails and data quality notes
6. Assemble into full skill document following Growth Analytics template format

## Success Criteria
- [ ] All relevant tables discovered and documented
- [ ] Partition columns and must-filter requirements identified
- [ ] 10+ tested SQL queries with expected output descriptions
- [ ] Guardrails section complete
- [ ] Follows the format from [[Deliverables/analytics-skills-deep-dive]]

## Estimated Time
3-4 days (ground-up build)

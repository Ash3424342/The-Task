---
title: "Compass MCP Skills — Domain Skill Enhancement & Creation"
status: in-progress
priority: high
owner: me
due: 2026-05-31
deliverable: "Enhanced and new Compass MCP skill documents for all Growth domains"
sources:
  - "[[raw/claude.ai-Scoping a context engine and compass MCP integration.pdf]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
execution-log: "2026-04-18/19: CMP-1 (Engagement & Comms) fully completed — schemas, live enum validation, 45 SQL examples, cross-skill joins all confirmed via Compass MCP. See Deliverables/cmp-1-engagement-skill-spec.md."
---

# Project: Compass MCP Skills

## Overview
[[wiki/Compass-MCP|Compass MCP]] is [[wiki/Groww|Groww]]'s central data platform MCP (DataHub + Superset + Trino, 80+ tools) sitting over 5000+ raw tables and 25000+ derived tables. Skills are schema reference documents that teach the LLM how to query a specific domain's tables with correct joins, filters, guardrails, and best practices.

**16 skills already exist.** My job is to **enhance existing skills + add new ones** for all Growth domains.

## Domains Assigned (All Under Growth)
1. User Events Analytics
2. Mutual Fund Analytics
3. MTF & Pledge Analytics
4. Credit Analytics
5. Growth Analytics (the Growth domain knowledge, analytics-funnel documented in detail)
6. Commodities Trading Analytics
7. **Engagement & Communications** (CTR/conversion data channels, campaigns) — most relevant to [[Projects/content-engine|Content Engine]]
8. F&O Analytics
9. Stocks & Trading Analytics
10. User Master & Growth Analytics
11. TTU/NTU/FID
12. 915 Trading Analytics

## Skill Structure (Based on Growth Analytics Example)
Each skill document includes:
- Table definitions with row counts and key dimensions
- Join keys and relationships
- Query best practices and guardrails
- Common filters and partition requirements
- Example queries
- Stale table registry
- Must-filter columns

## Key Tables (Engagement Skill — Most Relevant)
- `engagement_pn_narad_master` — 250M rows/day
- `engagement_email_backend_master` — 3-6M rows

## Current State: 16 Skills with Depth Assessment

| # | Skill | Depth | Status | Enhancement Opportunity |
|---|---|---|---|---|
| 1 | Stocks & Trading | Deep | ⬜ todo | Add column-level docs, more tested SQL |
| 2 | F&O Analytics | Deep | ⬜ todo | Future-date issue documented; add more SQL |
| 3 | Mutual Funds | Deep | ⬜ todo | No explicit guardrails — add them |
| 4 | User Master & Growth | Deep | ⬜ todo | Add cross-skill join patterns |
| 5 | User Events | Deep (unique) | ⬜ todo | Pinot + Iceberg, sampling documented |
| 6 | Onboarding | Deep | ⬜ todo | Stale tables well-flagged |
| 7 | UPI Payments | Medium | ⬜ todo | 2 stale tables; rolling window limitations |
| 8 | Engagement & Comms | Deep | **✅ done (CMP-1)** | All 9 tables fully cataloged, 45 SQL examples, 17 guardrails, cross-skill joins |
| 9 | MTF & Pledge | Medium | ⬜ todo | Stale `mtf_transactions_v2` flagged |
| 10 | Commodities | Medium | ⬜ todo | No partition on `commodity_order_master` |
| 11 | 915 Trading | Medium | ⬜ todo | Data starts June 2025 |
| 12 | Help & Support | Medium | ⬜ todo | 1 BROKEN table, 2 stale |
| 13 | Context Push | **Minimal** | **⚠️ blocked — Trino access denied** | Catalog-quality spec done; 4 tables found (cep/dashboards_bgv/ds_user_data schemas); Trino access request filed |
| 14 | User Financial Health | **Minimal** | ⬜ todo | Major enhancement opportunity |
| 15 | Datahub Navigation | **Minimal** | ⬜ todo | Meta-skill, may not need deep enhancement |
| 16 | MF Extended | **Minimal** | ⬜ todo | Major enhancement opportunity |

**CMP-1 completion summary (2026-04-18/19):** 9 tables cataloged, 17 guardrails, 45 SQL examples (Q1 live-confirmed), 2 cross-skill joins live-confirmed. Spec: `Deliverables/cmp-1-engagement-skill-spec.md`.

## Priority Tiers
1. **Tier 1 (foundational)**: Engagement & Comms, User Master, User Events — these directly feed the Content Engine
2. **Tier 2 (minimal skills, ground-up)**: Context Push, Financial Health, MF Extended
3. **Tier 3 (medium skills)**: Add depth to existing medium-documented skills
4. **Tier 4 (deep skills)**: Incremental polish on already well-documented skills

## Workflow Per Skill
- **Minimal (ground-up)**: 3-4 days — schema discovery, write from scratch
- **Medium (add depth)**: 2 days — enrich existing documentation
- **Deep (incremental)**: 1 day — polish and add edge cases

## Target Pace
1 skill every 2-3 days once rolling

## Critical Data Guardrails (Apply to ALL Skills)
- Universal join key: `user_account_id` (format `ACC<digits>`) — but **NOT actually universal**: PN/WA tables use `useraccountid` (no underscore); session/DAU/DND tables use `cuid` (UUID format, requires user master cross-ref)
- **Future-date defensive filter**: any time-series engagement table may carry future-dated or sentinel-value rows. Always bound: `WHERE <date_col> <= current_date`. Confirmed cases: `engagement_sms_backend_master` (event_date to 2026-05-01), `engagement_dnd_fact` (date to 2038-01-18 Unix int32 sentinel), `fno_order_master.pt` (to 2090).
- Pinot events are SAMPLED — must use `boosting_factor`, last 1-2 days broken
- `engagement_session_indepth` = 58.1B rows total / **60M rows/day** — SINGLE DAY queries only
- `engagement_app_fact` = 16.2B rows — `MAX(week)` scan times out; always partition-filter with known Sunday date (e.g., `week = DATE '2026-04-13'`)
- Trino safety: read-only, partition enforcement, 600s timeout, 5000 row limit
- Trino syntax: use `current_date - interval '1' day` (NOT `current_date - 1`); use `SUBSTR(str, LENGTH(str)-n+1)` (no `RIGHT()`)
- **NEVER use**: `stocks_order` (stale Oct 2025), `invest_user_fid_info` (stale Jun 2024), `hns_freshchat_tickets_master_trino` (BROKEN), `mtf_transactions_v2` (stale Nov 2024)
- For user-master joins, prefer `growth_user_master_ultimate` (114 cols, has `dim_user_status` pre-built TTU/NTU/SignUp) over bare `growth_user_master`

## Compass Setup
- Proxy on port 3100, VPN required
- Tools: `search`, `get_database_tables`, `list_schema_fields`, `run_trino_query`
- Workflow is conversational from Claude Code, not scripted

## Long-Term Vision
Analytics → Segmentation → Content → Campaign → [[wiki/NARAD|NARAD]] → Approve (fully automated loop, months away)

## Deliverables (Completed Planning Docs)
- [[Deliverables/task-description-v2]] — Full project spec
- [[Deliverables/project-plan-v2]] — Phased plan with skill inventory
- [[Deliverables/implementation-plan-v2]] — Technical blueprint
- [[Deliverables/bootstrap-prompt]] — Claude Code session starter

## Related
- [[Projects/content-engine]] (parallel system, indirect connection via Engagement skill)
- [[wiki/Compass-MCP|Compass MCP]]

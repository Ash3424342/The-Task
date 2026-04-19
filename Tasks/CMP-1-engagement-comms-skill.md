---
title: "Enhance Engagement & Communications Compass Skill"
status: in-progress
priority: high
owner: me
due: 2026-04-25
deliverable: "Enhanced skill document for Engagement & Comms domain in Compass MCP"
sources:
  - "[[raw/claude.ai-Scoping a context engine and compass MCP integration.pdf]]"
  - "[[raw/compass-mcp-docs/Dataplatform · Analytics Skills Deep Dive - Coda.pdf]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
task-id: "CMP-1"
project: "[[Projects/compass-mcp-skills]]"
spec: "[[Deliverables/cmp-1-engagement-skill-spec]]"
execution-log: |
  2026-04-07: Task extracted from scoping session — Tier 1 priority, directly feeds Content Engine
  2026-04-18: Rebaselined (was 4 days overdue). Status todo → in-progress. Drafted full enhancement spec at Deliverables/cmp-1-engagement-skill-spec.md — 9 tables enumerated, per-table doc template defined, draft column maps for the 2 hero tables, 10 discovery queries scripted for next Compass session. Acceptance criteria locked.
---

# Enhance Engagement & Communications Compass Skill

## What
The Engagement & Comms skill is the most important for the [[Projects/content-engine|Content Engine]] integration. It covers CTR/conversion data channels, campaigns, email backend data.

> **Detailed enhancement spec:** [[Deliverables/cmp-1-engagement-skill-spec]] — defines tables in scope, per-table template, draft column maps, 10-step discovery agenda for the next Compass-connected session, and acceptance criteria.

## Status

**In progress as of 2026-04-18.** The enhancement spec is checked in. Next step requires a live Compass connection (proxy on :3100, VPN). When connected, walk through §6 of the spec (10 discovery queries) and fill the per-table sections in §3 of the spec.

## Key Tables (9 total — see spec for full list)
- `engagement_pn_narad_master` — 250M rows/day (push notifications via [[wiki/NARAD|NARAD]])
- `engagement_email_backend_master` — 3-6M rows (email delivery/engagement data) — **primary focus**
- `engagement_sms_backend_master`, `engagement_whatsapp_backend_master`, `engagement_user_session_daily`, `engagement_dau_indepth`, `engagement_session_indepth` (57B total — single day only), `engagement_app_fact`, `engagement_dnd_fact`

## What to Document (now mapped to spec sections)
- [x] Scope and table list locked (spec §1, §2)
- [x] Per-table doc template defined (spec §3)
- [x] Cross-skill joins enumerated (spec §5)
- [x] Acceptance criteria locked (spec §7)
- [ ] All table schemas with column descriptions — **needs Compass session** (spec §4 has draft column maps for 2 hero tables; needs validation + completion for all 9)
- [ ] Join keys and relationships between tables — partial (spec §5 has cross-skill; intra-skill joins still TBD)
- [ ] Partition requirements (these are huge tables) — partition column known for `pn_narad_master` (event_date), `session_indepth` (session_date); rest TBD
- [ ] Must-filter columns — partial
- [ ] Query guardrails (avoid full scans) — partial
- [ ] Example queries for common use cases — **5 minimum per table required, 0 currently checked-in**
- [ ] Stale table indicators — none known in Engagement domain; confirm in Compass

## Priority
Tier 1 — foundational, directly feeds Content Engine's email analytics. Pattern for CMP-2/3/4.

## Blockers
- Requires Compass MCP connection (VPN + proxy on :3100) to execute §6 discovery and §7 acceptance criteria. Without it, work is bounded to the spec layer.

# Groww — Project Plan (v2 — Updated)

**Owner:** Intern, Growth Team  
**Date:** April 6, 2026  
**Version:** 2.0 — Updated with Compass MCP documentation, Skills Deep Dive, and Setup Guide

---

## Task Overview

| | Content Engine | Compass MCP Skills |
|---|---|---|
| **Priority** | Side project | Primary work |
| **Timeline** | 1–2 weeks | Ongoing |
| **What it is** | AI-powered HTML email generator (MCP server + Claude Code) | Enhance 16 existing + build new domain skill documents for Compass |
| **Deliverable** | Working MCP server that generates production-ready HTML emails | Deeper skills with more tables, tested SQL, guardrails; new skills for uncovered domains |
| **Relationship** | Parallel systems — not integrated. Connected indirectly through human decisions. |

---

# THE 4-BOX PIPELINE — WHERE EVERYTHING SITS

| Box | Owner | System | My Role |
|---|---|---|---|
| Analytics | Compass MCP + Superset | 80+ tools, 16 skills, DataHub + Superset + Trino | Enhance & expand the skills |
| Segmentation | Audience Manager + NARAD + ML | BQ → AM → WebEngage | Not my scope |
| Content | **My Content Engine** | 18 blocks, 8 recipes, MCP tools | Building this |
| Campaign | WebEngage + NARAD + Netcore/FCM | Existing delivery infrastructure | Not my scope |

Key clarification: Compass and Content Engine are **parallel**, not sequential. Compass answers "what happened?" (analytics). Content Engine answers "what email should we send?" (content). WBR/MBR reports are handled by Compass's scheduled report feature — not by my Content Engine.

---

# TASK 2 — Compass MCP Skills (Primary Work)

## What Compass Actually Is

Compass is NOT just text-to-SQL. It is a unified data platform interface connecting three systems via MCP with 80+ tools:

- **DataHub** — Data catalog (dataset discovery, schema exploration, metadata management, lineage, event instrumentation metadata with GitHub source links)
- **Superset** — Visualization (chart creation, dashboard building, scheduled reports via email/Slack/webhook, data alerts)
- **Trino** — SQL engine (ad-hoc querying with safety guardrails: read-only, partition enforcement, 600s timeout, 5000 row limit)

Accessed via Claude Code, Cursor, VS Code, IntelliJ. Runs on Groww's VPN with local proxy on `127.0.0.1:3100`.

## What Skills Are

Skills are pre-built knowledge packages that encode deep domain knowledge: which tables to use, partition columns, column semantics, all filter values, optimized SQL patterns, data quality warnings, stale table alerts, and join keys. Users ask questions in plain English — the skill handles the rest.

## Current State: 16 Skills Already Exist

My job is NOT to write these from scratch. 16 skills already exist with varying depth. My work is to enhance existing skills and build new ones.

### Skill Inventory with Documentation Depth Assessment

| # | Skill | Depth | Tables Documented | Key Gaps / Enhancement Opportunities |
|---|---|---|---|---|
| 1 | Stocks & Trading | Deep | 3 tables, dimensions, guardrails | Add column-level docs; more tested SQL |
| 2 | F&O Analytics | Deep | 10 tables, dimensions, guardrails | Future-date issue well-documented; add more SQL |
| 3 | Mutual Funds | Deep | 10+ tables, table selection matrix | No explicit guardrails listed — add them |
| 4 | User Master & Growth | Deep | 10 tables, value distributions | Add cross-skill join patterns |
| 5 | User Events | Deep (unique) | Pinot + Iceberg, dual-level metadata | Sampling correction well-documented; app filters complete |
| 6 | Onboarding | Deep | 9 tables, full funnel, PII warnings | Stale tables well-flagged |
| 7 | UPI Payments | Medium | 6 tables, freshness flags | 2 stale tables; rolling window limitations |
| 8 | Engagement & Comms | Deep | 9 tables, daily volumes, PN funnel | Most relevant to Content Engine; well-documented |
| 9 | MTF & Pledge | Medium | 9 tables, freshness status | Stale `mtf_transactions_v2` flagged |
| 10 | Commodities | Medium | 6 tables, top underlyings | No partition on `commodity_order_master` — document workaround |
| 11 | 915 Trading | Medium | 7 tables, pre-computed segments | Data starts June 2025; cohort table gotcha |
| 12 | Help & Support | Medium | 7 tables, channel codes | 1 BROKEN table, 2 stale; chatbot is rolling 3-day window |
| 13 | Context Push | **Minimal** | Summary only | **Major enhancement opportunity** |
| 14 | User Financial Health | **Minimal** | Summary only | **Major enhancement opportunity** |
| 15 | Datahub Navigation | **Minimal** | Meta-skill, summary only | May not need deep enhancement |
| 16 | MF Extended | **Minimal** | Summary only | **Major enhancement opportunity** |

### Enhancement Priority

**Tier 1 — Deepen first (minimal documentation, clear gaps):**
1. Context Push Analytics — expand from summary to full skill
2. User Financial Health Analytics — expand from summary to full skill
3. MF Analytics Extended — expand from summary to full skill

**Tier 2 — Add tested SQL and column docs (already have good structure):**
4. Engagement & Communications — most relevant to Content Engine; add column-level docs matching the Growth Analytics reference format; write and test 10+ SQL queries
5. Commodities Trading — relevant to Content Engine; document the no-partition workaround for `commodity_order_master`
6. Mutual Funds — add explicit guardrails (currently missing)

**Tier 3 — Polish and cross-link (already deep):**
7. Stocks & Trading — add more tested SQL
8. F&O Analytics — add more tested SQL
9. User Master & Growth — add cross-skill join patterns
10. Onboarding, UPI, MTF, 915, H&S — incremental improvements

**Tier 4 — New skills:**
11. Any new domains as they emerge or as data engineering creates new tables

## Workflow for Skill Enhancement

### For Minimal Skills (Context Push, Financial Health, MF Extended)

These need to be built up from summary to full skill. Effectively a ground-up build:

**Day 1 — Discovery:**
- [ ] Identify all tables in the domain's catalog/schema via Compass (`search` tool or `get_database_tables`)
- [ ] Gather table metadata: columns, types, row counts, partition columns, freshness
- [ ] Interview domain data owner (find via Datahub ownership)
- [ ] Ask: top 10 business questions, common query mistakes, authoritative vs deprecated tables, known data quality issues
- [ ] Find existing Superset dashboards for the domain

**Day 2 — Draft Structure:**
- [ ] Write Tables Overview (table, schema, rows, grain, partition, purpose)
- [ ] Document Key Dimensions with exact enum values (run `SELECT DISTINCT` queries)
- [ ] Write Key Guardrails (partition filters, stale tables, type casting, dedup rules)

**Day 3 — Queries & Testing:**
- [ ] Write 10+ example questions the skill should answer
- [ ] Write and test SQL for each question via Compass's `run_trino_query`
- [ ] Document any query failures → add to guardrails
- [ ] Write Query Strategy Reference table

**Day 4 — Review & Publish:**
- [ ] Self-review against quality checklist
- [ ] Domain owner review
- [ ] Publish to Datahub

### For Medium Skills (UPI, MTF, Commodities, 915, H&S)

These have good structure but need depth:

**Day 1–2:**
- [ ] Add column-level documentation for primary tables (matching Growth Analytics reference format)
- [ ] Write and test 5–10 additional SQL queries
- [ ] Add any new tables discovered since initial skill creation
- [ ] Expand data quality notes based on real-world query failures

**Day 3:**
- [ ] Add cross-skill join patterns (e.g., "join commodity orders to user_master for demographic analysis")
- [ ] Review and publish

### For Deep Skills (Stocks, F&O, MF, User Master, Events, Onboarding, Engagement)

These are already well-documented. Enhancement is incremental:

- [ ] Write and test additional SQL queries (especially complex multi-table joins)
- [ ] Add column-level docs where missing
- [ ] Add cross-skill patterns
- [ ] Flag any new stale tables or data quality changes

## Quality Checklist (Per Skill)

Use this before marking any skill enhancement as done:

- [ ] Every table has: schema, rows, grain, freshness, purpose, partition column
- [ ] Key dimensions table with exact enum values
- [ ] At least 8 example questions listed
- [ ] At least 5 key guardrails documented (partition filters, stale tables, type casting)
- [ ] Stale table warnings with "Use Instead" alternatives
- [ ] At least 10 SQL queries tested against real data via Compass
- [ ] No references to broken/stale tables without warnings
- [ ] PII columns flagged
- [ ] Published on Datahub

## Critical Reference Data (From Deep Dive)

### Stale Table Registry — Never Use These

| Table | Last Data | Skill | Use Instead |
|---|---|---|---|
| `stocks_order` | Oct 2025 | Stocks | `core_stocks_order` |
| `invest_user_fid_info` | Jun 2024 | Onboarding | `fact_onboarding_invest` FID columns |
| `daily_onb_conversions` | EMPTY | Onboarding | `daily_onboarding_and_fid` |
| `fno_usersegmentation_monthly` | Oct 2025 | F&O | Use order tables |
| `groww_upi_debit_fact_table` | Feb 2024 | UPI | Historical only |
| `hns_freshchat_tickets_master_trino` | BROKEN | H&S | Do not use |
| `hns_csat_dashboard` | Mar 2024 | H&S | Historical CSAT only |
| `mtf_transactions_v2` | Nov 2024 | MTF | `aay_mtf_fid_agg` |

### Schema Directory

| Schema | Tables | Domain |
|---|---|---|
| `platform_iceberg.core_invest` | Stocks, F&O, MF, MTF, Commodity, 915 | Trading & Investment |
| `platform_iceberg.core_bgv` | Growth, Onboarding, UPI, Engagement, H&S | User Lifecycle & Operations |
| `platform_iceberg.dwh_invest` | MF enriched | Mutual Funds Data Warehouse |
| `platform_iceberg.dump_invest` | 915 cohorts | Periodic Snapshots |
| `platform_iceberg.ems` | Event data (Iceberg) | Raw Events |
| `platform_dp_pinot.default` | Event data (Pinot — sampled!) | Sampled Events |

### Largest Tables (Must-Filter Reference)

| Table | Rows | Skill | Must Filter On |
|---|---|---|---|
| `engagement_session_indepth` | 57B total | Engagement | `session_date` (SINGLE DAY only!) |
| `user_product_txn_master_fact` | 8.3B | User Master | `order_date` |
| `fact_transaction_stocks` | 7.1B | Stocks | `order_date` |
| `fno_order_master` | 3.8B | F&O | `pt` (warning: contains future dates to 2090!) |
| `core_stocks_order` | 2.5B | Stocks | `order_date` |
| `fno_pnl_master` | 1.9B | F&O | `pt` |
| `engagement_pn_narad_master` | 250M/day | Engagement | `event_date` |
| `growth_user_master_ultimate` | 117M | User Master | Recommended for all user analytics |
| `onb_usercheckpoints_master` | 117M | Onboarding | `groww_signup_time` |
| `commodity_order_master` | 71.6M | Commodities | **No partition!** Use `commodity_day_view_seg` instead |

### Universal Join Key

All user-level tables across ALL skills: `user_account_id` (format: `ACC<digits>`)

## Compass Setup (For My Own Reference)

```bash
# Prerequisites: Node.js 18+, VPN connected, Groww email & password

# Option 1: Auto setup
./scripts/compass-mcp-setup.sh

# Option 2: Manual — Claude Code
claude mcp add compass --transport http http://127.0.0.1:3100/mcp

# Diagnostics
./scripts/compass-mcp-setup.sh --check

# Proxy logs
tail -f ~/.compass-mcp/proxy.log
```

## Timeline Estimate (Skills)

| Work Item | Estimated Days | Notes |
|---|---|---|
| Context Push (minimal → full) | 3–4 | Ground-up build |
| User Financial Health (minimal → full) | 3–4 | Ground-up build |
| MF Extended (minimal → full) | 3–4 | Ground-up build |
| Engagement & Comms enhancement | 2–3 | Add column docs + tested SQL |
| Commodities enhancement | 2 | No-partition workaround + SQL |
| Mutual Funds — add guardrails | 1–2 | Currently missing guardrails |
| Deep skills — incremental | 1 each | Additional SQL, cross-skill patterns |
| New skills (as assigned) | 3–4 each | Full build |

---

# TASK 1 — Content Engine (Side Project, 1–2 Weeks)

## Architecture

```
Claude Code / LLM (CM interacts in natural language)
          │ MCP tool calls
          ▼
Content Engine MCP Server (Python)
  ├── Recipe Management: list_recipes, get_recipe, recommend_recipe
  ├── Generation Engine: generate_email (orchestrates LLM + charts + render)
  ├── Chart Renderer: generate_chart (Playwright → retina PNG)
  ├── HTML Renderer: render recipe JSON → production email HTML
  └── Export: export_email (HTML + image URLs → ready for NARAD)

Data Layer:
  /recipes/*.json    /blocks/*.json    /brands/*.json
  /charts/templates/ /renderer/templates/ /output/
```

## Component Checklist

| Component | What It Does | Input | Output |
|---|---|---|---|
| Email Analyzer | Parse 450 HTML emails from WebEngage | Raw HTML files | Classified + decomposed JSON per email |
| Block Library | Define 18 block types as schemas | Analysis output + manual curation | Pydantic models + JSON schemas |
| Recipe System | Define 8 email template patterns | Block library + email clustering | Recipe JSON files with generation hints |
| Chart Renderer | Generate 5 chart types as retina PNGs | Structured data (prices, labels) | PNG files via Playwright |
| HTML Renderer | Assemble recipe into email HTML | Recipe JSON + filled block data | Production HTML (table-based, inline CSS) |
| MCP Server | Expose 7 tools for Claude Code | Natural language via MCP | Orchestrates all above |

## Sprint Plan

### Week 1: Foundation

**Day 1 — Setup + Email Export**
- [ ] Set up project repository and Python environment
- [ ] Export emails from WebEngage (manual or API)
- [ ] Organize raw HTML files
- [ ] Set up Playwright for chart rendering
- Checkpoint: 450 HTML files organized and accessible

**Day 2 — Email Analyzer**
- [ ] Build HTML parser (BeautifulSoup)
- [ ] Build block detector for 6 most common blocks
- [ ] Build email type classifier (rule-based)
- [ ] Run on 50 emails, validate manually
- Checkpoint: Analyzer at ~80% accuracy on 50 emails

**Day 3 — Analyzer Refinement + Block Library**
- [ ] Add remaining 12 block detectors
- [ ] Run on full 450-email corpus
- [ ] Fix misclassifications
- [ ] Start block JSON schemas
- Checkpoint: Full corpus analyzed, 10+ schemas drafted

**Day 4 — Block Library + Recipe System**
- [ ] Complete all 18 block schemas
- [ ] Extract brand configs from real emails
- [ ] Cluster emails → define 5–8 recipes
- [ ] Write recipe JSON files
- Checkpoint: Complete block library + recipes

**Day 5 — HTML Renderer**
- [ ] Write email boilerplate HTML
- [ ] Write Jinja2 templates for each block (table-based, inline CSS)
- [ ] Build recipe → HTML renderer
- [ ] Test against original emails
- Checkpoint: Can render a recipe into valid HTML

### Week 2: AI Layer + MCP

**Day 6 — Chart Renderer (Part 1)**
- [ ] Build `price_card` template matching crude oil example
- [ ] Build `listing_card` template
- [ ] Test at retina resolution
- Checkpoint: 2 chart types working

**Day 7 — Chart Renderer (Part 2)**
- [ ] Build `comparison`, `portfolio`, `metric` templates
- [ ] Image upload function (staging)
- Checkpoint: All 5 chart types working

**Day 8 — MCP Server**
- [ ] Initialize MCP server with Python SDK
- [ ] Implement read-only tools: list_recipes, get_recipe, list_blocks
- [ ] Implement generate_chart
- [ ] Test from Claude Code
- Checkpoint: MCP server running, basic tools working

**Day 9 — Generation Engine**
- [ ] Implement generate_email (core orchestrator)
- [ ] Write LLM prompts for copy generation (Anthropic API)
- [ ] Implement preview_html and export_email
- [ ] End-to-end test
- Checkpoint: Full pipeline working

**Day 10 — Testing + Demo**
- [ ] Run 5 generation scenarios, compare against real emails
- [ ] Fix edge cases
- [ ] Write README
- [ ] Demo to manager
- Checkpoint: Demo-ready

## Technical Decisions

| Decision | Choice | Reason |
|---|---|---|
| Language | Python | Comfortable, good MCP SDK |
| Chart rendering | Playwright (HTML → screenshot) | Best fidelity to Groww's design |
| Image hosting | Local first, staging bucket later | Cloud access TBD |
| LLM for copy | Claude Sonnet via Anthropic API | Best quality for marketing copy |
| Email testing | Manual (Gmail Android) | Primary user base |

## Engagement Skill Connection

The Engagement & Communications skill documents the email performance tables that are directly relevant to the Content Engine:

- `engagement_email_backend_master` (3–6M rows/day) — every email sent. Could be used to map campaign performance back to email templates.
- `engagement_pn_narad_master` (250M/day) — push notification data via NARAD.
- PN status distribution: SENT 72%, OPTED_OUT 9%, VENDOR_FAILURE 9%, FREQUENCY_CAPPED 8.6%
- Push notification funnel: 3M total → 2M sent (66%) → 440K received (22%) → 19K clicked (4.3% CTR)

This data can inform recipe design — which email types have the highest engagement? But this is an indirect connection, not a direct integration.

---

# PARALLEL WORK SCHEDULE

```
Week 1:  [Content Engine: Foundation]     +  [Skills: Set up Compass, explore minimal skills]
Week 2:  [Content Engine: AI + MCP]       +  [Skills: Context Push (ground-up build)]
Week 3:  [Content Engine: Done ✓]         →  [Skills: User Financial Health (ground-up build)]
Week 4:                                       [Skills: MF Extended (ground-up) + Engagement enhancement]
Week 5:                                       [Skills: Commodities + Mutual Funds enhancement]
Week 6:                                       [Skills: Deep skills incremental improvement]
Week 7:                                       [Skills: Cross-skill patterns + new skills as assigned]
Week 8+:                                      [Skills: Ongoing enhancement + Content Engine v2]
```

Content Engine is a focused 2-week sprint. Skills work is ongoing and scales to fill available time. During Week 1–2, skill work is limited to Compass setup and initial exploration of the minimal skills, since Content Engine takes priority during that window.

---

# OPEN QUESTIONS FOR MANAGER

**Priority questions (resolve before starting):**

1. **Compass setup** — Have I run the setup guide? Do I have VPN + credentials?
2. **WebEngage access** — How do I export the 90-day email archive?
3. **Skill enhancement priority** — Should I start with the 3 minimal skills (Context Push, Financial Health, MF Extended), or does my manager have different priorities?
4. **What counts as "done" for skill enhancement?** Adding 5 more queries? Full column-level docs? Testing coverage?
5. **Anthropic API key** — Do I have one for copy generation?

**Secondary questions (can resolve during Week 1):**

6. Brand guidelines per sub-brand (colors, fonts, logos)?
7. SEBI disclaimer text templates?
8. Groww cloud upload access for chart images?
9. Skill review process — who validates before Datahub publish?
10. Existing Superset dashboards per domain I can reference?
11. Skill format preference — Growth Analytics reference format (17-page detailed) or Deep Dive format (summary with dimensions + guardrails)?
12. Is there a backlog of requested new skills, or do I identify gaps myself?

---

# SUCCESS CRITERIA

## Content Engine v1 is Done When:
- [ ] CM can describe an email → receives production HTML
- [ ] 5 generated emails compared against real ones and approved
- [ ] HTML renders on Gmail Android
- [ ] All 4 sub-brands theme correctly
- [ ] MCP server stable, all 7 tools working
- [ ] Manager has seen a demo

## A Skill Enhancement is Done When:
- [ ] New tables/columns documented where gaps existed
- [ ] 5+ new SQL queries tested via Compass
- [ ] Additional guardrails added from real failures
- [ ] Published/updated on Datahub
- [ ] Compass answers 5 new NL questions correctly

## A New Skill (Ground-Up) is Done When:
- [ ] All tables documented (schema, rows, grain, freshness, partition, purpose)
- [ ] Key dimensions with exact enum values
- [ ] 8+ example questions
- [ ] 5+ key guardrails
- [ ] 10+ tested SQL queries
- [ ] Stale table warnings included
- [ ] Quality checklist passes
- [ ] Domain owner reviewed
- [ ] Published on Datahub

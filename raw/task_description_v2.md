# Task Description Document (v2 — Updated)
## Content Engine & Compass MCP Skills — Groww, Growth Team

**Author:** Intern, Growth Team  
**Assigned by:** Manager  
**Date:** April 6, 2026  
**Status:** Active  
**Version:** 2.0 — Updated with Compass MCP documentation, Skills Deep Dive, and Setup Guide

---

# Table of Contents

1. Project Overview
2. Company & Domain Context
3. The 4-Box Pipeline — Where My Work Fits
4. Task 1: Content Engine (Side Project)
5. Task 2: Compass MCP Skills (Primary Work)
6. Long-Term Vision
7. Technical Architecture
8. My Specific Responsibilities
9. Scope Boundaries
10. Deliverables Checklist
11. Success Criteria
12. Dependencies & Risks
13. Open Questions

---

# 1. Project Overview

## What I'm Building

I have been assigned two tasks that sit in different parts of Groww's data and marketing infrastructure:

**Task 1 — Content Engine (side project, 1–2 weeks):** An AI-powered HTML email generator delivered as an MCP server. Campaign managers describe what email they want in natural language via Claude Code. The system selects an email template ("recipe"), generates the copy and chart images using AI, and outputs production-ready HTML that gets pasted into NARAD (Groww's campaign distribution platform) for sending. This is the "Content" box in Groww's marketing pipeline.

**Task 2 — Compass MCP Skills (primary ongoing work):** Enhancing existing and building new domain-specific skill documents for Compass — Groww's unified data platform MCP that connects DataHub (catalog), Superset (visualization), and Trino (SQL engine) into a single conversational interface with 80+ tools. 16 skills already exist covering all major business domains. My work involves deepening these existing skills (adding more tables, queries, guardrails) and building new skills for domains not yet covered.

## Why This Matters

**Content Engine:** Today, creating a single marketing email at Groww involves a multi-team, multi-day process: the creative team designs in Figma, a UX/engineering team or third party converts the design to HTML, and the output is manually pasted into NARAD. The Content Engine replaces this with a prompt-driven workflow where a campaign manager can generate a production-ready email in minutes, not days.

**Compass Skills:** Groww's data platform has hundreds of tables across multiple schemas, with billions of rows, inconsistent naming, and complex join relationships. Even experienced analysts spend significant time finding the right table, understanding column semantics, and writing efficient queries. Skills are pre-built knowledge packages that tell the AI assistant exactly which tables, columns, filters, and query patterns to use for each business domain. Instead of users needing to know the data schema, they just ask questions in plain English — the skill handles the rest. Better skills mean more accurate answers, fewer expensive query mistakes, and broader coverage of business questions.

## How the Two Tasks Relate

These tasks are **parallel systems**, not directly integrated. They serve different purposes in Groww's marketing pipeline:

- **Compass** answers "what happened?" — analytics, querying, dashboards, reporting
- **Content Engine** answers "what email should we send?" — content generation

The connection is **indirect**: Compass tells the team which cohorts are performing (e.g., "commodity traders dropped 15% this month"), and the Content Engine generates the emails for those cohorts. They don't call each other. A human (the campaign manager or analyst) bridges the gap by using Compass insights to decide what email to create, then using the Content Engine to generate it.

---

# 2. Company & Domain Context

## About Groww

Groww is one of India's largest retail investment platforms. Users can trade stocks, mutual funds, futures & options (F&O), and commodities (via MCX). The platform is regulated by SEBI (Securities and Exchange Board of India), which means all user-facing communications must comply with financial advertising regulations — no investment advice, no guaranteed returns, mandatory disclaimers.

Groww operates multiple sub-brands, each with its own visual identity for emails:

- **Groww** — the main brand (green accent, #00D09C)
- **915** — separate trading app focused on F&O and derivatives, distinct from the main Groww app
- **AMC** — asset management sub-brand
- **W** — additional sub-brand

The platform serves approximately 117 million registered users (72M signup-only, 27M onboarded & transacted, 17M onboarded & non-transacted), with a large base on Android mobile (98M of 117M users).

## Key Internal Systems

**NARAD:** Groww's campaign distribution platform. Handles push notification delivery (170–250M push notifications per day), email sending, and campaign orchestration. Emails are sent through NARAD by pasting HTML into its interface. NARAD handles delivery, targeting (via segment IDs), and approval workflows.

**WebEngage:** Marketing engagement platform used to manage campaigns. Historical emails (approximately 150 per month) are stored here. Also handles user segmentation and campaign triggering. The segmentation flow is: BigQuery → Audience Manager → WebEngage.

**Compass MCP:** Groww's unified data platform interface — NOT just text-to-SQL. It connects three systems into a single conversational MCP with 80+ tools:

- **DataHub** — Data catalog (dataset discovery, schema exploration, metadata management, lineage tracking, event metadata with GitHub source links)
- **Superset** — Visualization (chart creation, dashboard building, scheduled reports, data alerts)
- **Trino** — SQL engine (ad-hoc querying against the Iceberg data lake with safety guardrails: read-only, partition enforcement, timeout protection, row limits)

Compass is accessed via Claude Code, Cursor, VS Code, or any MCP-compatible client. It runs on Groww's internal VPN network, with a local proxy on port 3100 handling connectivity.

Compass has 16 domain-specific analytics skills that encode deep knowledge about which tables to use, how to filter, what columns mean, and which queries are safe to run. These skills are the "brain" of Compass — the single most powerful feature.

**Groww Internal Cloud:** Image hosting for email assets. Chart images and other email graphics are uploaded here, and the resulting URLs are embedded in email HTML.

## Data Infrastructure Scale

The data infrastructure is massive. Key reference points from the Skills Deep Dive:

**Largest tables by row count:**
- `engagement_session_indepth` — 57 billion rows (SINGLE DAY queries only!)
- `user_product_txn_master_fact` — 8.3 billion rows (must filter on `order_date`)
- `fact_transaction_stocks` — 7.1 billion rows (must filter on `order_date`)
- `fno_order_master` — 3.8 billion rows (must filter on `pt`)
- `core_stocks_order` — 2.5 billion rows (must filter on `order_date`)
- `fno_pnl_master` — 1.9 billion rows (must filter on `pt`)
- `engagement_pn_narad_master` — 170–250 million rows per day
- `growth_user_master_ultimate` — 117 million rows

**Schema directory:**
- `platform_iceberg.core_invest` — Stocks, F&O, MF, MTF, Commodity, 915 (Trading & Investment)
- `platform_iceberg.core_bgv` — Growth, Onboarding, UPI, Engagement, H&S (User Lifecycle & Operations)
- `platform_iceberg.dwh_invest` — MF enriched (Mutual Funds Data Warehouse)
- `platform_iceberg.dump_invest` — 915 cohorts (Periodic Snapshots)
- `platform_iceberg.ems` — Event data (Raw Events on Iceberg)
- `platform_dp_pinot.default` — Event data (Sampled Events on Pinot)

**Universal join key:** `user_account_id` (format: `ACC<digits>`) across ALL user-level tables in ALL skills.

---

# 3. The 4-Box Pipeline — Where My Work Fits

Groww's marketing pipeline has four boxes. Each box has a clear owner and system:

| Box | Owner | System | What It Does |
|---|---|---|---|
| **Analytics** | Compass MCP + Superset | Compass queries data, creates charts/dashboards. 16 skills cover all domains. | Answers "what happened?" |
| **Segmentation** | Audience Manager + NARAD + ML models | BQ → AM → WebEngage. Compass can query segments but doesn't create them. | Answers "who should we target?" |
| **Content** | **My Content Engine** | 18 blocks, recipes, MCP tools. I'm building this. | Answers "what email should we send?" |
| **Campaign** | WebEngage + NARAD + Netcore/FCM | Existing delivery infrastructure. | Handles "how do we deliver it?" |

**My Content Engine is the Content box.** Everything else exists. I plug in between Segmentation (who) and Campaign (deliver).

**Key distinction:** The Analytics box (Compass) and Content box (my engine) are parallel, not sequential. Compass handles automated dashboard PDFs and data alerts on a schedule. My system handles beautiful branded HTML emails with blocks and recipes. They don't overlap.

**WBR/MBR (Weekly/Monthly Business Reviews):** These are handled by Compass's scheduled reports feature, not by my Content Engine. Compass can "send this dashboard as a PDF every Monday at 9am" via email, Slack, or webhook. My system generates marketing/educational/onboarding emails — designed HTML with blocks and recipes.

---

# 4. Task 1: Content Engine

## 4.1 What It Is

The Content Engine is an MCP server (written in Python) that exposes tools for AI-powered email generation. It is callable from Claude Code, meaning a campaign manager interacts with it through natural language conversation. The system handles the full pipeline: template selection, copy generation, chart image rendering, and HTML email assembly.

## 4.2 The Current Workflow (Being Replaced)

1. Campaign manager identifies the need for an email
2. Creative team designs the email layout in Figma
3. UX/engineering team or third-party vendor converts Figma design to HTML
4. Images are manually created, uploaded to Groww's cloud, and URLs embedded in HTML
5. Final HTML is pasted into NARAD for distribution
6. NARAD sends the email to the targeted user segment

This process takes days and involves multiple teams.

## 4.3 The New Workflow (What I'm Building)

1. Campaign manager opens Claude Code (connected to the Content Engine MCP server)
2. CM describes what they want: "Generate a commodity educational email about crude oil price movement this month for active traders"
3. The system recommends a recipe (or the CM picks one manually)
4. The system generates: email copy (headlines, body text, factor cards), chart images (price cards, listings), email subject line and preheader
5. The system renders everything into a single production-ready HTML file with all image URLs embedded
6. CM pastes the HTML into NARAD and sends

## 4.4 Components

The Content Engine consists of six components I am building from scratch:

### Component 1: Email Analyzer Pipeline

Ingests approximately 450 raw HTML emails exported from WebEngage (90 days of history at ~150 emails/month), parses their HTML structure, auto-classifies each email by type, and decomposes them into reusable blocks.

**Why it's needed:** There is no existing standardized template system. The analyzer reverse-engineers patterns from real emails to bootstrap the block library and recipe system.

**Input:** Raw HTML email files from WebEngage.

**Output per email:** A structured JSON containing the email's detected classification (educational, onboarding, promotional, commodity update, trading alert, feature announcement, weekly digest), a list of detected blocks in order, and metadata (subject line, word count, image count, CTA count).

**Technical approach:** HTML parsing via BeautifulSoup, heuristic pattern matching for block detection, rule-based classification combining subject line keywords, block patterns, and content analysis.

### Component 2: Block Library

Defines 18 reusable email building blocks as formal JSON schemas. Each block has a defined structure (required fields, optional fields), brand-specific style variants, and an associated HTML template.

**The 18 blocks:**

1. **header_logo** — Brand banner with the appropriate logo (Groww, 915, AMC, or W)
2. **hero_text** — Main headline and subtitle, with optional accent-colored highlight word
3. **hero_image** — Full-width image (not a chart)
4. **chart_image** — Programmatically generated chart PNG (triggers the chart renderer)
5. **article_section** — Heading followed by one or more paragraphs of body copy
6. **factor_card** — Mint/teal-tinted explainer card with bold title and description
7. **cta_button** — Call-to-action button (green or blue)
8. **social_proof_banner** — Trust/credibility strip
9. **footer_standard** — SEBI registration, social media links, unsubscribe link (mandatory for regulatory compliance)
10. **divider** — Horizontal line separator
11. **card_wrapper** — Container that wraps other blocks in a card-style visual group
12. **contract_row** — Single commodity contract line (instrument, exchange, price, change, lot size)
13. **filter_pills** — Tab-style filter chips
14. **listing_header** — Section header for list sections
15. **step_instruction** — Numbered how-to step with title and description
16. **app_screenshot_pair** — Side-by-side app screenshots
17. **product_category_grid** — 2×2 grid of product category tiles
18. **social_showcase** — Social proof with specific metrics

### Component 3: Recipe System

Recipes are pre-defined ordered sequences of blocks that represent a specific email type. They include metadata (subject line template, preheader template) and generation hints (tone, reading level, word count target, things to avoid).

**Initial recipes (8):**

1. **commodity_educational** — Commodity price movement explainer (~15–17 blocks)
2. **onboarding_welcome** — Welcome email for new signups (~9 blocks)
3. **product_discovery** — Product category introduction (~10–12 blocks)
4. **trading_alert** — Price movement alerts (~8–10 blocks)
5. **feature_announcement** — New feature/product launch (~10–12 blocks)
6. **educational_how_to** — Step-by-step tutorial guides (~12–15 blocks)
7. **weekly_digest** — Weekly market summary (~12–15 blocks)
8. **promotional** — Campaign/offer emails (~8–10 blocks)

### Component 4: Chart Image Renderer

Programmatically generates five types of chart images as retina (2x) PNGs for embedding in emails:

1. **price_card** — Single instrument with price, change percentage, and sparkline chart
2. **listing_card** — Multiple instruments in a list format with filter pills
3. **comparison** — Side-by-side comparison of two instruments or periods
4. **portfolio** — Portfolio breakdown visualization
5. **metric** — Single large metric with context label

**Technical approach:** HTML/CSS templates rendered in headless Chromium via Playwright, screenshotted at 2x resolution.

### Component 5: HTML Email Renderer

Takes a filled recipe (blocks with generated content) and renders it into a single production-ready HTML email file. Must use table-based layout with inline CSS only (no modern CSS, no JavaScript). Must be responsive and mobile-first. Must include SEBI compliance footer and unsubscribe link. Must support all four sub-brand themes.

**Technical approach:** Jinja2 templates for each of the 18 block types, concatenated and wrapped in email boilerplate HTML.

### Component 6: MCP Server

Wraps all components into an MCP server callable from Claude Code.

**Tools exposed:**
- **list_recipes** — Browse available recipe templates
- **get_recipe** — Get a specific recipe's full structure
- **recommend_recipe** — Describe what you want, get ranked suggestions
- **list_blocks** — Browse all available block types
- **generate_email** — Main tool: takes recipe + brief, generates copy via LLM, renders charts, produces HTML
- **generate_chart** — Standalone chart image generation
- **export_email** — Final export: HTML file + image URLs ready for NARAD

## 4.5 Reference Example: The Crude Oil Email

The crude oil email (provided as a reference image) maps to the Content Engine as follows:

**Recipe:** commodity_educational

**Block decomposition:**
1. header_logo (Groww logo)
2. hero_text ("What moved Crude Oil prices this month?" with "Crude Oil" in green)
3. chart_image (price_card: Crude Oil Mini 20 Apr Fut, MCX, ₹9,044.00, +48.29%, 1M sparkline)
4. article_section ("One narrow passage, one big move" — Strait of Hormuz content)
5. divider
6. article_section ("So what moves Crude Oil prices?")
7. factor_card ("Geopolitics & supply routes")
8. factor_card ("OPEC+ output decisions")
9. factor_card ("Weekly inventory data")
10. article_section (italic note about News section)
11. article_section ("Two ways to trade Crude Oil")
12. chart_image (listing_card: All Commodities list with Crude Oil variants)
13. article_section (smaller position and lower risk)
14. cta_button ("EXPLORE NOW", green)
15. social_proof_banner
16. footer_standard (SEBI disclaimer, social links, unsubscribe)

---

# 5. Task 2: Compass MCP Skills

## 5.1 What Compass MCP Is (Corrected Understanding)

Compass is **NOT** just a text-to-SQL layer. It is a unified data platform interface that connects three systems via MCP:

- **DataHub** — Data catalog for discovering datasets, exploring schemas, managing metadata (descriptions, ownership, tags, glossary terms, domains), tracking data lineage (upstream/downstream with configurable depth), and browsing event instrumentation metadata (event definitions, triggers, GitHub source code links, tech owners)
- **Superset** — Visualization for creating charts (line, bar, pie, scatter, pivot, mixed timeseries — all configurable through conversation), building dashboards (auto-positioned grid layout), scheduling reports (PDF/PNG/CSV via email/Slack/webhook on cron), and setting data alerts (SQL-based threshold alerts)
- **Trino** — SQL engine for ad-hoc querying against the Iceberg data lake, with safety guardrails (read-only, partition enforcement, configurable timeouts up to 600s, row limits up to 5,000)

Compass has **80+ tools** organized across: Analytics Skills (16), SQL Execution (5), Charts (8), Dashboards (6), Data Discovery (6), Metadata (10), Lineage (1), Reports & Alerts (4), Datasets (4), Databases (3), Tags (4), Identity & Admin (4).

Users access Compass via Claude Code, Cursor, VS Code (Copilot or Kilo Code), IntelliJ (Cline), or any MCP-compatible client. It runs on Groww's internal VPN network with a local proxy on port 3100 (`http://127.0.0.1:3100/mcp`).

## 5.2 What Skills Are

Skills are pre-built knowledge packages that tell the AI assistant exactly which tables, columns, filters, and query patterns to use for each business domain. Instead of users needing to know the data schema, they just ask questions in plain English — the skill handles the rest.

Each skill encodes:

- **Which tables to use** — "For stock orders, use `core_stocks_order`, NOT the stale `stocks_order`"
- **Partition columns** — "Always filter on `order_date` — this table has 2.5B rows"
- **Column semantics** — "`product = 'CNC'` means delivery, `'MIS'` means intraday"
- **All filter values** — "Order status values: `EXECUTED`, `CANCELLED`, `REJECTED`, `FAILED`"
- **Optimized SQL patterns** — Pre-written templates for the 10 most common questions per domain
- **Data quality warnings** — "This table has future-dated rows — always cap your date range"
- **Stale table alerts** — "DO NOT USE `invest_user_fid_info` — stale since Jun 2024"
- **Join keys** — "Universal key: `user_account_id` (format `ACC<digits>`)"

**User experience comparison:**

Without skills: User opens SQL Lab, searches for the right table, guesses column names, writes SQL, debugs partition errors, interprets raw results.

With skills: User asks "What was the stock order success rate yesterday?" and gets "Yesterday's success rate was 87.3% across 14.2M orders" in seconds.

## 5.3 The 16 Existing Skills

The following 16 skills already exist. My work involves enhancing and expanding them, plus adding new skills for uncovered domains.

| # | Skill | Domain | Key Tables | Scale | Example Question |
|---|---|---|---|---|---|
| 1 | Stocks & Trading | Equities | `core_stocks_order`, `fact_transaction_stocks` | 2.5B, 7.1B rows | "CNC vs MIS turnover this week?" |
| 2 | F&O Analytics | Derivatives | `fno_order_master`, `fno_pnl_master` | 3.8B, 1.9B rows | "Options PnL distribution" |
| 3 | Mutual Funds | MF | `fact_mutual_fund_order_details_enriched`, `view_mf_user_aum_latest` | Millions | "SIP creation trends this month?" |
| 4 | User Master & Growth | Growth | `growth_user_master_ultimate`, `user_product_txn_master_fact` | 117M, 8.3B rows | "Signup → FID conversion by source" |
| 5 | User Events | Instrumentation | DataHub userEvent entities + `ems_raw` | 57B+ events | "What events fire on the home screen?" |
| 6 | Onboarding | Activation | `onb_usercheckpoints_master`, `fact_onboarding_invest` | 117M, 45M rows | "Where do users drop off in KYC?" |
| 7 | UPI Payments | Payments | `groww_upi_onb_user_fact`, `groww_upi_mandate_fact_table` | 66.9M, 27.2M rows | "UPI mandate lifecycle analysis" |
| 8 | MTF & Pledge | Margin | `aay_mtf_book_agg`, `fact_pledge_transaction` | 243, 50M rows | "MTF book size and market share" |
| 9 | Commodities | MCX | `commodity_day_view_seg`, `commodity_order_master` | 684K, 71.6M rows | "Gold vs Silver turnover" |
| 10 | 915 Trading | 915 App | `userday_details_915`, `mis_central_915_ab` | 1.9M, 17 rows | "915 DAU and MTU trends" |
| 11 | Help & Support | Support | `hns_helpdesktickets_master_trino`, `hns_chatbot_fact_table_v3` | 12M, 135K rows | "Chatbot self-service rate" |
| 12 | Engagement & Comms | Communications | `engagement_pn_narad_master`, `engagement_user_session_daily` | 250M/day, 8M/day | "Push notification CTR by campaign" |
| 13 | Context Push | Notifications | Context-aware push tables | Varies | "Contextual push delivery analysis" |
| 14 | User Financial Health | Wellness | Financial health tables | Varies | "User investment diversification" |
| 15 | Datahub Navigation | Meta | DataHub entities | N/A | "How do I find dataset lineage?" |
| 16 | MF Analytics Extended | MF Extended | Extended MF tables | Varies | "Scheme-level NAV performance" |

## 5.4 My Skill Work — What I'm Actually Doing

My work on skills falls into two categories:

### A. Enhancing Existing Skills

The 16 existing skills cover the major domains, but the Deep Dive document reveals they vary significantly in depth. Some skills (Stocks, F&O, Engagement) have detailed table inventories, key dimensions, example questions, and guardrails. Others (Context Push, User Financial Health, MF Extended) are listed but appear less developed.

Enhancement work includes:
- Adding more tables that weren't included in the original skill
- Writing more example queries (moving from "example questions it can answer" to actual tested SQL)
- Adding more data quality notes and guardrails based on real-world query failures
- Expanding column-level documentation (the Growth Analytics reference skill has detailed column tables — most skills in the Deep Dive have summary-level documentation)
- Adding cross-skill join patterns (how to combine data from multiple skills)
- Documenting new tables as they get created by data engineering

### B. Building New Skills

New skills for domains not yet covered by the 16. As these emerge, they follow the same format: Tables Overview, Key Dimensions, Example Questions, Key Guardrails, plus the detailed column-level documentation and tested SQL from the reference skill format.

## 5.5 Skill Content — What Each Existing Skill Already Covers

Based on the Deep Dive document, here is what's already documented per skill, which informs what enhancement work is needed:

### Stocks & Trading (Skill 1) — Well Documented
- 3 tables with row counts, schema, grain, partition columns
- Key dimensions table (product, direction, status, exchange, platform, order type, special flags with exact enum values)
- 8 example questions
- 4 guardrails (including stale table warning: `stocks_order` last updated Oct 2025, use `core_stocks_order`)

### F&O Analytics (Skill 2) — Well Documented
- 10 tables with row counts and purpose
- Key dimensions (contract type, call/put, moneyness, underlying, power trader, basket order, fast exit, API flag)
- 11 example questions
- 4 guardrails (critical: `fno_order_master.pt` contains future dates up to 2090 — must cap date range)

### Mutual Fund (Skill 3) — Well Documented
- 10+ tables with schema and purpose
- Table Selection Strategy matrix (analysis type → table to use)
- 8 example questions
- No explicit guardrails listed (enhancement opportunity)

### User Master & Growth (Skill 4) — Well Documented
- 10 tables with row counts and purpose
- Key dimensions with exact value distributions (Status: 72M SignUp, 27M Onboarded & Transacted; Source: 63M performance, 47M organic++; Platform: 98M android)
- 6 example questions
- 4 guardrails

### User Events (Skill 5) — Unique Two-Level Structure
- Event Metadata (DataHub entities with name, description, trigger, GitHub source URL, tech owner, screen names, category, app)
- Event Data (Trino with Pinot vs Iceberg selection guidance)
- Critical Pinot sampling warning (must use boosting_factor, last 1-2 days broken)
- App identification filters (6 app variants with exact filter SQL)
- 8 example questions

### Onboarding (Skill 6) — Well Documented
- 9 tables with row counts, schema, purpose
- Full onboarding funnel documented (Signup → Email → Mobile → PAN → Bank → Signature → eSign → KYC → Stock Onboarding → F&O → Commodity)
- 9 example questions
- 4 guardrails (critical: `esign_details` contains PII; stale table warnings for `invest_user_fid_info` and `daily_onb_conversions`)

### UPI Payments (Skill 7) — Documented
- 6 tables with status flags (FRESH vs STALE)
- UPI onboarding funnel
- Mandate status distribution with exact percentages
- 6 example questions
- 3 guardrails (timestamps are epoch ms; rolling window tables)

### Engagement & Communications (Skill 8) — Well Documented
- 9 tables with daily volumes per channel:
  - `engagement_pn_narad_master` — 170–250M rows/day (push notifications via NARAD)
  - `engagement_email_backend_master` — 3–6M rows/day (email)
  - `engagement_sms_backend_master` — 500–650K rows/day (SMS)
  - `engagement_whatsapp_backend_master` — 100–140K rows/day (WhatsApp)
  - `engagement_user_session_daily` — 8M rows/day (DAU aggregated)
  - `engagement_dau_indepth` — 8M rows/day (app engagement)
  - `engagement_session_indepth` — 65M rows/day, 57B total (per-session detail)
  - `engagement_app_fact` — weekly (install/reachability)
  - `engagement_dnd_fact` — 70K (DND state changes)
- Push notification funnel with typical day numbers (3M total → 2M sent → 440K received → 19K clicked, 4.3% CTR)
- PN status distribution (SENT 72%, OPTED_OUT 9%, VENDOR_FAILURE 9%, FREQUENCY_CAPPED 8.6%, EXPIRED 1.7%)
- 10 example questions
- 4 guardrails (critical: `engagement_session_indepth` 57B rows — single day queries ONLY; PN `os` column has mixed casing; DND `s7` is VARCHAR not boolean)

### MTF & Pledge (Skill 9) — Documented
- 9 tables with freshness status
- 8 example questions
- 3 guardrails (prefer fresh `aay_mtf_fid_agg` over stale `mtf_transactions_v2`)

### Commodities (Skill 10) — Documented
- 6 tables with row counts and purpose
- Top underlyings listed (GOLD, GOLDM, SILVER, SILVERM, CRUDEOIL, CRUDEOILM, etc.)
- 7 example questions
- 4 guardrails (prefer pre-aggregated `commodity_day_view_seg` over 71.6M-row `commodity_order_master`; timestamps are epoch ms; all trading on MCX)

### 915 Trading (Skill 11) — Documented
- 7 tables
- Pre-computed user segments (scalper, buyer/seller, profitability, platform, recency, frequency)
- 9 example questions
- 3 guardrails (data starts June/July 2025; cohort tables include broader F&O users; duplicates in `mis_central_915_ab`)

### Help & Support (Skill 12) — Documented
- 7 tables with freshness status (2 stale, 1 broken)
- Channel source codes mapping (1=Email, 2=Portal, 3=Chat, 7=Feedback Widget, 10=Outbound)
- 8 example questions
- 4 guardrails (critical: `hns_freshchat_tickets_master_trino` is BROKEN — do not use)

### Other Skills (13–16) — Summary Level Only
- Context Push, User Financial Health, Datahub Navigation, MF Extended are listed with minimal detail
- These represent the clearest enhancement opportunities

## 5.6 Known Stale Tables (Critical Reference)

From the Deep Dive's Cross-Skill Reference — tables that must never be used:

| Table | Last Data | Skill | Use Instead |
|---|---|---|---|
| `stocks_order` | Oct 2025 | Stocks | `core_stocks_order` |
| `invest_user_fid_info` | Jun 2024 | Onboarding | `fact_onboarding_invest` FID columns |
| `daily_onb_conversions` | EMPTY | Onboarding | `daily_onboarding_and_fid` |
| `fno_usersegmentation_monthly` | Oct 2025 | F&O | Use order tables for current data |
| `groww_upi_debit_fact_table` | Feb 2024 | UPI | Historical only |
| `hns_freshchat_tickets_master_trino` | BROKEN | H&S | Do not use |
| `hns_csat_dashboard` | Mar 2024 | H&S | Historical CSAT only |
| `mtf_transactions_v2` | Nov 2024 | MTF | `aay_mtf_fid_agg` |

---

# 6. Long-Term Vision

The ultimate goal (not immediate scope) is a more connected marketing pipeline. But importantly, this does NOT mean Compass and Content Engine directly integrate. The vision is:

1. **Compass (Analytics)** surfaces insights: "Commodity traders dropped 15% this month" — using Commodities skill + User Master skill
2. **Human decision** (analyst or CM): "We should send a re-engagement email to lapsed commodity traders"
3. **Audience Manager (Segmentation):** Creates segment "users who traded gold in last 30 days but not in 15" via BQ → AM → WebEngage → Segment ID
4. **Content Engine (Content):** CM uses the Content Engine to generate a commodity educational email about gold, with price charts and market analysis
5. **NARAD (Campaign):** CM pastes HTML into NARAD with the Segment ID, gets approval, sends
6. **Compass (Measurement):** Engagement & Comms skill tracks open rate, CTR, conversion for the campaign

Over time, the human decision step may become more automated (Compass alerts triggering content requests), but the systems remain parallel. Compass does not call Content Engine. Content Engine does not call Compass.

---

# 7. Technical Architecture

## 7.1 Content Engine Architecture

Python application structured as an MCP server with five internal components.

**Data flow:** Claude Code → MCP Server → Recipe Registry selects template → Copy Writer calls Anthropic API for text → Chart Renderer uses Playwright for PNGs → HTML Renderer assembles via Jinja2 → Final HTML + image URLs returned.

**Tech stack:** Python 3.11+, MCP SDK (Python), BeautifulSoup, Pydantic, Jinja2, Playwright, Anthropic SDK, Typer + Rich (CLI).

## 7.2 Compass MCP Architecture

Already exists. For reference:

```
AI Assistant (Claude Code / Cursor) → Compass MCP Server → DataHub (catalog)
                                                          → Superset (charts/dashboards)
                                                          → Trino (SQL engine)
```

Connection: Local proxy on `127.0.0.1:3100` → VPN → `prod-dp-ai-datahub-gms.growwinfra.in/mcp`

Setup: Run `compass-mcp-setup.sh` or configure manually. Requires Node.js 18+, VPN, Groww email/password.

Claude Code config: `claude mcp add compass --transport http http://127.0.0.1:3100/mcp`

## 7.3 How They Relate (Not Integrated)

```
┌────────────────────────┐     ┌──────────────────────────┐
│    Compass MCP         │     │   Content Engine MCP     │
│                        │     │                          │
│  "What happened?"      │     │  "What email to send?"   │
│                        │     │                          │
│  DataHub + Superset    │     │  Recipes + Blocks +      │
│  + Trino + 16 Skills   │     │  Charts + HTML Renderer  │
│                        │     │                          │
│  80+ tools             │     │  7 tools                 │
└───────────┬────────────┘     └──────────┬───────────────┘
            │                             │
            │     ┌───────────┐           │
            └────►│  Human    │◄──────────┘
                  │  (CM /    │
                  │  Analyst) │
                  └─────┬─────┘
                        │
                        ▼
              ┌───────────────┐
              │    NARAD      │
              │  (Campaign    │
              │  Delivery)    │
              └───────────────┘
```

---

# 8. My Specific Responsibilities

## What I Own

**Content Engine — full build (side project, 1–2 weeks):**
- Email analyzer pipeline (450 emails from WebEngage)
- Block library (18 blocks, 4 brand variants)
- Recipe system (8 recipes)
- Chart image renderer (5 chart types, Playwright)
- HTML email renderer (Jinja2, table-based, inline CSS)
- MCP server (7 tools)
- LLM prompts for copy generation
- Testing and demo

**Compass MCP Skills — enhance and expand (primary, ongoing):**
- Deepening existing skills: more tables, more tested SQL queries, more guardrails, column-level documentation
- Building new skills for uncovered domains
- Supporting toolkit (schema discovery, query tester, skill validator)
- Publishing to Datahub

## What I Don't Own

- Compass MCP core platform (existing, built by Saurabh Dubey / data platform team)
- NARAD (existing campaign delivery system)
- WebEngage (existing, I only export data from it)
- Audience Manager / segmentation (existing, separate system)
- Superset / Trino infrastructure (existing)
- SEBI compliance review of generated content
- Brand guidelines / design system creation

---

# 9. Scope Boundaries

## In Scope (Immediate)

- Content Engine MCP server (Python, Claude Code integration)
- Email analyzer, block library, recipe system, chart renderer, HTML renderer
- Enhancing existing Compass skills (more queries, tables, guardrails)
- Adding new Compass skills for uncovered domains
- Skill authoring toolkit

## In Scope (Future — Not Immediate)

- Chart images uploaded to Groww's cloud (needs access)
- Content Engine improvements based on CM feedback
- More skills as new data domains emerge

## Out of Scope

- Building or modifying Compass MCP core platform
- Building or modifying NARAD
- Building a web UI for the Content Engine
- Direct integration between Content Engine and Compass (they're parallel systems)
- Segmentation / Audience Manager
- Sending emails (NARAD's job)
- WBR/MBR report generation (Compass's scheduled reports feature handles this)
- SEBI compliance review process

---

# 10. Deliverables Checklist

## Task 1: Content Engine

- [ ] Email Analyzer Pipeline — 450 HTML emails analyzed, classified, decomposed
- [ ] Block Library — 18 block schemas with Pydantic models and brand variants
- [ ] Recipe System — 8 recipe templates as JSON
- [ ] Chart Image Renderer — 5 chart types generating retina PNGs
- [ ] HTML Email Renderer — Jinja2 templates, table-based, responsive, SEBI-compliant
- [ ] MCP Server — 7 tools, full orchestration
- [ ] Copy Generation — LLM prompts via Anthropic API
- [ ] CLI — analyze, generate-chart, render-email, serve-mcp commands
- [ ] End-to-end demo — 5 different email types generated and compared against real emails

## Task 2: Compass MCP Skills

### Enhancement of existing skills:
- [ ] Add column-level documentation (matching the Growth Analytics reference format) to skills that only have summary-level coverage
- [ ] Write and test additional SQL queries for each skill
- [ ] Add new tables discovered since initial skill creation
- [ ] Expand data quality notes based on real-world query failures
- [ ] Add cross-skill join patterns

### New skills:
- [ ] Build new skills for domains not covered by the 16 existing ones
- [ ] Each new skill follows the standardized format: Tables Overview, Key Dimensions, Example Questions, Key Guardrails, tested SQL

### Toolkit:
- [ ] Schema discovery script
- [ ] Query testing framework
- [ ] Skill completeness validator

---

# 11. Success Criteria

## Content Engine v1

- [ ] CM can describe an email in natural language → receives production-ready HTML
- [ ] System correctly recommends recipes based on descriptions
- [ ] Generated copy matches Groww's tone (educational, conversational, accessible to retail investors)
- [ ] No financial advice, no guaranteed returns, SEBI-compliant language
- [ ] Chart images match Groww's design language (verified by visual comparison)
- [ ] HTML renders correctly on Gmail for Android (primary user base)
- [ ] Table-based layout, inline CSS only
- [ ] Functioning unsubscribe link and SEBI disclaimer footer
- [ ] All four sub-brands render with correct theming
- [ ] MCP server starts cleanly, all 7 tools work
- [ ] End-to-end generation under 60 seconds
- [ ] Manager has seen a live demo

## Compass Skills — Per Skill Enhancement

- [ ] New tables added that weren't in the original skill
- [ ] All new SQL queries tested against real data via Compass
- [ ] Column-level documentation expanded (at minimum for primary tables)
- [ ] Additional guardrails documented based on real failures
- [ ] Compass generates correct SQL for at least 5 new natural language questions
- [ ] Published/updated on Datahub

## Compass Skills — New Skill

- [ ] All tables documented with schema, grain, freshness, purpose, row counts
- [ ] Key dimensions with exact enum values
- [ ] At least 10 example questions listed
- [ ] At least 5 key guardrails documented
- [ ] 10+ SQL queries tested against real data
- [ ] Stale table warnings included
- [ ] Published on Datahub

---

# 12. Dependencies & Risks

## Dependencies

| Dependency | Owner | Status | Impact if Blocked |
|---|---|---|---|
| WebEngage email export access | Marketing team | Unknown | Cannot build email analyzer |
| Groww cloud upload access | Infrastructure team | Unknown | Chart images can't be hosted |
| Compass MCP access (VPN + credentials) | Data platform team | Available (setup guide exists) | Cannot test skills |
| Brand guidelines for all 4 sub-brands | Design team | Unknown | Email rendering may not match brand |
| SEBI disclaimer templates | Compliance team | Unknown | Footer may not have correct regulatory text |
| Domain data owner availability | Various teams | Unknown | Skills can't be validated |
| Anthropic API key | Manager | Unknown | Copy generation won't work |

## Risks

**Risk 1: Email HTML parsing accuracy.** If emails were created by different teams with wildly different HTML patterns, block detection may have low accuracy. Mitigation: manual review of 20-30 emails, iterative refinement, accept ~80% for v1.

**Risk 2: Email rendering across clients.** Table-based email that looks perfect in Gmail may break elsewhere. Mitigation: prioritize Gmail Android for v1.

**Risk 3: LLM copy quality.** AI-generated copy may not match Groww's voice. Mitigation: detailed generation hints in recipes, real emails as few-shot examples, iterative prompt refinement.

**Risk 4: Skill accuracy.** As an intern, I may miss domain nuances. Mitigation: interview domain owners, get review before publishing.

**Risk 5: Data access.** Intern-level access may not cover all schemas. Mitigation: identify gaps early, request in batch.

**Risk 6: Skill enhancement scope creep.** With 16 existing skills, "enhance" is open-ended. Mitigation: prioritize based on skill completeness gaps (summary-only skills first), set clear per-skill goals.

---

# 13. Open Questions

## Access & Permissions

1. **WebEngage email export** — Do I have API access, or manual export? Who grants access?
2. **Groww cloud image hosting** — Can I get write access? URL pattern? Staging bucket?
3. **Compass MCP setup** — Have I run the setup guide? Do I have VPN access and credentials?
4. **Anthropic API key** — Existing key or need to set up?
5. **Table-level access** — Do I have read access to all schemas in `core_invest`, `core_bgv`, `dwh_invest`?

## Process & Prioritization

6. **Which existing skills to enhance first?** The summary-only skills (Context Push, User Financial Health, MF Extended) are clear enhancement targets. But should I prioritize skills my manager cares about most?
7. **What counts as "done" for skill enhancement?** Adding 5 more queries? Full column-level docs? Both?
8. **Content Engine demo timeline?** When does my manager want to see v1?
9. **Skill review process** — Who validates? Domain data owners, my manager, or the data platform team?

## Design & Compliance

10. **Brand guidelines** — Documented style guide per sub-brand?
11. **SEBI compliance text** — Specific templates for email footers?
12. **Email content restrictions** — Words/phrases that must never appear?

## Technical

13. **Engagement tables for Content Engine** — The Engagement skill shows `engagement_email_backend_master` has email data. Can I use this to enhance the email analyzer (map campaign performance back to email HTML templates)?
14. **Skill format** — Should enhanced skills match the Growth Analytics reference format (17-page detailed document with full column tables), or the Skills Deep Dive format (summary with key dimensions and guardrails)?
15. **New skill requests** — Is there a backlog of requested skills, or do I identify gaps myself?

---

*This document reflects the corrected understanding of both tasks, incorporating the Compass MCP BETA documentation, Analytics Skills Deep Dive, and Setup Guide. Compass and the Content Engine are parallel systems — Compass for analytics, Content Engine for email generation — with a human bridging the gap. The task description should be updated as open questions are resolved and scope evolves.*

# Groww — Detailed Project Plan

**Owner:** Intern, Growth Team  
**Manager-assigned tasks:** (1) Content Engine (side project), (2) Compass MCP Skills (primary)  
**Date:** April 2026  
**Last updated:** April 6, 2026

---

## Executive Summary

Two tasks have been assigned following a manager meeting:

**Task 1 — Content Engine:** Build an AI-powered HTML email generator for Groww's marketing team. The system takes a campaign manager's natural language prompt, selects or recommends an email recipe (a JSON template of reusable blocks), generates copy and chart images via LLM, and outputs production-ready HTML that gets pasted into NARAD for distribution. Delivered as an MCP server callable from Claude Code. Timeline: 1–2 weeks.

**Task 2 — Compass MCP Skills:** Write structured skill documents for every Growth domain so that Compass (Groww's central text-to-SQL MCP with access to 5000+ raw tables and 25000+ derived tables) can accurately query any part of Groww's data. Each skill teaches the LLM the schema, join keys, query patterns, and gotchas for a specific domain. This is ongoing primary work.

**Long-term convergence:** Once Compass has enough skills and the Content Engine is live, the full automated loop becomes possible — Compass auto-analyzes data → identifies user segments → Content Engine generates targeted email → NARAD sends → Engagement skill measures results → cycle repeats.

---

# TASK 2 — Compass MCP Skills (Primary Work)

## 2.1 What Compass MCP Is

Compass is Groww's company-wide MCP (Model Context Protocol) server. It sits on top of all of Groww's data infrastructure — 5000+ raw tables and 25000+ derived tables across every product domain. It provides a text-to-SQL interface that allows LLMs (via Claude Code or other tools) to query Groww's databases using natural language.

The problem: without domain-specific "skills," Compass doesn't know which tables to use, how to join them, what filters are mandatory, or what the common business questions are. It needs structured reference documents — skills — to query each domain accurately.

## 2.2 What a "Skill" Is (Anatomy)

Based on the Growth Analytics skill (the reference example), every skill document follows this exact structure. This is the template to follow for every new skill:

**Section 1 — Header Block**
- Domain name and description
- Catalog paths (e.g., `platform_iceberg.growth`, `platform_iceberg.ems`)
- Last validated date
- Owner

**Section 2 — Tables Overview**
A summary table listing every table in the domain with:
- Table name and schema
- Grain (what one row represents — e.g., "1 row per user per product")
- Freshness (near-realtime via Kafka, daily batch, ~4x/day, monthly)
- Purpose (one-line description)

This section is critical because it's what the LLM reads first to decide which table to use.

**Section 3 — Join Key Reference**
A table showing every identifier used across the domain:
- Column name, format, which tables it appears in
- Notes on when to use each key
- The "Critical rule" — e.g., "For any cross-table join, use `user_account_id`"

This prevents the LLM from doing incorrect joins (a common failure mode).

**Section 4 — Key Columns by Table**
For each table, a detailed column listing grouped by logical category:
- Column name, type, description
- Categories: identity & timing, user identity, device context, campaign attribution, geography, web-specific, pipeline metadata, domain-specific fields
- Nested/complex columns (like `environmentsettings.PRODUCTION.rules[].coverage`) documented with sub-field tables

This is the most time-consuming section but also the most important for query accuracy.

**Section 5 — Common Filter Values**
Enumerated values that the LLM needs to know about:
- Event names and their daily volumes (helps with query planning)
- Status codes, type enums, channel classifications
- Platform values (`android`, `ios`, `web`)
- Any boolean flags and what they mean

Without this, the LLM will guess at filter values and often get them wrong.

**Section 6 — Critical Query Best Practices**
Numbered rules that prevent expensive mistakes:
- Mandatory partition filters (e.g., "ALWAYS filter `ems_events` by `pt` — a single day is 100M+ rows")
- Type casting requirements (e.g., "AppsFlyer `event_time` is varchar, not timestamp")
- Deduplication rules (e.g., "Filter `WHERE rnm = 1` for `growth_signup_attributes`")
- Join patterns (e.g., "Use `customer_user_id = user_account_id` for AppsFlyer joins")
- Standard exclusions (e.g., "Always exclude `%@xyby.net%` and `credit_only_flag = 0`")

These rules prevent query failures, expensive full-table scans, and incorrect results.

**Section 7 — Example Queries (10–15)**
Real, tested SQL queries covering the most common business questions. Each query:
- Has a descriptive title
- Shows the full SQL with comments
- Uses CTEs for multi-step analysis
- Demonstrates correct joins, filters, and aggregations
- Is ordered from simple to complex

**Section 8 — Query Strategy Reference**
A lookup table mapping business questions to the right tables and filters:
- "How many new signups today by channel?" → `growth_user_master`, filter `DATE(signup_time) = CURRENT_DATE`, use `mapping0`
- Format: Question → Best Table(s) → Key Filter/Note

This is the LLM's decision tree for routing questions to the right data.

**Section 9 — Data Quality Notes**
Known issues, edge cases, and gotchas:
- NULLable columns and what NULL means in context
- Freshness gaps (e.g., "not real-time, updated ~4x/day")
- Superseded tables (e.g., "`features_v2` supersedes `features`")
- Parsing requirements (e.g., "`_dp_event_data` is unparsed JSON")
- Deduplication requirements

## 2.3 Skill Inventory — Full List with Prioritization

### Tier 1 — Build First (Foundation + Content Engine dependency)

These skills are either foundational (every other skill joins to them) or directly required for the Content Engine's future integration with Compass.

**Skill 1: Engagement & Communications Analytics**
- Why first: Directly powers the Content Engine — this is where campaign CTR, email open rates, conversion data, channel performance, and communication history live. You need this to analyze the 90-day email archive and to eventually measure Content Engine output effectiveness.
- Expected tables: WebEngage event tables, campaign tables, notification delivery tables, email/push/SMS performance tables
- Key questions it answers: "What's the open rate for commodity emails?", "Which email template converts best?", "What channels reach this segment?", "CTR by campaign type last month?"
- Discovery steps:
  - [ ] Get catalog paths for engagement/comms schemas
  - [ ] Identify all tables related to email, push, SMS, in-app notifications
  - [ ] Map WebEngage event data (`webengage_event_raw_v2` already in EMS) — understand what campaign events look like
  - [ ] Document delivery, open, click, conversion event types
  - [ ] Find how campaign performance is currently measured (existing dashboards/queries?)

**Skill 2: User Master & Growth Analytics**
- Why first: The `growth_user_master` table (117M rows) is the central dimension table. Every skill joins to it. If this skill is wrong, every other skill's joins break.
- Note: The Growth Analytics skill already covers this partially. Your job may be to expand it into a standalone skill with deeper documentation of `growth_user_master_ultimate` and all enrichment columns.
- Expected tables: `growth_user_master`, `growth_user_master_ultimate`, `growth_utm_mapping`, `growth_realtime_productwise_fids`, `growth_signup_attributes`
- Key questions: "How many NTU (new-to-Groww users) this week?", "TTU (total transacting users) by product?", "FID (first investment date) conversion by channel?", "User demographics breakdown?"
- Discovery steps:
  - [ ] Get full column list for `growth_user_master_ultimate` (the enriched version)
  - [ ] Document all derived flags: `affluent`, `power_trader`, `credit_only_flag`, etc.
  - [ ] Map TTU/NTU/FID definitions precisely — these are key business metrics
  - [ ] Document the ~4x/day refresh cadence and what "stale" means for recent signups

**Skill 3: User Events Analytics (v2)**
- Why first: EMS events are the behavioral backbone. Segmentation for email targeting depends on being able to query "users who did X in last N days." Without this skill, Compass can't build segments.
- Expected tables: `ems_events`, `ems_raw`, `growth_events`, `webengage_event_raw_v2`
- Key questions: "What events did user X fire today?", "How many users hit the PAN screen this week?", "Session count by platform?", "Pre-login to post-login funnel?"
- Discovery steps:
  - [ ] Get complete list of `event_name` values in `ems_events` (there could be hundreds)
  - [ ] Categorize events by funnel stage: acquisition, onboarding, activation, engagement, retention
  - [ ] Document `_dp_event_data` JSON payload structure for top 20 events
  - [ ] Map the pre-login identity resolution flow (LUID → user_account_id) in detail

### Tier 2 — Core Product Domains (High query volume)

These are the product verticals that generate the most business questions and email campaigns.

**Skill 4: Stocks & Trading Analytics**
- Covers: CNC (delivery) and MIS (intraday) stock trades
- Expected tables: Order tables, trade execution tables, position tables, holding tables
- Key questions: "Daily traded volume?", "Top traded stocks?", "User trading frequency distribution?", "Average order value by segment?"
- Discovery steps:
  - [ ] Identify all tables in the stocks/trading schema
  - [ ] Understand order lifecycle: placed → confirmed → executed → settled
  - [ ] Document order types, statuses, and edge cases (AMO, GTT, etc.)
  - [ ] Map to `growth_user_master` via `user_account_id`

**Skill 5: Mutual Fund Analytics**
- Covers: SIP, lumpsum, AMC sub-brand, fund categories
- Expected tables: SIP tables, MF order tables, fund master, AMC tables, folio tables
- Key questions: "Active SIP count?", "SIP cancellation rate?", "AUM by fund category?", "Top performing funds by inflows?"
- Discovery steps:
  - [ ] Identify MF schema and all tables
  - [ ] Understand SIP lifecycle: created → active → paused → cancelled → completed
  - [ ] Document fund categorization (equity, debt, hybrid, etc.)
  - [ ] Map AMC sub-brand specific tables

**Skill 6: F&O Analytics**
- Covers: Futures and Options trading
- Expected tables: F&O order tables, position tables, margin tables, P&L tables
- Key questions: "F&O active traders?", "Average margin utilization?", "Most traded contracts?", "P&L distribution?"
- Discovery steps:
  - [ ] Identify F&O schema
  - [ ] Understand contract types: futures vs options, calls vs puts
  - [ ] Document expiry-related logic (contracts expire, data has temporal sensitivity)
  - [ ] Map SEBI regulation-relevant fields (margin requirements, position limits)

**Skill 7: Commodities Trading Analytics**
- Covers: MCX contracts (crude oil, gold, silver, natural gas, etc.)
- Directly relevant to Content Engine since commodity emails are a key email type
- Expected tables: Commodity order tables, contract tables, price tables
- Key questions: "Commodity traders this month?", "Most traded commodity?", "Crude oil contract volume?", "New commodity traders (first FID)?"
- Discovery steps:
  - [ ] Identify commodities schema
  - [ ] Understand MCX contract structure (lot sizes, expiry dates, mini vs standard)
  - [ ] Document commodity-specific fields
  - [ ] Reference the crude oil email example — what data points would the Content Engine need?

### Tier 3 — Specialized Domains

**Skill 8: 915 Trading Analytics**
- Covers: 915 sub-brand (trading-focused sub-brand)
- Expected tables: 915-specific order/trade tables, 915 F&O tables
- Discovery: Understand what makes 915 different from main Groww trading

**Skill 9: Credit Analytics**
- Covers: Credit product line (loans, credit lines)
- Expected tables: Credit application, disbursement, repayment, eligibility tables
- Discovery: Understand credit funnel: eligibility → application → KYC → disbursement → repayment

**Skill 10: MTF & Pledge Analytics**
- Covers: Margin Trading Facility, pledge/unpledge flows
- Expected tables: MTF position tables, pledge tables, margin call tables
- Discovery: Understand MTF lifecycle and margin call triggers

**Skill 11: Onboarding Analytics**
- Covers: Signup → PAN → eSign → bank verification → first trade funnel
- Expected tables: Onboarding step tables, KYC tables, verification tables
- Note: Partially covered by User Events skill but needs its own dedicated view for funnel analysis
- Discovery: Map every onboarding step and its corresponding event/table

### Tier 4 — Supporting & Emerging

**Skill 12+:** UPI Payments & Transactions, User Financial Health, Help & Support (HnS), Context Push, and any new domains as they emerge. Build these as bandwidth allows or as business need arises.

## 2.4 Skill Development Workflow (Detailed)

For each skill, follow this process:

### Phase 1: Discovery (Day 1)

**Step 1.1 — Identify the schema and catalog**
- Find the catalog path (e.g., `platform_iceberg.commodities`)
- List all tables in the schema using Compass or direct DB access
- Note: some domains span multiple schemas — check for related tables in `core_bgv`, `ems`, and domain-specific schemas

**Step 1.2 — Gather table metadata**
- For each table: column names, data types, row count, partition columns, freshness
- Use `DESCRIBE TABLE` or equivalent metadata queries
- Note which tables are derived vs raw

**Step 1.3 — Interview the domain expert**
- Find the data owner or analyst for this domain (check Datahub for "Owners")
- Ask these specific questions:
  - "What are the top 10 business questions people ask about this domain?"
  - "What are the most common query mistakes people make?"
  - "Which tables are authoritative vs deprecated?"
  - "Are there any tables that look similar but serve different purposes?"
  - "What are the known data quality issues?"
  - "Are there any mandatory filters I should know about (like partition filters)?"

**Step 1.4 — Find existing queries**
- Check if there are existing dashboards (Superset, Metabase, Looker) for this domain
- Look at saved queries or notebooks from the team
- These become the basis for your Example Queries section

### Phase 2: Drafting (Day 2–3)

**Step 2.1 — Tables Overview**
- Write the summary table: name, schema, grain, freshness, purpose
- Order tables from most-used to least-used
- Be precise about grain — "1 row per user" vs "1 row per user per day" vs "1 row per event" matters

**Step 2.2 — Join Key Reference**
- Identify every identifier column across all tables
- Document which tables each key appears in
- Write the "Critical rule" for cross-table joins
- Note any identity resolution steps required (like LUID → user_account_id)

**Step 2.3 — Key Columns**
- For each table, group columns by logical category
- Write clear, unambiguous descriptions
- For complex/nested columns, add sub-field documentation
- Flag PII columns
- Note computed vs raw columns

**Step 2.4 — Common Filter Values**
- Query the actual data to get enum values: `SELECT DISTINCT column FROM table`
- For event tables, get top event names by volume
- Document boolean flags and what each value means
- Note platform values, status codes, type classifiers

**Step 2.5 — Query Best Practices**
- Identify partition columns → write "ALWAYS FILTER BY" rules
- Note type casting requirements (varchar dates, epoch timestamps, JSON strings)
- Document deduplication rules
- Note standard exclusions (test accounts, internal users)
- Document table preferences (e.g., "Use `gobbler_s2s_events` over device-side events")

### Phase 3: Example Queries & Testing (Day 3–4)

**Step 3.1 — Write 10–15 example queries**
- Cover the business questions from Step 1.3
- Start simple (single table, basic aggregation) and build to complex (multi-table joins, CTEs, funnels)
- Every query must demonstrate correct filter usage, join patterns, and best practices from Section 6
- Add comments explaining non-obvious logic

**Step 3.2 — Test every query**
- Run each query against real data via Compass
- Verify results make sense (sanity check row counts, values)
- Note any performance issues (long-running queries → add optimization notes)
- Fix any errors and update the skill document

**Step 3.3 — Write Query Strategy Reference**
- Create the lookup table: business question → table(s) → key filter
- Cover at least 15–20 common questions
- Include edge cases ("What if they ask about a specific user?", "What if the date range is very large?")

### Phase 4: Quality & Publication (Day 4–5)

**Step 4.1 — Self-review using checklist**
- [ ] Every table has schema, grain, freshness, purpose
- [ ] Every important column has type and description
- [ ] Join keys are explicit with cross-table mapping
- [ ] Partition columns have "ALWAYS FILTER" warnings
- [ ] At least 10 example queries, all tested and working
- [ ] Common filter values are enumerated
- [ ] Data quality notes cover known issues
- [ ] Query Strategy Reference has 15+ entries
- [ ] No references to deprecated tables without noting the replacement
- [ ] PII columns are flagged

**Step 4.2 — Domain owner review**
- Send to the data owner identified in Step 1.3
- Ask them to verify: table descriptions, column meanings, query correctness
- Incorporate feedback

**Step 4.3 — Publish to Datahub**
- Upload the skill document to Compass via Datahub
- Set status to "Published"
- Set yourself as owner
- Tag with relevant domain tags

**Step 4.4 — Validation test**
- After publishing, test the skill by asking Compass natural language questions about the domain
- Verify it generates correct SQL
- Note any failures → update the skill document

## 2.5 Timeline Estimate

| Skill | Estimated Days | Cumulative |
|---|---|---|
| Engagement & Communications | 4–5 (first skill, learning curve) | Week 1 |
| User Master & Growth | 3–4 | Week 2 |
| User Events Analytics v2 | 3–4 | Week 2–3 |
| Stocks & Trading | 2–3 | Week 3 |
| Mutual Fund | 2–3 | Week 3–4 |
| F&O | 2–3 | Week 4 |
| Commodities | 2–3 | Week 4–5 |
| 915, Credit, MTF, Onboarding | 2–3 each | Week 5–7 |
| Tier 4 skills | 2 each | Week 7+ |

Note: These estimates assume you've ramped up on the format after the first skill. The first skill always takes longest. Parallel work with the Content Engine (1–2 weeks) means the early skills may take slightly longer.

---

# TASK 1 — Content Engine (Side Project, 1–2 Weeks)

## 1.1 System Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                     Claude Code / LLM                          │
│              (CM interacts in natural language)                 │
│                                                                │
│  Example: "Generate a commodity email about crude oil          │
│           price movement this month for active traders"        │
└────────────────────────────┬───────────────────────────────────┘
                             │ MCP tool calls
                             ▼
┌────────────────────────────────────────────────────────────────┐
│                  Content Engine MCP Server                      │
│                                                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │
│  │ Recipe       │  │ Generation   │  │ Rendering            │ │
│  │ Management   │  │ Engine       │  │ Pipeline             │ │
│  │              │  │              │  │                      │ │
│  │ list_recipes │  │ generate_copy│  │ render_html          │ │
│  │ get_recipe   │  │ generate_    │  │ generate_chart_image │ │
│  │ recommend_   │  │   chart_data │  │ upload_image         │ │
│  │   recipe     │  │              │  │ export_final_html    │ │
│  │ create_recipe│  │              │  │                      │ │
│  └──────────────┘  └──────────────┘  └──────────────────────┘ │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Data Layer                             │  │
│  │  /recipes/*.json    /blocks/*.json    /output/*.html      │  │
│  │  /charts/*.png      /brands/*.json    /analysis/*.json    │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

## 1.2 Component Breakdown

### Component 1: Email Analyzer Pipeline

**Purpose:** Ingest ~450 HTML emails from WebEngage (90 days × ~150/month), parse their HTML structure, auto-classify them by type, and decompose them into blocks. This bootstraps the recipe/block library.

**Input:** Raw HTML email files exported from WebEngage

**Output per email:**
```json
{
  "source_file": "email_2026_01_15_commodity_crude.html",
  "classification": {
    "type": "educational",
    "sub_type": "commodity_explainer",
    "brand": "groww",
    "confidence": 0.92
  },
  "detected_blocks": [
    { "order": 1, "type": "header_logo", "brand": "groww" },
    { "order": 2, "type": "hero_text", "content": { "headline": "What moved Crude Oil prices this month?", "subtitle": "..." } },
    { "order": 3, "type": "chart_image", "chart_type": "price_card", "src": "..." }
  ],
  "metadata": {
    "subject": "What moved Crude Oil prices this month?",
    "word_count": 342,
    "image_count": 4,
    "cta_count": 1,
    "block_count": 17
  }
}
```

**Implementation approach:**
- Use an HTML parser (BeautifulSoup in Python or Cheerio in Node) to traverse the DOM
- Build heuristic classifiers for each block type based on HTML patterns:
  - `header_logo`: look for first `<img>` with brand logo URL pattern
  - `hero_text`: look for large font-size `<h1>`/`<h2>` near the top
  - `chart_image`: look for `<img>` tags with chart-related URL patterns or specific dimensions
  - `cta_button`: look for `<a>` tags styled as buttons (background-color, border-radius, centered)
  - `footer_standard`: look for SEBI disclaimer text, unsubscribe links, social media icons
  - And so on for all 18 block types
- Email type classification: use a combination of subject line keywords, block patterns, and content analysis
  - Educational: has `article_section` + `chart_image` + `factor_card` pattern
  - Onboarding: has `product_category_grid` + welcome keywords
  - Commodity: has `contract_row` or commodity keywords (crude, gold, silver, MCX)
  - Promotional: has strong CTA focus, fewer educational blocks

**Sub-tasks:**
- [ ] 1.1.1: Write WebEngage email export script (or manual export if API isn't available)
- [ ] 1.1.2: Build HTML parser that extracts DOM structure into a normalized tree
- [ ] 1.1.3: Build block detector — pattern matching for each of 18 block types
- [ ] 1.1.4: Build email type classifier (rule-based first, can add ML later)
- [ ] 1.1.5: Run analyzer on full 450-email corpus
- [ ] 1.1.6: Manual review of 20–30 analyzed emails to validate accuracy
- [ ] 1.1.7: Fix misclassifications and edge cases
- [ ] 1.1.8: Output final analysis JSON files

### Component 2: Block Library

**Purpose:** Define the 18 reusable email blocks as JSON schemas. Each block has a defined structure, required/optional fields, and brand-specific style variants.

**Block schema format (per block type):**
```json
{
  "block_name": "hero_text",
  "description": "Main headline and subtitle at the top of the email",
  "required_fields": {
    "headline": { "type": "string", "max_length": 80, "description": "Primary headline text" },
    "subtitle": { "type": "string", "max_length": 200, "description": "Supporting subtitle text" }
  },
  "optional_fields": {
    "highlight_word": { "type": "string", "description": "Word in headline to highlight in brand color" },
    "alignment": { "type": "enum", "values": ["center", "left"], "default": "center" }
  },
  "brand_variants": {
    "groww": { "headline_color": "#44475B", "highlight_color": "#00D09C", "font": "Inter" },
    "915": { "headline_color": "...", "highlight_color": "...", "font": "..." },
    "amc": { "headline_color": "...", "highlight_color": "...", "font": "..." },
    "w": { "headline_color": "...", "highlight_color": "...", "font": "..." }
  },
  "html_template": "<tr><td style='...'>{{headline}}</td></tr><tr><td style='...'>{{subtitle}}</td></tr>"
}
```

**Full block inventory (18 blocks):**

| # | Block | Required Fields | Notes |
|---|---|---|---|
| 1 | `header_logo` | `brand` | Auto-selects logo image URL based on brand |
| 2 | `hero_text` | `headline`, `subtitle` | Optional `highlight_word` for brand color emphasis |
| 3 | `hero_image` | `image_url`, `alt_text` | Full-width image, retina dimensions |
| 4 | `chart_image` | `chart_type`, `chart_data` | Triggers chart renderer (see Component 4) |
| 5 | `article_section` | `heading`, `paragraphs[]` | Main body content block |
| 6 | `factor_card` | `title`, `description` | Mint/teal tinted card for explainers |
| 7 | `cta_button` | `text`, `url` | Optional `color` override (green default, blue variant) |
| 8 | `social_proof_banner` | `text` | Trust strip ("10M+ users", ratings, etc.) |
| 9 | `footer_standard` | `brand` | Auto-includes SEBI disclaimer, social links, unsubscribe |
| 10 | `divider` | (none) | Simple horizontal line separator |
| 11 | `card_wrapper` | `children[]` | Container block that wraps other blocks in a card |
| 12 | `contract_row` | `commodity`, `exchange`, `expiry`, `price`, `change`, `change_pct` | Single commodity contract line |
| 13 | `filter_pills` | `pills[]` | Tab-style filter chips (e.g., "All", "Crude Oil", "Gold") |
| 14 | `listing_header` | `title` | Section header for lists |
| 15 | `step_instruction` | `step_number`, `title`, `description` | Numbered how-to step |
| 16 | `app_screenshot_pair` | `left_image_url`, `right_image_url` | Side-by-side app screenshots |
| 17 | `product_category_grid` | `categories[]` (name, icon, url) | 2×2 product category tiles |
| 18 | `social_showcase` | `metrics[]` (label, value) | Social proof with specific numbers |

**Sub-tasks:**
- [ ] 1.2.1: Create JSON schema for each of the 18 blocks
- [ ] 1.2.2: Extract brand color palettes and font specs for Groww, 915, AMC, W
- [ ] 1.2.3: Write HTML template snippet for each block (table-based email HTML for compatibility)
- [ ] 1.2.4: Validate schemas against the analyzed email corpus — can every real email be represented?
- [ ] 1.2.5: Store as `/blocks/*.json` in the MCP server's data directory

### Component 3: Recipe System

**Purpose:** Recipes are ordered combinations of blocks that define the structure of a specific email type. A recipe is what the CM selects (or the LLM recommends) as the starting point for generation.

**Recipe format:**
```json
{
  "recipe_id": "commodity_educational",
  "name": "Commodity Educational Email",
  "description": "Explains commodity price movements with charts and factor analysis",
  "brand": "groww",
  "meta": {
    "type": "educational",
    "subtype": "commodity_explainer",
    "subject_template": "What moved {{commodity}} prices this month?",
    "preheader_template": "{{commodity}} prices don't move without reason. Let's understand what's driving them."
  },
  "blocks": [
    { "block": "header_logo", "data": {} },
    { "block": "hero_text", "data": { "highlight_word": "{{commodity}}" } },
    { "block": "chart_image", "data": { "chart_type": "price_card" } },
    { "block": "article_section", "data": { "heading_template": "One narrow passage, one big move" } },
    { "block": "divider", "data": {} },
    { "block": "article_section", "data": { "heading_template": "So what moves {{commodity}} prices?" } },
    { "block": "factor_card", "data": {} },
    { "block": "factor_card", "data": {} },
    { "block": "factor_card", "data": {} },
    { "block": "article_section", "data": {} },
    { "block": "chart_image", "data": { "chart_type": "listing_card" } },
    { "block": "article_section", "data": {} },
    { "block": "cta_button", "data": { "text": "EXPLORE NOW" } },
    { "block": "social_proof_banner", "data": {} },
    { "block": "footer_standard", "data": {} }
  ],
  "generation_hints": {
    "tone": "educational, conversational, confident",
    "reading_level": "accessible to retail investors",
    "word_count_target": 300,
    "avoid": ["jargon", "financial advice", "guaranteed returns"]
  }
}
```

**Initial recipes to build (from the email analysis):**

| # | Recipe | Block Count | Use Case |
|---|---|---|---|
| 1 | `commodity_educational` | 15–17 | Commodity price movement explainer (like the crude oil example) |
| 2 | `onboarding_welcome` | 9 | Welcome email for new signups |
| 3 | `product_discovery` | 10–12 | Introduce users to product categories |
| 4 | `trading_alert` | 8–10 | Price movement alerts for stocks/F&O |
| 5 | `feature_announcement` | 10–12 | New feature or product launch |
| 6 | `educational_how_to` | 12–15 | Step-by-step guides (using step_instruction blocks) |
| 7 | `weekly_digest` | 12–15 | Weekly market summary |
| 8 | `promotional` | 8–10 | Campaign/offer emails |

**Sub-tasks:**
- [ ] 1.3.1: Cluster the analyzed emails to identify the most common patterns
- [ ] 1.3.2: Define 5–8 recipe templates based on clusters
- [ ] 1.3.3: Write `generation_hints` for each recipe (tone, constraints, SEBI compliance notes)
- [ ] 1.3.4: Validate each recipe against 3–5 real emails of that type
- [ ] 1.3.5: Store as `/recipes/*.json`

### Component 4: Chart Image Renderer

**Purpose:** Programmatically generate the 5 chart image types as retina PNGs for embedding in emails.

**Chart types and their specifications:**

**4a. `price_card`**
- Shows: single instrument price with sparkline chart
- Example: The crude oil price card showing ₹9,044.00, +2,985.00 (48.29%), 1M sparkline
- Input data: `{ instrument: "Crude Oil Mini 20 Apr Fut", exchange: "MCX", price: 9044, change: 2985, change_pct: 48.29, lot_size: 10, sparkline_data: [array of prices], period: "1M" }`
- Output: Retina PNG (~600×350px at 2x)
- Design: White card with shadow, instrument name, price in large font, change in green/red, sparkline chart, period selector

**4b. `listing_card`**
- Shows: multiple instruments in a list format
- Example: The "All Commodities" list showing Crude Oil 20 Apr, 18 May, Mini variants
- Input data: `{ title: "All Commodities", filters: ["All", "Crude Oil", "Gold", ...], items: [{ name, exchange, price, change, change_pct }] }`
- Output: Retina PNG (~600×500px at 2x)

**4c. `comparison`**
- Shows: side-by-side comparison of two instruments or periods
- Input data: Two sets of price/performance data
- Output: Retina PNG

**4d. `portfolio`**
- Shows: portfolio breakdown (pie chart or bar chart with asset allocation)
- Input data: `{ segments: [{ name, value, percentage }] }`
- Output: Retina PNG

**4e. `metric`**
- Shows: single large metric with context (e.g., "10M+ users", "₹1L Cr+ invested")
- Input data: `{ metric: "10M+", label: "happy investors", subtext: "..." }`
- Output: Retina PNG

**Implementation approach:**
- Option A (recommended): Build charts as HTML/CSS → render with Puppeteer → screenshot as PNG. This gives precise control over styling to match Groww's design language.
- Option B: Use Python (matplotlib + Pillow) for chart rendering. Less precise for matching the exact Groww UI look.
- Option C: Use a headless chart library like Chart.js rendered via Puppeteer.

**Sub-tasks:**
- [ ] 1.4.1: Set up Puppeteer (or chosen rendering approach)
- [ ] 1.4.2: Build `price_card` chart template (HTML/CSS) matching the crude oil example exactly
- [ ] 1.4.3: Build `listing_card` chart template
- [ ] 1.4.4: Build `comparison` chart template
- [ ] 1.4.5: Build `portfolio` chart template
- [ ] 1.4.6: Build `metric` chart template
- [ ] 1.4.7: Write rendering function: input JSON → output retina PNG
- [ ] 1.4.8: Test all 5 chart types against real email examples
- [ ] 1.4.9: Add image upload function (to Groww's internal cloud or staging bucket)

### Component 5: HTML Email Renderer

**Purpose:** Takes a filled recipe (blocks with content) and outputs production-ready HTML email that works across all email clients.

**Email HTML constraints (critical):**
- Must use table-based layout (not flexbox/grid — email clients don't support modern CSS)
- Inline CSS only (no `<style>` blocks — many email clients strip them)
- No JavaScript
- Images must use absolute URLs (hosted on Groww's cloud)
- Must be responsive (mobile-first, most Groww users are on Android)
- Must include SEBI compliance text in footer
- Must include unsubscribe link
- Max width: 600px (standard email width)

**Sub-tasks:**
- [ ] 1.5.1: Write email boilerplate HTML (DOCTYPE, meta tags, outer container)
- [ ] 1.5.2: Write HTML template for each of 18 block types (table-based, inline CSS)
- [ ] 1.5.3: Build the renderer: recipe JSON → iterate blocks → fill templates → concatenate → wrap in boilerplate
- [ ] 1.5.4: Add brand theming (swap colors/logo based on brand field)
- [ ] 1.5.5: Test output HTML in email clients (Gmail Android as primary)
- [ ] 1.5.6: Compare rendered output against original emails visually

### Component 6: MCP Server

**Purpose:** Wraps all components into an MCP server that Claude Code can call via tool use.

**Tools to expose:**

| Tool | Input | Output | Description |
|---|---|---|---|
| `list_recipes` | `{ brand?, type? }` | Array of recipe summaries | Browse available recipes with optional filters |
| `get_recipe` | `{ recipe_id }` | Full recipe JSON | Get a specific recipe's structure |
| `recommend_recipe` | `{ description }` | Ranked recipe suggestions | LLM-powered recommendation based on CM's description |
| `list_blocks` | `{ category? }` | Array of block definitions | Browse available block types |
| `generate_email` | `{ recipe_id, content_brief, brand, data? }` | Generated email JSON (filled blocks) | Main generation tool — fills recipe blocks with AI-generated content |
| `generate_chart` | `{ chart_type, chart_data }` | `{ image_url, dimensions }` | Renders a chart image and returns hosted URL |
| `preview_html` | `{ email_id }` | Raw HTML string | Renders the generated email as HTML |
| `export_email` | `{ email_id }` | `{ html_url, images: [] }` | Final export — HTML file + all image URLs ready for NARAD |

**MCP server file structure:**
```
content-engine/
├── src/
│   ├── server.ts              # MCP server setup and tool registration
│   ├── tools/
│   │   ├── recipes.ts         # list_recipes, get_recipe, recommend_recipe
│   │   ├── blocks.ts          # list_blocks
│   │   ├── generate.ts        # generate_email (orchestrates LLM + chart + render)
│   │   ├── charts.ts          # generate_chart
│   │   └── export.ts          # preview_html, export_email
│   ├── renderer/
│   │   ├── html.ts            # Recipe JSON → HTML conversion
│   │   └── templates/         # HTML templates per block type
│   ├── charts/
│   │   ├── renderer.ts        # Puppeteer-based chart renderer
│   │   └── templates/         # HTML templates for each chart type
│   └── analyzer/
│       ├── parser.ts          # HTML email parser
│       └── classifier.ts      # Email type classifier
├── data/
│   ├── blocks/                # Block JSON schemas
│   ├── recipes/               # Recipe JSON templates
│   ├── brands/                # Brand config (colors, logos, fonts)
│   └── analysis/              # Output from email analyzer
├── output/                    # Generated emails and chart images
├── package.json
└── README.md
```

**Sub-tasks:**
- [ ] 1.6.1: Initialize MCP server project with SDK
- [ ] 1.6.2: Register all tools with input/output schemas
- [ ] 1.6.3: Implement `list_recipes`, `get_recipe`, `list_blocks` (simple file reads)
- [ ] 1.6.4: Implement `recommend_recipe` (semantic matching against recipe descriptions)
- [ ] 1.6.5: Implement `generate_email` (the core orchestrator)
- [ ] 1.6.6: Implement `generate_chart` (calls chart renderer)
- [ ] 1.6.7: Implement `preview_html` and `export_email`
- [ ] 1.6.8: End-to-end test: natural language → MCP tools → final HTML
- [ ] 1.6.9: Write Claude Code configuration to connect to the MCP server

## 1.3 Sprint Plan — Day by Day

### Week 1: Foundation

**Day 1 — Setup + Email Export**
- [ ] Set up project repository
- [ ] Export emails from WebEngage (manual or API)
- [ ] Organize raw HTML files by date and type
- [ ] Set up development environment (Node.js + Puppeteer or Python)
- End of day: 450 HTML files organized and accessible

**Day 2 — Email Analyzer**
- [ ] Build HTML parser
- [ ] Build block detector for the 6 most common blocks (header_logo, hero_text, article_section, chart_image, cta_button, footer_standard)
- [ ] Build email type classifier (rule-based)
- [ ] Run analyzer on 50 emails, validate manually
- End of day: Analyzer working on 50 emails with ~80% block detection accuracy

**Day 3 — Analyzer Refinement + Block Library**
- [ ] Add remaining 12 block detectors
- [ ] Run analyzer on full 450-email corpus
- [ ] Fix misclassifications
- [ ] Start defining block JSON schemas based on analysis
- End of day: Full corpus analyzed, 10+ block schemas drafted

**Day 4 — Block Library + Recipe System**
- [ ] Complete all 18 block JSON schemas
- [ ] Extract brand configs (colors, logos, fonts) from real emails
- [ ] Cluster analyzed emails → define 5–8 recipe templates
- [ ] Write recipe JSON files with generation hints
- End of day: Complete block library + 5–8 recipes defined

**Day 5 — HTML Renderer**
- [ ] Write email boilerplate HTML
- [ ] Write HTML templates for each block type (table-based, inline CSS)
- [ ] Build the recipe → HTML renderer
- [ ] Test: render a recipe and compare against original email
- End of day: Can render a recipe into valid, decent-looking HTML

### Week 2: AI Layer + MCP

**Day 6 — Chart Renderer (Part 1)**
- [ ] Set up Puppeteer
- [ ] Build `price_card` chart template (matching the crude oil example)
- [ ] Build `listing_card` chart template
- [ ] Test rendering at retina resolution
- End of day: 2 chart types rendering clean PNGs

**Day 7 — Chart Renderer (Part 2)**
- [ ] Build `comparison`, `portfolio`, `metric` chart templates
- [ ] Write image upload function (staging bucket)
- [ ] Test all 5 chart types
- End of day: All 5 chart types working

**Day 8 — MCP Server Setup**
- [ ] Initialize MCP server with SDK
- [ ] Implement read-only tools: `list_recipes`, `get_recipe`, `list_blocks`
- [ ] Implement `generate_chart` tool
- [ ] Test from Claude Code: can it browse recipes and generate charts?
- End of day: MCP server running, basic tools working

**Day 9 — Generation Engine**
- [ ] Implement `generate_email` — the core tool that orchestrates everything
- [ ] Write the LLM prompt for content generation (tone, constraints, SEBI compliance)
- [ ] Implement `preview_html` and `export_email`
- [ ] End-to-end test: "Generate a commodity email about crude oil"
- End of day: Full pipeline working end-to-end

**Day 10 — Testing + Polish + Demo**
- [ ] Run 5 different email generation scenarios, compare against real emails
- [ ] Fix edge cases and rendering issues
- [ ] Write README with setup instructions
- [ ] Prepare demo for manager
- End of day: Demo-ready, documented

## 1.4 Key Technical Decisions

| Decision | Options | Recommendation | Notes |
|---|---|---|---|
| Language | Node.js (TypeScript) vs Python | Either works; pick your stronger language | Node has better MCP SDK support |
| Chart rendering | Puppeteer vs matplotlib vs Chart.js | Puppeteer (HTML → screenshot) | Best fidelity to Groww's design language |
| Image hosting | Groww cloud vs S3 staging vs local | Start local, move to staging bucket | Need to ask about cloud access |
| LLM for generation | Claude via MCP or direct API | Depends on your setup | The MCP server calls out to an LLM for copy generation |
| Email testing | Litmus vs Email on Acid vs manual | Manual first, formal testing later | Cross-client testing is nice-to-have for v1 |

---

# Cross-Task Dependencies & Parallel Work Schedule

```
Week 1:  [Content Engine: Foundation]  +  [Skill 1: Engagement & Comms - Discovery]
Week 2:  [Content Engine: AI + MCP]    +  [Skill 1: Engagement & Comms - Draft]
Week 3:  [Content Engine: Done ✓]      →  [Skill 2: User Master]  +  [Skill 1: Review]
Week 4:                                    [Skill 3: User Events]  +  [Skill 2: Review]
Week 5:                                    [Skill 4: Stocks]  +  [Skill 5: Mutual Fund]
Week 6:                                    [Skill 6: F&O]  +  [Skill 7: Commodities]
Week 7:                                    [Skill 8–11: 915, Credit, MTF, Onboarding]
Week 8+:                                   [Tier 4 skills]  +  [Content Engine v2]
```

The Content Engine's email analysis work (Component 1) directly feeds into the Engagement & Communications skill (Skill 1). The analyzed email patterns help you understand what campaign data looks like, which tables store email performance data, and what questions the Comms skill needs to answer. Running these in parallel during Week 1–2 is efficient.

---

# The Full Automated Loop (Long-Term Vision)

Once both tasks mature, they converge into a fully automated email campaign system:

```
Compass MCP (with all skills)
       │
       ├── 1. AUTO-ANALYTICS
       │      "Commodity traders down 15% this month"
       │      Uses: Commodities Trading skill + User Master skill
       │
       ├── 2. SEGMENTATION
       │      "Users who traded gold in last 30 days but not in 15"
       │      Uses: Audience Manager + Segment ID generation
       │      Powered by: User Events skill + User Master skill
       │
       ├── 3. CONTENT GENERATION
       │      Content Engine generates targeted email
       │      Recipe: commodity_educational
       │      Chart: gold price_card with live data from Compass
       │      Text: AI-generated copy about gold trends
       │
       ├── 4. CAMPAIGN ORCHESTRATION
       │      NARAD receives: Segment ID + HTML + image URLs
       │      Approval workflow → send to targeted users
       │
       └── 5. MEASUREMENT & FEEDBACK
              Engagement & Comms skill tracks:
              - Open rate, CTR, conversion for this campaign
              - Compares against historical benchmarks
              - Feeds insights back into next cycle
```

**Your skills work (Task 2) enables steps 1, 2, and 5.**
**Your Content Engine (Task 1) enables step 3.**
**NARAD (existing) handles step 4.**

---

# Open Questions for Manager

**Access & Permissions:**
1. Do you have API/export access to WebEngage to pull the 90-day email archive, or will this be manual?
2. Can you get write access to Groww's internal image cloud for chart uploads?
3. Do you currently have Compass query access to test skills against real data?
4. For each skill's tables, do you have read access, or do you need to request per schema?

**Process & Review:**
5. Who reviews and approves each skill before it goes live? Domain data owners? Your manager? Both?
6. Is the skill prioritization (Engagement first, then User Master, then User Events) aligned with your manager's expectations?
7. When does your manager want to see a first working Content Engine demo?

**Design & Compliance:**
8. Is there a design system or style guide for each sub-brand's email styling (Groww, 915, AMC, W)?
9. Are there specific SEBI disclaimer text templates that must be in every email footer?
10. Who owns compliance review for generated email content?

**Infrastructure:**
11. Are there existing Superset/Metabase dashboards for each domain that you can reference?
12. What's the preferred tech stack at Groww — Node.js or Python for internal tools?

---

# Success Criteria

**Content Engine v1 is done when:**
- [ ] A CM can describe an email in natural language via Claude Code
- [ ] The system recommends or accepts a recipe
- [ ] It generates copy that matches Groww's tone and style
- [ ] It renders chart images that match Groww's design language
- [ ] It outputs production HTML that can be pasted into NARAD
- [ ] The HTML renders correctly on Gmail (Android) — Groww's primary user base

**A Compass Skill is done when:**
- [ ] Published on Datahub with status "Published"
- [ ] All tables in the domain are documented with grain, freshness, and purpose
- [ ] All important columns have type and description
- [ ] Join keys and partition filters are explicitly documented
- [ ] 10+ example queries are tested and working against real data
- [ ] Query Strategy Reference covers 15+ common business questions
- [ ] Data quality notes cover all known gotchas
- [ ] Compass can answer the top 10 business questions for that domain via natural language
- [ ] Domain data owner has reviewed and approved

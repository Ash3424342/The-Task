# Task Description Document
## Content Engine & Compass MCP Skills — Groww, Growth Team

**Author:** Intern, Growth Team  
**Assigned by:** Manager  
**Date:** April 6, 2026  
**Status:** Active

---

# Table of Contents

1. Project Overview
2. Company & Domain Context
3. Task 1: Content Engine (Side Project)
4. Task 2: Compass MCP Skills (Primary Work)
5. Long-Term Vision: The Full Automated Loop
6. Technical Architecture
7. My Specific Responsibilities
8. Scope Boundaries
9. Deliverables Checklist
10. Success Criteria
11. Dependencies & Risks
12. Open Questions

---

# 1. Project Overview

## What I'm Building

I have been assigned two tasks that together form the foundation of Groww's automated email marketing intelligence system:

**Task 1 — Content Engine (side project, 1–2 weeks):** An AI-powered HTML email generator delivered as an MCP server. Campaign managers describe what email they want in natural language via Claude Code. The system selects an email template ("recipe"), generates the copy and chart images using AI, and outputs production-ready HTML that gets pasted into NARAD (Groww's campaign distribution platform) for sending.

**Task 2 — Compass MCP Skills (primary ongoing work):** Structured skill documents for Groww's central text-to-SQL MCP called Compass. Compass sits on top of all of Groww's data infrastructure (5,000+ raw tables, 25,000+ derived tables) and lets LLMs query any database via natural language. Without domain-specific "skills," Compass doesn't know which tables to use, how to join them, or what filters are mandatory. I am writing skills for every domain under Growth — over 10 skill documents covering stocks, mutual funds, F&O, commodities, user events, engagement, onboarding, credit, and more.

## Why This Matters

Today, creating a single marketing email at Groww involves a multi-team, multi-day process: the creative team designs in Figma, a UX/engineering team or third party converts the design to HTML, and the output is manually pasted into NARAD. This is slow, expensive, and doesn't scale.

The Content Engine replaces this with a prompt-driven workflow where a campaign manager can generate a production-ready email in minutes, not days.

The Compass MCP skills unlock a different but equally important capability: the ability for anyone at Groww to query the company's entire data infrastructure using natural language. Once enough skills are written, Compass can auto-generate insights ("commodity traders are down 15% this month"), identify user segments ("users who traded gold in the last 30 days but stopped"), and feed that intelligence directly into the Content Engine to generate targeted emails — creating a fully automated marketing loop.

## How the Two Tasks Connect

These tasks are independent in the short term but converge in the long term. The Content Engine generates emails. The Compass MCP skills enable the data intelligence that decides what emails to send, to whom, and when. Together, they form the backbone of an automated, data-driven email marketing system.

---

# 2. Company & Domain Context

## About Groww

Groww is one of India's largest retail investment platforms. Users can trade stocks, mutual funds, futures & options (F&O), and commodities (via MCX). The platform is regulated by SEBI (Securities and Exchange Board of India), which means all user-facing communications must comply with financial advertising regulations — no investment advice, no guaranteed returns, mandatory disclaimers.

Groww operates multiple sub-brands, each with its own visual identity for emails:

- **Groww** — the main brand (green accent, #00D09C)
- **915** — trading-focused sub-brand
- **AMC** — asset management sub-brand
- **W** — additional sub-brand

The platform serves approximately 117 million registered users, with a large base on Android mobile.

## Key Internal Systems

**NARAD:** Groww's campaign distribution platform. Emails are sent through NARAD by pasting HTML into its interface. NARAD handles delivery, targeting (via segment IDs), and approval workflows. It is not being built or modified — it is an existing system that the Content Engine outputs feed into.

**WebEngage:** The marketing engagement platform currently used to manage and send campaigns. Historical emails (approximately 150 per month across standard, ad-hoc, and push notification types) are stored here. These emails are the training data for the Content Engine's email analyzer.

**Compass MCP:** Groww's company-wide Model Context Protocol server. It provides a text-to-SQL interface over Groww's full data infrastructure, hosted on Iceberg tables accessible via Trino. Compass is an existing system that I am enhancing by writing domain-specific skill documents. It is connected to Claude Code and other LLMs for direct querying.

**Datahub:** Groww's internal metadata catalog (at prod-dp-ai-datahub.growwinfra.in). This is where Compass skills are published, and where table metadata, ownership, and documentation are stored. Existing skills (like the Growth Analytics skill) are published here.

**Groww Internal Cloud:** Image hosting for email assets. Chart images and other email graphics are uploaded here, and the resulting URLs are embedded in email HTML. This avoids external CDN dependencies for a large audience.

---

# 3. Task 1: Content Engine

## 3.1 What It Is

The Content Engine is an MCP server (written in Python) that exposes tools for AI-powered email generation. It is callable from Claude Code, meaning a campaign manager (or I, during development) interacts with it through natural language conversation. The system handles the full pipeline: template selection, copy generation, chart image rendering, and HTML email assembly.

## 3.2 The Current Workflow (What Exists Today)

The current email creation process is manual and multi-step:

1. A campaign manager identifies the need for an email (e.g., "we need a commodity education email about crude oil")
2. The creative team designs the email layout in Figma
3. A UX/engineering team or a third-party vendor converts the Figma design into HTML
4. Images (charts, screenshots, etc.) are manually created, uploaded to Groww's internal cloud, and URLs are embedded in the HTML
5. The final HTML is pasted into NARAD for distribution
6. NARAD sends the email to the targeted user segment

This process takes days and involves multiple teams. There is no standardized template system — each email is designed from scratch or loosely copied from previous ones.

## 3.3 The New Workflow (What I'm Building)

The Content Engine replaces steps 1–4 with an AI-driven flow:

1. Campaign manager opens Claude Code (connected to the Content Engine MCP server)
2. CM describes what they want: "Generate a commodity educational email about crude oil price movement this month for active traders"
3. The system recommends a recipe (or the CM picks one manually)
4. The system generates: email copy (headlines, body text, factor cards), chart images (price cards, listings), and the full email subject line and preheader
5. The system renders everything into a single production-ready HTML file with all image URLs embedded
6. CM pastes the HTML into NARAD and sends

## 3.4 Components

The Content Engine consists of six components that I am building from scratch:

### Component 1: Email Analyzer Pipeline

**What it does:** Ingests approximately 450 raw HTML emails exported from WebEngage (90 days of history at ~150 emails/month), parses their HTML structure, auto-classifies each email by type, and decomposes them into reusable blocks.

**Why it's needed:** There is no existing standardized template system. The analyzer reverse-engineers the patterns from real emails to bootstrap the block library and recipe system. It also trains the system on Groww's email design language — what blocks appear in what order, what copy patterns are used, what chart types are common.

**Input:** Raw HTML email files from WebEngage.

**Output per email:** A structured JSON containing the email's detected classification (educational, onboarding, promotional, commodity update, trading alert, feature announcement, weekly digest), a list of detected blocks in order, and metadata (subject line, word count, image count, CTA count).

**Technical approach:** HTML parsing via BeautifulSoup, heuristic pattern matching for block detection (e.g., header_logo detected by finding the first image with "logo" in the URL or alt text near the top of the email; cta_button detected by finding anchor tags with background-color, border-radius, and padding styles), and rule-based classification combining subject line keywords, block patterns, and content analysis.

### Component 2: Block Library

**What it does:** Defines 18 reusable email building blocks as formal JSON schemas. Each block has a defined structure (required fields, optional fields), brand-specific style variants, and an associated HTML template.

**Why it's needed:** Blocks are the atomic units of the email system. Every email is a sequence of blocks. Standardizing them as schemas ensures consistency across all generated emails and makes it possible for the LLM to fill them with generated content.

**The 18 blocks:**

1. **header_logo** — Brand banner with the appropriate logo (Groww, 915, AMC, or W)
2. **hero_text** — Main headline and subtitle at the top of the email, with optional accent-colored highlight word
3. **hero_image** — Full-width image (not a chart)
4. **chart_image** — Programmatically generated chart PNG (triggers the chart renderer)
5. **article_section** — Heading followed by one or more paragraphs of body copy
6. **factor_card** — Mint/teal-tinted explainer card with a bold title and description (used for explaining market factors like "OPEC+ output decisions")
7. **cta_button** — Call-to-action button (green or blue, linked to a Groww app deeplink)
8. **social_proof_banner** — Trust/credibility strip ("10M+ happy investors", app store ratings)
9. **footer_standard** — Standard footer with SEBI registration, social media links, and unsubscribe link (mandatory for regulatory compliance)
10. **divider** — Horizontal line separator between sections
11. **card_wrapper** — Container that wraps other blocks in a card-style visual group
12. **contract_row** — Single commodity contract line showing instrument name, exchange, price, change, and lot size
13. **filter_pills** — Tab-style filter chips (e.g., "All", "Crude Oil", "Gold", "Natural Gas", "Silver")
14. **listing_header** — Section header for list sections
15. **step_instruction** — Numbered how-to step with title and description
16. **app_screenshot_pair** — Side-by-side app screenshots
17. **product_category_grid** — 2×2 grid of product category tiles (Stocks, Mutual Funds, F&O, Gold)
18. **social_showcase** — Social proof with specific metrics

Each block schema includes brand variants for all four sub-brands with their respective colors, fonts, and logo URLs.

### Component 3: Recipe System

**What it does:** Recipes are pre-defined ordered sequences of blocks that represent a specific email type. They include metadata (subject line template, preheader template) and generation hints (tone, reading level, word count target, things to avoid).

**Why it's needed:** Recipes provide the structural blueprint for email generation. When a CM says "make a commodity email," the system uses the commodity_educational recipe to know that the email should start with a header_logo, then hero_text, then a price_card chart, then article sections with factor cards, then a listing_card chart, then a CTA, then social proof, then footer.

**Initial recipes (8):**

1. **commodity_educational** — Explains commodity price movements with charts and factor analysis (like the crude oil email example). ~15–17 blocks.
2. **onboarding_welcome** — Welcome email for new signups with product category grid. ~9 blocks.
3. **product_discovery** — Introduces users to product categories they haven't explored. ~10–12 blocks.
4. **trading_alert** — Price movement alerts for stocks or F&O contracts. ~8–10 blocks.
5. **feature_announcement** — New feature or product launch announcements. ~10–12 blocks.
6. **educational_how_to** — Step-by-step tutorial guides using step_instruction blocks. ~12–15 blocks.
7. **weekly_digest** — Weekly market summary with multiple charts and sections. ~12–15 blocks.
8. **promotional** — Campaign or offer-focused emails with strong CTAs. ~8–10 blocks.

Recipes can be provided by the CM or recommended by the LLM based on a natural language description.

### Component 4: Chart Image Renderer

**What it does:** Programmatically generates five types of chart images as retina (2x) PNGs for embedding in emails. These replace the current process of manually creating chart screenshots in Figma or taking app screenshots.

**Why it's needed:** The crude oil email example contains two chart images: a price card (showing ₹9,044.00 with a sparkline) and a listing card (showing multiple commodity contracts). These are currently created manually. The chart renderer automates this.

**The 5 chart types:**

1. **price_card** — Single instrument with price, change percentage, and sparkline chart. Matches the Groww app's price card design (white card, shadow, MCX label, ₹ price, green/red change, period selector).
2. **listing_card** — Multiple instruments in a list format with filter pills at the top. Matches the Groww app's commodity listing view.
3. **comparison** — Side-by-side comparison of two instruments or time periods.
4. **portfolio** — Portfolio breakdown visualization (pie or bar chart).
5. **metric** — Single large metric with context label (e.g., "10M+ happy investors").

**Technical approach:** Each chart type has an HTML/CSS template that replicates the Groww app's visual design. The template is populated with data, rendered in a headless Chromium browser (via Playwright), and screenshotted at 2x resolution to produce a retina PNG. This approach gives pixel-perfect control over styling to match Groww's design language.

**Image hosting:** Generated PNGs need to be uploaded to Groww's internal cloud to get hosted URLs. These URLs are then embedded in the email HTML. For the initial version, local file paths or a staging bucket may be used until I get cloud upload access.

### Component 5: HTML Email Renderer

**What it does:** Takes a filled recipe (blocks with generated content) and renders it into a single production-ready HTML email file.

**Why it's needed:** Email HTML is fundamentally different from web HTML. Email clients (especially Outlook, Gmail on mobile, and older Android email apps) do not support modern CSS. The renderer must produce table-based layouts with inline CSS — no flexbox, no grid, no external stylesheets, no JavaScript.

**Email HTML constraints:**

- Table-based layout only (nested tables for structure)
- All CSS must be inline (no style blocks — many clients strip them)
- No JavaScript
- Images use absolute URLs (hosted on Groww's cloud)
- Must be responsive and mobile-first (most Groww users are on Android)
- Maximum width: 600px (email standard)
- Must include SEBI compliance disclaimer in footer
- Must include unsubscribe link
- Must support all four sub-brand themes (Groww, 915, AMC, W)

**Technical approach:** Jinja2 templates for each of the 18 block types. The renderer iterates through the recipe's block list, fills each block's template with content data, concatenates the rendered blocks, and wraps everything in email boilerplate HTML (DOCTYPE, meta tags, outer table container, preheader text).

### Component 6: MCP Server

**What it does:** Wraps all the above components into an MCP (Model Context Protocol) server that Claude Code can interact with via tool use.

**Why it's needed:** The MCP server is the interface. Without it, all the other components are just libraries. The MCP server makes them accessible as natural language tools: "list available recipes," "generate a commodity email about gold," "render a price card chart for crude oil."

**Tools exposed:**

- **list_recipes** — Browse available recipe templates with optional brand/type filters
- **get_recipe** — Get full details of a specific recipe including block structure
- **recommend_recipe** — Describe what you want, get ranked recipe suggestions
- **list_blocks** — Browse all available block types and their schemas
- **generate_email** — The main tool. Takes a recipe ID and content brief, generates copy via LLM, renders chart images, and produces final HTML
- **generate_chart** — Standalone chart image generation from structured data
- **export_email** — Export a generated email as final HTML ready for NARAD

**Tech stack:** Python, using the official MCP SDK. The server runs as a stdio process launched by Claude Code.

## 3.5 Reference Example: The Crude Oil Email

To ground all of the above in a concrete example, here is how the crude oil email (provided as a reference image) maps to the Content Engine's architecture:

**Recipe used:** commodity_educational

**Block decomposition of the actual email:**
1. header_logo (Groww logo)
2. hero_text (headline: "What moved Crude Oil prices this month?", subtitle: "Commodities prices don't move without reason. Let's understand what's driving them.", highlight_word: "Crude Oil")
3. chart_image (type: price_card, data: Crude Oil Mini 20 Apr Fut, MCX, ₹9,044.00, +2,985.00, +48.29%, 1M sparkline)
4. article_section (heading: "One narrow passage, one big move", paragraphs about Strait of Hormuz and oil supply)
5. divider
6. article_section (heading: "So what moves Crude Oil prices?")
7. factor_card (title: "Geopolitics & supply routes", description about conflicts near oil regions)
8. factor_card (title: "OPEC+ output decisions", description about production controls)
9. factor_card (title: "Weekly inventory data", description about US inventory releases)
10. article_section (italic note about tracking in the News section)
11. article_section (heading: "Two ways to trade Crude Oil", body about MCX contract types)
12. chart_image (type: listing_card, data: All Commodities list showing Crude Oil 20 Apr, 18 May, Mini 20 Apr, Mini 18 May with prices)
13. article_section (body about smaller position and lower risk)
14. cta_button (text: "EXPLORE NOW", color: green, url: groww.in/commodities)
15. social_proof_banner
16. footer_standard (SEBI disclaimer, social links, unsubscribe)

This is exactly what the Content Engine would produce when a CM says: "Generate a commodity educational email about crude oil price movement this month."

---

# 4. Task 2: Compass MCP Skills

## 4.1 What Compass MCP Is

Compass is Groww's company-wide MCP (Model Context Protocol) server that provides a text-to-SQL interface over the entire data infrastructure. It is connected to Claude Code and other LLMs, allowing anyone to query Groww's databases by typing natural language questions like "How many new users signed up this week from organic channels?" Compass translates this into SQL, executes it against the Iceberg/Trino data layer, and returns results.

The data infrastructure is massive: 5,000+ raw tables and 25,000+ derived tables across every product and operational domain.

## 4.2 What a Skill Is

A skill is a structured reference document that teaches Compass how to query a specific domain accurately. Without skills, Compass doesn't know which of the 30,000 tables to use, how to join them, what filters are mandatory, or what common business questions look like as SQL.

A reference example exists: the Growth Analytics skill, which I have studied in detail. It covers user acquisition, attribution, referral programs, feature flags, and behavioral events. It is a 17-page document published on Datahub.

Every skill document follows a standardized structure with 9 sections:

1. **Header** — Domain name, catalog paths, last validated date, owner
2. **Tables Overview** — Every table in the domain listed with its schema, grain (what one row represents), freshness (how often it's updated), and purpose (one-line description)
3. **Join Key Reference** — Every identifier column across the domain's tables, showing which tables it appears in and how to join them. Includes a "critical rule" stating the primary join strategy (typically: "use user_account_id for cross-table joins")
4. **Key Columns by Table** — Detailed column-level documentation for every table, grouped by category (identity, device context, campaign attribution, etc.), with column name, data type, and description
5. **Common Filter Values** — Enumerated values the LLM needs to know: event names with daily volumes, status codes, type enums, platform values, boolean flag meanings
6. **Critical Query Best Practices** — Numbered rules that prevent expensive mistakes: mandatory partition filters, type casting requirements, deduplication rules, standard exclusions, table preferences
7. **Example Queries** — 10–15 real, tested SQL queries covering common business questions, ordered from simple to complex, with comments explaining non-obvious logic
8. **Query Strategy Reference** — A lookup table mapping business questions to the right tables and filters (e.g., "How many signups today?" → growth_user_master, filter by signup_time, use mapping0 for channel)
9. **Data Quality Notes** — Known issues and edge cases: NULLable columns, freshness gaps, superseded tables, parsing requirements, deduplication needs

## 4.3 Skills I Need to Build

I am responsible for writing skills for every domain under Growth. The full inventory, prioritized by importance:

### Tier 1 — Foundation (Build First)

These are either foundational tables that every other skill joins to, or directly required for the Content Engine's future Compass integration.

**Skill 1: Engagement & Communications Analytics**
Covers: Campaign CTR, email open rates, conversion data, channel performance, WebEngage events, notification delivery, push/SMS/email metrics. This skill directly powers the Content Engine — it's how we measure whether generated emails are performing well.

**Skill 2: User Master & Growth Analytics**
Covers: The central user dimension table (growth_user_master, 117M rows) and its enriched variant (growth_user_master_ultimate). User demographics, KYC status, signup attribution, product-wise first investment dates, affluent/power trader flags. Every other skill joins to this table — if it's wrong, everything else breaks. Key business metrics: TTU (total transacting users), NTU (new-to-Groww users), FID (first investment date).

**Skill 3: User Events Analytics v2**
Covers: Behavioral event streams (ems_events, growth_events). All user actions on the app and web — screen views, clicks, transactions, onboarding steps. This is the backbone for behavioral segmentation: "users who viewed the commodities page but didn't trade." Critical gotchas include mandatory partition filters (a single day of EMS data is 100M+ rows) and pre-login identity resolution via LUID mapping.

### Tier 2 — Core Product Domains

**Skill 4: Stocks & Trading Analytics** — CNC (delivery) and MIS (intraday) stock trades, order lifecycle, traded volume, unique traders.

**Skill 5: Mutual Fund Analytics** — SIP lifecycle, lumpsum orders, fund categorization (equity/debt/hybrid), AUM calculations, AMC sub-brand specific tables.

**Skill 6: F&O Analytics** — Futures and options trading, contract structure, position lifecycle, margin utilization, P&L distributions.

**Skill 7: Commodities Trading Analytics** — MCX contracts (crude oil, gold, silver, natural gas), lot sizes, mini vs standard contracts, commodity-specific metrics. Directly relevant to the Content Engine since commodity emails are a key email type.

### Tier 3 — Specialized Domains

**Skill 8: 915 Trading Analytics** — 915 sub-brand specific trading data.

**Skill 9: Credit Analytics** — Credit product line: eligibility, application, KYC, disbursement, repayment.

**Skill 10: MTF & Pledge Analytics** — Margin Trading Facility positions, pledge/unpledge flows, margin calls.

**Skill 11: Onboarding Analytics** — Full onboarding funnel: signup → PAN → eSign → bank verification → first trade. Partially covered by User Events but needs its own dedicated funnel view.

### Tier 4 — Supporting & Emerging

**Skills 12+:** UPI Payments & Transactions, User Financial Health, Help & Support, Context Push, and any new domains as they emerge. Built as bandwidth allows or business need arises.

## 4.4 How I Write Each Skill

Each skill follows a 4-phase development process:

**Phase 1: Discovery (Day 1)** — Identify the schema and catalog paths. List all tables using SHOW TABLES or Datahub. Gather table metadata (columns, types, row counts, partition columns, freshness). Interview the domain's data owner or analyst to learn the top 10 business questions, common query mistakes, authoritative vs deprecated tables, and known data quality issues. Find existing dashboards (Superset, Metabase) and saved queries as references.

**Phase 2: Drafting (Day 2–3)** — Write the Tables Overview (name, schema, grain, freshness, purpose). Document Join Key Reference with cross-table mapping. Write Key Columns for each table grouped by category. Enumerate Common Filter Values by running SELECT DISTINCT queries. Write Critical Query Best Practices (partition filters, type casting, deduplication, exclusions).

**Phase 3: Example Queries & Testing (Day 3–4)** — Write 10–15 example queries covering the business questions from Phase 1. Test every query against real data via Compass. Verify results for sanity. Write the Query Strategy Reference lookup table (15+ entries). Document Data Quality Notes.

**Phase 4: Review & Publication (Day 4–5)** — Self-review against completeness checklist. Get domain data owner to validate. Publish to Datahub. Test the skill by asking Compass natural language questions and verifying it generates correct SQL.

**Target pace:** The first skill will take 4–5 days (learning curve). Subsequent skills should take 2–3 days each once I'm in rhythm.

---

# 5. Long-Term Vision: The Full Automated Loop

The ultimate goal — not in immediate scope but what everything is building toward — is a fully automated email campaign system. Once Compass has enough skills and the Content Engine is integrated, the flow becomes:

**Step 1 — Auto-Analytics (Compass):** Compass automatically detects patterns and anomalies. Example: "Commodity traders are down 15% month-over-month." This uses the Commodities Trading skill and User Master skill.

**Step 2 — Segmentation (Compass + Audience Manager):** Compass generates user segments via text-to-SQL. Example: "Users who traded gold in the last 30 days but haven't traded in 15 days." This produces a Segment ID via the Audience Manager MCP. This uses the User Events skill and User Master skill.

**Step 3 — Content Generation (Content Engine):** The Content Engine automatically generates a targeted email for the segment. It selects the commodity_educational recipe, generates copy about gold price trends, renders a gold price_card chart with live data from Compass, and outputs production HTML.

**Step 4 — Campaign Orchestration (NARAD):** NARAD receives the Segment ID and HTML, runs an approval workflow, and sends the email to the targeted users.

**Step 5 — Measurement (Compass):** The Engagement & Communications skill tracks open rates, CTR, and conversion for the campaign. Results are compared against historical benchmarks and fed back into the next analysis cycle.

My skills work (Task 2) enables Steps 1, 2, and 5. My Content Engine (Task 1) enables Step 3. NARAD (existing) handles Step 4. The integration with Compass (Step 3 pulling live data) is future scope, several months out, and contingent on me getting the necessary data access permissions.

---

# 6. Technical Architecture

## 6.1 Content Engine Architecture

The Content Engine is a Python application structured as an MCP server with five internal components:

**Data flow:** Claude Code sends a tool call (e.g., generate_email) → MCP Server receives the call → Recipe Registry selects the template → Copy Writer calls the Anthropic API to generate text for each block → Chart Renderer uses Playwright to generate chart PNGs → HTML Renderer assembles blocks into a final email using Jinja2 templates → the HTML file and image URLs are returned to the CM.

**Tech stack:** Python 3.11+, MCP SDK (Python), BeautifulSoup (HTML parsing), Pydantic (schema validation), Jinja2 (HTML templating), Playwright (chart screenshot rendering), Anthropic SDK (LLM content generation), Typer + Rich (CLI).

**Data layer:** All recipes, block schemas, and brand configs are stored as JSON files in a local data directory. Generated outputs (HTML emails, chart PNGs) go to an output directory. No database is needed for v1.

## 6.2 Compass Skills Architecture

Skills are Markdown documents published to Datahub. The supporting tooling is a small Python toolkit:

**Schema discovery script** — generates skill scaffolds from table metadata by querying Compass or Trino for SHOW TABLES and DESCRIBE TABLE, producing a pre-filled Markdown template.

**Query testing framework** — parses .sql test files (annotated with expected behavior: row count expectations, mandatory columns, max runtime), validates SQL against known best practices (checks for partition filters on large tables, standard exclusions on growth_user_master), and optionally executes queries against real data.

**Skill validator** — checks a completed skill document against the quality checklist: does it have a Tables Overview with actual entries? Join Key Reference? Key Columns for every table? At least 10 tested example queries? Query Strategy Reference with 15+ entries? Data Quality Notes? No remaining TODO/FILL placeholders?

## 6.3 Integration Points

**Content Engine → NARAD:** Output is a single HTML file with hosted image URLs. CM manually pastes this into NARAD. No API integration needed.

**Content Engine → Groww Cloud:** Chart PNGs need to be uploaded to get hosted URLs. This requires cloud upload access (currently an open question).

**Content Engine → Anthropic API:** The copy generation step calls Claude Sonnet via the Anthropic API to fill recipe blocks with email copy.

**Compass Skills → Datahub:** Skills are published as documents on Datahub.

**Content Engine → Compass MCP (future):** A future tool (fetch_live_data) will call Compass to get real-time data (stock prices, user counts, market metrics) for populating emails dynamically. This is months away and requires data access permissions I don't currently have.

---

# 7. My Specific Responsibilities

## What I Own Completely

**Content Engine — everything:**
- Exporting and analyzing 90 days of emails from WebEngage (~450 emails)
- Defining the block library (18 block types as JSON schemas with brand variants)
- Defining the recipe system (8 recipe templates)
- Building the chart image renderer (5 chart types as retina PNGs)
- Building the HTML email renderer (table-based, inline CSS, responsive, SEBI-compliant)
- Building the MCP server (7 tools, full orchestration)
- Writing the LLM prompts for copy generation
- Testing and demoing the end-to-end system

**Compass MCP Skills — everything:**
- Discovery, drafting, testing, and publishing for all 10+ skill domains
- Building the supporting toolkit (schema discovery, query tester, skill validator)
- Interviewing domain data owners for each skill
- Testing all example queries against real data
- Getting each skill reviewed and approved by domain owners
- Publishing to Datahub

## What I Don't Own

- NARAD (existing system, not being modified)
- WebEngage (existing system, I'm only exporting data from it)
- Compass MCP core platform (existing system, I'm only adding skills to it)
- Groww's internal cloud hosting infrastructure (existing, I need upload access)
- Audience Manager / segmentation platform (existing, future integration)
- SEBI compliance review of generated email content (needs compliance team involvement)
- Brand guidelines and design system (need to extract from existing emails or get from design team)

---

# 8. Scope Boundaries

## In Scope (Immediate)

- Content Engine MCP server (Python, Claude Code integration)
- Email analyzer pipeline (450 emails from WebEngage)
- Block library (18 blocks, 4 brand variants each)
- Recipe system (8 recipes)
- Chart renderer (5 chart types, retina PNGs, Playwright-based)
- HTML email renderer (Jinja2 templates, table-based, inline CSS)
- Copy generation via Anthropic API (Claude Sonnet)
- CLI tooling for local testing and development
- All Compass MCP skills under the Growth domain (10+ skills)
- Skill authoring toolkit (scaffold generator, query tester, validator)

## In Scope (Future — Not Immediate)

- Content Engine integration with Compass MCP for live data (contingent on data access)
- Automated segment → content → campaign pipeline
- Chart images uploaded to Groww's cloud (needs cloud access)
- Content Engine v2 improvements based on CM feedback
- Tier 4 skills (UPI, Financial Health, Help & Support, etc.)
- WBR/MBR auto-generation via Compass (weekly/monthly business review emails)

## Out of Scope

- Building or modifying NARAD
- Building or modifying the Compass MCP core platform
- Building a web UI for the Content Engine (it's CLI/MCP only)
- Sending emails (that's NARAD's job)
- User segmentation logic (that's the Audience Manager's job)
- SEBI compliance review process
- Brand/design system creation (I extract from existing emails)
- Mobile app changes
- Any work on 915, AMC, or W products beyond their email branding

---

# 9. Deliverables Checklist

## Task 1: Content Engine

- [ ] Email Analyzer Pipeline — ingests ~450 HTML emails, classifies types, decomposes into blocks, outputs structured JSON
- [ ] Block Library — 18 block types defined as JSON schemas with Pydantic models, brand variants for all 4 sub-brands
- [ ] Recipe System — 8 recipe templates as JSON files with generation hints
- [ ] Chart Image Renderer — 5 chart types (price_card, listing_card, comparison, portfolio, metric) generating retina PNGs via Playwright
- [ ] HTML Email Renderer — Jinja2 templates for all 18 blocks, table-based layout, inline CSS, responsive, SEBI footer
- [ ] MCP Server — 7 tools (list_recipes, get_recipe, recommend_recipe, list_blocks, generate_email, generate_chart, export_email) running as stdio server
- [ ] Copy Generation — LLM prompt templates for email content generation via Anthropic API
- [ ] CLI — Command-line interface for analyze, generate-chart, render-email, serve-mcp
- [ ] End-to-end demo — at least 5 different email types generated and compared against real emails

## Task 2: Compass MCP Skills

- [ ] Skill 1: Engagement & Communications Analytics
- [ ] Skill 2: User Master & Growth Analytics
- [ ] Skill 3: User Events Analytics v2
- [ ] Skill 4: Stocks & Trading Analytics
- [ ] Skill 5: Mutual Fund Analytics
- [ ] Skill 6: F&O Analytics
- [ ] Skill 7: Commodities Trading Analytics
- [ ] Skill 8: 915 Trading Analytics
- [ ] Skill 9: Credit Analytics
- [ ] Skill 10: MTF & Pledge Analytics
- [ ] Skill 11: Onboarding Analytics
- [ ] Skills 12+: Additional domains as assigned

Supporting deliverables:
- [ ] Skill template (standardized Markdown template matching the Growth Analytics skill format)
- [ ] Schema discovery script (auto-generates skill scaffolds from table metadata)
- [ ] Query testing framework (validates SQL queries against best practices and real data)
- [ ] Skill validator (checks document completeness against quality checklist)

---

# 10. Success Criteria

## Content Engine — Definition of Done

The Content Engine v1 is complete when all of the following are true:

**Functional criteria:**
- A campaign manager can describe an email in natural language via Claude Code and receive a production-ready HTML file
- The system correctly recommends recipes based on natural language descriptions
- Generated copy matches Groww's tone: educational, conversational, confident, accessible to retail investors
- Generated copy contains no financial advice, no guaranteed returns, and includes appropriate SEBI-compliant language
- Chart images match Groww's design language (verified by visual comparison against real emails)
- The HTML output renders correctly on Gmail for Android (Groww's primary user base)
- The HTML uses table-based layout with inline CSS only (no modern CSS that email clients would strip)
- The HTML includes a functioning unsubscribe link and SEBI disclaimer footer
- All four sub-brands (Groww, 915, AMC, W) render with correct theming

**Technical criteria:**
- The MCP server starts cleanly and all 7 tools respond without errors
- The email analyzer successfully processes at least 80% of the 450-email corpus without failures
- Chart renderer produces retina PNGs at the correct dimensions for all 5 chart types
- End-to-end generation (prompt → HTML) completes in under 60 seconds

**Validation:**
- At least 5 generated emails (one per major type) have been compared side-by-side against real Groww emails and approved as visually and tonally comparable
- Manager has seen a live demo

## Compass MCP Skills — Definition of Done (Per Skill)

An individual skill is complete when all of the following are true:

**Content criteria:**
- All tables in the domain are documented with schema, grain, freshness, and purpose
- All important columns have data type and description documented
- Join keys are explicitly documented with cross-table mapping
- Partition columns have "ALWAYS FILTER BY" warnings with specific instructions
- Common filter values are enumerated (event names, status codes, type enums)
- At least 10 example queries are included, all tested against real data and returning correct results
- Query Strategy Reference covers at least 15 common business questions
- Data Quality Notes document all known gotchas (NULLs, type casting, freshness gaps, deduplication)
- No TODO, FILL IN, or placeholder text remains in the document

**Validation criteria:**
- Domain data owner has reviewed and confirmed accuracy of table descriptions, column meanings, and query correctness
- Published on Datahub with status "Published"
- After publishing, at least 5 natural language questions have been asked via Compass, and Compass generates correct SQL for all of them

---

# 11. Dependencies & Risks

## Dependencies

| Dependency | Owner | Status | Impact if Blocked |
|---|---|---|---|
| WebEngage email export access | Marketing team / manager | Unknown | Cannot build email analyzer without raw HTML files |
| Groww cloud upload access for images | Infrastructure team | Unknown | Chart images can't be hosted; HTML will have local paths only |
| Compass query access for testing skills | Data platform team | Partially available (intern access restrictions) | Cannot test example queries against real data |
| Brand guidelines (colors, fonts, logos) for all 4 sub-brands | Design team | Unknown (may need to extract from existing emails) | Chart and email rendering may not match brand exactly |
| SEBI disclaimer text templates | Compliance/legal team | Unknown | Footer may not have exact required regulatory text |
| Domain data owner availability for skill reviews | Various teams | Unknown | Skills can be written but not validated and published |
| Table-level read access for each skill's domain | Data platform team | Unknown (may need per-schema access requests) | Cannot discover schemas or test queries |
| Anthropic API key for copy generation | Manager / engineering | Unknown | Copy generation won't work; can only produce structural output |

## Risks

**Risk 1: WebEngage HTML emails may not be structurally consistent.** If different teams or vendors created emails with wildly different HTML patterns, the block detector's heuristic approach may have low accuracy. Mitigation: manual review of 20–30 emails after initial analysis, iterative detector refinement, and acceptance of ~80% accuracy for v1.

**Risk 2: Email HTML rendering across clients is notoriously fragile.** A table-based email that looks perfect in Gmail may break in Outlook or Apple Mail. Mitigation: prioritize Gmail Android (Groww's primary user base) for v1, defer cross-client testing to v2.

**Risk 3: LLM-generated copy may not consistently match Groww's voice.** The tone and style of AI-generated email copy may need significant prompt engineering. Mitigation: include detailed generation hints in recipes, use real Groww emails as few-shot examples in prompts, and plan for iterative prompt refinement.

**Risk 4: Compass skill accuracy depends on domain expertise I don't have.** As an intern, I may not understand the nuances of every financial product domain. Mitigation: interview domain data owners for every skill, get explicit review and sign-off before publishing.

**Risk 5: Data access restrictions may delay Compass skills.** My intern-level access may not cover all schemas and tables needed for every skill. Mitigation: identify access gaps early, request access in batch, and work on accessible skills first while waiting for permissions.

---

# 12. Open Questions

These are unresolved items that need follow-up with my manager or other teams:

## Access & Permissions

1. **WebEngage email export** — Do I have API access to export emails programmatically, or will this be a manual process? Who can grant access if I don't have it?

2. **Groww cloud image hosting** — Can I get write access to the internal cloud for uploading chart images? What's the URL pattern for hosted images? Is there a staging bucket I can use during development?

3. **Compass query access** — Do I have full read access to all Growth-domain tables in Trino, or is it restricted per schema? What's the process for requesting access to additional schemas?

4. **Anthropic API key** — Is there an existing API key I should use for the Content Engine's copy generation, or do I need to set one up?

5. **Table-level access for each skill domain** — For Tier 2 and Tier 3 skills (Stocks, MF, F&O, etc.), do I need to request separate access to each schema?

## Process & Review

6. **Skill review and approval** — Who reviews and approves each Compass skill before it goes live on Datahub? Is it the domain data owner, my manager, or both? Is there a formal sign-off process?

7. **Skill priority** — Is my prioritization (Engagement & Comms first, then User Master, then User Events, then product domains) aligned with what my manager expects? Are there specific skills that are more urgent?

8. **Content Engine demo timeline** — When does my manager want to see the first working Content Engine demo?

9. **Existing dashboards** — Are there existing Superset or Metabase dashboards for each product domain that I can reference when writing skills? Who can point me to them?

## Design & Compliance

10. **Brand guidelines** — Is there a documented design system or style guide for each sub-brand's email styling (Groww, 915, AMC, W)? Specifically: exact hex colors, font specifications, logo image URLs, and spacing rules.

11. **SEBI compliance text** — Are there specific SEBI disclaimer text templates that must appear in every email footer? Are they different for different email types or products? Who owns compliance review for AI-generated email content?

12. **Email content restrictions** — Are there specific words, phrases, or claims that must never appear in Groww marketing emails (beyond the obvious no-financial-advice rule)? Is there a compliance checklist I should build into the generation prompts?

## Technical

13. **Tech stack preference** — Is Python the right choice for internal tools at Groww, or would the team prefer Node.js? Are there existing internal MCP servers I should study for patterns?

14. **WebEngage email format** — What format are emails stored in WebEngage? Raw HTML files, or do I need to extract them from an API response that includes metadata? Is there a campaign-to-email mapping I can use to identify email types?

15. **Image dimensions** — What are the exact retina dimensions expected for chart images in Groww emails? Are there existing chart image assets I can reference for pixel-perfect matching?

---

*This document reflects the full scope of both assigned tasks as understood from the manager briefing and subsequent scoping conversation. It should be treated as a living document and updated as open questions are resolved and scope evolves.*

# Groww — Project Plan

**Owner:** Intern, Growth Team  
**Manager-assigned tasks:** (1) Content Engine, (2) Compass MCP Skills  
**Date:** April 2026

---

## Task Overview

| | Content Engine | Compass MCP Skills |
|---|---|---|
| **Priority** | Side project | Primary work |
| **Timeline** | 1–2 weeks | Ongoing |
| **What it is** | AI-powered HTML email generator (MCP server + Claude Code) | Domain skill documents that teach Compass how to query Groww's data |
| **Deliverable** | Working MCP server that generates production-ready HTML emails | 10+ skill docs (like the Growth Analytics skill) covering all Growth domains |
| **Long-term convergence** | Compass auto-analyzes → segments users → Content Engine generates email → NARAD sends |

---

# TASK 2 — Compass MCP Skills (Primary)

## What a "Skill" Is

A skill is a structured reference document that teaches the Compass text-to-SQL layer how to query a specific domain. Based on the Growth Analytics skill example, each skill contains:

1. **Header** — Domain, catalogs, last validated date
2. **Tables Overview** — Table name, schema, grain, freshness, purpose
3. **Join Key Reference** — How tables connect (primary key: `user_account_id`)
4. **Key Columns by Table** — Column-level detail with types and descriptions
5. **Common Filter Values** — Enums, known values, partition columns
6. **Critical Query Best Practices** — Mandatory filters, gotchas, performance tips
7. **Example Queries** — 10–15 real queries covering common business questions
8. **Query Strategy Reference** — "If someone asks X, use table Y with filter Z"
9. **Data Quality Notes** — Known issues, caveats, edge cases

## Skills to Build (Prioritized)

### Tier 1 — High frequency, directly relevant to your Content Engine work
| # | Skill | Why first |
|---|---|---|
| 1 | **Engagement & Communications Analytics** | Directly feeds Content Engine — campaign CTR, conversion, channel data |
| 2 | **User Master & Growth Analytics** | Foundation table (117M users) — every other skill joins to this |
| 3 | **User Events Analytics** | EMS events power behavioral segmentation for targeting |

### Tier 2 — Core product domains (high query volume)
| # | Skill | Domain |
|---|---|---|
| 4 | **Stocks & Trading Analytics** | Core product — CNC, MIS trades |
| 5 | **Mutual Fund Analytics** | SIP, lumpsum, AMC sub-brand |
| 6 | **F&O Analytics** | Futures & Options trading |
| 7 | **Commodities Trading Analytics** | MCX contracts, crude oil, gold etc. |

### Tier 3 — Specialized domains
| # | Skill | Domain |
|---|---|---|
| 8 | **915 Trading Analytics** | 915 sub-brand specific |
| 9 | **Credit Analytics** | Credit product line |
| 10 | **MTF & Pledge Analytics** | Margin trading facility |
| 11 | **Onboarding Analytics** | Signup → KYC → eSign → FID funnel |

### Tier 4 — Supporting / emerging
| # | Skill | Domain |
|---|---|---|
| 12+ | UPI Payments, User Financial Health, Help & Support, Context Push, etc. | As assigned |

## Workflow for Writing Each Skill

### Step 1 — Discovery (Day 1)
- [ ] Identify all tables in the domain's catalog/schema
- [ ] Get access to table metadata (column names, types, row counts, freshness)
- [ ] Talk to the domain's data owner or analyst — ask: "What are the top 10 questions people ask about this domain?"
- [ ] Identify join keys back to `user_account_id` and `growth_user_master`

### Step 2 — Draft Structure (Day 1–2)
- [ ] Write Tables Overview (table, schema, grain, freshness, purpose)
- [ ] Document Join Key Reference
- [ ] Write Key Columns for each table (copy the format from Growth Analytics skill exactly)
- [ ] List Common Filter Values and enums

### Step 3 — Query Best Practices & Examples (Day 2–3)
- [ ] Identify partition columns and mandatory filters (performance-critical)
- [ ] Write 10–15 example queries covering the "Query Strategy Reference" patterns
- [ ] Test every query against real data via Compass to verify correctness
- [ ] Document Data Quality Notes (NULLs, type casting, deduplication, freshness gaps)

### Step 4 — Review & Publish (Day 3)
- [ ] Self-review: does every table have join keys documented? Are all partition filters noted?
- [ ] Get domain owner to validate
- [ ] Publish to Compass (Datahub)

**Target pace:** 1 skill every 2–3 days once you're in rhythm (first one will take longer).

## Quality Checklist (Per Skill)

Use this before marking any skill as done:

- [ ] Every table has: schema, grain, freshness, purpose
- [ ] Every column that matters has: type, description
- [ ] Join keys are explicit — how does this domain connect to `growth_user_master`?
- [ ] Partition columns are called out with "ALWAYS FILTER BY" warnings
- [ ] At least 10 example queries, all tested
- [ ] Common filter values listed (enums, known strings)
- [ ] Data quality notes cover: NULLs, type casting, deduplication, freshness
- [ ] "Query Strategy Reference" table: business question → table → filter

---

# TASK 1 — Content Engine (Side Project, 1–2 Weeks)

## Architecture

```
┌─────────────────────────────────────────────────┐
│                 Claude Code / LLM                │
│         (CM talks here in natural language)      │
└──────────────────────┬──────────────────────────┘
                       │ MCP tool calls
┌──────────────────────▼──────────────────────────┐
│            Content Engine MCP Server             │
│                                                  │
│  Tools exposed:                                  │
│  ├── list_recipes()                              │
│  ├── list_blocks()                               │
│  ├── recommend_recipe(description)               │
│  ├── generate_email(recipe, content_brief)       │
│  ├── generate_chart_image(type, data)            │
│  ├── preview_email(email_id)                     │
│  └── export_html(email_id)                       │
└──────┬───────────┬───────────┬──────────────────┘
       │           │           │
┌──────▼──┐ ┌──────▼──┐ ┌─────▼───────┐
│ Recipe  │ │ Chart   │ │ HTML        │
│ Library │ │ Renderer│ │ Renderer    │
│ (JSON)  │ │ (PNG)   │ │ (Templates) │
└─────────┘ └─────────┘ └─────────────┘
```

## Sprint Plan

### Week 1 — Foundation

**Days 1–2: Email Analysis Pipeline**
- [ ] Export ~450 HTML emails from WebEngage (last 90 days)
- [ ] Write a parser that extracts structure from raw HTML (identify headers, images, CTAs, text blocks, dividers)
- [ ] Auto-classify emails into types: educational, onboarding, promotional, commodity update, trading alert, etc.
- [ ] Output: structured JSON per email showing detected blocks and classification

**Days 3–4: Block Library + Recipe System**
- [ ] Define the 18 block JSON schemas (based on analysis + the block table you already have)
- [ ] Each block schema: name, required fields, optional fields, default styles, brand variants (Groww / 915 / AMC / W)
- [ ] Create 5–8 recipe templates from the most common email patterns found in analysis
- [ ] Validate: can every analyzed email be represented as a combination of these blocks?

**Day 5: HTML Renderer**
- [ ] Build the recipe JSON → HTML conversion engine
- [ ] Each block type maps to an HTML partial/template
- [ ] Support brand theming (Groww green, 915 brand, AMC brand, W brand)
- [ ] Test: render a recipe and compare output visually against original email

### Week 2 — AI Layer + MCP

**Days 6–7: Chart Image Renderer**
- [ ] Build 5 chart generators: price_card, listing_card, comparison, portfolio, metric
- [ ] Input: structured data (prices, dates, labels) → Output: retina PNG
- [ ] Use a library like Puppeteer (render HTML chart → screenshot) or Python (matplotlib/Pillow)
- [ ] Test against the crude oil email example — does the price_card output match?

**Days 8–9: MCP Server**
- [ ] Set up MCP server (Node.js or Python)
- [ ] Expose tools: `list_recipes`, `recommend_recipe`, `generate_email`, `generate_chart_image`, `export_html`
- [ ] Wire up: Claude Code calls `generate_email` → picks recipe → LLM generates text for each block → chart renderer makes images → HTML renderer assembles final email
- [ ] Image upload to Groww's internal cloud (or placeholder URLs for now)

**Day 10: Testing + Demo**
- [ ] End-to-end test: "Generate a commodity email about crude oil price movement" → get final HTML
- [ ] Compare output against real crude oil email
- [ ] Document known limitations and next steps
- [ ] Demo to manager

## Content Engine — Component Checklist

| Component | Input | Output | Status |
|---|---|---|---|
| Email Analyzer | Raw HTML from WebEngage | Classified + decomposed JSON | ☐ |
| Block Library | Analysis output + manual curation | 18 block JSON schemas | ☐ |
| Recipe System | Block library | 5–8 recipe templates (JSON) | ☐ |
| Chart Renderer | Structured data (prices, labels) | Retina PNG images (5 types) | ☐ |
| HTML Renderer | Recipe JSON + filled block data | Production HTML email | ☐ |
| MCP Server | Natural language via Claude Code | Orchestrates all above | ☐ |

---

# Long-Term Vision — The Full Loop

Once both tasks mature, they converge:

```
Compass MCP (with all skills)
       │
       ├── 1. AUTO-ANALYTICS: "Commodity traders down 15% this month"
       │
       ├── 2. SEGMENTATION: "Users who traded gold in last 30 days but not in 15"
       │         → Segment ID from Audience Manager
       │
       ├── 3. CONTENT: Content Engine generates targeted email
       │         → Recipe: commodity_educational
       │         → Chart: gold price_card with live data from Compass
       │         → Text: AI-generated copy about gold trends
       │
       ├── 4. CAMPAIGN: NARAD orchestrates send
       │         → Segment ID + HTML + approval
       │
       └── 5. MEASUREMENT: Engagement & Comms skill tracks CTR/conversion
                 → Feeds back into next cycle
```

**Your skills work (Task 2) enables steps 1, 2, and 5.**  
**Your Content Engine (Task 1) enables step 3.**  
**NARAD (existing) handles step 4.**

---

# Open Questions to Resolve with Manager

1. **WebEngage access** — Do you have API/export access to pull the 90-day email archive?
2. **Image hosting** — Can you get write access to Groww's internal cloud for chart image uploads, or should you use a staging bucket?
3. **Compass MCP access** — When can you get query access to test your skills against real data?
4. **Skill review process** — Who validates each skill before it goes live? Domain data owners?
5. **Content Engine demo** — When does your manager want to see a first working version?
6. **Brand guidelines** — Are there documented brand/style guides for email HTML (colors, fonts, spacing) for each sub-brand?

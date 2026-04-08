---
title: "Claude Session: Scoping Content Engine & Compass MCP Integration"
date: 2026-04-07
source: "[[raw/claude.ai-Scoping a context engine and compass MCP integration.pdf]]"
screenshots: "[[raw/claude.ai-Scoping a context engine and compass MCP integration - Claude-fpscreenshot.pdf]]"
participants: Ashish (user), Claude (AI)
session-type: scoping / requirements gathering
related-projects:
  - "[[Projects/content-engine]]"
  - "[[Projects/compass-mcp-skills]]"
status: done
---

# Claude Session Analysis: Scoping Content Engine & Compass MCP

## Session Summary
A structured scoping session (9 rounds of Q&A) between Ashish and Claude to fully define two tasks assigned by Ashish's manager at [[wiki/Groww|Groww]]:

1. **Task 1 — Content Engine**: Build an AI-powered HTML email generator from scratch
2. **Task 2 — Compass MCP Skills**: Write/enhance skill documents for [[wiki/Compass-MCP|Compass MCP]] domains

## Key Discoveries (Round by Round)

### Round 1: Business Context
- [[wiki/Groww|Groww]] = Indian retail investment platform (stocks, MF, F&O, commodities)
- This project is specifically Groww's **email marketing system** HTML email generator
- Sub-brands: Groww, [[wiki/915-Brand|915]] (trading), AMC, W
- [[wiki/Compass-MCP|Compass MCP]] = central MCP with access to 5000+ raw tables and 25000+ derived tables

### Round 2: Content Engine Architecture
- **Current flow**: Creative team designs in Figma → converts to HTML → sends via [[wiki/NARAD|NARAD]]
- **New flow (CM flow)**: (1) Choose template, (2) Prompt what to build, (3) Images via prompt, (4) Text via prompt
- Content Engine = AI-powered email builder where CMs select a recipe (JSON template of blocks), then use prompts to generate copy and chart images
- Output: production-ready HTML email → goes to [[wiki/NARAD|NARAD]]

### Round 3: Build Scope
- Recipe/block system does NOT already exist properly — needs to be built from scratch
- Will analyze last 90 days of sent emails (~450 from [[wiki/WebEngage|WebEngage]]) to reverse-engineer block/recipe patterns
- The workflow will auto-classify email type, analyze HTML, images, everything

### Round 4: Pipeline End-to-End
- Existing emails stored in [[wiki/WebEngage|WebEngage]] (~150/month)
- Classification partially user-defined, partially data-discovered
- Compass MCP integration is **future scope** (not immediate) — restricted access for interns
- **Immediate scope**: Content Engine with manual data input
- **Future scope**: Full loop — Analytics → Segmentation → Content → Campaign → [[wiki/NARAD|NARAD]] → Approve

### Round 5: Immediate Deliverables
1. Pull ~450 emails from [[wiki/WebEngage|WebEngage]], parse HTML, classify, decompose into blocks
2. Define the block library (18 blocks) as JSON components
3. Define recipes (standard block combinations per email type)
4. Build AI generation layer (pick recipe → generate images → generate text → output HTML)

### Round 6: Technical Architecture
- **Component 1 — Email Analyzer**: HTML emails → parse → classify → decompose into blocks → structured recipes
- **Component 2 — Recipe/Block Library**: JSON block definitions + recipe templates
- **Component 3 — Content Engine MCP Server**: Tools like `list_recipes`, `generate_email`, `preview_email`
- Tech stack: **Python**, MCP server + Claude Code workflow (no web app)
- Image generation: needs to be built from scratch (5 chart templates: price_card, listing_card, comparison, portfolio, metric)

### Round 7: Full Scope Confirmed
1. Email Analyzer Pipeline (~450 HTML emails)
2. Block Library (18 block definitions as JSON)
3. Recipe System (CM-provided or LLM-recommended)
4. Chart Image Renderer (5 templates, retina PNGs)
5. HTML Email Renderer (recipe JSON → production HTML via Jinja2)
6. Content Engine MCP Server (wraps all above as tools)
- Final output: chart PNGs → uploaded to Groww cloud → get URLs → production HTML with embedded URLs → CM pastes into [[wiki/NARAD|NARAD]]

### Round 8: Compass MCP Scoping
- Full automated loop: Analytics → Segmentation → Content → Campaign → [[wiki/NARAD|NARAD]] → Approve
- Compass MCP side: Text-to-SQL layer, Segmentation (Audience Manager), Skillsets, New Biz targeting, Comms outputs
- Compass MCP integration is **future scope** — months away
- Segmentation goes BQ → Audience Manager → [[wiki/WebEngage|WebEngage]]

### Round 9: Compass MCP Skills Work
- All domains under Growth assigned to Ashish: User Events Analytics, MF Analytics, MTF & Pledge Analytics, Credit Analytics, Growth Analytics, Commodities Trading Analytics, Engagement & Communications, F&O Analytics, Stocks & Trading Analytics, User Master
- **16 skills already exist** — Ashish's job is to **enhance existing + add new ones**
- Each skill = schema reference document teaching LLM how to query domain tables
- Target pace: 1 skill every 2-3 days

### Final Confirmed Priorities
- **Primary work**: Compass MCP skills (ongoing, high priority)
- **Side project**: Content Engine (1-2 weeks timeframe)

## Documents Produced
1. Project Plan v2 (detailed with Bases views)
2. Task Description v2 (corrected with Deep Dive doc findings)
3. Implementation Plan v2 (full Python code architecture)
4. Claude Code Bootstrap Prompt (for new sessions)

## Key Corrections (from later docs)
- Compass is **DataHub + Superset + Trino** unified via MCP (80+ tools), NOT just text-to-SQL
- Content Engine and Compass are **parallel systems**, not integrated — connected only indirectly through a human
- 16 skills already exist with significant detail — task is enhance + add, not write from scratch
- Segmentation flow: BQ → Audience Manager → [[wiki/WebEngage|WebEngage]]
- WBR/MBR moved out of scope — Compass's scheduled reports handle those
- Key tables for Engagement skill: `engagement_pn_narad_master` (250M/day), `engagement_email_backend_master` (3-6M)

# Claude Code Bootstrap Prompt
## Copy everything below this line into a new Claude Code session

---

You are my technical partner on two projects at Groww (Indian fintech / stock trading platform, 117M users, SEBI-regulated). I'm an intern on the Growth team. I'll give you full context, then we'll start working.

## MY TWO TASKS

**Task 1 — Content Engine (side project, 1–2 weeks):** Build an AI-powered HTML email generator as an MCP server. Campaign managers describe an email in natural language → system picks a recipe (JSON template of blocks) → generates copy + chart images via LLM → outputs production HTML → CM pastes into NARAD (our campaign platform) to send.

**Task 2 — Compass MCP Skills (primary, ongoing):** Enhance existing and build new domain-specific skill documents for Compass — Groww's unified data platform MCP (DataHub + Superset + Trino, 80+ tools, 16 skills already exist). Skills teach the AI which tables to use, how to filter, what columns mean, and which queries are safe to run.

These are PARALLEL systems — Compass answers "what happened?" (analytics), Content Engine answers "what email should we send?" (content). They don't call each other. A human bridges the gap.

## COMPANY CONTEXT

- Groww: stocks, mutual funds, F&O, commodities (MCX). Regulated by SEBI.
- Sub-brands: Groww (main, green #00D09C), 915 (separate trading app), AMC, W
- NARAD: campaign distribution — we paste HTML into it, it sends to users
- WebEngage: current marketing platform, stores ~150 emails/month
- Audience Manager: segmentation (BQ → AM → WebEngage). Not my scope.
- Compass MCP: at `http://127.0.0.1:3100/mcp` via local proxy on VPN. Has `search`, `run_trino_query`, `list_schema_fields`, `get_database_tables`, `generate_chart`, and 75+ other tools.

## THE 4-BOX PIPELINE (my place in it)

| Box | Owner | My Role |
|---|---|---|
| Analytics | Compass MCP + Superset (16 skills) | Enhance & expand skills |
| Segmentation | Audience Manager + NARAD + ML | Not my scope |
| **Content** | **My Content Engine** | **Building this** |
| Campaign | WebEngage + NARAD + Netcore/FCM | Not my scope |

WBR/MBR (weekly/monthly business review reports) are handled by Compass's scheduled reports feature — NOT by my Content Engine.

## CONTENT ENGINE — WHAT I'M BUILDING

**6 components:**
1. **Email Analyzer** — Parse ~450 HTML emails from WebEngage (90 days), detect block types, classify email type, output structured JSON
2. **Block Library** — 18 reusable email blocks as Pydantic schemas (header_logo, hero_text, hero_image, chart_image, article_section, factor_card, cta_button, social_proof_banner, footer_standard, divider, card_wrapper, contract_row, filter_pills, listing_header, step_instruction, app_screenshot_pair, product_category_grid, social_showcase)
3. **Recipe System** — 8 recipe templates: commodity_educational, onboarding_welcome, product_discovery, trading_alert, feature_announcement, educational_how_to, weekly_digest, promotional
4. **Chart Renderer** — 5 chart types as retina PNGs via Playwright: price_card, listing_card, comparison, portfolio, metric
5. **HTML Renderer** — Jinja2 templates for all 18 blocks → table-based email HTML with inline CSS (no modern CSS — email clients strip it). Must include SEBI footer, unsubscribe link, support all 4 brands.
6. **MCP Server** — 7 tools: list_recipes, get_recipe, recommend_recipe, list_blocks, generate_email, generate_chart, export_email

**Tech stack:** Python 3.11+, MCP SDK, BeautifulSoup, Pydantic, Jinja2, Playwright, Anthropic SDK (Claude Sonnet for copy generation), Typer+Rich CLI.

**Reference email:** A commodity educational email about crude oil — has header_logo → hero_text ("What moved Crude Oil prices this month?") → price_card chart (₹9,044, +48.29%) → article sections → 3 factor cards (Geopolitics, OPEC+, Inventory) → listing_card chart → CTA ("EXPLORE NOW") → social proof → SEBI footer. This is the gold standard to match.

## COMPASS SKILLS — WHAT I'M DOING

16 skills already exist with varying depth. I'm enhancing existing ones and building new ones.

**Minimal skills (need ground-up build):**
- Context Push Analytics
- User Financial Health Analytics  
- MF Analytics Extended

**Key existing skills I'm enhancing (most relevant to my work):**
- Engagement & Communications — 9 tables including `engagement_email_backend_master` (3-6M/day), `engagement_pn_narad_master` (250M/day), `engagement_user_session_daily` (8M/day). PN funnel: 3M → 2M sent → 440K received → 19K clicked (4.3% CTR). Needs column-level docs and tested SQL.
- Commodities Trading — 6 tables. `commodity_order_master` has 71.6M rows with NO partition. Always use `commodity_day_view_seg` (684K, pre-aggregated). Timestamps are epoch ms.
- Mutual Funds — 10+ tables with table selection matrix but NO guardrails documented.

**Critical reference data:**
- Universal join key across ALL tables: `user_account_id` (format `ACC<digits>`)
- Schemas: `platform_iceberg.core_invest` (trading), `platform_iceberg.core_bgv` (user lifecycle), `platform_iceberg.dwh_invest` (MF), `platform_iceberg.ems` (events raw), `platform_dp_pinot.default` (events sampled)
- Pinot is SAMPLED — raw COUNT(*) is wrong. Must use boosting_factor. Last 1-2 days broken.
- `engagement_session_indepth` has 57 BILLION rows — SINGLE DAY queries only
- `fno_order_master` has future dates up to 2090 — must cap date range
- Stale tables to NEVER use: `stocks_order` (Oct 2025), `invest_user_fid_info` (Jun 2024), `hns_freshchat_tickets_master_trino` (BROKEN), `mtf_transactions_v2` (Nov 2024)

## WHAT I'VE ALREADY DONE (Prior Work)

Here's the work I did before this scoping session. It was exploratory / basic / sometimes in the wrong direction. Some of it is usable, some needs to be reworked:

[TODO — FILL THIS IN WITH YOUR ACTUAL PRIOR WORK. Examples of what to write here:]

- "I explored the emails in WebEngage and manually exported ~50 HTML files to understand the structure"
- "I wrote a basic HTML parser that extracts images and text but doesn't detect block types yet"
- "I mapped out what the engagement team does — their campaign types, email frequency, tools they use"
- "I started looking at Compass and ran some test queries on the engagement tables"
- "I have a rough recipe JSON for the commodity email type"
- "I set up the project repo at [path] with [structure]"
- "I built a prototype chart renderer that works for price_card but the styling doesn't match Groww's design"
- "I compiled notes from talking to [person] about [topic]"

[BE SPECIFIC — mention file paths, what works, what's broken, what's half-done]

## WHAT NEEDS TO CHANGE FROM MY PRIOR DIRECTION

Based on the scoping I did with my manager and the research into Compass docs:

1. **Compass is NOT just text-to-SQL** — it's DataHub + Superset + Trino with 80+ tools. My skill work should use Compass's own tools (`search`, `list_schema_fields`, `run_trino_query`) for discovery, not custom scripts.
2. **Content Engine and Compass are parallel** — I was possibly thinking they'd integrate directly. They don't. Compass = analytics. Content Engine = email generation. Human bridges the gap.
3. **16 skills already exist** — I don't write them from scratch. I enhance existing ones (add column docs, tested SQL, guardrails) and build new ones for uncovered domains.
4. **Skills priority is my primary work** — Content Engine is a 1-2 week side project. Skills are ongoing.
5. **WBR/MBR is Compass's job** — scheduled reports via Superset, not my Content Engine.

## HOW TO WORK WITH ME

- **Skills work:** Use Compass tools directly. When I say "discover the tables for X domain," use `search`, `get_database_tables`, `list_schema_fields`, `run_trino_query` via the Compass MCP connection. Help me write and test SQL, then assemble the skill document.
- **Content Engine work:** Help me build the 6 components in Python. We'll go component by component. Start from wherever my prior work left off.
- **Always ask before assuming** — if something in my prior work conflicts with the scoped plan above, flag it and ask which direction to go.
- **Reference the real data** — when writing skill SQL, use actual table names from the schemas above. When building email templates, match Groww's design language (green #00D09C, Inter font, 600px email width, table-based layout).

## LET'S START

First, look at my current project directory and tell me:
1. What exists already
2. What's usable as-is
3. What needs to be reworked to match the plan above
4. What's the most impactful next step

Start by exploring the project files.

I'm an intern at Groww (Indian fintech, 117M users, stocks/MF/F&O/commodities). I was assigned two tasks by my manager. I've already done some exploratory/basic work on this project, but I just finished a deep scoping session that clarified exactly what I need to build and corrected some wrong assumptions I had. 

I need you to understand the scoped plan, then explore my project directory to see what I've already built, and tell me: what's reusable as-is, what needs reworking, and what's the best next step.

---

## TASK 1: Content Engine (side project, 1-2 weeks)

An MCP server (Python) that generates production-ready HTML marketing emails. A campaign manager talks to Claude Code → picks a recipe (JSON template of ordered blocks) → system generates copy via LLM + chart images via Playwright → outputs final HTML that gets pasted into NARAD (Groww's email/push distribution platform).

**6 components I need:**
1. Email Analyzer — parse ~450 HTML emails from WebEngage, detect 18 block types, classify email type, output JSON
2. Block Library — 18 block types as Pydantic schemas with brand variants (Groww/#00D09C, 915, AMC, W)
3. Recipe System — 8 recipe templates (commodity_educational, onboarding_welcome, product_discovery, trading_alert, feature_announcement, educational_how_to, weekly_digest, promotional)
4. Chart Renderer — 5 chart types as retina PNGs via Playwright (price_card, listing_card, comparison, portfolio, metric)
5. HTML Renderer — Jinja2 templates, table-based layout, inline CSS only (email client compatible), 600px max width, SEBI footer, unsubscribe link, 4 brand themes
6. MCP Server — 7 tools (list_recipes, get_recipe, recommend_recipe, list_blocks, generate_email, generate_chart, export_email)

Stack: Python 3.11+, MCP SDK, BeautifulSoup, Pydantic, Jinja2, Playwright, Anthropic SDK, Typer/Rich CLI.

## TASK 2: Compass MCP Skills (primary ongoing work)

Compass is Groww's unified data platform MCP — DataHub (catalog) + Superset (visualization) + Trino (SQL) with 80+ tools. 16 analytics skills already exist. My job is to enhance existing skills (add column-level docs, tested SQL, guardrails) and build new skills for uncovered domains.

**Skills needing ground-up build:** Context Push, User Financial Health, MF Extended (currently minimal/summary only)

**Skills needing enhancement:** Engagement & Communications (most relevant to Content Engine — has `engagement_email_backend_master` 3-6M/day, `engagement_pn_narad_master` 250M/day), Commodities (no partition on `commodity_order_master` 71.6M rows — must use `commodity_day_view_seg`), Mutual Funds (no guardrails documented)

**For skill work I use Compass's own tools** (`search`, `get_database_tables`, `list_schema_fields`, `run_trino_query`) — not custom scripts. Compass is at `http://127.0.0.1:3100/mcp`.

## KEY CORRECTIONS FROM SCOPING (things I may have gotten wrong in my earlier work)

1. **Compass and Content Engine are PARALLEL systems** — they don't integrate or call each other. Compass = analytics ("what happened?"). Content Engine = email generation ("what to send?"). A human bridges the gap.
2. **16 skills already exist** — I don't write them from scratch. I enhance existing + build new for uncovered domains.
3. **Compass is NOT just text-to-SQL** — it's DataHub + Superset + Trino unified. 80+ tools.
4. **WBR/MBR reports are Compass's scheduled reports** — NOT my Content Engine. My system makes branded marketing emails only.
5. **Segmentation is not my scope** — that's Audience Manager (BQ → AM → WebEngage).
6. **Skills are my PRIMARY work**, Content Engine is a side project.

## CRITICAL DATA REFERENCES

- Universal join key: `user_account_id` (format `ACC<digits>`)
- Schemas: `platform_iceberg.core_invest` (trading), `platform_iceberg.core_bgv` (user lifecycle), `platform_iceberg.dwh_invest` (MF), `platform_iceberg.ems` (events)
- Pinot events are SAMPLED — must use boosting_factor, last 1-2 days broken
- `engagement_session_indepth` = 57B rows, SINGLE DAY queries only
- `fno_order_master` has future dates to 2090 — must cap
- NEVER use: `stocks_order` (stale Oct 2025), `invest_user_fid_info` (stale Jun 2024), `hns_freshchat_tickets_master_trino` (BROKEN), `mtf_transactions_v2` (stale Nov 2024)

---

Now explore my project directory. Look at everything — code, configs, data files, notes, any work I've done. Then give me:
1. **Inventory** — what exists, organized by which of the 6 Content Engine components (or skill work) it maps to
2. **Assessment** — for each piece: usable as-is / needs rework / wrong direction / irrelevant
3. **Gaps** — what's completely missing that I need to build
4. **Recommended next step** — the single most impactful thing to do right now

# Today's Commands — 2026-04-07

## Priority: Compass MCP Skills (Primary Work)

### Start with: Engagement & Communications Skill (CMP-1)
This is Tier 1 priority — directly feeds the Content Engine.

**For Claude Code (with Compass MCP connected):**
```
Connect to Compass MCP. I need to enhance the Engagement & Communications skill.

Start by discovering the schema:
1. Search for tables matching "engagement" — list all with row counts
2. Get full schema for `engagement_pn_narad_master` and `engagement_email_backend_master`
3. Identify partition columns, join keys, and must-filter columns
4. Run a sample query to validate table access

Key tables I know about:
- engagement_pn_narad_master (250M rows/day)
- engagement_email_backend_master (3-6M rows)

Output: A complete skill document following the Growth Analytics template format.
```

## Side Project: Content Engine (Week 1 Foundation)

### Day 1: Email Analyzer (CE-1)
**For Claude Code:**
```
I'm building an email analyzer pipeline for Groww's Content Engine.
Task: Parse raw HTML emails from WebEngage, classify by type (educational, onboarding, promotional, commodity, transactional), and decompose into constituent blocks.

Start by:
1. Set up Python project structure with `uv init`
2. Build the HTML parser that extracts structure from email HTML
3. Create the block detector (pattern matching for 18 known block types)
4. Build the email classifier

Tech: Python, BeautifulSoup/lxml for parsing
Output: JSON recipe format per email
```

## Completed
- [x] Initial vault setup
- [x] Scoping session ingested (2 PDFs → AI session + 2 projects + 5 tasks + 5 wiki pages)

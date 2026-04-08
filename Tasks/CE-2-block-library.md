---
title: "Define Block Library (18 blocks)"
status: todo
priority: medium
owner: me
due: 2026-04-14
deliverable: "18 block definitions as JSON schemas"
sources: "[[raw/claude.ai-Scoping a context engine and compass MCP integration.pdf]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
task-id: "CE-2"
project: "[[Projects/content-engine]]"
execution-log: "2026-04-07: Task extracted from scoping session"
---

# Define Block Library

## What
Create JSON schema definitions for all 18 email blocks. These are the building blocks that compose any Groww email.

## Known Blocks
header_logo, hero_text, hero_image, chart_image, article_section, factor_card, cta_button, social_proof_banner, divider, footer_standard, card_wrapper, step_instruction, contract_row, filter_pills, listing_header, and more.

## Success Criteria
- [ ] Each block has a JSON schema with required/optional fields
- [ ] Blocks are composable (can be arranged in any order)
- [ ] Schema supports both [[wiki/Groww|Groww]] and [[wiki/915-Brand|915]] brand variants
- [ ] Validated against real email decompositions from analyzer

## Blocked By
- [[Tasks/CE-1-email-analyzer]] (iteratively — analyzer output informs block definitions)

---
title: "Build Email Analyzer Pipeline"
status: todo
priority: medium
owner: me
due: 2026-04-14
deliverable: "Pipeline that ingests ~450 HTML emails from WebEngage, classifies type, decomposes into blocks"
sources: "[[raw/claude.ai-Scoping a context engine and compass MCP integration.pdf]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
task-id: "CE-1"
project: "[[Projects/content-engine]]"
execution-log: "2026-04-07: Task extracted from scoping session"
---

# Build Email Analyzer Pipeline

## What
Ingest ~450 HTML emails from [[wiki/WebEngage|WebEngage]] (last 90 days, ~150/month). Parse the raw HTML, classify each email by type (educational, onboarding, promotional, commodity update, etc.), and decompose into constituent blocks.

## Success Criteria
- [ ] Can ingest raw HTML email files
- [ ] Correctly classifies into 5+ email types
- [ ] Decomposes each email into ordered list of blocks (matching the 18 block types)
- [ ] Outputs structured JSON (recipe format) per email
- [ ] Handles both [[wiki/Groww|Groww]] and [[wiki/915-Brand|915]] brand emails

## Dependencies
- Access to WebEngage email exports
- Block type definitions (can be iterative)

## Blocks
- [[Tasks/CE-2-block-library]]

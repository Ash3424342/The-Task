---
title: WebEngage
type: entity
---

# WebEngage

Campaign management platform used by [[Groww]]. Stores existing sent emails — the source for the Content Engine's email analysis pipeline.

## Key Facts
- ~150 emails per month (standard, adhoc, push, etc.)
- Last 90 days = ~450 emails available for analysis
- Email types: educational, onboarding, promotional, commodity update, etc.

## Role in Pipeline
- **Source**: Existing emails stored here → analyzed by Content Engine to reverse-engineer block/recipe patterns
- **Segmentation target**: BQ → Audience Manager → WebEngage (segments are pushed here)

## Related
- [[Groww]]
- [[NARAD]]
- [[Compass-MCP]]

---
title: NARAD
type: entity
---

# NARAD

[[Groww]]'s internal email and push notification sending/distribution platform. Campaign managers paste HTML + images into NARAD, which handles the actual delivery to users.

## How It Works
- Contains a page where you paste the HTML and images
- Images are hosted on Groww's internal cloud — URLs are embedded in the email HTML
- Handles wide audience distribution

## Key Table
- `engagement_pn_narad_master` — 250M rows/day (push notification data)

## Role in Pipeline
Content Engine generates HTML → CM pastes into NARAD → NARAD sends to users

## Related
- [[Groww]]
- [[Compass-MCP]]
- [[WebEngage]]

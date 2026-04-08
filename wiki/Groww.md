---
title: Groww
type: entity
---

# Groww

Indian retail investment platform. Products include stocks, mutual funds, F&O (futures & options), and commodities trading. Subject to SEBI regulation (Securities and Exchange Board of India).

## User Base
- 117M registered users total
- 72M signup-only
- 27M onboarded & transacted
- 17M onboarded & non-transacted
- 98M of 117M on Android mobile

## Sub-brands
- **Groww** — main brand
- **[[915-Brand|915]]** — trading sub-brand
- **AMC** — asset management
- **W** — sub-brand

## Key Internal Systems
- **[[Compass-MCP]]** — Central data platform MCP (DataHub + Superset + Trino, 80+ tools, 5000+ raw tables, 25000+ derived tables)
- **[[NARAD]]** — Email/push notification sending platform
- **[[WebEngage]]** — Campaign management and email storage (~150 emails/month)
- **Content Engine** — AI-powered HTML email generator (being built by [[Ashish]])

## Critical Data References
- Universal join key across ALL tables: `user_account_id` (format `ACC<digits>`)
- Schemas: `platform_iceberg.core_invest` (trading), `platform_iceberg.core_bgv` (user lifecycle), `platform_iceberg.dwh_invest` (MF), `platform_iceberg.ems` (events)
- Pinot events are SAMPLED — must use `boosting_factor`, last 1-2 days broken
- `engagement_session_indepth` = 57B rows — SINGLE DAY queries only
- **Stale tables (NEVER use)**: `stocks_order` (Oct 2025), `invest_user_fid_info` (Jun 2024), `hns_freshchat_tickets_master_trino` (BROKEN), `mtf_transactions_v2` (Nov 2024)
- `fno_order_master` has future dates to 2090 — must cap date range

## Data Scale
- 5000+ raw tables
- 25000+ derived tables
- `engagement_pn_narad_master`: 250M rows/day
- `engagement_email_backend_master`: 3-6M rows
- `engagement_session_indepth`: 57B rows (SINGLE DAY queries only)
- `user_product_txn_master_fact`: 8.3B rows
- `fact_transaction_stocks`: 7.1B rows
- `fno_order_master`: 3.8B rows
- `core_stocks_order`: 2.5B rows
- `fno_pnl_master`: 1.9B rows
- NARAD: 170-250M push notifications/day

## Related
- [[Projects/content-engine]]
- [[Projects/compass-mcp-skills]]

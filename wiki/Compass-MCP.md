---
title: Compass MCP
type: entity
---

# Compass MCP

[[Groww]]'s unified data platform MCP server. Connects AI assistants (Claude, Cursor, etc.) to Groww's data infrastructure via Model Context Protocol. **Not just text-to-SQL** — it unifies DataHub + Superset + Trino into a single conversational interface.

Author: Saurabh Dubey. Status: BETA.

## Architecture
```
Claude/Cursor → Local Proxy (127.0.0.1:3100) → Compass MCP → DataHub (catalog)
                                                     ↓              ↓
                                               Superset (viz)    Trino (SQL)
```

## Three Unified Systems
- **DataHub** — Data catalog: dataset discovery, schema exploration, metadata management, lineage tracking, event instrumentation metadata with GitHub source links
- **Superset** — Visualization: chart creation (12 types), dashboard building, scheduled reports (Email/Slack/Webhook), data alerts
- **Trino** — SQL engine: ad-hoc querying against Iceberg data lake with safety guardrails (read-only, partition enforcement, 600s timeout, 5000 row limit)

## 80+ Tools (by Category)

| Category | Count | Key Tools |
|---|---|---|
| Analytics Skills | 16 | Auto-loaded per domain |
| SQL Execution | 5 | `run_trino_query`, `superset > execute_sql` |
| Charts | 8 | `generate_chart`, `generate_explore_link`, `update_chart` |
| Dashboards | 6 | `generate_dashboard`, `add_chart_to_existing_dashboard` |
| Data Discovery | 6 | `search`, `list_schema_fields`, `get_database_tables` |
| Metadata | 10 | `update_description`, `add_owners`, `add_tags` |
| Lineage | 1 | `get_lineage` (upstream/downstream, configurable depth) |
| Reports & Alerts | 4 | Schedule reports, data alerts |
| Datasets | 4 | Create, get info, list, delete |
| Databases | 3 | List databases, schemas, tables |
| Tags (Superset) | 4 | Create, add, list, delete |
| Identity & Admin | 4 | Identity, MCP token, instance info, health check |

## 16 Analytics Skills
Skills are the **single most powerful feature**. Pre-built knowledge packages that tell the AI exactly which tables, columns, filters, and query patterns to use per domain.

| # | Skill | Domain | Key Tables | Size |
|---|---|---|---|---|
| 1 | Stocks & Trading | Equities | `core_stocks_order`, `fact_transaction_stocks` | 2.5B, 7.1B |
| 2 | F&O Analytics | Derivatives | `fno_order_master`, `fno_pnl_master` | 3.8B, 1.9B |
| 3 | Mutual Funds | MF | `fact_mutual_fund_order_details_enriched` | Millions |
| 4 | User Master & Growth | Growth | `growth_user_master_ultimate`, `user_product_txn_master_fact` | 117M, 8.3B |
| 5 | User Events | Instrumentation | DataHub events + `ems_raw` (Pinot sampled) | 57B+ |
| 6 | Onboarding | Activation | `onb_usercheckpoints_master` | 117M |
| 7 | UPI Payments | Payments | `groww_upi_onb_user_fact` | 66.9M |
| 8 | Engagement & Comms | Comms | `engagement_pn_narad_master`, `engagement_email_backend_master` | 250M/day, 3-6M |
| 9 | MTF & Pledge | Margin | `aay_mtf_book_agg`, `fact_pledge_transaction` | 243, 50M |
| 10 | Commodities | MCX | `commodity_day_view_seg`, `commodity_order_master` | 684K, 71.6M |
| 11 | Help & Support | Support | `hns_helpdesktickets_master_trino` | 12M |
| 12 | 915 Trading | 915 App | `userday_details_915` | 1.9M |
| 13 | Context Push | Notifications | Context-aware push tables | Varies |
| 14 | User Financial Health | Wellness | Financial health tables | Varies |
| 15 | Datahub Navigation | Meta | DataHub entities | N/A |
| 16 | MF Analytics Extended | MF | Extended MF tables | Varies |

## Setup
- Proxy on port 3100 (macOS LaunchAgent, auto-starts on login)
- VPN required
- Supported clients: Claude Code, Cursor, Kilo Code, Cline, VS Code Copilot
- Setup: `./scripts/compass-mcp-setup.sh` or manual proxy + IDE config
- Support: **#dataplatform-helpdesk** Slack channel

## Key Guardrails
- Read-only (SELECT only)
- Partition enforcement on billion-row tables
- Timeout protection (default 120s, max 600s)
- Row limits (default 500, max 5000)
- Stale table prevention
- Date range capping (prevents future-date scans)

## Relationship to Content Engine
**Parallel system, not integrated.** Compass answers "what happened?" (analytics); Content Engine answers "what email should we send?" (content). Connection is indirect through a human.

## Official Documentation
- [[Deliverables/compass-mcp-product-doc|Product Overview]] (14 pages)
- [[Deliverables/analytics-skills-deep-dive|Analytics Skills Deep Dive]] (18 pages) — the definitive skill reference
- [[Deliverables/compass-setup-guide|Setup Guide]] (5 pages)

## Related
- [[Groww]]
- [[NARAD]]
- [[WebEngage]]
- [[Projects/compass-mcp-skills]]

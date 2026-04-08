---
title: "Analytics Skills Deep Dive — All 16 Skills Reference"
status: done
type: reference
sources:
  - "[[raw/compass-mcp-docs/Dataplatform · Analytics Skills Deep Dive - Coda.pdf]]"
author: Saurabh Dubey
date: 2026-04-04
---

# Analytics Skills Deep Dive

Source: Dataplatform Coda page (18 pages). The definitive reference for all 16 [[wiki/Compass-MCP|Compass MCP]] skills. Essential reading for skill enhancement work.

## What Each Skill Contains
- Tables overview (row counts, freshness, partition columns)
- Key columns reference (type, description, sample values)
- Common filter values (exact enumerations)
- 10+ example queries (production-ready SQL templates)
- Query strategy table ("if user asks X, use table Y with approach Z")
- Table relationships (join keys, cross-table navigation)
- Data quality notes (stale tables, naming inconsistencies, PII)
- Critical guardrails (partition requirements, row count warnings, timeout prevention)

## All 16 Skills Summary

| # | Skill | Domain | Key Tables | Table Sizes |
|---|---|---|---|---|
| 1 | Stocks & Trading | Equities | `core_stocks_order`, `fact_transaction_stocks` | 2.5B, 7.1B |
| 2 | F&O Analytics | Derivatives | `fno_order_master`, `fno_pnl_master` | 3.8B, 1.9B |
| 3 | Mutual Funds | MF | `fact_mutual_fund_order_details_enriched`, `view_mf_user_aum_latest` | Millions |
| 4 | User Master & Growth | Growth | `growth_user_master_ultimate`, `user_product_txn_master_fact` | 117M, 8.3B |
| 5 | User Events | Instrumentation | DataHub `userEvent` entities + `ems_raw` | 57B+ events |
| 6 | Onboarding | Activation | `onb_usercheckpoints_master`, `fact_onboarding_invest` | 117M, 45M |
| 7 | UPI Payments | Payments | `groww_upi_onb_user_fact`, `groww_upi_mandate_fact_table` | 66.9M, 27.2M |
| 8 | Engagement & Comms | Comms | `engagement_pn_narad_master`, `engagement_email_backend_master` | 250M/day, 3-6M |
| 9 | MTF & Pledge | Margin | `aay_mtf_book_agg`, `fact_pledge_transaction` | 243, 50M |
| 10 | Commodities | MCX | `commodity_day_view_seg`, `commodity_order_master` | 684K, 71.6M |
| 11 | Help & Support | Support | `hns_helpdesktickets_master_trino`, `hns_chatbot_fact_table_v3` | 12M, 135K |
| 12 | 915 Trading | 915 App | `userday_details_915`, `mis_central_915_ab` | 1.9M, 17 |
| 13 | Context Push | Notifications | Context-aware push tables | Varies |
| 14 | User Financial Health | Wellness | Financial health tables | Varies |
| 15 | Datahub Navigation | Meta | DataHub entities | N/A |
| 16 | MF Analytics Extended | MF | Extended MF tables | Varies |

## Cross-Skill Reference

### Schema Directory
| Schema | Tables | Domain |
|---|---|---|
| `platform_iceberg.core_invest` | Stocks, F&O, MF, MTF, Commodity, 915 | Trading & Investment |
| `platform_iceberg.core_bgv` | Growth, Onboarding, UPI, Engagement, H&S | User Lifecycle |
| `platform_iceberg.dwh_invest` | MF (enriched) | MF Data Warehouse |
| `platform_iceberg.dump_invest` | 915 cohorts | Periodic Snapshots |
| `platform_iceberg.ems` | Event data | Raw Events (Iceberg) |
| `platform_dp_pinot.default` | Event data | Sampled Events (Pinot) |

### Largest Tables (Must-Filter)
| Table | Rows | Skill | Must Filter? |
|---|---|---|---|
| `engagement_session_indepth` | 57B total | Engagement | `session_date` (1 day only!) |
| `user_product_txn_master_fact` | 8.3B | User Master | `order_date` |
| `fact_transaction_stocks` | 7.1B | Stocks | `order_date` |
| `fno_order_master` | 3.8B | F&O | `pt` |
| `core_stocks_order` | 2.5B | Stocks | `order_date` |
| `engagement_pn_narad_master` | 250M/day | Engagement | `event_date` |

### Stale Table Warnings
| Table | Last Data | Use Instead |
|---|---|---|
| `stocks_order` | Oct 2025 | `core_stocks_order` |
| `invest_user_fid_info` | Jun 2024 | `fact_onboarding_invest` FID columns |
| `daily_onb_conversions` | EMPTY | `daily_onboarding_and_fid` |
| `fno_usersegmentation_monthly` | Oct 2025 | Use order tables for current data |
| `groww_upi_debit_fact_table` | Feb 2024 | Historical only |
| `hns_freshchat_tickets_master_trino` | BROKEN | Do not use |
| `hns_csat_dashboard` | Mar 2024 | Historical CSAT only |
| `mtf_transactions_v2` | Nov 2024 | `aay_mtf_fid_agg` |

## Per-Skill Details

### Key Guardrails by Skill
- **Stocks**: Always filter `order_date` (2.5B rows). Prefer `core_stocks_order` over stale `stocks_order`. Never query `fact_transaction_stocks` >7 days without aggregation.
- **F&O**: `fno_order_master.pt` has future dates to 2090 — add `pt < date('2027-01-01')` or `is_tradingday = true`. PnL: `sell_value - buy_value` for longs, reverse for shorts.
- **MF**: Table selection matrix is critical (order volumes → `fact_mutual_fund_order_details_enriched`, AUM → `view_mf_user_aum_latest`, SIPs → `fact_mutual_fund_sip_details`).
- **User Master**: `user_product_txn_master_fact` is 8.3B — MUST filter on `order_date`. Product naming inconsistent: FID uses `cnc`/`sip`; txn_master uses `Stocks`/`Mf`. Prefer `growth_user_master_ultimate` over `growth_user_master`.
- **User Events**: Pinot is SAMPLED — use `SUM(boosting_factor * COUNT(*))` via CTE. Last 1-2 days broken (~1% accuracy). App filters by `app_package_name`.
- **Onboarding**: `esign_details` contains PII — never SELECT *. `fact_onboarding_fno` has 154 columns — select only needed funnel steps.
- **UPI**: Timestamps are epoch milliseconds — use `from_unixtime(col / 1000)`. Rolling window tables for recent data only.
- **Engagement**: `engagement_session_indepth` is 57B rows — SINGLE DAY queries only. PN Narad `os` column has mixed casing — always use `LOWER(os)`. DND `s7` is VARCHAR `'true'`/`'false'`, not boolean.
- **MTF**: Prefer `aay_mtf_fid_agg` (fresh daily) over `mtf_transactions_v2` (stale Nov 2024). `aay_mtf_market_share_data` values already in crores.
- **Commodities**: Prefer `commodity_day_view_seg` (684K, pre-aggregated) over `commodity_order_master` (71.6M, no partition). Timestamps epoch ms. Underlying naming varies: `GOLD` vs `GOLDM` (mini).
- **H&S**: `hns_freshchat_tickets_master_trino` is BROKEN. Chatbot is 3-day rolling window. Exotel is balanced sample (20 per category). Time fields in seconds not ms.
- **915**: Data starts June/July 2025. `user_cohorts_915` has 5.5M rows but 915 ATU only ~26K — inner join with `onb_915_master`. `mis_central_915_ab` has duplicates — use `WHERE refresh_month = month`.

## Full document: [[raw/compass-mcp-docs/Dataplatform · Analytics Skills Deep Dive - Coda.pdf]]

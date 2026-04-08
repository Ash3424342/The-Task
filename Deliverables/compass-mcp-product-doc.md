---
title: "Compass MCP — Product Documentation (Official)"
status: done
type: reference
sources:
  - "[[raw/compass-mcp-docs/Dataplatform · Compass MCP - BETA - Coda.pdf]]"
author: Saurabh Dubey
date: 2026-04-04
---

# Compass MCP — Official Product Doc

Source: Dataplatform Coda page (14 pages), authored by Saurabh Dubey.

## What Compass Is
A Model Context Protocol (MCP) server that connects AI assistants (Claude, Cursor, etc.) to [[wiki/Groww|Groww]]'s data infrastructure. Unifies **DataHub** (data catalog), **Superset** (visualization), and **Trino** (SQL engine) into a single conversational interface.

## 9 Hero Features

### 1. Domain-Specific Analytics Skills — "The Brain"
- 16 skills, pre-built knowledge packages
- Each skill: tables, partition columns, column semantics, filter values, query patterns, guardrails, stale table alerts, join keys
- Skills are auto-loaded when question matches domain

### 2. Trino SQL Querying — "The Engine"
- Run SQL directly from conversation
- Two engines: **Trino Direct** (`run_trino_query`, 5000 rows, 600s timeout) and **Superset SQL** (`superset > execute_sql`, 10000 rows, 300s timeout)
- Built-in guardrails: read-only, partition enforcement, timeout, row limits, stale table prevention, date range capping

### 3. Dashboards & Visualization — "The Face"
- Create charts conversationally: ASK → PREVIEW → ITERATE → SAVE → DASHBOARD → SHARE
- 12 chart types: Line, Bar, Stacked Bar, Horizontal Bar, Area, Scatter, Pie/Donut, Table, Interactive Table, Pivot Table, Mixed Timeseries
- Chart features: filters, group by, time grain, axis formatting, legend, stacking, sorting

### 4. Data Discovery & Catalog
- Search datasets, explore schemas, browse tables, find saved queries, search user events
- Tools: `search`, `list_schema_fields`, `get_entities`, `get_dataset_queries`, `get_database_schemas`, `get_database_tables`

### 5. Metadata Management
- Update descriptions, manage ownership, add tags, glossary terms, set domains, Superset tags
- Safe merge workflow: fetch → merge → review → write

### 6. Data Lineage
- Upstream/downstream with configurable depth (`max_hops`)
- Tool: `get_lineage`

### 7. Reports & Alerts
- Schedule reports (PNG, PDF, CSV, TEXT) via Email, Slack, Webhook with cron scheduling
- Data alerts with SQL-based thresholds

### 8. User Identity & Access
- Identity check, MCP token generation, instance info, health check

### 9. Architecture
```
Claude/Cursor → Compass MCP → DataHub (catalog)
                     ↓              ↓
               Superset (viz)    Trino (SQL)
```

## Summary of Tools (80+ capabilities)

| Category | Count |
|---|---|
| Analytics Skills | 16 |
| SQL Execution | 5 |
| Charts | 8 |
| Dashboards | 6 |
| Data Discovery | 6 |
| Metadata | 10 |
| Lineage | 1 |
| Reports & Alerts | 4 |
| Datasets | 4 |
| Databases | 3 |
| Tags (Superset) | 4 |
| Identity & Admin | 4 |

## Supported Clients
Claude Code (CLI), Cursor IDE, Kilo Code (VS Code), Cline (IntelliJ), VS Code Copilot — all fully supported.

## Full document: [[raw/compass-mcp-docs/Dataplatform · Compass MCP - BETA - Coda.pdf]]

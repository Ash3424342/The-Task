---
title: "CMP-2 — Context Push Analytics Compass Skill: Discovery Spec"
status: in-progress
type: deliverable
date: 2026-04-19
owner: me
task: "[[Tasks/CMP-2-context-push-skill]]"
---

# CMP-2 — Context Push Analytics Skill (Skill #13 of 16)

> **Ground-up build.** Unlike CMP-1 (Engagement & Comms, which had a known 9-table list), CMP-2 started with no confirmed table list. This spec documents the Prompt 1 discovery (2026-04-19 via live Compass MCP).
>
> **Relationship to CMP-1:** CMP-1 = *delivery events* (did the PN reach the user, was it clicked?). CMP-2 = *campaign orchestration* (what campaign is configured, which users were targeted, which PN did C-MAB decide to send, did the session attribute back to a comms channel?).

---

## Section 1: Catalog search results

| Query | Total results | Signal |
|---|---|---|
| "context push" | 6 | Low — matched `transaction_context` column, not push campaigns |
| "context_push" | 10,000 | Too broad — generic push-related tables across all schemas |
| "cep campaign" | 107 | **HIGH** — revealed `platform_iceberg.cep.*` and `platform_iceberg.cep_release.*` schemas |
| "narad campaign" | 151 | **HIGH** — revealed `derived_bgv.narad_campaign_type`, `invest_iceberg.growth.narad_campaign_metadata` |
| "push trigger eligibility" | 0 | None |
| "cmab" | 2,151 | **HIGH** — C-MAB lives in `platform_iceberg.ds_user_data.*` schema (2K+ tables, mostly Hive temp) |
| "comms attribution" | 67 | **HIGH** — revealed `dashboards_bgv.engagement_comms_attribution` (7.66B rows, live) |

---

## Section 2: Candidate table assessment

| Table (schema-qualified) | Total rows | Last updated | Confidence | Why |
|---|---|---|---|---|
| `platform_iceberg.cep.campaign_info` | 70,008 | 2026-04-18 12:31 (today!) | **HIGH** | Live campaign config master — MongoDB CDC, 6 commits/day. `campaignid` joins to PN narad. Contains rules, templates, schedule, state. |
| `platform_iceberg.dashboards_bgv.engagement_comms_attribution` | **7.66B** | 2026-04-18 04:24 | **HIGH** | Production attribution table linking msg_id → app session (cuid/suid). Partitioned by `session_date`. 2 commits/day. |
| `platform_iceberg.ds_user_data.cmab_user_store_v3` | 14.7M | 2026-02-26 | **MEDIUM** | C-MAB user feature store (85 cols). Last refresh 2 months ago — periodic ML batch table, not daily. No partition (COR). |
| `platform_iceberg.ds_user_data.cmab_interaction_logs` | 10.5M | 2025-12-01 | **MEDIUM** | C-MAB inference logs: user, pn_id_sent, time_slot, score, reward, tg_cg. Stale (4+ months). Historical training data. |
| `platform_iceberg.cep_release.generic_campaign_snapshot` | 346M | 2025-08-16 | **STALE** | Was the user-level campaign snapshot (per instanceid). Last commit Aug 2025 — superseded by new arch. |
| `invest_iceberg.growth.narad_campaign_metadata` | 232K | 2025-11-18 | **STALE** | Old campaign metadata in `invest_iceberg` catalog. Nov 2025 last snapshot. Replaced by `cep.campaign_info`. |
| `platform_iceberg.derived_bgv.narad_campaign_type` | 0 | 2025-07-10 | **EMPTY** | 0 rows, COR table. Empty/deprecated. |
| `platform_iceberg.cep_release.*` (all others) | 0-3 rows | 2025-06-20 | **STALE** | `campaign_7_master`, `generic_campaign_payload`, `generic_campaign_response`, `generic_campaign_user_metadata`, `journey_action_instance_meta` — all empty or near-zero. Legacy CEP release tables, superseded. |

**⚠ Critical discovery: `cep_release` schema is DEAD.** All production Context Push tables have migrated to the `cep` schema. Any queries against `cep_release` will return empty or stale results.

---

## Section 3: Per-table schemas (high-confidence tables)

### `platform_iceberg.cep.campaign_info` ✅ HIGH CONFIDENCE
**Schema home:** `platform_iceberg.cep` (live production CEP schema)
**Partition:** `is_partitioned=True`, partition_count=100 — partition column not directly exposed; likely on `campaignid` or internal key
**Rows:** 70,008 | **Last updated:** 2026-04-18 12:31 | **Cadence:** 6 commits/day
**Source:** MongoDB CDC (Kafka topic: `mongo-db-campaign-service-v4.dataplatform.dataplatform.campaign`)
**Join key:** `campaignid` (bigint) — maps to `engagement_pn_narad_master.campaignid` (varchar — may need CAST)

| Column | Type | Description |
|---|---|---|
| `campaignid` | bigint | Campaign identifier — joins to PN narad `campaignid` (note: varchar in PN narad, bigint here — CAST needed) |
| `name` | varchar | Human-readable campaign name (e.g., "17Apr'26 - Stock - Signup - Educational") |
| `campaigntype` | varchar | Type: GENERIC, JOURNEY, etc. |
| `campaigncategory` | varchar | Category classification |
| `state` | varchar | Campaign state: LIVE, DRAFT, PAUSED, ARCHIVED |
| `isapproved` | boolean | Whether campaign is approved |
| `current` | boolean | True = this is the current/active version of the campaign |
| `notificationdeliverypriority` | varchar | Delivery priority |
| `productentity` | varchar | Product vertical (Stocks, MF, F&O, Credit, etc.) |
| `org` | varchar | Organization/team |
| `tags` | array(varchar) | Campaign tags |
| `createdby` | varchar | Creator email |
| `createdat` | row(_date bigint) | Creation timestamp (epoch ms in nested struct) |
| `modifiedat` | row(_date bigint) | Last modified timestamp |
| `lastapprovedat` | row(_date bigint) | Approval timestamp |
| `lastliveat` | row(_date bigint) | Last went live timestamp |
| `version` | bigint | Version number of campaign config |
| `doc_id` | varchar | MongoDB document ID |
| `rules` | array(row(...)) | Targeting eligibility rules: attributeName, operator, values, attributeSpace |
| `exittriggers` | array(row(...)) | Conditions to exit campaign journey |
| `schedule` | row(...) | Schedule config: cronExpression, startTime, endTime, triggers (event-based) |
| `actions` | array(row(...)) | Action configs per channel: notificationTemplate (push), emailTemplate, whatsappTemplate, feedTemplate, storyTemplate, rewardTemplate |
| `dagview` | row(...) | Visual DAG representation of campaign journey (nodes, edges) |

**Key insight:** `cep.campaign_info` is the **source of truth for all campaign metadata**. The `name` column here directly matches the `campaign_name` in `engagement_pn_narad_master`. The `rules` array contains the targeting logic (which users qualify for this campaign). The `actions` array contains template content (what push body/title was sent).

**⚠ Guardrail:** `createdat`, `modifiedat` etc. are stored as `row(_date bigint)` — the actual epoch ms is nested. Use `createdat._date` to access the timestamp value.

---

### `platform_iceberg.dashboards_bgv.engagement_comms_attribution` ✅ HIGH CONFIDENCE
**Schema home:** `platform_iceberg.dashboards_bgv`
**Partition:** `session_date` (date) — confirmed, partition_count=1,566
**Rows:** 7.66B | **Last updated:** 2026-04-18 04:24 | **Cadence:** 2 commits/day
**Join keys:** `cuid` (UUID), `suid` (session UUID), `msg_id` (links to PN/SMS/WA msg_id)

| Column | Type | Description |
|---|---|---|
| `session_date` | date | **Partition column.** Date of the attributed session |
| `session_event_time` | timestamp(6) tz | Exact timestamp of the session event |
| `cuid` | varchar | User CUID — joins to `engagement_session_indepth.cuid` and `engagement_dau_indepth.cuid` |
| `suid` | varchar | Session UUID — joins to `engagement_session_indepth.suid` |
| `event_name` | varchar | App event that opened the attributed session |
| `msg_id` | varchar | Message ID of the communication that drove this session — joins to `engagement_pn_narad_master.msg_id` |
| `source` | varchar | Communication source (EMS, NARAD, etc.) |
| `campaign_tag` | varchar | Campaign tag from the original communication |
| `campaign_type` | varchar | Campaign type |

**Key insight:** This is the **attribution bridge** — it links a communication (msg_id → PN delivery event) to an app session (cuid + suid → session_indepth). This answers "did this push cause the user to open the app?". The `comm_session` column in `engagement_session_indepth` carries the same attribution info as a CEP string, but this table provides it in a normalized, queryable form.

**⚠ Important: 7.66B rows** — always filter by `session_date`. Mandatory single-day filter.

---

### `platform_iceberg.ds_user_data.cmab_user_store_v3` ⚠ MEDIUM (periodic)
**Schema home:** `platform_iceberg.ds_user_data`
**Partition:** NONE (`is_partitioned=False`, COR table)
**Rows:** 14.7M | **Last updated:** 2026-02-26 | **Cadence:** Periodic ML batch (not daily)
**Join key:** `user_account_id` (varchar)

Key columns (85 total):
| Column | Type | Description |
|---|---|---|
| `snap_date` | date | Date of snapshot |
| `user_account_id` | varchar | User join key |
| `third_party_id` | varchar | Push token / FCM ID |
| `is_stocks_invested` / `is_mf_invested` / `is_fno_invested` | integer | Product investment flags |
| `days_since_fid` / `days_since_*_onboarding` | bigint | Days since key events |
| `cnc_trades_l90d`, `mis_trades_l90d`, `fno_trades_l90d` | bigint | Trading activity L90D |
| `sessions_l3d/7d/30d`, `dau_l3d/7d/30d` | bigint | Session engagement features |
| `comms_session_ratio` | decimal | % of sessions driven by communications |
| `intraday_ratio`, `options_ratio`, etc. | decimal | Trading style features |
| `large_cap_value_ratio`, `mid_cap_value_ratio`, `small_cap_value_ratio` | double | Portfolio composition |

**Key insight:** This is the **feature store for C-MAB** — user behavioral features used as context for the contextual bandit to decide which PN campaign to show. Input to the C-MAB model. Refreshed every few weeks, not daily.

---

### `platform_iceberg.ds_user_data.cmab_interaction_logs` ⚠ MEDIUM (stale)
**Schema home:** `platform_iceberg.ds_user_data`
**Partition:** NONE (`is_partitioned=False`, COR table)
**Rows:** 10.5M | **Last updated:** 2025-12-01 (4+ months ago) | **Cadence:** Batch ML job
**Join key:** `user_id` (varchar — unclear if this is user_account_id or cuid)
**Note:** Catalog description says "stub for AM upstream lineage" — this may be a schema-only stub with data elsewhere

| Column | Type | Description |
|---|---|---|
| `log_id` | varchar | Unique log ID |
| `prediction_timestamp` | timestamp(6) | When C-MAB made the prediction |
| `user_id` | varchar | User identifier (format TBD — ACC? or UUID?) |
| `pn_id_sent` | varchar | Push notification ID sent (links to msg_id?) |
| `time_slot` | varchar | PN time slot (slot1, slot2, slot3, slot4 — 4 daily slots) |
| `model_version` | varchar | C-MAB model version |
| `score` | double | Model score for the selected candidate |
| `context_vector` | varchar | JSON serialized context features |
| `context_vector_hash` | varchar | Hash of context vector |
| `candidates_considered` | varchar | All candidates C-MAB evaluated |
| `scores_all_candidates` | varchar | Scores for all candidates |
| `feedback_timestamp` | varchar | When feedback (click/no-click) was received |
| `reward` | varchar | Reward signal: '1' = clicked, '0' = not clicked |
| `tg_cg` | varchar | Treatment/Control group assignment |

---

## Section 4: Schema home for Context Push

| Schema | Status | Tables |
|---|---|---|
| `platform_iceberg.cep` | **PRIMARY — LIVE** | `campaign_info` (70K rows, updated daily) |
| `platform_iceberg.dashboards_bgv` | **LIVE** | `engagement_comms_attribution` (7.66B rows) |
| `platform_iceberg.ds_user_data` | **ML batch** | `cmab_user_store_v3`, `cmab_interaction_logs` |
| `platform_iceberg.cep_release` | **DEAD/STALE** | All tables empty or Aug 2025 last updated |
| `invest_iceberg.growth` | **STALE** | `narad_campaign_metadata` (Nov 2025) |

**Primary schema home: `platform_iceberg.cep`** (live production CEP service CDC tables)
**Attribution analytics: `platform_iceberg.dashboards_bgv`**
**ML/C-MAB: `platform_iceberg.ds_user_data`**

---

## Section 5: Architecture insight

Context Push = **CEP** (Campaign Execution Platform). The CEP service manages campaigns (configured in MongoDB → replicated to `cep.campaign_info`). When a campaign fires:

```
CEP decides target users
   ↓ (uses rules from cep.campaign_info)
C-MAB selects which PN to send per user
   ↓ (cmab_user_store_v3 provides features)
NARAD dispatches the PN
   ↓ (engagement_pn_narad_master records delivery)
User opens app → attributed to comm
   ↓ (engagement_comms_attribution links msg_id → session)
Session recorded
   ↓ (engagement_session_indepth.comm_session = "CEP-{instanceId}-...")
```

The `CEP-{campaignInstanceId}-{bucket}-CAMPAIGN-PRODUCTION-{userId}-{counter}` format in `comm_session` uses `instanceId` = the campaign instance numeric ID (e.g., "84316"), NOT `campaignid`. There may be a `cep.campaign_instance` table not yet found.

---

## Section 6: Open questions for Prompt 2

1. **Campaign instance tracking:** `comm_session` uses `CEP-{campaignInstanceId}` but `cep.campaign_info` has `campaignid`. There must be an instance table — search for `cep.campaign_instance` or similar in a Trino query on the `cep` schema.

2. **Type mismatch:** `cep.campaign_info.campaignid` = bigint; `engagement_pn_narad_master.campaignid` = varchar. Confirm join works: `CAST(ci.campaignid AS varchar) = pn.campaignid`.

3. **`cmab_interaction_logs.user_id`:** Is this `user_account_id` (ACC...) or `cuid` (UUID)? Needs sample data to confirm.

4. **Live C-MAB table:** `cmab_user_store_v3` is 2 months stale. Is there a v4 or a daily-refresh CMAB feature table? Search for `cmab_user_store_v4` or `cmab_pn_feature_store`.

5. **`engagement_comms_attribution.msg_id` join:** Does this match `engagement_pn_narad_master.msg_id` directly? Or is there a transformation?

6. **`cep.campaign_info` partition column:** `is_partitioned=True` with 100 partitions but the partition column isn't visible in the schema. Need a Trino `SHOW PARTITIONS` or sample query to confirm.

7. **`state` enum values** in `cep.campaign_info`: What are the valid campaign states? Expected: LIVE, DRAFT, PAUSED, ARCHIVED — confirm via sample.

8. **`campaigntype` enum values**: Expected: GENERIC, JOURNEY — confirm.

---

## Section 7: Confirmed CMP-2 scope (working table list)

| # | Table | Schema home | Rows | Status |
|---|---|---|---|---|
| 1 | `cep.campaign_info` | `platform_iceberg.cep` | 70K | ✅ LIVE — hero table |
| 2 | `dashboards_bgv.engagement_comms_attribution` | `platform_iceberg.dashboards_bgv` | 7.66B | ✅ LIVE — hero table |
| 3 | `ds_user_data.cmab_user_store_v3` | `platform_iceberg.ds_user_data` | 14.7M | ⚠️ Periodic refresh (Feb 2026) |
| 4 | `ds_user_data.cmab_interaction_logs` | `platform_iceberg.ds_user_data` | 10.5M | ⚠️ Stale (Dec 2025) — may be superseded |

**Next step (Prompt 2):** Sample data + enum validation for these 4 tables, plus search for missing campaign instance table.

---

## 10. Prompt 2 results — Sample data + enum validation [UNVERIFIED, 2026-04-19]

*All queries attempted via live Compass MCP Trino. Date: 2026-04-19.*

---

### §10.1 Section 1: Per-table samples + enum values

**CRITICAL FINDING: All 4 CMP-2 tables are Trino-access-denied.**

| Table | Trino access | Error |
|---|---|---|
| `platform_iceberg.cep.campaign_info` | ❌ DENIED | `Access Denied: Cannot select from table platform_iceberg.cep.campaign_info` |
| `platform_iceberg.dashboards_bgv.engagement_comms_attribution` | ❌ DENIED | `Access Denied: Cannot select from table platform_iceberg.dashboards_bgv.engagement_comms_attribution` |
| `platform_iceberg.ds_user_data.cmab_user_store_v3` | ❌ DENIED | `Access Denied: Cannot select from table platform_iceberg.ds_user_data.cmab_user_store_v3` |
| `platform_iceberg.ds_user_data.cmab_interaction_logs` | ❌ DENIED | `Access Denied: Cannot select from table platform_iceberg.ds_user_data.cmab_interaction_logs` |

**Contrast with CMP-1:** All `platform_iceberg.core_bgv.*` tables (9 engagement tables) were freely queryable via this MCP token. CMP-2 tables span three different schemas (`cep`, `dashboards_bgv`, `ds_user_data`) that have elevated access controls. The MCP Trino token has read access to `core_bgv` but not to these schemas.

**Implication for skill:** SQL examples for CMP-2 tables are **theoretical** — they cannot be live-run and confirmed by standard analytics users via this Trino connection. The skill must document this access requirement explicitly.

---

### §10.2 Section 2: Future-date check

Not executable via Trino (access denied). From catalog metadata:

| Table | Partition col | Last snapshot | Status |
|---|---|---|---|
| `cep.campaign_info` | unknown (100 partitions) | 2026-04-18 12:31 | ✅ fresh — no future-date concern (it's a config table, not event-based) |
| `dashboards_bgv.engagement_comms_attribution` | `session_date` | 2026-04-18 04:24 | ✅ fresh |
| `ds_user_data.cmab_user_store_v3` | none (COR) | 2026-02-26 | ⚠️ 52 days old |
| `ds_user_data.cmab_interaction_logs` | none (COR) | 2025-12-01 | ⚠️ 139 days old |

---

### §10.3 Section 3: Trigger lineage (confirmed via PN narad)

`platform_iceberg.core_bgv.engagement_pn_narad_master` IS accessible. Using it as a proxy:

```sql
SELECT campaignid, campaign_name, campaign_tag, productentity,
       COUNT(*) AS pn_attempts
FROM platform_iceberg.core_bgv.engagement_pn_narad_master
WHERE event_date = current_date - interval '1' day
GROUP BY campaignid, campaign_name, campaign_tag, productentity
ORDER BY pn_attempts DESC LIMIT 10
```

**✅ Confirmed:** `campaignid` in PN narad is **varchar** (e.g., "33982", "37401"). Top campaigns:

| campaignid | campaign_name | campaign_tag | productentity | pn_attempts |
|---|---|---|---|---|
| "33982" | 03 Oct'25 - Credit - RPL - Low Freq. Targeting - Megh | credit - teleport - rpl - low freq | Personal Loans | 8,839,349 |
| "37401" | 17Apr'26 - Stock - Signup - Educational | stocks educational | Stocks | 7,919,378 |
| "36624" | Credit - Teleport - PA, MLR, Repeat - No Credit comms. received L7D | credit - teleport - pa - comms not received | Personal Loans | 6,442,174 |
| "37400" | 17Apr'26 - Stock - NTU PA & Onb - Educational | stocks educational | Stocks | 5,569,546 |
| "32375" | MF - NTU PA - Value props | mf value prop | MF | 4,929,207 |

**Join formula confirmed:** `CAST(ci.campaignid AS varchar) = pn.campaignid` — bigint in `cep.campaign_info` to varchar in PN narad. No direct Trino join tested (access denied on CEP side), but the mapping is structurally sound.

**Campaign tag taxonomy observed from PN narad:**
- `credit - teleport - *` — Credit/personal loans campaigns
- `stocks educational` — Educational stock content
- `f&o educational` — F&O educational  
- `mf value prop` — MF value proposition
- `groww_amc_*` — AMC/NFO campaigns
- `stocks opening bell`, `pers - price alert - *` — DMA/personalized stock alerts

---

### §10.4 Section 4: New guardrails discovered (Prompt 2)

| ID | Table | Guardrail |
|---|---|---|
| G1 | All CMP-2 tables | **Schema-level Trino access restriction.** `cep`, `dashboards_bgv`, `ds_user_data` schemas require elevated permissions not available via standard MCP token. SQL examples in the skill are catalog-confirmed but Trino-unrunnable for standard users. |
| G2 | `cep.campaign_info` | **Type mismatch on join:** `cep.campaign_info.campaignid` = bigint; `engagement_pn_narad_master.campaignid` = varchar. Always use `CAST(ci.campaignid AS varchar) = pn.campaignid`. |
| G3 | `cep.campaign_info` | **MongoDB CDC source** (Kafka: `mongo-db-campaign-service-v4`) — table is a CDC dump, not a traditional analytics table. Updates are incremental (change events), not daily batch. The `current = true` filter is mandatory to get latest version of each campaign. |
| G4 | `cep.campaign_info` | **Nested struct timestamps:** `createdat`, `modifiedat`, `lastliveat` etc. are `row(_date bigint)` — the epoch ms is at `createdat._date`, not directly at the column. Use: `createdat._date` to access. |
| G5 | `ds_user_data.cmab_user_store_v3` | **Stale (52 days).** Last refresh Feb 2026. Not suitable for daily analytics. Use as reference/ML feature store, not current state. |
| G6 | `ds_user_data.cmab_interaction_logs` | **Very stale (139 days).** Last refresh Dec 2025. May be superseded by a newer CMAB logging table. |
| G7 | All CMP-2 hero tables | **CMP-2 is catalog-only** — schemas confirmed via DataHub (list_schema_fields), but live Trino validation impossible without elevated access. Skill documentation is schema-accurate but SQL examples cannot be verified as runnable by standard analytics users. |

---

### §10.5 Resolution path

Since Trino access is denied, the CMP-2 skill can still be built at **catalog quality** (not Trino-confirmed quality):

1. **Schemas:** Confirmed via `list_schema_fields` ✅
2. **Row counts + freshness:** Confirmed via DataHub `datasetProperties` ✅  
3. **Campaign linkage:** Confirmed via PN narad proxy ✅
4. **Sample data / enums / SQL tests:** ❌ not possible without access grant

**Recommended path forward:** Request Trino access to `cep`, `dashboards_bgv`, and `ds_user_data` schemas. Document the access request in the task. Meanwhile, write the skill at catalog quality — it will still be useful for LLM context even without live SQL examples.

**Alternative:** Check if `core_bgv` has any derived views or summary tables from CEP/attribution data that are accessible. The `comms_attribution` data from session_indepth's `comm_session` column is in `core_bgv` and IS accessible — this can serve as a partial substitute.

CMP-2 is **done** when:

- [ ] All confirmed tables documented with schema, partition, row scale, refresh cadence
- [ ] `cep.campaign_info` campaign type/state enums confirmed via live Trino query
- [ ] `cep.campaign_info.campaignid ↔ engagement_pn_narad_master.campaignid` join confirmed (type cast)
- [ ] `engagement_comms_attribution.msg_id ↔ engagement_pn_narad_master.msg_id` join confirmed
- [ ] Campaign instance ID mystery resolved (what table has instanceId = 84316 etc.)
- [ ] CMAB user_id column format confirmed (ACC... or UUID)
- [x] All confirmed tables discovered with schema, rows, freshness (✅ catalog-quality via DataHub)
- [~] SQL examples — see §11 (theoretical for restricted tables; working alternatives via core_bgv)
- [ ] Trino access grant needed: request access to `cep`, `dashboards_bgv`, `ds_user_data` schemas
- [ ] Wiki `compass-analytics-skills.md` Skill #13 updated
- [ ] Project doc row 13 updated

---

## 11. SQL examples — catalog-quality (2026-04-19) [UNVERIFIED — Trino access denied]

> **Access status:** `cep`, `dashboards_bgv`, `ds_user_data` schemas are Trino-restricted. Queries below are schema-confirmed (columns/types verified via DataHub) but **cannot be run** via the standard MCP Trino token. They will work once elevated access is granted.
>
> **Workaround:** The final 2 queries use only `core_bgv` tables (fully accessible) and approximate the same answers via `comm_session` in `engagement_session_indepth`.

---

### cep.campaign_info — 5 queries (schema-confirmed, Trino-restricted)

```sql
-- Q1: All live campaigns with their product and category
-- ⚠️ REQUIRES cep schema access
SELECT campaignid, name, campaigntype, campaigncategory,
       productentity, state, notificationdeliverypriority,
       isapproved, current, version
FROM platform_iceberg.cep.campaign_info
WHERE current = true
  AND state = 'LIVE'
ORDER BY campaignid DESC
LIMIT 50;

-- Q2: Campaign by product entity and type (what's running today)
-- ⚠️ REQUIRES cep schema access
SELECT productentity, campaigntype, campaigncategory,
       COUNT(*) AS campaign_count
FROM platform_iceberg.cep.campaign_info
WHERE current = true
  AND state = 'LIVE'
GROUP BY productentity, campaigntype, campaigncategory
ORDER BY campaign_count DESC;

-- Q3: Look up campaign metadata for a known campaign_id from PN narad
-- ⚠️ REQUIRES cep schema access
SELECT campaignid, name, campaigncategory, productentity,
       state, notificationdeliverypriority,
       createdat._date AS created_epoch_ms,
       lastliveat._date AS last_live_epoch_ms
FROM platform_iceberg.cep.campaign_info
WHERE current = true
  AND campaignid = 37401;
-- Note: campaignid is bigint here; in PN narad it's varchar '37401'

-- Q4: Campaigns recently approved (last 7 days)
-- ⚠️ REQUIRES cep schema access
SELECT campaignid, name, productentity, campaigntype, state,
       lastapprovedat._date AS approved_epoch_ms
FROM platform_iceberg.cep.campaign_info
WHERE current = true
  AND lastapprovedat._date > (CAST(current_date - interval '7' day AS timestamp) - TIMESTAMP '1970-01-01 00:00:00') * 1000
ORDER BY lastapprovedat._date DESC
LIMIT 20;

-- Q5: Join campaign config to delivery events (requires both schemas)
-- ⚠️ REQUIRES BOTH cep AND core_bgv access
SELECT ci.name AS campaign_name, ci.campaigncategory, ci.productentity,
       SUM(CASE WHEN pn.status = 'SENT' THEN 1 ELSE 0 END) AS sent,
       SUM(CASE WHEN pn.click_time IS NOT NULL THEN 1 ELSE 0 END) AS clicked,
       ROUND(SUM(CASE WHEN pn.click_time IS NOT NULL THEN 1 ELSE 0 END) * 100.0
             / NULLIF(SUM(CASE WHEN pn.status = 'SENT' THEN 1 ELSE 0 END), 0), 3) AS ctr_pct
FROM platform_iceberg.cep.campaign_info ci
JOIN platform_iceberg.core_bgv.engagement_pn_narad_master pn
  ON CAST(ci.campaignid AS varchar) = pn.campaignid
WHERE ci.current = true
  AND pn.event_date = current_date - interval '1' day
GROUP BY ci.name, ci.campaigncategory, ci.productentity
ORDER BY sent DESC
LIMIT 20;
```

---

### dashboards_bgv.engagement_comms_attribution — 5 queries (schema-confirmed, Trino-restricted)

```sql
-- Q1: PN attribution rate — what % of sent PNs drove a session?
-- ⚠️ REQUIRES dashboards_bgv schema access
SELECT source, campaign_type,
       COUNT(DISTINCT msg_id) AS attributed_msgs,
       COUNT(DISTINCT cuid) AS attributed_users
FROM platform_iceberg.dashboards_bgv.engagement_comms_attribution
WHERE session_date = current_date - interval '1' day
GROUP BY source, campaign_type
ORDER BY attributed_msgs DESC
LIMIT 20;

-- Q2: Attribution within N minutes of PN send — (join comms_attribution to PN narad)
-- ⚠️ REQUIRES dashboards_bgv AND core_bgv access
SELECT pn.campaign_name, pn.campaign_tag,
       COUNT(DISTINCT attr.cuid) AS attributed_users,
       COUNT(DISTINCT attr.suid) AS attributed_sessions
FROM platform_iceberg.dashboards_bgv.engagement_comms_attribution attr
JOIN platform_iceberg.core_bgv.engagement_pn_narad_master pn
  ON attr.msg_id = pn.msg_id
WHERE attr.session_date = current_date - interval '1' day
  AND pn.event_date = current_date - interval '1' day
GROUP BY pn.campaign_name, pn.campaign_tag
ORDER BY attributed_sessions DESC
LIMIT 20;

-- Q3: Session events triggered by communications (what users do after clicking PN)
-- ⚠️ REQUIRES dashboards_bgv schema access
SELECT event_name, source, COUNT(*) AS attributed_sessions
FROM platform_iceberg.dashboards_bgv.engagement_comms_attribution
WHERE session_date = current_date - interval '1' day
GROUP BY event_name, source
ORDER BY attributed_sessions DESC
LIMIT 20;

-- Q4: Attribution trend over 7 days
-- ⚠️ REQUIRES dashboards_bgv schema access
SELECT session_date, source,
       COUNT(DISTINCT msg_id) AS attributed_msgs,
       COUNT(DISTINCT cuid) AS unique_users
FROM platform_iceberg.dashboards_bgv.engagement_comms_attribution
WHERE session_date BETWEEN current_date - interval '7' day AND current_date - interval '1' day
GROUP BY session_date, source
ORDER BY session_date, attributed_msgs DESC;

-- Q5: Top campaigns by attribution (sessions driven by each campaign)
-- ⚠️ REQUIRES dashboards_bgv schema access
SELECT campaign_tag, campaign_type,
       COUNT(DISTINCT cuid) AS unique_attributed_users,
       COUNT(*) AS total_attributed_sessions
FROM platform_iceberg.dashboards_bgv.engagement_comms_attribution
WHERE session_date = current_date - interval '1' day
GROUP BY campaign_tag, campaign_type
ORDER BY unique_attributed_users DESC
LIMIT 20;
```

---

### Working alternatives via core_bgv (fully accessible ✅)

These queries use `engagement_session_indepth.comm_session` to approximate attribution analytics without needing `dashboards_bgv` access.

```sql
-- ALT-1: Sessions attributed to any PN campaign (from comm_session column)
-- ✅ RUNS — uses core_bgv only; confirms trigger lineage
SELECT COUNT(*) AS pn_attributed_sessions,
       COUNT(DISTINCT cuid) AS unique_attributed_users
FROM platform_iceberg.core_bgv.engagement_session_indepth
WHERE session_date = current_date - interval '1' day
  AND comm_session IS NOT NULL
  AND comm_session LIKE 'CEP-%';
-- comm_session format: CEP-{instanceId}-{bucket}-CAMPAIGN-PRODUCTION-{userId}-{counter}

-- ALT-2: Extract campaign instance ID from comm_session and join to PN campaign name
-- ✅ RUNS — parses CEP attribution from comm_session
SELECT SUBSTR(comm_session, 5, STRPOS(comm_session, '-', 5) - 5) AS campaign_instance_id,
       COUNT(*) AS attributed_sessions,
       COUNT(DISTINCT cuid) AS unique_users
FROM platform_iceberg.core_bgv.engagement_session_indepth
WHERE session_date = current_date - interval '1' day
  AND comm_session LIKE 'CEP-%'
GROUP BY SUBSTR(comm_session, 5, STRPOS(comm_session, '-', 5) - 5)
ORDER BY attributed_sessions DESC
LIMIT 20;

-- ALT-3: Top campaigns by PN volume (accessible proxy for cep.campaign_info)
-- ✅ RUNS — uses core_bgv only
SELECT campaignid, campaign_name, campaign_tag, productentity,
       COUNT(*) AS total_attempts,
       SUM(CASE WHEN status = 'SENT' THEN 1 ELSE 0 END) AS sent,
       SUM(CASE WHEN click_time IS NOT NULL THEN 1 ELSE 0 END) AS clicked,
       ROUND(SUM(CASE WHEN click_time IS NOT NULL THEN 1 ELSE 0 END) * 100.0
             / NULLIF(SUM(CASE WHEN status = 'SENT' THEN 1 ELSE 0 END), 0), 3) AS ctr_pct
FROM platform_iceberg.core_bgv.engagement_pn_narad_master
WHERE event_date = current_date - interval '1' day
GROUP BY campaignid, campaign_name, campaign_tag, productentity
ORDER BY sent DESC LIMIT 20;
```

---

### Access request (action item)

**Request Trino access for:** `platform_iceberg.cep`, `platform_iceberg.dashboards_bgv`, `platform_iceberg.ds_user_data`

**Requester:** MCP Trino token (the token used by the Compass MCP server on port 3100)

**Reason:** CMP-2 Context Push Analytics skill build requires sample queries and enum validation on CEP campaign tables. Current token has `core_bgv` access only.

**Contact:** Data Platform team (Saurabh Dubey per wiki) — whoever manages Trino row-level/schema-level ACLs.

Once granted: re-run Prompts 2 + 3 from `Manager/CMP-2-Compass-Prompts.md` to complete live validation.

---

## 12. Prompt 3 results — Cross-skill joins + hero SQL + stale sweep [UNVERIFIED, 2026-04-19]

*All queries via live Compass MCP Trino. Date used: `current_date - interval '1' day` = 2026-04-18 (Saturday). All run on accessible core_bgv tables only — CMP-2 tables remain access-denied.*

---

### §12.1 Section A: Cross-skill join validations

**Join 1 (CMP-2 ⨝ growth_user_master_ultimate):** ❌ All CMP-2 tables Trino-access-denied. Not runnable.

**Join 2 (CMP-2 ⨝ PN narad via campaign instance):**
- Direct join (`SPLIT_PART(comm_session,'-',2) = pn.campaigninstanceid` on 60M × 249M rows): ❌ **TIMED OUT** (300s)
- Two-step approach ✅: (1) aggregate comm_session to get top instanceIds → (2) look up PN narad with IN filter

```sql
-- Step 1: extract top instance IDs from comm_session
SELECT SPLIT_PART(comm_session, '-', 2) AS campaign_instance_id,
       COUNT(*) AS attributed_sessions
FROM platform_iceberg.core_bgv.engagement_session_indepth
WHERE session_date = current_date - interval '1' day
  AND comm_session LIKE 'CEP-%'
GROUP BY SPLIT_PART(comm_session, '-', 2)
ORDER BY attributed_sessions DESC LIMIT 20;

-- Step 2: look up campaign metadata for those instance IDs
SELECT campaigninstanceid, campaign_name, campaign_tag, productentity,
       COUNT(*) AS pn_sent
FROM platform_iceberg.core_bgv.engagement_pn_narad_master
WHERE event_date = current_date - interval '1' day
  AND campaigninstanceid IN ('84496','84479','84476','84474','84470','84471','84475','84490')
  AND status = 'SENT'
GROUP BY campaigninstanceid, campaign_name, campaign_tag, productentity
ORDER BY pn_sent DESC;
```

**Combined result (top campaigns by attributed sessions, 2026-04-18):**

| campaign_instance_id | campaign_name | campaign_tag | attributed_sessions | pn_sent | attribution_rate |
|---|---|---|---|---|---|
| 84496 | 03 Oct'25 - Credit - RPL - Low Freq. Targeting - Megh | credit - teleport - rpl - low freq | 124,387 | 4,594,807 | 2.7% |
| 84479 | 18 Apr'26 - AMC - Arbitrage fund NFO - Mid | groww_amc_Arbitrage_Fund_nfo | 116,753 | 1,351,039 | **8.6%** |
| 84476 | Credit - All Pa Users Teleport - Saturday - Farhan | credit - pa - all propensity - prism | 95,874 | 3,521,258 | 2.7% |
| 84474 | Credit - Teleport - PA, MLR, Repeat | credit - teleport - pa - comms not received | 59,032 | 4,393,214 | 1.3% |
| 84470 | 17Apr'26 - Stock - NTU PA & Onb - Educational | stocks educational | 26,733 | 4,469,948 | 0.6% |
| 84471 | 17Apr'26 - Stock - Signup - Educational | stocks educational | 15,560 | 5,481,931 | 0.3% |

**Join key confirmed:** `SPLIT_PART(comm_session, '-', 2)` = `campaigninstanceid` in PN narad — both varchar, no type coercion. Join 84452 not found in PN narad (numeric-only instanceids match; CAMP-INSTANCE-* format does not).

**Join 3 (session_indepth ⨝ dau_indepth on cuid):** ✅ confirmed — both core_bgv, clean cuid UUID join.

---

### §12.2 Section B: Hero SQL results (5 queries via comm_session proxy)

All queries use `platform_iceberg.core_bgv.engagement_session_indepth` — the only accessible attribution table.

**Q1: Total PN-attributed sessions and users yesterday** ✅ 1 row

```sql
SELECT COUNT(*) AS pn_attributed_sessions,
       COUNT(DISTINCT cuid) AS unique_attributed_users
FROM platform_iceberg.core_bgv.engagement_session_indepth
WHERE session_date = current_date - interval '1' day
  AND comm_session IS NOT NULL AND comm_session LIKE 'CEP-%'
```
**Result:** 523,874 sessions | 463,862 unique users (2026-04-18, Saturday)

---

**Q2: Top campaigns by attributed sessions (two-step via PN narad lookup)** ✅ 9 rows (lookup step)

See §12.1 above. Best performer: AMC Arbitrage NFO at 8.6% attribution rate (vs 0.3-2.7% for others).

---

**Q3: Order conversion among PN-attributed sessions** ✅ 1 row

```sql
SELECT SUM(CASE WHEN orders > 0 THEN 1 ELSE 0 END) AS order_sessions,
       COUNT(*) AS total_cep_sessions,
       ROUND(SUM(CASE WHEN orders > 0 THEN 1 ELSE 0 END)*100.0/COUNT(*),2) AS order_conversion_pct
FROM platform_iceberg.core_bgv.engagement_session_indepth
WHERE session_date = current_date - interval '1' day
  AND comm_session LIKE 'CEP-%'
```
**Result:** 12 order sessions / 523,874 total → **0.00% order conversion** (Saturday — market closed, no stock orders).

---

**Q4: TTU vs NTU split among PN-attributed users** ✅ 2 rows

```sql
SELECT d.ttu_flag,
       COUNT(DISTINCT s.cuid) AS unique_users,
       ROUND(COUNT(DISTINCT s.cuid)*100.0/SUM(COUNT(DISTINCT s.cuid)) OVER (),2) AS pct
FROM platform_iceberg.core_bgv.engagement_session_indepth s
JOIN platform_iceberg.core_bgv.engagement_dau_indepth d
  ON s.cuid = d.cuid
WHERE s.session_date = current_date - interval '1' day
  AND d.session_date = current_date - interval '1' day
  AND s.comm_session LIKE 'CEP-%'
GROUP BY d.ttu_flag ORDER BY unique_users DESC
```
**Result:** TTU (ttu_flag=1): **402,388 users (86.7%)** | NTU: 61,474 users (13.3%)  
CEP campaigns are heavily TTU-targeted — makes sense for re-engagement / cross-sell push.

---

**Q5: 7-day trend of PN-attributed sessions** ✅ 7 rows (no timeout — LIKE filter reduces scan efficiently)

```sql
SELECT session_date, COUNT(*) AS cep_attributed_sessions,
       COUNT(DISTINCT cuid) AS unique_users
FROM platform_iceberg.core_bgv.engagement_session_indepth
WHERE session_date BETWEEN current_date - interval '7' day AND current_date - interval '1' day
  AND comm_session LIKE 'CEP-%'
GROUP BY session_date ORDER BY session_date
```

| Date | CEP sessions | Unique users |
|---|---|---|
| 2026-04-12 (Sun) | 142,629 | 129,026 |
| 2026-04-13 (Mon) | 8,979,904 | 3,408,978 |
| 2026-04-14 (Tue) | 813,648 | 642,027 |
| 2026-04-15 (Wed) | 9,013,840 | 3,457,289 |
| 2026-04-16 (Thu) | 10,586,969 | 3,607,971 |
| 2026-04-17 (Fri) | 9,347,425 | 3,449,322 |
| 2026-04-18 (Sat) | 523,874 | 463,862 |

**⚠️ New guardrail G9:** Weekend/holiday drop is extreme — **98.5% fewer CEP sessions on weekends** (142K Sun, 524K Sat) vs weekdays (9-10M). NSE/BSE market closed → stock/F&O campaigns don't fire → attribution collapses. Apr 14 (Tue) also low (813K) — likely Ram Navami / market holiday. Always check day-of-week when interpreting CEP attribution numbers.

---

### §12.3 Section C: Stale sweep (accessible tables only)

| Table | MAX(partition) | Days behind | Status |
|---|---|---|---|
| `core_bgv.engagement_pn_narad_master` | **2026-04-19** | 0 | ✅ fresh — proxy for CEP campaign freshness |
| `core_bgv.engagement_session_indepth` | **2026-04-19** | 0 | ✅ fresh — proxy for attribution freshness |
| `cep.campaign_info` | ❌ access denied | — | from catalog: 2026-04-18 (1 day), 6 commits/day ✅ |
| `dashboards_bgv.engagement_comms_attribution` | ❌ access denied | — | from catalog: 2026-04-18 (1 day) ✅ |
| `ds_user_data.cmab_user_store_v3` | ❌ access denied | — | from catalog: 2026-02-26 (52 days) ⚠️ |
| `ds_user_data.cmab_interaction_logs` | ❌ access denied | — | from catalog: 2025-12-01 (139 days) ⚠️ |

No future-date anomalies found. PN narad and session_indepth are both current as of today.

---

### §12.4 New guardrails (Prompt 3, G8–G9)

| ID | Guardrail |
|---|---|
| G8 | **Direct session_indepth × PN narad join on SPLIT_PART times out.** 60M × 249M rows with string function is too expensive (>300s). Always use two-step: (1) aggregate comm_session to get instanceIds, (2) look up PN narad with IN filter on specific instanceIds. |
| G9 | **Weekend/market-holiday attribution collapse.** CEP-attributed sessions drop 98%+ on weekends (NSE/BSE closed, stock/F&O campaigns don't fire). Apr 14 (Ram Navami) also dropped 91% vs adjacent weekdays. Always check `day_of_week` or join to market calendar before interpreting attribution trends. |

- `Deliverables/cmp-1-engagement-skill-spec.md` — CMP-1 (Engagement & Comms, completed)
- `Manager/CMP-2-Compass-Prompts.md` — Prompt templates for Prompts 2 + 3
- `Tasks/CMP-2-context-push-skill.md` — task tracking
- `Projects/compass-mcp-skills.md` — project overview

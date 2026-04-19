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

## 8. Acceptance criteria

CMP-2 is **done** when:

- [ ] All confirmed tables documented with schema, partition, row scale, refresh cadence
- [ ] `cep.campaign_info` campaign type/state enums confirmed via live Trino query
- [ ] `cep.campaign_info.campaignid ↔ engagement_pn_narad_master.campaignid` join confirmed (type cast)
- [ ] `engagement_comms_attribution.msg_id ↔ engagement_pn_narad_master.msg_id` join confirmed
- [ ] Campaign instance ID mystery resolved (what table has instanceId = 84316 etc.)
- [ ] CMAB user_id column format confirmed (ACC... or UUID)
- [ ] 10+ tested SQL examples for 2 hero tables
- [ ] Wiki `compass-analytics-skills.md` Skill #13 updated
- [ ] Project doc row 13 updated to ✅ done

---

## 9. Related

- `Deliverables/cmp-1-engagement-skill-spec.md` — CMP-1 (Engagement & Comms, completed)
- `Manager/CMP-2-Compass-Prompts.md` — Prompt templates for Prompts 2 + 3
- `Tasks/CMP-2-context-push-skill.md` — task tracking
- `Projects/compass-mcp-skills.md` — project overview

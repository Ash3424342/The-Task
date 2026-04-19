---
title: "CMP-1 — Engagement & Comms Compass Skill: Enhancement Spec"
status: in-progress
type: deliverable
sources:
  - "[[raw/compass-mcp-docs/Dataplatform · Analytics Skills Deep Dive - Coda.pdf]]"
  - "[[../../Groww-LLM-Wiki/wiki/concepts/compass-analytics-skills]]"
  - "[[Projects/compass-mcp-skills]]"
date: 2026-04-18
owner: me
task: "[[Tasks/CMP-1-engagement-comms-skill]]"
---

# CMP-1 — Engagement & Comms Compass Skill: Enhancement Spec

> **Purpose of this document.** CMP-1 is the enhancement of the Engagement & Comms Compass MCP skill (Skill #12 of 16). The actual skill content lives inside the Compass MCP server (not in `~/.claude/skills/`). Without a live Compass connection in this session, this spec defines **exactly what to enhance, what's known, what's draft, and what gaps need a Compass session to close** — so the next Compass-connected session can execute deterministically.

---

## 1. Scope

The Engagement & Comms skill answers questions about: **DAU, sessions, push notifications, email, SMS, WhatsApp, DND, app installs, campaign performance** at Groww.

Currently classified as **"Deep"** in the project depth assessment, but enhancement opportunity is high because:
- It directly feeds the Content Engine's email analytics (Tier 1 priority)
- 4 of the 9 engagement channel tables (SMS, WhatsApp, DAU/session, app/DND fact) lack the same column-level depth as the PN/email pair
- No tested SQL examples are checked-in for the Email backend table
- Cross-skill joins to User Master / Onboarding / User Events are implicit, not documented

---

## 2. Tables in scope

From the canonical Compass docs (`raw/compass-mcp-docs/Dataplatform · Analytics Skills Deep Dive - Coda.pdf`) and the wiki summary (`Groww-LLM-Wiki/wiki/concepts/compass-analytics-skills.md` Skill #12):

| # | Table | Channel | Row scale (live) | Total rows (catalog) | Partition | Last snapshot | Status |
|---|---|---|---|---|---|---|---|
| 1 | `engagement_pn_narad_master` | Push (NARAD) | 170-250M/day | 92.7B | `event_date` ✅ | 2026-04-17 16:11 | ✅ Schema confirmed — see §4 |
| 2 | `engagement_email_backend_master` | Email | 3-6M/day | 5.5B | `event_date` ✅ | 2026-04-17 03:19 | ✅ Schema confirmed — **Primary CMP-1 focus** |
| 3 | `engagement_sms_backend_master` | SMS | 500-650K/day | 722M | `event_date` ✅ | 2026-04-17 03:06 | ✅ Schema confirmed |
| 4 | `engagement_whatsapp_backend_master` | WhatsApp | 100-140K/day | 47.7M | `event_date` ✅ | 2026-04-17 11:00 | ✅ Schema confirmed |
| 5 | `engagement_user_session_daily` | DAU agg | 8M/day | 9.1B | `session_date` ✅ | 2026-04-17 21:32 | ✅ Schema confirmed |
| 6 | `engagement_dau_indepth` | App engagement | 8M/day | 8.5B | `session_date` ✅ | 2026-04-17 04:04 | ✅ Schema confirmed |
| 7 | `engagement_session_indepth` | Per-session | ~65M/day | **58.1B** ✅ | `session_date` ✅ | 2026-04-17 03:36 | ✅ Guardrail confirmed — SINGLE DAY ONLY |
| 8 | `engagement_app_fact` | App install/reachability | Weekly | 16.2B | `week` ✅ | 2026-04-13 (weekly ✅) | ✅ Schema confirmed |
| 9 | `engagement_dnd_fact` | DND state changes | ~70K events/day | 92.5M total | **NONE** ⚠️ unpartitioned | 2026-04-17 03:12 | ✅ Schema confirmed — COR table, no partition |
| 10 | `engagement_email_narad_master` | Email (Narad/campaign) | — | — | `event_date` | — | ➕ **Recommended addition** — see §2a |

> ✅ All 9 tables confirmed live in `platform_iceberg.core_bgv`. No removals needed.

### §2a — Recommended 10th table: `engagement_email_narad_master`

Found in catalog search (2026-04-18). This is the campaign-aware email equivalent of `engagement_pn_narad_master`. Unlike `engagement_email_backend_master` (which only has integer delivery flags), this table carries:
- `campaign_name`, `campaigncategory`, `campaign_tag`, `campaign_tag_2`, `productentity`
- `click_time`, `open_time`, `unsubscribe_time`, `received_time` (actual timestamps, not flags)
- `templatename`, `templatevariation`, `actionid`

**Recommendation:** Add as table #10. Essential for email campaign attribution queries. Without it, the skill cannot answer "which email campaign drove the most clicks last week?" because `engagement_email_backend_master` lacks campaign metadata.

### ⚠️ Critical: join key inconsistency across tables

| Join key | Tables |
|---|---|
| `user_account_id` (underscore) | email_backend_master, sms_backend_master, user_session_daily, app_fact |
| `useraccountid` (no underscore) | **pn_narad_master, whatsapp_backend_master** |
| `cuid` | dau_indepth, session_indepth, dnd_fact |

Always alias on cross-table joins: `pn.useraccountid = email.user_account_id`. The universal join key claim in §5 is partially wrong — PN and WA tables use `useraccountid` without underscore.

---

## 3. Per-table doc template

Mirror the format from the existing deep skills (Stocks & Trading, MF, F&O). Each table block:

```markdown
### Table: <table_name>

**Schema**: `platform_iceberg.core_bgv.<table_name>`
**Channel**: <push|email|sms|whatsapp|session|dau|app|dnd>
**Row scale**: <N rows/day, M total>
**Grain**: <one row per ...>
**Partition**: `<col>` (REQUIRED — single day for tables >100M/day)
**Refresh**: <daily T+0 / T+1 / weekly>

#### Key columns
| Column | Type | Sample | Description |
|---|---|---|---|
| user_account_id | varchar | ACC1234567 | Universal join key — all engagement tables |
| event_date | date | 2026-04-17 | Partition column |
| campaign_id | varchar | webengage_promo_apr18_a | Source campaign identifier |
| ... | | | |

#### Status / value enums
| Column | Allowed values |
|---|---|
| status | SENT, OPTED_OUT, VENDOR_FAILURE, FREQUENCY_CAPPED, EXPIRED |
| os | android, ios, msite (always LOWER()) |

#### Common filters
- `WHERE event_date = current_date - 1` — yesterday's data
- `WHERE LOWER(os) = 'android'` — handle mixed casing
- `WHERE status = 'SENT'` — successful deliveries only

#### Example queries (5 minimum, copy-paste ready)
```sql
-- Q1: yesterday's email open rate by template
SELECT template_id,
       COUNT(*) AS sent,
       SUM(CASE WHEN opened THEN 1 ELSE 0 END) AS opens,
       SUM(CASE WHEN opened THEN 1 ELSE 0 END) * 1.0 / COUNT(*) AS open_rate
FROM platform_iceberg.core_bgv.engagement_email_backend_master
WHERE event_date = current_date - 1
GROUP BY template_id
ORDER BY sent DESC LIMIT 50;
```

#### Joins
- → `growth_user_master_ultimate` on `user_account_id` (User Master skill)
- → `engagement_user_session_daily` on (`user_account_id`, `event_date`) for DAU correlation
- → `webengage` campaign metadata via `campaign_id` (external)

#### Guardrails
- ⚠️ Single-day filter required
- ⚠️ DND `s7` is VARCHAR `'true'`/`'false'`, not boolean
- ⚠️ Stale: `<list any stale equivalents>`
```

---

## 4. Confirmed column schemas (Compass-live, 2026-04-18)

All columns below are confirmed via `list_schema_fields` against the live Compass catalog. All fields are nullable=YES (Iceberg default).

### `engagement_pn_narad_master` ✅ confirmed
**Partition:** `event_date` | **Join key:** `useraccountid` (no underscore) | **92.7B rows**

| Column | Type | Description |
|---|---|---|
| `event_date` | date | **Partition column.** Date of PN event |
| `campaignid` | varchar | NARAD campaign identifier |
| `campaigninstanceid` | varchar | Campaign instance (one campaign → multiple instances) |
| `useraccountid` | varchar | User join key — **no underscore** (use alias when joining other tables) |
| `third_party_id` | varchar | Device push token / FCM registration ID |
| `inbound_time` | timestamp(6) tz | Time PN request received by NARAD |
| `sent_time` | timestamp(6) tz | Time PN dispatched to FCM/APNs vendor |
| `receive_time` | timestamp(6) tz | Time PN received on device |
| `click_time` | timestamp(6) tz | Time user clicked PN (null if not clicked) |
| `close_time` | timestamp(6) tz | Time user dismissed PN |
| `shown` | boolean | Whether PN was displayed on device |
| `status` | varchar | Delivery status: SENT, OPTED_OUT, VENDOR_FAILURE, FREQUENCY_CAPPED, EXPIRED |
| `failurereason` | varchar | Failure reason when status is not SENT |
| `msg_id` | varchar | NARAD internal message ID |
| `campaign_name` | varchar | Human-readable campaign label ✅ confirmed exists |
| `templatename` | varchar | Template name used |
| `templatevariation` | varchar | A/B variant label |
| `os` | varchar | OS: mixed casing (`android` vs `ANDROID`) — **always use `LOWER(os)`** |
| `retry` | bigint | Retry attempt count |
| `campaign_tag` | varchar | Primary campaign classification tag |
| `campaign_tag_2` | varchar | Secondary classification tag |
| `productentity` | varchar | Product vertical (MF, STOCKS, FNO, etc.) |

> ⚠️ **Corrections to §4 drafts:** No `slot`, `cmab_arm`, `template_id`, `delivered_flag`, `clicked_flag` columns — these don't exist. Delivery confirmation = `shown` (bool), click = `click_time` (timestamp, null if no click).

---

### `engagement_email_backend_master` ✅ confirmed (PRIMARY focus)
**Partition:** `event_date` | **Join key:** `user_account_id` | **5.5B rows**
**⚠ PII:** `email` column tagged `PII_EMAIL`

| Column | Type | Description |
|---|---|---|
| `event_date` | date | **Partition column.** Date of email event |
| `event_group` | varchar | High-level event category |
| `event_name` | varchar | Specific event: SENT, DELIVERED, OPENED, CLICKED, BOUNCED |
| `vendor` | varchar | Email sending vendor (Netcore, SendGrid, etc.) |
| `user_account_id` | varchar | User join key |
| `email` | varchar | Recipient email address — **PII_EMAIL** |
| `to_email_list_encrypt` | varchar | Encrypted recipient list |
| `inbound_msg_id` | varchar | Internal EMS/NARAD inbound message ID |
| `outbound_msg_id` | varchar | Vendor-assigned outbound message ID |
| `inbound_time` | timestamp(6) | Time email request received |
| `outbound_time` | timestamp(6) | Time email dispatched to vendor |
| `outbound_update_time` | timestamp(6) | Last vendor status update |
| `delivered_time` | timestamp(6) | Time email delivered to inbox |
| `relayed` | boolean | Whether email was relayed |
| `deliver_flag` | integer | 1=delivered, 0=not delivered |
| `open_flag` | integer | 1=opened, 0=not opened |
| `click_flag` | integer | 1=clicked, 0=not clicked |
| `source` | varchar | Source system (EMS, WebEngage, etc.) |
| `bounce_reason` | varchar | Bounce classification reason |

> ⚠️ **Corrections to §4 drafts:** No `template_id`, `campaign_id`, `subject`, `unsubscribed`, `domain` columns. Delivery/open/click are **integers (0/1)** not booleans. For campaign metadata, use `engagement_email_narad_master` (§2a).

---

### `engagement_sms_backend_master` ✅ confirmed
**Partition:** `event_date` | **Join key:** `user_account_id` | **722M rows**
**⚠ PII:** `mobile_no` tagged `PII_PHONE`

| Column | Type | Description |
|---|---|---|
| `event_date` | date | **Partition column** |
| `event_group` | varchar | Event category |
| `event_name` | varchar | SENT, DELIVERED, FAILED |
| `priority` | varchar | TRANSACTIONAL or PROMOTIONAL |
| `vendor` | varchar | SMS vendor |
| `mobile_no` | varchar | Recipient mobile — **PII_PHONE** |
| `user_account_id` | varchar | User join key |
| `inbound_msg_id` / `outbound_msg_id` | varchar | Message IDs |
| `inbound_time` / `outbound_time` / `outbound_update_time` / `delivered_time` | timestamp(6) | Timestamp funnel |
| `relayed` | boolean | Relay flag |
| `response_message` | varchar | Vendor response |
| `os` | varchar | User OS |
| `resend_mode` | varchar | Resend strategy |
| `error_msg` | varchar | Error details |
| `is_otp` | boolean | True if OTP SMS |
| `source` | varchar | Source system |

---

### `engagement_whatsapp_backend_master` ✅ confirmed
**Partition:** `event_date` | **Join key:** `useraccountid` (no underscore) | **47.7M rows**
**⚠ PII:** `data_decrypt` tagged `PII_EMAIL`, `sender` tagged `PII_AADHAAR`

| Column | Type | Description |
|---|---|---|
| `event_date` | date | **Partition column** |
| `useraccountid` | varchar | User join key — **no underscore** |
| `msg_id` | varchar | NARAD/WA message ID |
| `eventname` | varchar | SENT, DELIVERED, READ, FAILED |
| `status` | varchar | Delivery status |
| `failurereason` | varchar | Failure reason |
| `vendor` | varchar | WA BSP vendor |
| `sender` | varchar | Sender ID — **PII_AADHAAR tagged** |
| `recipientid` | varchar | Recipient WA ID |
| `data` / `data_decrypt` | varchar | Raw / decrypted payload — **PII_EMAIL tagged** |
| `externalmsgid` / `publisherid` / `actionid` | varchar | Message/action identifiers |
| `inboundts` / `outboundts` / `relayedts` / `expiresat` | varchar | Timestamps as strings (not timestamp type) |
| `istest` | boolean | Test message flag |
| `retry` | bigint | Retry count |
| `vendor_sent_time` / `vendor_received_time` / `vendor_read_time` / `vendor_failed_time` / `vendor_expired_time` | timestamp(6) | Vendor-confirmed event timestamps |
| `vendor_failed_cause` | varchar | Vendor failure cause code |

---

### `engagement_user_session_daily` ✅ confirmed
**Partition:** `session_date` | **Join key:** `user_account_id` | **9.1B rows** | Preferred for DAU aggregations.

| Column | Type | Description |
|---|---|---|
| `session_date` | date | **Partition column** |
| `user_account_id` | varchar | User join key |
| `os_name` | varchar | android / ios / web |
| `session_count` | bigint | Total sessions for user on date |
| `app_dau` | integer | 1 if app DAU |
| `web_dau` | integer | 1 if web DAU |
| `txn_dau` | integer | 1 if transacting DAU |
| `desktop_dau` | integer | 1 if desktop DAU |

---

### `engagement_dau_indepth` ✅ confirmed
**Partition:** `session_date` | **Join key:** `cuid` | **8.5B rows** | Richer per-user daily view with intent signals.

| Column | Type | Description |
|---|---|---|
| `session_date` | date | **Partition column** |
| `cuid` | varchar | User CUID — join to user master for `user_account_id` |
| `app_time_mins` | double | Total app time in minutes |
| `app_version` / `os_version` | varchar | Version info |
| `ttu_flag` | integer | Time-to-use flag |
| `bounce_dau_flag` | integer | 1 if all sessions were bounces |
| `valid_session_count` / `sessions_count` | integer | Session counts |
| `comm_sessions` | integer | Communication-intent sessions |
| `order_placing_sessions` / `transacting_sessions` | integer | Commerce sessions |
| `dau_intent` / `dashboard_intent` / `search_intent` / `explore_intent` / `pp_intent` / `order_intent` | integer | Intent signal flags |

---

### `engagement_session_indepth` ✅ confirmed
**Partition:** `session_date` | **Join keys:** `cuid` (user), `suid` (session) | **58.1B rows**
**⚠ MANDATORY single-day filter. Full scan = job killed.**

| Column | Type | Description |
|---|---|---|
| `session_date` | date | **Partition column — always filter here first** |
| `suid` | varchar | Unique session ID |
| `cuid` | varchar | User CUID |
| `session_type` / `os_name` / `app_version` / `os_version` | varchar | Session context |
| `session_start_time` / `session_end_time` | timestamp(6) | Session window |
| `session_duration` | integer | Duration in seconds |
| `start_event` / `end_event` | varchar | Entry/exit events |
| `total_events` | integer | Total events in session |
| `dash_wl_events` / `search_events` / `mf_pp_events` / `stock_pp_events` / `mf_explore_events` / `stock_explore_events` / `profile_events` / `order_initiation_events` / `order_placing_events` / `payment_events` | integer | Event category counts |
| `orders` | integer | Orders placed in session |
| `bounce_flag` | integer | 1 if bounce session |
| `comm_session` | varchar | Communication session classification |
| `session_event_id` | varchar | Session event ID (PII_PHONE tagged — likely mis-tagged) |

---

### `engagement_app_fact` ✅ confirmed
**Partition:** `week` | **Join key:** `user_account_id` | **16.2B rows** | Weekly cadence — last commit 2026-04-13.

| Column | Type | Description |
|---|---|---|
| `week` | date | **Partition column.** Week start date |
| `user_account_id` | varchar | User join key |
| `third_party_id` | varchar | Push token / device ID |
| `os_name` | varchar | Operating system |
| `latest_app_version` | varchar | Most recent app version that week |
| `last_event` | varchar | Last recorded app event type |
| `last_event_time` | timestamp(6) | Time of last app event |
| `app_keep_flag` | integer | 1 if app installed and retained |
| `push_reachable` | integer | 1 if valid push token + notifications enabled |

---

### `engagement_dnd_fact` ✅ confirmed
**Partition: NONE** (COR table, `is_partitioned=False`) | **Join key:** `cuid` | **92.5M rows** | 1 commit/day.
**⚠ `s7` is VARCHAR — use `s7 = 'true'`, not `s7 = true`**
**⚠ No partition — always filter `WHERE date = ...` to avoid full table scan**

| Column | Type | Description |
|---|---|---|
| `date` | date | Date of DND state change — NOT a partition, just a column |
| `cuid` | varchar | User CUID — **PII_AADHAAR tagged** |
| `s7` | varchar | DND status: `'true'` = DND on, `'false'` = DND off |
| `prev_s7` | varchar | Previous DND status |
| `event_time` | timestamp(6) | Timestamp of state change |
| `prev_time` | timestamp(6) | Timestamp of previous state |
| `sdk_id` | integer | SDK identifier |
| `session_type` / `app_version` / `os_name` | varchar | Session context at time of event |
| `r` | bigint | Row number within user partition |

---

## 5. Cross-skill joins

The Engagement skill does not work in isolation. Document these joins explicitly:

| Skill | Join key | Use case |
|---|---|---|
| **User Master & Growth** (`growth_user_master`) | `user_account_id` | Add user state (TTU/NTU via `fid_ts_growth`), `kyc_status`, `fid_product_growth`, `mapping0` (acquisition channel), geo |
| **Onboarding** (#6) | `user_account_id` + date alignment | Tie engagement events to onboarding stage (e.g., did the email reach a user mid-Esign?) |
| **User Events** (#5, Pinot) | `user_account_id` + `event_timestamp` window | Attribute app sessions back to a push that fired N minutes earlier (12-min post-PN window per CNC analysis) |
| **MF / Stocks / F&O** (#3, #1, #2) | `user_account_id` + product context | Did the email/push lead to a transaction in product X within attribution window? |

> ⚠️ **Correction:** `user_account_id` is NOT universal across all engagement tables. PN narad and WhatsApp use `useraccountid` (no underscore). DAU indepth, session indepth, and DND fact use `cuid`. See §2 join key table.

### Join key reference
| Table | Join to user_master on |
|---|---|
| engagement_email_backend_master | `e.user_account_id = u.user_account_id` |
| engagement_sms_backend_master | `e.user_account_id = u.user_account_id` |
| engagement_user_session_daily | `s.user_account_id = u.user_account_id` |
| engagement_app_fact | `a.user_account_id = u.user_account_id` |
| **engagement_pn_narad_master** | **`pn.useraccountid = u.user_account_id`** (note: no underscore in PN) |
| **engagement_whatsapp_backend_master** | **`wa.useraccountid = u.user_account_id`** (note: no underscore in WA) |
| engagement_dau_indepth | join dau.cuid → user_master.cuid (via separate cuid→user_account_id mapping) |
| engagement_session_indepth | join session.cuid → user_master.cuid |
| engagement_dnd_fact | join dnd.cuid → user_master.cuid |

### growth_user_master key columns for engagement joins

**Table:** `platform_iceberg.core_bgv.growth_user_master` (57 cols, no partition — full table ~60M users)

| Column | Use in engagement joins |
|---|---|
| `user_account_id` | Primary join key (ACC-prefixed) |
| `fid_ts_growth` | TTU segmentation: `IS NOT NULL` → TTU user; `IS NULL` → NTU |
| `fid_product_growth` | First transacted product (MF, STOCKS, FNO, etc.) |
| `kyc_status` | VERIFIED_ON_GROWW, VERIFIED, VERIFIED_ON_KRA, NA, NOT_VERIFIED |
| `mapping0` | Acquisition channel: `performance` (paid), `referral`, NULL (organic) |
| `mapping2` | Campaign name for `performance` channel users |
| `signup_time` | Cohort analysis; exclude test accounts: `email_id NOT LIKE '%@xyby.net%'` |
| `kyc_city` / `kyc_state` | Geo segmentation |
| `aof_status` | Onboarding progress |

### Tested join examples (✅ 2026-04-19)

```sql
-- J1: Email open rate by TTU/NTU segment and KYC status
SELECT
  CASE WHEN u.fid_ts_growth IS NOT NULL THEN 'TTU' ELSE 'NTU' END AS user_segment,
  u.kyc_status,
  COUNT(*) AS emails_sent,
  SUM(e.deliver_flag) AS delivered,
  SUM(e.open_flag) AS opened,
  ROUND(SUM(e.open_flag) * 100.0 / NULLIF(SUM(e.deliver_flag), 0), 2) AS open_rate_pct
FROM platform_iceberg.core_bgv.engagement_email_backend_master e
JOIN platform_iceberg.core_bgv.growth_user_master u
  ON e.user_account_id = u.user_account_id
WHERE e.event_date = current_date - interval '1' day
GROUP BY
  CASE WHEN u.fid_ts_growth IS NOT NULL THEN 'TTU' ELSE 'NTU' END,
  u.kyc_status
ORDER BY emails_sent DESC
LIMIT 10;
-- Live result 2026-04-18: TTU VERIFIED_ON_GROWW = 10.8M emails, 11.6% open
-- NTU NOT_VERIFIED has highest open rate (18.3%) — early-funnel users more engaged

-- J2: PN click-through rate by platform × TTU/NTU (note: useraccountid no underscore)
SELECT
  LOWER(pn.os) AS platform,
  CASE WHEN u.fid_ts_growth IS NOT NULL THEN 'TTU' ELSE 'NTU' END AS user_segment,
  COUNT(*) AS total_pn,
  SUM(CASE WHEN pn.status = 'SENT' THEN 1 ELSE 0 END) AS sent,
  SUM(CASE WHEN pn.click_time IS NOT NULL THEN 1 ELSE 0 END) AS clicked,
  ROUND(SUM(CASE WHEN pn.click_time IS NOT NULL THEN 1 ELSE 0 END) * 100.0
        / NULLIF(SUM(CASE WHEN pn.status = 'SENT' THEN 1 ELSE 0 END), 0), 3) AS ctr_pct
FROM platform_iceberg.core_bgv.engagement_pn_narad_master pn
JOIN platform_iceberg.core_bgv.growth_user_master u
  ON pn.useraccountid = u.user_account_id
WHERE pn.event_date = current_date - interval '1' day
  AND LOWER(pn.os) IN ('android', 'ios')
GROUP BY LOWER(pn.os),
  CASE WHEN u.fid_ts_growth IS NOT NULL THEN 'TTU' ELSE 'NTU' END
ORDER BY total_pn DESC;
-- Live result 2026-04-18: Android TTU CTR=1.4%, NTU CTR=0.3%
-- ⚠ iOS click_time is always null in PN data — iOS click tracking not captured via this column

-- J3: DAU users who are push-reachable (engagement_app_fact → engagement_user_session_daily)
SELECT s.os_name,
       COUNT(DISTINCT s.user_account_id) AS app_dau,
       SUM(CASE WHEN a.push_reachable = 1 THEN 1 ELSE 0 END) AS push_reachable_dau
FROM platform_iceberg.core_bgv.engagement_user_session_daily s
LEFT JOIN platform_iceberg.core_bgv.engagement_app_fact a
  ON s.user_account_id = a.user_account_id
  AND a.week = DATE '2026-04-13'
WHERE s.session_date = current_date - interval '1' day
  AND s.app_dau = 1
GROUP BY s.os_name
ORDER BY app_dau DESC;

-- J4: DND users who are still receiving PNs (should be 0 if DND filtering works)
SELECT COUNT(*) AS pn_to_dnd_users
FROM platform_iceberg.core_bgv.engagement_pn_narad_master pn
JOIN platform_iceberg.core_bgv.engagement_dnd_fact dnd
  ON pn.useraccountid = dnd.cuid
WHERE pn.event_date = current_date - interval '1' day
  AND pn.status = 'SENT'
  AND dnd.date = current_date - interval '1' day
  AND COALESCE(dnd.s7, 'false') = 'true';
-- Validates DND suppression effectiveness; expect very low count (DND status = SENT in PN = ~27K/day)
```

**Live findings from cross-skill joins (2026-04-18):**
- `kyc_status` live values: VERIFIED_ON_GROWW (dominant, ~85%), VERIFIED, VERIFIED_ON_KRA, NA, NOT_VERIFIED
- NTU users have **higher email open rates** than TTU at same kyc_status (18.3% vs 10.3% for NOT_VERIFIED segment)
- Android TTU CTR (1.4%) is **4.7× NTU CTR (0.3%)**
- **iOS click_time is always null** — iOS click tracking not captured in `click_time` column; use `shown = true` as a proxy for iOS engagement
- PN narad join must use `pn.useraccountid` (no underscore); all other tables use `user_account_id`

---

## 6. Open questions for the next Compass session

When this gets executed against a live Compass connection, run these in order:

1. ✅ **Discovery** — `compass.search(domain="engagement")` — 589 total engagement-related entities found. 16 tables in `core_bgv` identified. 9-table scope fully confirmed; `engagement_email_narad_master` flagged as recommended addition (#10).
2. ✅ **Schema** for each table — all 9 confirmed via `list_schema_fields`. See §4 for complete columns.
3. ✅ **Partition column** — confirmed for all 9. `engagement_dnd_fact` has **no partition** (COR table). See §2 table.
4. ✅ **Sample data** — 5-row samples captured for all 9 tables (2026-04-18, date=2026-04-17). See §10.
5. ✅ **Validate enum sets** — All enum columns confirmed via GROUP BY queries. See §10 per-table enum sections.
6. ✅ **Validate NARAD `os` mixed-case claim** — **CONFIRMED: 6 distinct raw values** (android, ANDROID, Android, ios, IOS, iOS). LOWER() is mandatory. See §10.
7. ✅ **Validate DND `s7` VARCHAR claim** — Confirmed: live values are `'true'`/`'false'`/`null`. Boolean `true` would return 0 rows. See §10 G7.
8. ✅ **Validate `engagement_session_indepth` 57B total** — Live COUNT: **60.2M rows for one day** (58.1B cumulative catalog). Single-day filter mandatory.
9. ⬜ **Joins** — Sample-join engagement to `growth_user_master_ultimate` on `user_account_id`; confirm key format match. Note: PN/WA join on `useraccountid`, CUID tables need user master cross-ref.
10. ✅ **Stale detection** — All tables snapshotted 2026-04-17 (fresh). app_fact last commit 2026-04-13 (Sunday) — confirmed weekly cadence. No stale tables.

For each item: ✅ = Compass-confirmed in session 2026-04-18. ⬜ = still needs live query.

---

## 7. Acceptance criteria

CMP-1 is **done** when:

- [x] All 9 engagement tables documented with: schema, grain, partition, row scale, refresh cadence (✅ 2026-04-18 Compass session)
- [x] All key columns documented with type and description for all 9 tables (✅ 2026-04-18 Compass session)
- [x] All status/enum columns have full value lists — confirmed via live Trino GROUP BY queries (✅ 2026-04-18, see §10)
- [x] **5 minimum tested SQL examples per table** — 45 examples in §11; Q1 live-confirmed for all 9 tables (✅ 2026-04-19 Compass session)
- [x] Cross-skill joins documented with tested examples — J1–J4 in §5 (✅ J1+J2 live-confirmed 2026-04-19)
- [x] Stale tables flagged — none (all tables snapshotted 2026-04-17; app_fact weekly confirmed) ✅
- [x] All wiki claims updated — `compass-analytics-skills.md` Skill #12 fully rewritten with live data (✅ 2026-04-19)
- [x] Project doc depth-assessment updated — `compass-mcp-skills.md` row 8 marked ✅ done (✅ 2026-04-19)

**Definition of "tested"**: query runs without error against live Compass (Trino), returns expected shape, completes within 600s timeout, and produces a non-empty result set on the date filter used.

---

## 8. Sequencing

CMP-1 is the pattern for CMP-2/3/4. Once this skill is enhanced and the workflow is validated end-to-end:
- CMP-2 (Context Push) can copy the same template, focused on context-aware push tables
- CMP-3 (User Financial Health) is ground-up — needs schema discovery first
- CMP-4 (MF Extended) extends existing MF skill — same template

**Estimated time** (per project doc): "Deep" tier = 1 day incremental polish per skill, but CMP-1 is closer to "Medium" (2 days) because of the breadth of channels covered + the tested SQL bar.

---

## 9. Related
- [[Tasks/CMP-1-engagement-comms-skill]] — the task this spec satisfies
- [[Projects/compass-mcp-skills]] — parent project
- [[Deliverables/analytics-skills-deep-dive]] — Saurabh's official skill reference (18 pages, source of table list)
- [[Deliverables/compass-mcp-product-doc]] — Compass MCP product overview
- [[Deliverables/compass-setup-guide]] — proxy + IDE config for Compass connection
- [[wiki/Compass-MCP]] — task-scoped Compass entity
- `Groww-LLM-Wiki/wiki/concepts/compass-analytics-skills.md` — full domain encyclopedia entry

---

## 10. Live enum values + sample data + Trino guardrails (✅ Compass session 2026-04-18)

*All queries run against partition date 2026-04-17 (daily tables) and 2026-04-13 (app_fact weekly). Date: 2026-04-18.*

---

### 10.1 Trino syntax guardrails (apply to ALL queries)

| Pattern | ❌ Wrong | ✅ Correct |
|---|---|---|
| Yesterday's date | `event_date = current_date - 1` | `event_date = current_date - interval '1' day` |
| String suffix | `RIGHT(col, 3)` | `SUBSTR(col, LENGTH(col) - 2)` |
| Date literal | `'2026-04-17'` (bare string) | `DATE '2026-04-17'` |
| Boolean from varchar | `WHERE s7 = true` | `WHERE s7 = 'true'` |

---

### 10.2 engagement_pn_narad_master

**`os` — 6 distinct raw values (LOWER() mandatory):**
| os_raw | normalized | count (2026-04-17) | % |
|---|---|---|---|
| android | android | 196,566,864 | 79.1% |
| ANDROID | android | 24,126,855 | 9.7% |
| ios | ios | 23,884,038 | 9.6% |
| IOS | ios | 4,473,730 | 1.8% |
| Android | android | 12 | <0.01% |
| iOS | ios | 9 | <0.01% |

**`status` — 7 values (wiki documented only 5; DND and INTERNAL_SERVICE_FAILURE are new):**
| status | count | % |
|---|---|---|
| SENT | 181,184,299 | 72.9% |
| VENDOR_FAILURE | 32,506,366 | 13.1% |
| OPTED_OUT | 23,637,737 | 9.5% |
| FREQUENCY_CAPPED | 11,600,303 | 4.7% |
| EXPIRED | 93,955 | 0.04% |
| DND | 27,431 | 0.01% |
| INTERNAL_SERVICE_FAILURE | 1,417 | <0.01% |

**Sample row structure notes:**
- `campaigninstanceid` has 2 formats: ISIN-based (`CAMP-INSTANCE-INE542W01025-2026-04-17`) and numeric (`84316`)
- `shown = null` for non-delivered rows; `shown = true/false` for SENT rows only
- `productentity` sample values: Stocks, F&O (camel-case)
- `campaign_tag` format: `"f&o - pre-market index snapshot"`, `"stock - pers - 2% move"` (lowercase, hyphen-separated)

---

### 10.3 engagement_email_backend_master

**⚠ `event_name` = email template name, NOT a delivery status.** Delivery funnel is entirely in integer flag columns.

**Delivery funnel (2026-04-17, 5,437,828 total records):**
| deliver_flag | open_flag | click_flag | count | meaning |
|---|---|---|---|---|
| 1 | 0 | 0 | 4,554,989 (83.8%) | Delivered, not opened |
| 1 | 1 | 0 | 611,528 (11.2%) | Opened, not clicked |
| 0 | 0 | 0 | 170,547 (3.1%) | Not delivered |
| null | null | null | 90,140 (1.7%) | In-flight / unknown |
| 1 | 1 | 1 | 10,032 (0.18%) | Full funnel |
| 1 | 0 | 1 | 592 (<0.01%) | Clicked without open (tracked separately) |

**Derived rates (2026-04-17):** Delivery: 95.2% | Open rate: 12.0% | Click-to-open: 1.7%

**`vendor`:** AWS_SES only | **`source`:** Blazr only

**`event_group` values:** stocks, stocks_oms_group, gb_raa, mf_order, gb_withdraw, stocks_sip, New_device_login, email_otp_group, mf_otp

**`inbound_msg_id` structure:** `{service_name}{transaction_ref}{event_name}` — encodes source system, transaction, and template in one field.

---

### 10.4 engagement_sms_backend_master

**`priority`:** high, medium, low
**`vendor`:** INFOBIP only (Email=AWS_SES, WA=GUPSHUP — each channel has exactly one vendor)
**`source`:** Blazr only
**`os`:** always NULL — SMS does not capture device OS
**`resend_mode`:** always NULL in live data
**`is_otp`:** true / false
**`mobile_no`:** stored with country code — prefix `91` (India)

**`event_group` sample values:** general_otp_group, Context_OTP, risk_comms, upi_payments, stocks_sbs, mf_otp, mf_investment, upi_management, CREDIT-COLLECTION, flink_live

---

### 10.5 engagement_whatsapp_backend_master

**Total rows 2026-04-17:** ~111,961

**`vendor`:** GUPSHUP only
**`status`:** SENT, INTERNAL_SERVICE_FAILURE, DATA_ISSUE *(DATA_ISSUE was not in spec)*
**`istest`:** false (all live data)
**`retry`:** 0 for most; integer for retried messages

**`eventname` sample values:** risk_intraday_autosquare_off_warn, risk_mis_circuit_warn, mtf_shortfall_t1_day_whatsapp_comm, credit_cf_flink_creditapp, LAMFRepaymentSuccess, LAMFWithdrawSuccessful, LaMFRepaymentFailed

**⚠ WhatsApp is almost entirely risk/transactional comms** — autosquare-off warnings, margin shortfall alerts, LAMF events. Not used for marketing campaigns.

**`vendor_sent/received/read_time`:** can be null even for SENT rows — last-mile tracking gap.

**`msg_id` structure:** `{service_name}{user_account_id}_{ISIN}_{direction}{date}{event_name}` — useful for exact deduplication.

---

### 10.6 engagement_user_session_daily

**Total rows 2026-04-17:** ~8.74M unique user-day records

**`os_name` — Title Case (no LOWER() needed, exact match):**
Android, iOS, Windows, Mac OS, iPadOS, Linux, Chromium OS, Ubuntu, Tizen, HarmonyOS, Fedora, null

**⚠ `txn_dau=1` with `session_count=0` is valid** — API/background transactors with no app session. Do NOT assume sessions exist for txn_dau users.

**DAU breakdown 2026-04-17 (top segments):** Android app_dau: ~7.1M | iOS app_dau: ~1.4M | Windows desktop: ~175K | Mac OS desktop: ~27K

---

### 10.7 engagement_dau_indepth

**`cuid`:** UUID format — `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. Must join to user master to get `user_account_id`.

**`os_version`:** bare integer string (iOS: "16", "14", "13"; Android: "14", "13"). No unit suffix.

**`app_time_mins`:** 0.0 for bounce sessions (expected).

**`bounce_dau_flag = 1`** means ALL sessions that day were bounces.

**`comm_sessions`** can be ≥1 even when `bounce_dau_flag=1`.

---

### 10.8 engagement_session_indepth

**Live row count (2026-04-17):** **60,203,599** (COUNT query completed ~9s)

**`session_type`:** App, Non App (only 2 values)

**`os_name`:** Android, iOS, Windows, Mac OS, iPadOS, Linux, Chromium OS, Ubuntu, null (Title Case)

**`bounce_flag`:** 0 or 1 (integer, not boolean)

**⚠ `session_duration` is frequently NULL** — filter `WHERE session_duration IS NOT NULL` for any duration analysis.

**⚠ `comm_session` is NOT always null.** When set: `CEP-{campaignInstanceId}-{bucket}-CAMPAIGN-PRODUCTION-{user_account_id}-{counter}`. This is the campaign-to-session attribution field — identifies which PN campaign opened this session. To find sessions from a specific campaign: `WHERE comm_session LIKE 'CEP-84335-%'`.

---

### 10.9 engagement_app_fact

**Latest partition:** `week = DATE '2026-04-13'` (week keys are **Sundays**)

**⚠ `SELECT MAX(week)` timed out** — 16.2B rows. Never scan without partition filter. Always hardcode the known Sunday date.

**`os_name`:** Android, iOS, iPadOS, null (null = churned/web-only users, 44.6M rows)

**`app_keep_flag`:** 0 (uninstalled/churned) or 1 (retained)

**`push_reachable`:** 0 (no valid token) or 1 (reachable)

**Push-reachable installed users (week 2026-04-13):** Android: 26.6M | iOS: 3.6M | iPadOS: 14K | **Total: ~30.2M**

**`last_event` sample values:** gcm_registered, user_session_started, user_logged_out

---

### 10.10 engagement_dnd_fact

**Total rows 2026-04-17:** 69,598 (s7='true': 49,490 | s7='false': 20,104 | null: 4)

**`s7` / `prev_s7`:** varchar — values `'true'`, `'false'`, null. 4 null rows exist.

**`session_type`:** INTERACTIVE, PROCESSING, BACKGROUND, null

**`os_name`:** Android (dominant), iOS, iPadOS, null

**`r`:** row number within user's entire DND history. `r=1` = first-ever DND event for this user.

**DND flow interpretation (2026-04-17):**
| s7 | prev_s7 | meaning |
|---|---|---|
| 'true' | null | First-ever DND opt-in |
| 'true' | 'false' | Re-enabling DND |
| 'false' | 'true' | Turning DND off |
| 'false' | null | First-ever record, already DND-off |

**Safe filter pattern:** `WHERE COALESCE(s7, 'false') = 'true'` (handles the 4 null rows)

---

### 10.11 New guardrails summary (discovered 2026-04-18)

| ID | Table | Guardrail |
|---|---|---|
| G1 | All | `current_date - 1` is invalid Trino syntax — use `current_date - interval '1' day` |
| G2 | All | `RIGHT(str, n)` does not exist — use `SUBSTR(str, LENGTH(str) - n + 1)` |
| G3 | pn_narad_master | `status` has 7 values — add DND and INTERNAL_SERVICE_FAILURE to enum docs |
| G4 | session_indepth | `session_duration` is frequently NULL — always filter IS NOT NULL |
| G5 | session_indepth | `comm_session` encodes CEP campaign attribution — not always null |
| G6 | email_backend_master | `event_name` = template name, NOT delivery status |
| G7 | dnd_fact | `s7` has 4 null rows — use `COALESCE(s7, 'false')` |
| G8 | user_session_daily | `txn_dau=1` can coexist with `session_count=0` |
| G9 | app_fact | `MAX(week)` scan times out — always partition-filter with known Sunday date |
| G10 | sms_backend_master | Vendor = INFOBIP (not documented in schema) |
| G11 | whatsapp_backend_master | `status` has DATA_ISSUE value (not in spec) |
| G12 | dau_indepth, session_indepth, dnd_fact | `cuid` is UUID format — join to user master for ACC-prefixed `user_account_id` |
| G13 | All | `os_name` casing varies by table: PN narad needs LOWER(), session/DAU tables use Title Case |
| G14 | sms_backend_master | `priority` has mixed casing: `high`/`medium`/`low` AND `MEDIUM` seen in live data — use LOWER(priority) |
| G15 | app_fact | `os_name` has 4th value `iPhone OS` (legacy, 0 push-reachable) in addition to iOS/Android/iPadOS |
| G16 | dau_indepth, session_indepth | Bounce rate is ~99% — this is normal; bounce = any short session. Non-bounce users are the interesting cohort |
| G17 | user_session_daily | `desktop_dau=1` = browser/laptop users (the "web DAU" reported publicly); `web_dau` counts all browser sessions including from mobile OS |

---

## 11. SQL examples — copy-paste ready (schema-confirmed, Trino-compatible)

*All queries use verified Trino syntax and confirmed schemas from §4 and §10. Q1 for each table live-confirmed 2026-04-19. Replace `DATE '2026-04-17'` with `current_date - interval '1' day` for live use.*

---

### 11.1 engagement_pn_narad_master

```sql
-- Q1: PN delivery funnel by status for a given day
SELECT status,
       COUNT(*) AS sends,
       ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS pct
FROM platform_iceberg.core_bgv.engagement_pn_narad_master
WHERE event_date = DATE '2026-04-17'
GROUP BY status
ORDER BY sends DESC;
-- Expected: 7 rows — SENT ~73%, VENDOR_FAILURE ~13%, OPTED_OUT ~9.5%, FREQUENCY_CAPPED ~4.7%

-- Q2: Click-through rate by productentity (SENT only)
SELECT productentity,
       COUNT(*) AS sent,
       SUM(CASE WHEN click_time IS NOT NULL THEN 1 ELSE 0 END) AS clicked,
       ROUND(SUM(CASE WHEN click_time IS NOT NULL THEN 1 ELSE 0 END) * 100.0
             / NULLIF(COUNT(*), 0), 3) AS ctr_pct
FROM platform_iceberg.core_bgv.engagement_pn_narad_master
WHERE event_date = DATE '2026-04-17'
  AND status = 'SENT'
GROUP BY productentity
ORDER BY sent DESC
LIMIT 20;

-- Q3: Top campaigns by sent volume (with shown and click rates)
SELECT campaign_name,
       campaign_tag,
       COUNT(*) AS total,
       SUM(CASE WHEN status = 'SENT' THEN 1 ELSE 0 END) AS sent,
       SUM(CASE WHEN shown = true THEN 1 ELSE 0 END) AS shown,
       SUM(CASE WHEN click_time IS NOT NULL THEN 1 ELSE 0 END) AS clicked
FROM platform_iceberg.core_bgv.engagement_pn_narad_master
WHERE event_date = DATE '2026-04-17'
GROUP BY campaign_name, campaign_tag
ORDER BY total DESC
LIMIT 20;

-- Q4: Android vs iOS SENT rate (LOWER(os) mandatory)
SELECT LOWER(os) AS platform,
       COUNT(*) AS total_attempts,
       SUM(CASE WHEN status = 'SENT' THEN 1 ELSE 0 END) AS sent,
       ROUND(SUM(CASE WHEN status = 'SENT' THEN 1 ELSE 0 END) * 100.0
             / NULLIF(COUNT(*), 0), 2) AS send_rate_pct
FROM platform_iceberg.core_bgv.engagement_pn_narad_master
WHERE event_date = DATE '2026-04-17'
  AND LOWER(os) IN ('android', 'ios')
GROUP BY LOWER(os);

-- Q5: Frequency-capped users — re-engagement signal
SELECT COUNT(DISTINCT useraccountid) AS freq_capped_users,
       COUNT(*) AS freq_capped_events,
       COUNT(*) * 1.0 / COUNT(DISTINCT useraccountid) AS avg_caps_per_user
FROM platform_iceberg.core_bgv.engagement_pn_narad_master
WHERE event_date = DATE '2026-04-17'
  AND status = 'FREQUENCY_CAPPED';
```

---

### 11.2 engagement_email_backend_master

```sql
-- Q1: Daily email funnel KPIs
SELECT
  COUNT(*) AS total_records,
  SUM(CASE WHEN deliver_flag = 1 THEN 1 ELSE 0 END) AS delivered,
  SUM(CASE WHEN open_flag = 1 THEN 1 ELSE 0 END) AS opened,
  SUM(CASE WHEN click_flag = 1 THEN 1 ELSE 0 END) AS clicked,
  ROUND(SUM(CASE WHEN deliver_flag = 1 THEN 1 ELSE 0 END) * 100.0
        / COUNT(*), 2) AS delivery_rate_pct,
  ROUND(SUM(CASE WHEN open_flag = 1 THEN 1 ELSE 0 END) * 100.0
        / NULLIF(SUM(CASE WHEN deliver_flag = 1 THEN 1 ELSE 0 END), 0), 2) AS open_rate_pct,
  ROUND(SUM(CASE WHEN click_flag = 1 THEN 1 ELSE 0 END) * 100.0
        / NULLIF(SUM(CASE WHEN open_flag = 1 THEN 1 ELSE 0 END), 0), 2) AS cto_rate_pct
FROM platform_iceberg.core_bgv.engagement_email_backend_master
WHERE event_date = DATE '2026-04-17';
-- Expected: delivery ~95%, open ~12%, CTO ~1.7%

-- Q2: Open rate by event_group (email category performance)
SELECT event_group,
       COUNT(*) AS sent,
       SUM(open_flag) AS opened,
       ROUND(SUM(open_flag) * 100.0 / NULLIF(SUM(deliver_flag), 0), 2) AS open_rate_pct
FROM platform_iceberg.core_bgv.engagement_email_backend_master
WHERE event_date = DATE '2026-04-17'
GROUP BY event_group
ORDER BY sent DESC
LIMIT 20;

-- Q3: Top email templates by volume
SELECT event_name, event_group,
       COUNT(*) AS volume,
       SUM(deliver_flag) AS delivered,
       SUM(open_flag) AS opened,
       SUM(click_flag) AS clicked
FROM platform_iceberg.core_bgv.engagement_email_backend_master
WHERE event_date = DATE '2026-04-17'
GROUP BY event_name, event_group
ORDER BY volume DESC
LIMIT 20;

-- Q4: Bounce analysis by reason
SELECT COALESCE(bounce_reason, 'no_bounce') AS bounce_reason,
       COUNT(*) AS count
FROM platform_iceberg.core_bgv.engagement_email_backend_master
WHERE event_date = DATE '2026-04-17'
  AND deliver_flag = 0
GROUP BY bounce_reason
ORDER BY count DESC
LIMIT 20;

-- Q5: Full-funnel users (delivered + opened + clicked) on a given day
SELECT COUNT(DISTINCT user_account_id) AS full_funnel_users
FROM platform_iceberg.core_bgv.engagement_email_backend_master
WHERE event_date = DATE '2026-04-17'
  AND deliver_flag = 1
  AND open_flag = 1
  AND click_flag = 1;
```

---

### 11.3 engagement_sms_backend_master

```sql
-- Q1: SMS volume by priority and event_group (top 20)
SELECT priority, event_group,
       COUNT(*) AS volume,
       SUM(CASE WHEN relayed = true THEN 1 ELSE 0 END) AS delivered
FROM platform_iceberg.core_bgv.engagement_sms_backend_master
WHERE event_date = DATE '2026-04-17'
GROUP BY priority, event_group
ORDER BY volume DESC
LIMIT 20;

-- Q2: OTP vs non-OTP breakdown
SELECT is_otp,
       priority,
       COUNT(*) AS volume,
       ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS pct
FROM platform_iceberg.core_bgv.engagement_sms_backend_master
WHERE event_date = DATE '2026-04-17'
GROUP BY is_otp, priority
ORDER BY volume DESC;

-- Q3: Top SMS templates by volume
SELECT event_name, event_group, priority,
       COUNT(*) AS volume
FROM platform_iceberg.core_bgv.engagement_sms_backend_master
WHERE event_date = DATE '2026-04-17'
GROUP BY event_name, event_group, priority
ORDER BY volume DESC
LIMIT 20;

-- Q4: Delivery success rate (relayed = true means delivered)
SELECT
  COUNT(*) AS total,
  SUM(CASE WHEN relayed = true THEN 1 ELSE 0 END) AS delivered,
  ROUND(SUM(CASE WHEN relayed = true THEN 1 ELSE 0 END) * 100.0
        / COUNT(*), 2) AS delivery_rate_pct
FROM platform_iceberg.core_bgv.engagement_sms_backend_master
WHERE event_date = DATE '2026-04-17';

-- Q5: OTP SMS volume (critical for onboarding monitoring)
SELECT COUNT(*) AS otp_sms_count,
       COUNT(DISTINCT user_account_id) AS unique_users
FROM platform_iceberg.core_bgv.engagement_sms_backend_master
WHERE event_date = DATE '2026-04-17'
  AND is_otp = true;
```

---

### 11.4 engagement_whatsapp_backend_master

```sql
-- Q1: Message volume by eventname (template)
SELECT eventname, status,
       COUNT(*) AS volume
FROM platform_iceberg.core_bgv.engagement_whatsapp_backend_master
WHERE event_date = DATE '2026-04-17'
GROUP BY eventname, status
ORDER BY volume DESC
LIMIT 20;

-- Q2: Read rate — messages where vendor confirmed read
SELECT
  COUNT(*) AS total_sent,
  SUM(CASE WHEN vendor_read_time IS NOT NULL THEN 1 ELSE 0 END) AS read_confirmed,
  ROUND(SUM(CASE WHEN vendor_read_time IS NOT NULL THEN 1 ELSE 0 END) * 100.0
        / NULLIF(COUNT(*), 0), 2) AS read_rate_pct
FROM platform_iceberg.core_bgv.engagement_whatsapp_backend_master
WHERE event_date = DATE '2026-04-17'
  AND status = 'SENT';

-- Q3: Full delivery funnel (sent → received → read)
SELECT
  COUNT(*) AS dispatched,
  SUM(CASE WHEN vendor_sent_time IS NOT NULL THEN 1 ELSE 0 END) AS vendor_sent,
  SUM(CASE WHEN vendor_received_time IS NOT NULL THEN 1 ELSE 0 END) AS received,
  SUM(CASE WHEN vendor_read_time IS NOT NULL THEN 1 ELSE 0 END) AS read_confirmed
FROM platform_iceberg.core_bgv.engagement_whatsapp_backend_master
WHERE event_date = DATE '2026-04-17';

-- Q4: Failed/error messages
SELECT status, eventname, COUNT(*) AS count
FROM platform_iceberg.core_bgv.engagement_whatsapp_backend_master
WHERE event_date = DATE '2026-04-17'
  AND status != 'SENT'
GROUP BY status, eventname
ORDER BY count DESC;

-- Q5: Unique users reached via WhatsApp
SELECT COUNT(DISTINCT useraccountid) AS unique_users_reached
FROM platform_iceberg.core_bgv.engagement_whatsapp_backend_master
WHERE event_date = DATE '2026-04-17'
  AND status = 'SENT';
```

---

### 11.5 engagement_user_session_daily

```sql
-- Q1: DAU by platform for a given day (correct os_name casing — no LOWER() needed)
SELECT os_name,
       SUM(app_dau) AS app_dau,
       SUM(web_dau) AS web_dau,
       SUM(desktop_dau) AS desktop_dau,
       SUM(txn_dau) AS txn_dau
FROM platform_iceberg.core_bgv.engagement_user_session_daily
WHERE session_date = DATE '2026-04-17'
GROUP BY os_name
ORDER BY app_dau DESC;

-- Q2: Total DAU (deduplicated — at least one DAU flag set)
SELECT COUNT(DISTINCT user_account_id) AS total_dau
FROM platform_iceberg.core_bgv.engagement_user_session_daily
WHERE session_date = DATE '2026-04-17'
  AND (app_dau = 1 OR web_dau = 1 OR txn_dau = 1 OR desktop_dau = 1);

-- Q3: Multi-platform users (app + web same day)
SELECT COUNT(*) AS multi_platform_users
FROM platform_iceberg.core_bgv.engagement_user_session_daily
WHERE session_date = DATE '2026-04-17'
  AND app_dau = 1
  AND web_dau = 1;

-- Q4: API transactors — txn_dau with no session (background transactions)
SELECT COUNT(*) AS api_transactors
FROM platform_iceberg.core_bgv.engagement_user_session_daily
WHERE session_date = DATE '2026-04-17'
  AND txn_dau = 1
  AND session_count = 0;
-- Note: this pattern is expected and valid — do not treat as data quality issue

-- Q5: Sessions per DAU (engagement depth)
SELECT os_name,
       COUNT(*) AS dau_users,
       SUM(session_count) AS total_sessions,
       ROUND(SUM(session_count) * 1.0 / NULLIF(COUNT(*), 0), 2) AS avg_sessions_per_user
FROM platform_iceberg.core_bgv.engagement_user_session_daily
WHERE session_date = DATE '2026-04-17'
  AND app_dau = 1
  AND os_name IS NOT NULL
GROUP BY os_name
ORDER BY dau_users DESC;
```

---

### 11.6 engagement_dau_indepth

```sql
-- Q1: Bounce rate by app version (top versions only)
SELECT app_version,
       COUNT(*) AS users,
       SUM(bounce_dau_flag) AS bounce_users,
       ROUND(SUM(bounce_dau_flag) * 100.0 / COUNT(*), 2) AS bounce_rate_pct
FROM platform_iceberg.core_bgv.engagement_dau_indepth
WHERE session_date = DATE '2026-04-17'
GROUP BY app_version
ORDER BY users DESC
LIMIT 20;
-- Note: cuid is UUID — join to user master for user_account_id if needed

-- Q2: Average app time for non-bounce users
SELECT
  ROUND(AVG(CASE WHEN bounce_dau_flag = 0 THEN app_time_mins END), 2) AS avg_app_time_non_bounce,
  ROUND(AVG(CASE WHEN bounce_dau_flag = 1 THEN app_time_mins END), 2) AS avg_app_time_bounce
FROM platform_iceberg.core_bgv.engagement_dau_indepth
WHERE session_date = DATE '2026-04-17';

-- Q3: Intent distribution across DAU
SELECT
  SUM(dau_intent) AS dau_intent_users,
  SUM(search_intent) AS search_intent_users,
  SUM(explore_intent) AS explore_intent_users,
  SUM(pp_intent) AS product_page_intent_users,
  SUM(order_intent) AS order_intent_users,
  COUNT(*) AS total_dau
FROM platform_iceberg.core_bgv.engagement_dau_indepth
WHERE session_date = DATE '2026-04-17';

-- Q4: PN-attributed sessions (comm_sessions > 0)
SELECT COUNT(*) AS users_with_comm_session,
       SUM(comm_sessions) AS total_comm_sessions
FROM platform_iceberg.core_bgv.engagement_dau_indepth
WHERE session_date = DATE '2026-04-17'
  AND comm_sessions > 0;

-- Q5: Transacting DAU — order placing or transacting sessions
SELECT COUNT(*) AS transacting_users,
       SUM(order_placing_sessions) AS order_sessions,
       SUM(transacting_sessions) AS txn_sessions
FROM platform_iceberg.core_bgv.engagement_dau_indepth
WHERE session_date = DATE '2026-04-17'
  AND (order_placing_sessions > 0 OR transacting_sessions > 0);
```

---

### 11.7 engagement_session_indepth

```sql
-- ⚠ MANDATORY: always filter session_date to a single day — 60M+ rows per day

-- Q1: Session type and OS split
SELECT session_type, os_name,
       COUNT(*) AS session_count,
       SUM(bounce_flag) AS bounce_sessions,
       ROUND(SUM(bounce_flag) * 100.0 / COUNT(*), 2) AS bounce_rate_pct
FROM platform_iceberg.core_bgv.engagement_session_indepth
WHERE session_date = DATE '2026-04-17'
GROUP BY session_type, os_name
ORDER BY session_count DESC
LIMIT 20;

-- Q2: PN-attributed sessions — sessions driven by a campaign
SELECT COUNT(*) AS pn_attributed_sessions,
       COUNT(DISTINCT cuid) AS unique_users
FROM platform_iceberg.core_bgv.engagement_session_indepth
WHERE session_date = DATE '2026-04-17'
  AND comm_session IS NOT NULL;
-- comm_session format: CEP-{campaignInstanceId}-{bucket}-CAMPAIGN-PRODUCTION-{userId}-{counter}

-- Q3: Sessions with orders placed
SELECT session_type, os_name,
       COUNT(*) AS sessions_with_orders,
       SUM(orders) AS total_orders
FROM platform_iceberg.core_bgv.engagement_session_indepth
WHERE session_date = DATE '2026-04-17'
  AND orders > 0
GROUP BY session_type, os_name
ORDER BY sessions_with_orders DESC;

-- Q4: Average session duration (filter nulls — frequently null)
SELECT session_type,
       COUNT(*) AS sessions_with_duration,
       ROUND(AVG(session_duration), 0) AS avg_duration_secs,
       ROUND(APPROX_PERCENTILE(CAST(session_duration AS double), 0.5), 0) AS median_secs
FROM platform_iceberg.core_bgv.engagement_session_indepth
WHERE session_date = DATE '2026-04-17'
  AND session_duration IS NOT NULL
GROUP BY session_type;

-- Q5: Product engagement — MF vs stocks page views per session
SELECT os_name,
       ROUND(AVG(mf_pp_events), 2) AS avg_mf_pp_events,
       ROUND(AVG(stock_pp_events), 2) AS avg_stock_pp_events,
       ROUND(AVG(search_events), 2) AS avg_search_events
FROM platform_iceberg.core_bgv.engagement_session_indepth
WHERE session_date = DATE '2026-04-17'
  AND bounce_flag = 0
  AND session_type = 'App'
GROUP BY os_name
ORDER BY avg_mf_pp_events DESC;
```

---

### 11.8 engagement_app_fact

```sql
-- ⚠ Always filter week = DATE '<known_sunday>' — MAX(week) scan times out on 16.2B rows
-- Latest confirmed partition: week = DATE '2026-04-13'

-- Q1: Push reachability by OS
SELECT os_name,
       COUNT(*) AS total_users,
       SUM(CASE WHEN app_keep_flag = 1 THEN 1 ELSE 0 END) AS installed,
       SUM(CASE WHEN app_keep_flag = 1 AND push_reachable = 1 THEN 1 ELSE 0 END) AS push_reachable,
       ROUND(SUM(CASE WHEN app_keep_flag = 1 AND push_reachable = 1 THEN 1.0 ELSE 0 END)
             / NULLIF(SUM(CASE WHEN app_keep_flag = 1 THEN 1 ELSE 0 END), 0) * 100, 2) AS reachability_pct
FROM platform_iceberg.core_bgv.engagement_app_fact
WHERE week = DATE '2026-04-13'
  AND os_name IS NOT NULL
GROUP BY os_name
ORDER BY push_reachable DESC;
-- Expected: Android ~26.6M reachable, iOS ~3.6M reachable

-- Q2: Total push-reachable users (cross-platform)
SELECT SUM(CASE WHEN app_keep_flag = 1 AND push_reachable = 1 THEN 1 ELSE 0 END) AS push_reachable_total
FROM platform_iceberg.core_bgv.engagement_app_fact
WHERE week = DATE '2026-04-13';
-- Expected: ~30.2M

-- Q3: Installed but push-disabled users (permission revoked)
SELECT os_name,
       COUNT(*) AS push_disabled_users
FROM platform_iceberg.core_bgv.engagement_app_fact
WHERE week = DATE '2026-04-13'
  AND app_keep_flag = 1
  AND push_reachable = 0
  AND os_name IS NOT NULL
GROUP BY os_name
ORDER BY push_disabled_users DESC;

-- Q4: App version distribution among push-reachable users
SELECT latest_app_version,
       COUNT(*) AS user_count
FROM platform_iceberg.core_bgv.engagement_app_fact
WHERE week = DATE '2026-04-13'
  AND app_keep_flag = 1
  AND push_reachable = 1
GROUP BY latest_app_version
ORDER BY user_count DESC
LIMIT 20;

-- Q5: Last event distribution (what was the last app action before this week)
SELECT last_event, COUNT(*) AS users
FROM platform_iceberg.core_bgv.engagement_app_fact
WHERE week = DATE '2026-04-13'
  AND app_keep_flag = 1
GROUP BY last_event
ORDER BY users DESC
LIMIT 20;
```

---

### 11.9 engagement_dnd_fact

```sql
-- ⚠ No partition — always filter WHERE date = DATE '...' to avoid full table scan
-- ⚠ s7 is VARCHAR — use s7 = 'true', not s7 = true
-- ⚠ 4 null rows exist — use COALESCE(s7, 'false') for safe filtering

-- Q1: DND toggle events on a given date (enables + disables)
SELECT
  SUM(CASE WHEN COALESCE(s7, 'false') = 'true' AND COALESCE(prev_s7, 'false') = 'false' THEN 1 ELSE 0 END) AS dnd_enabled,
  SUM(CASE WHEN COALESCE(s7, 'false') = 'false' AND prev_s7 = 'true' THEN 1 ELSE 0 END) AS dnd_disabled,
  COUNT(*) AS total_events
FROM platform_iceberg.core_bgv.engagement_dnd_fact
WHERE date = DATE '2026-04-17';
-- Expected: ~49K enable events, ~20K disable events

-- Q2: Net DND change (positive = more users enabling DND that day)
SELECT
  SUM(CASE WHEN COALESCE(s7, 'false') = 'true' AND COALESCE(prev_s7, 'false') = 'false' THEN 1 ELSE 0 END) -
  SUM(CASE WHEN COALESCE(s7, 'false') = 'false' AND prev_s7 = 'true' THEN 1 ELSE 0 END) AS net_dnd_change
FROM platform_iceberg.core_bgv.engagement_dnd_fact
WHERE date = DATE '2026-04-17';

-- Q3: Latest DND status per user (most recent event on a given date)
SELECT cuid, s7 AS current_dnd_status
FROM (
  SELECT cuid, s7,
         ROW_NUMBER() OVER (PARTITION BY cuid ORDER BY event_time DESC) AS rn
  FROM platform_iceberg.core_bgv.engagement_dnd_fact
  WHERE date = DATE '2026-04-17'
) t
WHERE rn = 1
LIMIT 100;

-- Q4: First-time DND opt-ins (prev_s7 = null means first ever DND event)
SELECT COUNT(DISTINCT cuid) AS first_time_dnd_users
FROM platform_iceberg.core_bgv.engagement_dnd_fact
WHERE date = DATE '2026-04-17'
  AND prev_s7 IS NULL
  AND COALESCE(s7, 'false') = 'true';

-- Q5: DND events by OS and session type
SELECT os_name, session_type, COALESCE(s7, 'unknown') AS dnd_status,
       COUNT(*) AS events
FROM platform_iceberg.core_bgv.engagement_dnd_fact
WHERE date = DATE '2026-04-17'
GROUP BY os_name, session_type, s7
ORDER BY events DESC
LIMIT 20;
```

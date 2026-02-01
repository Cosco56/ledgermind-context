# 🧰 LedgerMind — assistant_context_pack (auto)
**Generated (UTC):** 2026-01-30T15:35:20.8076257Z
**Host:** DESKTOP-08IU10H

## Fast Status
- TruthGate nowrite lock: **True** (C:\ProgramData\LM\tx\exec\locks\tx_exec_truth_gate.nowrite)
- Bridge Task: TaskName:                             \LM-Bridge-TruthGate-1m; Next Run Time:                        N/A; Status:                               Disabled; Last Run Time:                        18/01/2026 11:29:01; Last Result:                          0; Scheduled Task State:                 Disabled; Start Time:                           10:19:00
- Open Run Once Task: TaskName:                             \LedgerMind\LM-Open-Run-Once-20260120-1620; Next Run Time:                        N/A; Status:                               Ready; Last Run Time:                        20/01/2026 16:32:00; Last Result:                          0; Scheduled Task State:                 Enabled; Start Time:                           16:32:00
- DATAQ_MARKET: mw=CLOSED status=OFF allow_new_trades=False top_reason=openready_v2|market_window!=OPEN

## Fingerprints (SHA256)
- `C:\ledgermind\tools\lm-open.enable_pack.ps1` | sha256=`F4DF478CE2233CDF66D6423E96538C9DCCEC2D6FEEC5F0C1393024CF4094E182` | lastWriteUtc=2026-01-18T09:29:09.1711912Z | bytes=1070
- `C:\ledgermind\tools\lm-weekend.disable_pack.ps1` | sha256=`D3D63EF9628AFE4652F3DFA7789F6091DC2D4A855AAA3C001882A3E2FDA5B31A` | lastWriteUtc=2026-01-21T07:25:08.1209790Z | bytes=973
- `C:\ledgermind\tools\lm-open.run.ps1` | sha256=`A91DFF6B6D95BB415BDCB6ED077CA5113BD292BABB3B94359B1156200540D9B7` | lastWriteUtc=2026-01-21T08:59:17.2619042Z | bytes=3128
- `C:\ProgramData\LM\tasks\tr\LM-Open-Run-Once.ps1` | sha256=`F67201A5F9448FA5719D68A602C04326320B6D19E5ED1E20ED63E4A63F341299` | lastWriteUtc=2026-01-18T09:08:46.5903911Z | bytes=1686
- `C:\ProgramData\LM\tasks\tr\LM-Bridge-TruthGate-1m.wrap.ps1` | sha256=`EEC3FE5CD19F76858E0A16BCBDE05F4FF3D629958B0F6034B111BF09E0D3F560` | lastWriteUtc=2026-01-24T09:49:49.6215588Z | bytes=1983
- `C:\ledgermind\tools\lm-doc.write_from_clipboard.ps1` | sha256=`4921DD7FD1008B475144A2FC9FEECC70F1DEFC55D3B2A0F1059CA233A8CA13B7` | lastWriteUtc=2026-01-18T13:42:41.9485952Z | bytes=2145
- `C:\ledgermind\data\lm\docs\transfer\LedgerMind.TransferSummary.Writer.latest.md` | sha256=`EF58A1F88F505213267F566CFD5E99C2886FD55D68350A53BCC2356523F275AF` | lastWriteUtc=2026-01-18T13:09:37.9387797Z | bytes=5379

## Transfer Summary (embedded)
source: `C:\ledgermind\data\lm\docs\transfer\LedgerMind.TransferSummary.Writer.latest.md`

# 🧰 LedgerMind — Transfer Summary (Writer / Control Room)
**Update:** 2026-01-18

## 0) כללי עבודה
- Windows + PowerShell 7 (Admin כשצריך)
- בכל בלוק PS: `Set-StrictMode -Version Latest; $ErrorActionPreference='Stop'; chcp 65001 > $null`
- Fail-Closed: אם יש ספק → `NO_TRADE` / `allow_new_trades=false`
- TruthGate: Single-Writer חובה (אין writers מקבילים)

---

## 1) Root Issues שנסגרו

### 1.1 Bars delayed (~15–16 דקות) → Recency OPEN לא מוכן
- `C:\ledgermind\data\md\bars_1m.latest.csv` lag ~900–1000s ⇒ `open_recency_ready=False`
- IBKR quotes כן חי, אבל bars delayed (חשד: מקור/entitlement bars)

### 1.2 TruthGate flapping בגלל writers שונים
- `tx_exec_truth_gate.latest.json` התהפך (PASS↔FAIL/RO) בגלל writers שונים
- נסגר: lock + guards + נטרול writer חיצוני + defense-in-depth

### 1.3 Open Run Once — runner חסר (נסגר)
- Task: `\LedgerMind\LM-Open-Run-Once-20260120-1620`
- Target: `C:\ProgramData\LM\tasks\tr\LM-Open-Run-Once.ps1` היה חסר ⇒ `Last Result=64`
- נסגר: runner נוצר + לוג עובד + ריצה ידנית `Last Result=0`
- Start time עודכן ל-**16:32** (אחרי 09:30 ET)

---

## 2) מה הוטמע

### 2.1 Recency v2 (OPEN fallback + כתיבה גם ב-CLOSED)
- Writer: `C:\ledgermind\tools\lm-dataq.recency.sample_1m.ps1`
- Task: `\LedgerMind\LM-DataQ-RecencySample-1m` (SYSTEM)
- ב-CLOSED: `recency_gate.latest.json status=OFF`
- ב-OPEN: מחשב `open_recency_ready_eff` (האמת ל-OPEN)

### 2.2 DataQ Market v2 (צרכן `open_recency_ready_eff`)
- Wrap: `C:\ledgermind\tools\lm-dataq.market.refresh_wrap_1m.ps1`
- Post: `C:\ledgermind\tools\lm-dataq.market.openready_v2.post.ps1`
- `dataq_market.latest.json` נחתם תמיד עם:
  - `status` (OFF/PASS/FAIL/CAUTION)
  - `allow_new_trades`
  - `top_reason=openready_v2|...`
  - `open_recency_ready_eff`

### 2.3 Open Run/Drill + Snapshot
- `C:\ledgermind\tools\lm-open.run.ps1` כולל RTH guard: `09:30–16:00 ET`
- אם `inRTH=False` → exit 0 (אין enable_pack ואין snapshot) — צפוי
- Snapshot כש-inRTH:
  - `C:\ledgermind\tools\lm-ops.snapshot_now.ps1` →
  - `C:\ledgermind\data\lm\ops\snapshots\ops_snapshot.<ts>.json`

### 2.4 Packs
- Enable: `C:\ledgermind\tools\lm-open.enable_pack.ps1`
- Weekend disable: `C:\ledgermind\tools\lm-weekend.disable_pack.ps1`
- `\LedgerMind\LM_MD_Refresh_OPEN_1m` נשאר Disabled

### 2.5 TruthGate Single-Writer
- Lock: `C:\ProgramData\LM\tx\exec\locks\tx_exec_truth_gate.nowrite`
- Toggle: `C:\ledgermind\tools\lm-truthgate.lock.toggle.ps1`

### 2.6 External TruthGate Writer (Bridge) — סגור
- Task: `\LM-Bridge-TruthGate-1m` (wscript→vbs→cmd chain)
- Wrap: `C:\ProgramData\LM\tasks\tr\LM-Bridge-TruthGate-1m.wrap.ps1`
- Guard קיים: `LM_TRUTHGATE_NOWRITE_GUARD_v1`
  - עם lock ON ⇒ לא כותב (`GUARD_TEST changed=False`)
- Policy: Task **Disabled כברירת מחדל** כדי לא להפר Single-Writer בזמן OPEN

#### Defense-in-depth (Disable אוטומטי גם ב-Packs)
- `lm-open.enable_pack.ps1`:
  - `LM_OPEN_ENABLE_DISABLE_BRIDGE_TRUTHGATE_TASK_v1` לפני `LM_OPEN_ENABLE_UNLOCK_TRUTHGATE_v1`
  - SHA256: `F4DF478CE2233CDF66D6423E96538C9DCCEC2D6FEEC5F0C1393024CF4094E182`
  - Backup: `C:\ledgermind\tools\lm-open.enable_pack.ps1.bak.disablebridge.20260118_112909`
- `lm-weekend.disable_pack.ps1`:
  - `LM_WEEKEND_DISABLE_DISABLE_BRIDGE_TRUTHGATE_TASK_v1`
  - Backup: `C:\ledgermind\tools\lm-weekend.disable_pack.ps1.bak.disablebridge.20260118_114050`

### 2.7 Open Run Once — runner + schedule
- Runner: `C:\ProgramData\LM\tasks\tr\LM-Open-Run-Once.ps1`
- Log: `C:\ProgramData\LM\tasks\logs\LM-Open-Run-Once.log`
- Next Run: `20/01/2026 16:32:00`

---

## 3) מצב נוכחי (Weekend)
- TruthGate lock קיים (Fail-Closed) ✅
- `\LM-Bridge-TruthGate-1m` Disabled ✅
- Packs מכילים disable אוטומטי ל-Bridge ✅
- DATAQ_MARKET ב-CLOSED ⇒ `status=OFF` + `allow_new_trades=False` ✅

---

## 4) מה לעשות ביום שלישי 20/01/2026 (OPEN הבא)
> 19/01 סגור בארה״ב

- Task אמור לרוץ ב-16:32
- אחרי 16:33 לבדוק:
  - `C:\ProgramData\LM\tasks\logs\LM-Open-Run-Once.log`
  - `C:\ledgermind\data\lm\ops\snapshots\ops_snapshot.*.json`

---

## 5) בדיקות קצרות (אחרי 16:32)
```powershell
$task="\LedgerMind\LM-Open-Run-Once-20260120-1620"
$logf="C:\ProgramData\LM\tasks\logs\LM-Open-Run-Once.log"
schtasks.exe /Query /TN $task /FO LIST /V | findstr /I "Last Run Time: Last Result: Status: Next Run Time:" | Out-Host
if(Test-Path $logf){ Get-Content $logf -Tail 220 -Encoding UTF8 | Out-Host } else { "LOG missing" | Out-Host }

schtasks.exe /Query /TN "\LM-Bridge-TruthGate-1m" /FO LIST /V | findstr /I "Status: Scheduled Task State:" | Out-Host
"LOCK_PRESENT=$((Test-Path -LiteralPath 'C:\ProgramData\LM\tx\exec\locks\tx_exec_truth_gate.nowrite'))" | Out-Host

$m="C:\ledgermind\data\dataq\market\dataq_market.latest.json"
$mk=Get-Content $m -Raw -Encoding UTF8 | ConvertFrom-Json -Depth 120
"DATAQ_MARKET mw=$($mk.market_window) status=$($mk.status) allow_new_trades=$($mk.allow_new_trades) top_reason=$($mk.top_reason)" | Out-Host
```





## Update 2026-01-29 — Demo/PaperLive hardening

### מצב סגירה (Fail-Closed)

- NOWRITE_PRESENT=True

- Bridge: SKIP|truthgate_secondary|reason=NOWRITE_LOCK|mw=CLOSED


### מה הצליח היום

- Paper Window RunOnce (פתיחה קצרה + refresh+eval) הגיע ל-TruthGate ok=True + allow_new_orders=True בתוך חלון.

### Known limitation
- LM-TX-Exec-TruthRefresh-1m (NETWORK SERVICE) לא מקדם broker_truth תחת nowrite; לדמו משתמשים ב-lm-tx.exec_truth.refresh_1m.run.ps1 בתוך חלון.

- PaperLive runs/KPI נכתבים תקין (runs jsonl + kpi daily/weekly).


### שינוי קוד

- Patch ב-lm-tx.paper_live_harness.run_v1.ps1: חסימה מחוץ לחלון על nowrite_lock + super_allow fallback (allow/allow_live_orders/allow_new_orders).


### אזהרה חשובה

- lm-tx.intent.enqueue.ps1 -? **מייצר intent אמיתי** (לא help). להשתמש ב-Get-Content/VSCode לעיון.


### Fingerprints
- C:\ledgermind\tools\lm-tx.paper_live_harness.run_v1.ps1  SHA256: 0D2DD1BCF38FBA0D894E2DC6D52C6F0A36A5B5FFFB008A64AA7EE2012600E868  LastWriteUtc: 2026-01-30T15:31:02.4550628Z
- C:\ledgermind\tools\lm-tx.exec_truth_gate.eval.ps1  SHA256: 55BE4BD6D0B2B7DC7C365E29E064A2129B6125227CF43D004C8B7FB5BDF72CE9  LastWriteUtc: 2026-01-27T16:45:05.2376559Z
- C:\ledgermind\tools\lm-tx.exec_truth.refresh_1m.run.ps1  SHA256: 439457173491459F68B81303146361DD6458C25A12E21948186E702C85E2705E  LastWriteUtc: 2026-01-24T09:49:56.8084649Z
### Tasks רלוונטיים
- \LedgerMind\LM-TX-Exec-TruthGate-Locked-1m | Status:                               Ready | Last Run Time:                        29/01/2026 23:32:01 | Last Result:                          0 | Next Run Time:                        29/01/2026 23:33:00 | Run As User:                          SYSTEM
- \LedgerMind\LM-TX-Exec-TruthRefresh-1m | Status:                               Ready | Last Run Time:                        29/01/2026 23:32:01 | Last Result:                          0 | Next Run Time:                        29/01/2026 23:33:00 | Run As User:                          NETWORK SERVICE
- \LedgerMind\LM-TX-PaperLive-5m | Status:                               Ready | Last Run Time:                        29/01/2026 23:32:01 | Last Result:                          0 | Next Run Time:                        29/01/2026 23:37:00 | Run As User:                          SYSTEM
- \LedgerMind\LM-TX-PaperLive-OnStart | Status:                               Ready | Last Run Time:                        02/01/2026 10:32:41 | Last Result:                          0 | Next Run Time:                        N/A | Run As User:                          SYSTEM
- \LedgerMind\LM-TX-IntentWorker-1m | Status:                               Ready | Last Run Time:                        29/01/2026 23:32:01 | Last Result:                          0 | Next Run Time:                        29/01/2026 23:33:00 | Run As User:                          SYSTEM
- \LedgerMind\LM-TX-IntentRuntime-1m | Status:                               Ready | Last Run Time:                        29/01/2026 23:32:01 | Last Result:                          0 | Next Run Time:                        29/01/2026 23:33:00 | Run As User:                          SYSTEM
- \LedgerMind\LM-TX-IntentStateMachine-1m | Status:                               Ready | Last Run Time:                        29/01/2026 23:32:01 | Last Result:                          0 | Next Run Time:                        29/01/2026 23:33:00 | Run As User:                          SYSTEM
- \LedgerMind\LM-TX-Live-ReadinessSuperGate-1m | Status:                               Ready | Last Run Time:                        29/01/2026 23:31:58 | Last Result:                          0 | Next Run Time:                        29/01/2026 23:32:57 | Run As User:                          SYSTEM
- \LedgerMind\LM-Bridge-TruthGate-1m | Status:                               Ready | Last Run Time:                        29/01/2026 23:32:11 | Last Result:                          0 | Next Run Time:                        29/01/2026 23:33:10 | Run As User:                          SYSTEM






### Gates mirror (LiveReadinessGate)
- Source-of-truth: C:\ledgermind\data\tx\gates\live_readiness_gate.latest.json
- Mirror: C:\ledgermind\data\lm\gates\live_readiness_gate.latest.json via Task \LedgerMind\LM-Gates-Mirror-LiveReadiness-1m (SYSTEM)
  - Script: C:\ProgramData\LM\ops\gates\mirror_live_readiness_gate_1m.ps1
  - Log: C:\ProgramData\LM\tasks\logs\LM-Gates-Mirror-LiveReadiness-1m.log




## Update 2026-01-30 — Pre-Open readiness
- mw(Bridge): SKIP|truthgate_secondary|reason=NOWRITE_LOCK|mw=CLOSED
- nowrite_present: True
- port_4002_ok: True

### Tasks (key)
- \LedgerMind\LM-Gates-Mirror-LiveReadiness-1m | Status:                               Ready | Last Run Time:                        30/01/2026 11:04:01 | Last Result:                          0 | Run As User:                          SYSTEM
- \LedgerMind\LM-Demo-OpenOnly-RunOnce-1m | Status:                               Ready | Last Run Time:                        30/01/2026 11:04:01 | Last Result:                          0 | Run As User:                          SYSTEM
- \LedgerMind\LM-TX-PaperLive-5m | Status:                               Ready | Last Run Time:                        30/01/2026 11:02:01 | Last Result:                          0 | Run As User:                          SYSTEM
- \LedgerMind\LM-TX-IntentWorker-1m | Status:                               Ready | Last Run Time:                        30/01/2026 11:04:01 | Last Result:                          0 | Run As User:                          SYSTEM

### Fingerprints
- C:\ledgermind\tools\lm-tx.paper_live_harness.run_v1.ps1  SHA256: 0D2DD1BCF38FBA0D894E2DC6D52C6F0A36A5B5FFFB008A64AA7EE2012600E868  LastWriteUtc: 2026-01-30T15:31:02.4550628Z
- C:\ProgramData\LM\ops\gates\mirror_live_readiness_gate_1m.ps1 | sha256=D83B5BF40B78383EFC3EAF3B577AE392216E588CB577F8B624DC281240B920F4 | lastWriteUtc=2026-01-30T07:23:15.6309443Z
- C:\ProgramData\LM\ops\demo\demo_open_only_runonce.ps1 | sha256=8DB428C1E84170AF6BED2655BFE7B87A76425A43312A7CBF0E64B775F37F0857 | lastWriteUtc=2026-01-30T08:06:18.3758928Z



## Update 2026-01-30 — Demo stability: super_allow_false fixed (final)
- harness_sha256: 0D2DD1BCF38FBA0D894E2DC6D52C6F0A36A5B5FFFB008A64AA7EE2012600E868
- harness_lastWriteUtc: 2026-01-30T15:31:02.4550628Z
- Fix: superAllow fallback reads SuperGate file (pass→root) when allow_live_orders=True; eliminates super_allow_false in window runs.



## Update 2026-01-30 — Demo: Submit RunOnce scaffolded
- Submit script created: C:\ProgramData\LM\ops\demo\demo_submit_runonce.ps1 (sha below)
- Submit task: \LedgerMind\LM-Demo-Submit-RunOnce-1m (SYSTEM) **DISABLED by default**
- OpenOnly remains enabled: \LedgerMind\LM-Demo-OpenOnly-RunOnce-1m

### Fingerprints (demo scripts)
- C:\ProgramData\LM\ops\demo\demo_submit_runonce.ps1 | sha256=91B734F4D3F14C6DD6951E523F4CABD08E72F6B4FC99E0F49CC303E678475597 | lastWriteUtc=2026-01-30T16:04:28.9373839Z | bytes=2841
- C:\ProgramData\LM\ops\demo\demo_open_only_runonce.ps1 | sha256=D3E0CC27711FE3810A8E80D8AF872E0B084453E1638FFE5718A24FB164DD344B | lastWriteUtc=2026-01-30T16:04:24.5301982Z | bytes=3399



## Update 2026-01-30 — Demo paper submit success
- submit_attempted: True
- submit_ok: True
- run_id: e67bbbd88dac48f8a336e55278199641
- coid: plv1|e67bbbd88dac48f8a336e55278199641
- ts_utc: 01/30/2026 19:31:21
- tool: C:\ledgermind\tools\lm-broker.submit.ps1


## Update 2026-01-30 — Demo OneShot Submit OK
- script: C:\ProgramData\LM\ops\demo\demo_one_shot_submit.ps1
- script_sha256: DD6E1BA2BA4D8A1AF140B633370EB093E904091AB7A2B8884682CAE6DBBD698E
- script_lastWriteUtc: 2026-01-30T19:40:55.6799835Z
- task: \LedgerMind\LM-Demo-OneShot-Submit-RunNow (SYSTEM, Disabled by default)
- log: C:\ProgramData\LM\tasks\logs\LM-Demo-OneShot-Submit.log
- result: 2026-01-30T19:46:07.0956476Z DEMO_ONESHOT_OK run_id=e065cb772af048559df4228b8b5d276c coid=plv1|e065cb772af048559df4228b8b5d276c


## Update 2026-01-30 — Demo Clean achieved (paper_orphans cleared)
- truth_gate: status=pass ok=True allow_new_orders=False reduce_only_mode=False
- unknown_open_orders_danger: 0
- note: paper_orphans autocancel succeeded (before=2 after=0); TruthGate now PASS


## Update 2026-01-30 — Demo OneShot Clean+Submit scaffolded
- script: C:\ProgramData\LM\ops\demo\demo_one_shot_clean_submit.ps1
- script_sha256: 1831A0ED6B272E5EE08285D6212B7A247D25F0DE01F7709BCF9F73718BEC49BE
- script_lastWriteUtc: 2026-01-30T20:14:54.7560418Z
- task: \LedgerMind\LM-Demo-OneShot-CleanSubmit-RunNow (SYSTEM, Disabled by default)
- log: C:\ProgramData\LM\tasks\logs\LM-Demo-OneShot-CleanSubmit.log
- flow: reset_orphans_lock -> autocancel -> refresh+eval verify clean -> submit -> verify attempted/ok -> relock


## Update 2026-01-30 — Demo OneShot Clean+Submit OK
- result: 2026-01-30T20:18:56.2072429Z DEMO_CLEANSUBMIT_OK run_id=47c1d05d962b4a5dbfe892cb7bb9299f coid=plv1|47c1d05d962b4a5dbfe892cb7bb9299f


## Update 2026-01-30 — Demo OneShot Clean+Submit lockfix
- script: C:\ProgramData\LM\ops\demo\demo_one_shot_clean_submit.ps1
- script_sha256: 78F1994F0E0FF7114257D387DE4E7B3CF3B82B93E246D555BB13FFC1014E6C80
- script_lastWriteUtc: 2026-01-30T20:29:21.7571325Z
- change: removed creation of persistent paper_orphans lock file (prevents LOCK_BUSY)


## Update 2026-01-30 — Demo OneShot Clean+Submit post-clean enabled
- script: C:\ProgramData\LM\ops\demo\demo_one_shot_clean_submit.ps1
- script_sha256: 42A974D0423DBBB3DFF8D1DD831B7E012B439FA2971C14FDFA3D1844D3AC5E8E
- script_lastWriteUtc: 2026-01-30T20:40:44.5303137Z
- change: added post-clean (autocancel+refresh+eval after submit) to leave paper demo clean (uod=0) automatically


## Update 2026-01-30 — Demo Clean+Submit FINAL OK (post-clean verified)
- result: 2026-01-30T20:49:18.4893258Z DEMO_CLEANSUBMIT_OK run_id=f81d7fd1f8a9486a91c1e21a404dac18 coid=plv1|f81d7fd1f8a9486a91c1e21a404dac18
- post_clean: 2026-01-30T20:49:18.4479901Z POST_CLEAN_CHECK ok=True status=pass uod=0 reasons=




<!-- LM_CTX_UPDATE_DEMO_CLEANSUBMIT_V2_20260131 -->
### Demo Clean+Submit — Patch V2 — 2026-01-31
- demo_one_shot_clean_submit.ps1: Seed-if-missing enabled [LM_PATCH_SEEDPAYLOAD_V1]
- demo_one_shot_clean_submit.ps1: Post-clean loop MaxAttempts=3 + POST_CLEAN_OK attempts=... [LM_PATCH_POST_CLEAN_LOOP_V1]
  - sha256: 6D6B7F2934D39FE336ECD33BB7F45AAC8F89C6C09AC412CE2479AE5B3D08DAF5
- demo_shutdown.ps1 created + Task \LedgerMind\LM-Demo-Shutdown-RunNow (SYSTEM, Disabled default)
  - sha256: 663EB9A28C298AF67D597AA5614879A6DFD3F7D8BC7667C13F5E7D0766C617EE
- \LedgerMind\LM-Demo-OpenOnly-RunOnce-1m disabled (prevent auto windows)
<!-- /LM_CTX_UPDATE_DEMO_CLEANSUBMIT_V2_20260131 -->

<!-- LM_CTX_UPDATE_DEMO_SHUTDOWN_PARAMFIX_V1_20260131 -->
### Demo Shutdown — Param Fix — 2026-01-31
- demo_shutdown.ps1: moved param() to top (fix -WhatIf under StrictMode) [LM_PATCH_SHUTDOWN_PARAM_TOP_V1]
  - sha256: 589DD592168C647109055EDF5BF21EBA0F41314D5257DE950CD4BE55389953CB
<!-- /LM_CTX_UPDATE_DEMO_SHUTDOWN_PARAMFIX_V1_20260131 -->

<!-- LM_CTX_UPDATE_PAPERLIVE_HARNESS_BOID_V1_20260131 -->
### PaperLive Harness — broker_order_id in run record — 2026-01-31
- lm-tx.paper_live_harness.run_v1.ps1: persist submit.broker_order_id (from lm_broker_submit.log) [LM_PLV1_SUBMIT_BOID_V1]
  - sha256: 53960847CD4D8E344D9CBC1EA0A84C33EA21D58FA00FB731F352E97DE9311633
<!-- /LM_CTX_UPDATE_PAPERLIVE_HARNESS_BOID_V1_20260131 -->

<!-- LM_CTX_UPDATE_PAPERLIVE_HARNESS_BOID_V2_20260131 -->
### PaperLive Harness — broker_order_id from tool output — 2026-01-31
- prefer submit tool output for broker_order_id; log is fallback [LM_PLV1_SUBMIT_BOID_V2_OBJOUT]
  - sha256: B139A948358F7BA4AD8BB8048A6CE4510C1A270F0693671647F0F18FFE0A7204
<!-- /LM_CTX_UPDATE_PAPERLIVE_HARNESS_BOID_V2_20260131 -->

<!-- LM_CTX_UPDATE_DEMO_SEED_ROBUST_V2_20260131 -->
### Demo Clean+Submit — Seed Robust + Template Shape — 2026-01-31
- demo_one_shot_clean_submit.ps1: seed robust (force array for templates; unique idempotency_key) [LM_PATCH_SEEDPAYLOAD_V1_ROBUST_V2]
  - sha256: 032FB6EEA632E91FDDBC545CFABCDD15A2335650BAA907A8C9188256CF7B527A
- order_payload.template.json: normalized minimal order shape (symbol/side/qty/order_type/limit_price)
  - sha256: 064467CC97941BB4FF7307F19EF8ABA696BA649AE9436619BECDF922F724F281
<!-- /LM_CTX_UPDATE_DEMO_SEED_ROBUST_V2_20260131 -->

<!-- LM_CTX_UPDATE_DEMO_SEED_IDK_UNIQ_V3_20260131 -->
### Demo Clean+Submit — Seed idempotency_key unique — 2026-01-31
- demo_one_shot_clean_submit.ps1: ensure idempotency_key unique on seed [LM_PATCH_SEEDPAYLOAD_V1_IDK_UNIQ_V3]
  - sha256: C2807B35615FA9A06F6E5A0288CEC206AA4E58B3CA0CFE494689182B8F0AC997
- order_payload.template.json (reference): sha256 064467CC97941BB4FF7307F19EF8ABA696BA649AE9436619BECDF922F724F281
<!-- /LM_CTX_UPDATE_DEMO_SEED_IDK_UNIQ_V3_20260131 -->

<!-- LM_CTX_UPDATE_ATTRIB_BROKERFILLS_TRFIX_V1_20260131 -->
### Attrib BrokerFills — Task exitcode fix — 2026-01-31
- Task \LedgerMind\LM-TX-Attrib-BrokerFills-5m: Task To Run changed to cmd.exe /c wrap.cmd (bypass vbs exitcode drift)
- wrap rewritten clean (rc propagated) sha256: BAD9AD574453E008B764C201496E60C660D6CAEEC581588845A3A6467EB34BBA
<!-- /LM_CTX_UPDATE_ATTRIB_BROKERFILLS_TRFIX_V1_20260131 -->

<!-- LM_CTX_UPDATE_AUTONOMY_MASTER_V1_20260131 -->
### Autonomy Master — observe (fail-closed) — 2026-01-31
- master script: C:\ledgermind\tools\lm-autonomy.master.ps1 sha256: BE4BFC9FB44150A3EC127133FB4A38394D49E9F72FF92EB56B21729644F7F307
- task: \LedgerMind\LM-Autonomy-Master-5m (SYSTEM, every 5m)
- mode file: C:\ledgermind\data\tx\autonomy\autonomy_mode.latest.json (mode=observe)
- safety: nowrite enforced; demo submit tasks forced disabled
<!-- /LM_CTX_UPDATE_AUTONOMY_MASTER_V1_20260131 -->

<!-- LM_CTX_UPDATE_AUTONOMY_MASTER_V2_20260131 -->
### Autonomy Master — v2 (log + state always, exit 0) — 2026-01-31
- master: C:\ledgermind\tools\lm-autonomy.master.ps1 sha256: 85C8383CB59E8E87B4DAE83E8E737479B787F313BF15FC9F195EA01A06C59393
- state: C:\ProgramData\LM\ops\autonomy\autonomy_state.latest.json
- log:   C:\ProgramData\LM\ops\autonomy\autonomy_master.log
<!-- /LM_CTX_UPDATE_AUTONOMY_MASTER_V2_20260131 -->

<!-- LM_CTX_UPDATE_AUTONOMY_MASTER_V4_20260131 -->
### Autonomy Master — v4 (observe, fail-closed) — 2026-01-31
- master: C:\ledgermind\tools\lm-autonomy.master.ps1 sha256: 2C3F7A5DF78A5E7AD0662C57F8936614C8C19AE96541AF6DFF25182CD334457F
- task: \LedgerMind\LM-Autonomy-Master-5m (SYSTEM, every 5m) last_result=0
- state: C:\ProgramData\LM\ops\autonomy\autonomy_state.latest.json
- mode: C:\ledgermind\data\tx\autonomy\autonomy_mode.latest.json (observe)
- safety: nowrite enforced; demo submit tasks forced disabled
<!-- /LM_CTX_UPDATE_AUTONOMY_MASTER_V4_20260131 -->

<!-- LM_CTX_UPDATE_AUTONOMY_PAPER_V5_20260131 -->
### Autonomy — PAPER v5 (arming + oneshot + auto-disarm) — 2026-01-31
- master: C:\ledgermind\tools\lm-autonomy.master.ps1 sha256: FEF9DAA70E1E58AC3214D688F88F4E538438BA6B4D8E79D82577E0B9626FAF3B
- arming:  C:\ledgermind\data\tx\exec\paper_submit_armed.latest.json sha256: 0AF0411DBD7B81248C2FF7E5D8CF73F8DB91169CDAF193D0DDA4BB0B59F40AA8 (armed=false, consumed_result=0)
- oneshot: \LedgerMind\LM-Demo-OneShot-CleanSubmit-RunNow last_result=0
<!-- /LM_CTX_UPDATE_AUTONOMY_PAPER_V5_20260131 -->

<!-- LM_CTX_UPDATE_AUTONOMY_ARM_PAPER_RUNONCE_V3_20260131 -->
### Autonomy — ARM_PAPER RunOnce v3 (wait + summary) — 2026-01-31
- runonce: C:\ledgermind\tools\lm-autonomy.arm_paper.runonce.ps1 sha256: 09D53C60529BCC83429F031F65B1C3C321C8F5439A1C9643A2E7ACF563144DE3
- behavior: creates arming token (TTL), runs master once, waits for completion, prints final state + auto-disarm
<!-- /LM_CTX_UPDATE_AUTONOMY_ARM_PAPER_RUNONCE_V3_20260131 -->

<!-- LM_CTX_UPDATE_AUTONOMY_POLICY_OBSERVE_RUNONCE_RESTORE_V1_20260131 -->
### Autonomy Policy — Observe Always + Paper via RunOnce — 2026-01-31
- autonomy_mode.latest.json: forced back to observe after RunOnce (policy #1) sha256: BDDECDA55CC78EB31FFE330342523A073C73FB23E2DA990A8F0868EA3B03A14A
- runonce: lm-autonomy.arm_paper.runonce.ps1 restores observe in finally sha256: 1A08CFB036397CE79BE7B71E1586E50689BB627CB361B48AF28B87C8F833F673
<!-- /LM_CTX_UPDATE_AUTONOMY_POLICY_OBSERVE_RUNONCE_RESTORE_V1_20260131 -->

<!-- LM_CTX_UPDATE_PAPER_PULSE_GATEBURN_V1_20260131 -->
### Daily Paper Pulse — gate-burn one-shot — 2026-01-31
- task: \LedgerMind\LM-Autonomy-PaperPulse-0905 (SYSTEM, Mon-Fri 09:05)
- wrap: C:\ProgramData\LM\tasks\tr\wrap\LedgerMind__LM-Autonomy-PaperPulse-0905.wrap.cmd
  - sha256: EF1ACAE5D649841E0F0E4D96D1FF952655E8F4CF7877B6A6C2C52AF1504E7272
- gate: C:\ledgermind\data\tx\autonomy\paper_pulse.enabled
  - missing => SKIP rc=0
  - present => DELETE gate immediately (one-shot burn) then run RunOnce (TTL 15m, MaxRuns=1)
- log: C:\ProgramData\LM\tasks\logs\LM-Autonomy-PaperPulse-0905.log
<!-- /LM_CTX_UPDATE_PAPER_PULSE_GATEBURN_V1_20260131 -->

<!-- LM_CTX_UPDATE_PAPER_PULSE_AUTOARM_V1_20260131 -->
### Daily Paper Pulse — AutoArm (09:04) + Pulse (09:05) — 2026-01-31
- autoarm script: C:\ProgramData\LM\ops\autonomy\paper_pulse.autoarm.ps1
  - sha256: 0DFC6DB4D954501E91BF684B52F651EBD9741CF79ABEF93AD9A4AEE7683E4D83
- task: \LedgerMind\LM-Autonomy-PaperPulse-AutoArm-0904 (SYSTEM, Mon-Fri 09:04)
  - Task To Run: pwsh.exe -File paper_pulse.autoarm.ps1 (no quoting drift)
- task: \LedgerMind\LM-Autonomy-PaperPulse-0905 (SYSTEM, Mon-Fri 09:05)
  - burns gate then runs RunOnce (TTL 15m, MaxRuns=1), restores observe
- gate: C:\ledgermind\data\tx\autonomy\paper_pulse.enabled
  - created by AutoArm
  - deleted by Pulse (one-shot burn)
<!-- /LM_CTX_UPDATE_PAPER_PULSE_AUTOARM_V1_20260131 -->

<!-- LM_EOD_20260131 -->
### EOD Snapshot — 20260131
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260131.md
  - sha256: A62FB5321D8FCE1A503C4211CF0230E2FFB82AD9EC77436678A7FE3CA2EBE82C
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260131 -->

<!-- LM_EOD_20260201 -->
### EOD Snapshot — 20260201
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260201.md
  - sha256: 896C1142E98EF1DE87E79F59CC9F27B1ED38BEB2A175EDED9751C4494387C21B
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260201 -->

<!-- EOD_CHAIN_KNOWN_GOOD_v1 BEGIN -->
## EOD Chain Known-Good (EOD_CHAIN_KNOWN_GOOD_v1)

updated_utc: 2026-02-01T18:13:45.1871645Z

onstart_delays:
- PackUpdate-OnStart: PT2M
- ReportLatest-OnStart: PT10M
- UploadKit-OnStart: PT12M
- GitHubSync-OnStart: PT14M

known_good_sha256:
- lm-eod.github.sync.ps1: B9AB7B3A6467CB1BA55116BAEC3784AC29B63AC2050F55CD604FFEFA8EFA5C33
- lm-eod.uploadkit.ps1: E5F78ECE9B8EF35C350AC4501478B24C37B98DF7FA6C9724A60C12E80D347868
- lm-eod.report.latest.refresh.ps1: 0E8C6C1843FE5343499E629CEA48D91207A57A48F39C2C8519FB77B9F9DE6308
<!-- EOD_CHAIN_KNOWN_GOOD_v1 END -->


## CR_DAILY v1 — 2026-02-23 (UTC=2026-02-23T11:17:02.2746729Z)

### Core Status
- NOWRITE_PRESENT=True
- IBGW_RUNNING=True 4002_LISTEN=True (pid=34212 session=1)
- DATAQ: mw=CLOSED dhs=PASS allow=False reason=market_window=CLOSED
- PRICES: ok=True ageMin=18.7655601 miss=0
- TRUTH: allow_new_orders=False reduce_only=True reason=nowrite_lock
- RISK: ok=True status=pass reasons= trade_days_30d=13
- MLR: ok=True

### System
- CPU=Intel(R) Core(TM) i7-14700F Cores=20 LP=28
- RAM_GB=128
- Realtek=1 Gbps Status=Up
- Power=Power Scheme GUID: 17bccb59-a850-4fd0-b768-b162ef063876  (Ultimate Performance (LM))

### Volumes
- B: freeGB=4 freePct=99.4 label=BIOS
- C: freeGB=149 freePct=16 label=
- D: freeGB=586.3 freePct=63.2 label=DATA
- E: freeGB=1584 freePct=85.1 label=LM-Data

### Tasks
- RunningCount=4
- Fails24hCount=0

### Notes (manual)
# NOTES
- (תוסיף ידנית: החלטות, חריגים, מה שחשוב לזכור)

## 2026-02-14 Session Summary (imported)
- NO_TRADE / Fail-Closed נשמר.
- NOWRITE latch: C:\ProgramData\LM\tx\exec\locks\tx_exec_truth_gate.nowrite
- Toggle:
  - lm-weekend.disable_pack.ps1 => lock ON
  - lm-open.enable_pack.ps1     => lock OFF (הופעל בטעות; הוחזר ל-ON)
- Disabled set via weekend pack + manual: LM-TX-GoNoGo-Chain-1m
- NOWRITE Creation(local)=2026-02-12 21:48:02; LastWrite(local)=2026-02-13 16:17:49

## 2026-02-14 Infra Upgrade (CR->GitHub + Chat Bundle)
- CR_DAILY.latest.md/json נבנה תקין.
- EOD Orchestrator משכפל CR ל-repo: context_packs\control_room\CR_DAILY.latest.md/.json + ארכיון יומי.
- GitHubSync היה Disabled והופעל: \LedgerMind\LM-EOD-GitHubSync-2358 Enabled=True, LastResult=0.
- LM_CHAT_CONTEXT_BUNDLE.latest.md נוצר אוטומטית + Copy-to-Clipboard + פתיחת Explorer לגרירה.
- Hardening Tasks: MultipleInstancesPolicy=IgnoreNew, ExecutionTimeLimit=PT2H.
- Note: Files panel בצ'אט לא מתעדכן אוטומטית מ-GitHub; לכן לגרור LM_CHAT_CONTEXT_BUNDLE.latest.md לעדכון מצב.

## Session Summary (imported) — 2026-02-15

(הדבק כאן את הסיכום מההודעה שלי למעלה — או תשאיר כמו שזה אם אתה מעדיף מינימלי)

## סיכום חדר בקרה — 15/02/2026 (LedgerMind) — CANARY + CR_DAILY + Fail-Stack Fixes

### מצב סופי (PASS)

* **Fail-Closed:** `NOWRITE_PRESENT=True` ✅
  * `mw=CLOSED`, `allow_new_trades=False` ✅
* **CR_DAILY:** `fails24h=0` ✅ (אמת־זמן ע״י injector)
* **CANARY READY 10 DAYS:** `CANARY_DAILY_OK=True`, `READY_NOW=False`, `DUE_UTC=2026-02-25T06:08:02Z` ✅
* **Micro Canary Split (SAFE):**
  * `ArmAuto`: `arm=True`, `disarm=False`, `nowrite=True`, `allow_new=False`, `lr=pass`, `mw=CLOSED` ✅
  * `Start-Safe`: `ok=True executed=False reason=market_window_not_OPEN:CLOSED` ✅ (צפוי כשסגור)

---

### מה הושלם (Highlights)

1. **PricesGate overlap/timeout**
* זוהתה ריצה חופפת (Event 322) + LastResult חריג.
* הוחל Patch: `ExecutionTimeLimit=PT2M`, `AllowHardTerminate=True` + בדיקות overlap ⇒ **נפתר**.

2. **CANARY READY 10 DAYS**
* נוצרו/עודכנו קבצים:
  * `C:\ProgramData\LM\ops\canary\canary_ready_10d.start.json`
  * `C:\ProgramData\LM\ops\canary\canary_daily.latest.json`
* נוצר Task: `\LedgerMind\LM-Canary-Ready10D-Daily` (Daily 08:20 + AtStartup, RunAs SYSTEM)
* בדיקות: NOWRITE + LiveReadiness + Health ages ⇒ **PASS**.

3. **Micro Canary Split — שרשרת אוטונומית מלאה**
* נמצא שחסרים סקריפטי canary start/stop תחת `C:\ledgermind\tools` ושוחזרו מגיבויים (hash-verified).
* תוקנו משתני מצב בסקריפט start (init של `*_status`) כדי למנוע StrictMode crashes ⇒ start/stop ידני **PASS**.
* נבנה wrapper בטוח:
  * `C:\ProgramData\LM\ops\canary\micro_canary_split.safe.ps1`
  * יוצר `micro_canary_split.latest.json` + `micro_canary_split.summary.latest.txt`
* נבנה ArmAuto:
  * `C:\ProgramData\LM\ops\canary\micro_canary_split.arm_auto.ps1`
  * Task: `\LedgerMind\LM-TX-Micro-CanarySplit-ArmAuto-1m`
  * outputs: `micro_canary_split.arm_auto.latest.json` + `micro_canary_split.arm_auto.summary.latest.txt`
* Tasks SAFE נוצרו תחת `\LedgerMind\`:
  * `LM-TX-Micro-CanarySplit-Start-Safe` (Daily 17:00 + RI 1 / DU 01:00)
  * `LM-TX-Micro-CanarySplit-Stop-Safe` (Daily 18:00)
  * `LM-TX-Micro-CanarySplit-Stop-Safe-OnStart` (OnStart)
* KillSwitch/Override:
  * `micro_canary_split.disarm` (override) + `micro_canary_split.arm`

4. **CR_DAILY + Chat Bundle Injection (Micro Canary)**
* נכתב Injector:
  * `C:\ProgramData\LM\ops\canary\micro_canary_split.cr_daily.inject.ps1`
  * מוסיף בלוק `<!-- LM_AUTO:MICRO_CANARY_BEGIN -->` ל־`CR_DAILY.latest.md` + `extensions.micro_canary_split` ב־`CR_DAILY.latest.json`
* נכתב Injector לבאנדל:
  * `C:\ProgramData\LM\ops\canary\micro_canary_split.chat_bundle.inject.ps1`
  * מוסיף אותו בלוק ל־`LM_CHAT_CONTEXT_BUNDLE.latest.md` (וגם לקובץ timestamp האחרון)
* חובר אוטומטית לשרשרת:
  * `lm-cr.githubsync.run.ps1` → מריץ injector לפני sync/bundle
  * `lm-chat.context.bundle.build.ps1` → מריץ bundle injector

5. **סגירת Fails24hCount=4 → 0 (אופרטיבי)**
* `LM-Health-FullAudit-DRBadge-Inject-5m`: תוקן Task XML: `-TimeoutSec 120` ⇒ `LastResult=0` ✅
* `LM-Health-FullAudit-RetentionPrune-Daily`: שוחזר `lm-hygiene.retention.prune.ps1` + guard למערכים ריקים ⇒ `LastResult=0` ✅
* `LM-IBGW-SessionGuard-5m`: תוקן Soft-Skip offhours/STOP (מנע ParserError) ⇒ `LastResult=0` ✅
* `LM-IBGW-Start-User-OnLogon`: wrapper בטוח/skip כש-mw≠OPEN ⇒ `LastResult=0` ✅

6. **Fails24h במודל “אמת בזמן אמת”**
* שודרג Injector CR כך ש־`tasks_fails_24h` נקבע לפי ScheduledTaskInfo בזמן אמת + סינון סטטוסים.
* תוצאה: `FAILS_NOW count=0` ו־`CR fails24h=0` ✅

---

### פתוחים (לא חוסם)

* `ibgw_4002_listen=false` בזמן `mw=CLOSED` — צפוי; לא חוסם (Fail-Closed).
* ROI להמשך: PreOpen/OPEN kit לניהול IBGW/4002 לפי חלון מסחר בלי false-fails.

DATAQ_CONTRACT ok=True status=pass missing=0 suffixUS=0 dup=0

### 0.1 LM_RISK_CHARTER v1 (Production 100K)
- **MaxDD=15%**
- **Zones:**
  - ירוק 0–5%: רגיל
  - צהוב 5–10%: ↓סיכון ~50% + Freeze Production
  - כתום 10–15%: Risk-Off; רק מוכח; ספק ב-Data/Truth/Execution ⇒ NO_TRADE
  - אדום ≥15%: Hard Stop + RCA; חזרה רק אחרי PASS
- **Time Stops:** יומי -0.8% עצירה ליום | שבועי -2.5% Risk-Off לשבוע | חודשי -5% כניסה לצהוב + Freeze Production
- **PASS Gates (לפני סקליילינג/חזרה):**
  - OPS: הכל טרי בתוך TTL; אחרת NO_TRADE
  - TRUTH: התאמה מול ברוקר; פער לא מוסבר ⇒ חסימה
  - EXECUTION: Reject/Duplicate/Slippage/Latency בתוך "תקציב ביצוע"; חריגה ⇒ הורדת סיכון/עצירה
  - EDGE: נטו חיובי אחרי עלויות על דגימה מספקת; Paper↔Micro-Live דומים; אחרת Research בלבד
- **TTL v0 (שמרני):** 1m>150s | 5m>12m | 15m>35m | daily>36h ⇒ OPS FAIL
- **Execution Budget v0:** Duplicates=0 (עצירה מיידית) | Rejects חוזרים ⇒ הורדת סיכון/עצירה | Slippage/Latency חריגים סדרתית ⇒ Risk-Off/עצירה
- **עיקרון:** Production יציב / Research חופשי; סקליילינג במדרגות (5K→10K→25K→50K→100K)

### 0.2 שינוי מהותי שמאפס הוכחה
- שינוי מקור דאטה / הגדרות ברוקר / Order Types
- שינוי Universe / Timeframe / טריגרי כניסה-יציאה
- שינוי מנוע ביצוע / Routing / מודל עלויות
- שינוי חוקי סיכון (חשיפה/גודל/עצירות)
=> כל שינוי מהותי מחזיר ל-Research עד PASS מחדש.

# 🧰 LedgerMind — assistant_context_pack (auto)
**Generated (UTC):** 2026-02-02T07:00:00.2640163Z
**Host:** DESKTOP-08IU10H

## Fast Status
- TruthGate nowrite lock: **True** (C:\ProgramData\LM\tx\exec\locks\tx_exec_truth_gate.nowrite)
- Bridge Task: TaskName:                             \LM-Bridge-TruthGate-1m; Next Run Time:                        N/A; Status:                               Disabled; Last Run Time:                        18/01/2026 11:29:01; Last Result:                          0; Scheduled Task State:                 Disabled; Start Time:                           10:19:00
- Open Run Once Task: TaskName:                             \LedgerMind\LM-Open-Run-Once-20260120-1620; Next Run Time:                        N/A; Status:                               Ready; Last Run Time:                        20/01/2026 16:32:00; Last Result:                          0; Scheduled Task State:                 Enabled; Start Time:                           16:32:00
- DATAQ_MARKET: mw=CLOSED status=OFF allow_new_trades=False top_reason=openready_v2|market_window!=OPEN

## Fingerprints (SHA256)
- `C:\ledgermind\tools\lm-open.enable_pack.ps1` | sha256=`80DBA23BFC051B43C845D2C1B015EE0F74E86F8166EF8F0F92882964071F6002` | lastWriteUtc=2026-01-30T16:42:42.8003486Z | bytes=1141
- `C:\ledgermind\tools\lm-weekend.disable_pack.ps1` | sha256=`A3935AE39F783789014232BD84B99C0A1D47D6C9880BD25400A947301ACC33CE` | lastWriteUtc=2026-01-30T16:42:47.8595957Z | bytes=1044
- `C:\ledgermind\tools\lm-open.run.ps1` | sha256=`E51CD14A214E015ABE85661178B3EADF69D4137087F469D84C3D328F7FABE7D3` | lastWriteUtc=2026-01-30T16:42:42.8562381Z | bytes=3199
- `C:\ProgramData\LM\tasks\tr\LM-Open-Run-Once.ps1` | sha256=`25BEE153C106D6AB5B81E9F1D117A07FFBA4CBBCBCDEB2A2CC25A640E9ED82F8` | lastWriteUtc=2026-01-30T15:31:19.6465730Z | bytes=2015
- `C:\ProgramData\LM\tasks\tr\LM-Bridge-TruthGate-1m.wrap.ps1` | sha256=`F0036D779D5B138D5E73023571FBF0555662F5D04499E8DC3F09EDECBBF2A1D4` | lastWriteUtc=2026-01-30T15:31:12.6101485Z | bytes=2312
- `C:\ledgermind\tools\lm-doc.write_from_clipboard.ps1` | sha256=`CD98F363E1AF2C120A78E0AD361CF5DEFDE001B943D6E60AB662BBA780EF8F3A` | lastWriteUtc=2026-01-30T16:42:29.2703410Z | bytes=2216
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





<!-- LM_EOD_20260202 -->
### EOD Snapshot — 20260202
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260202.md
  - sha256: 195882A636675219DB5E1F147AA81D845F485953ADE0671A24EF4906D4F9B33E
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260202 -->

<!-- LM_EOD_20260203 -->
### EOD Snapshot — 20260203
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260203.md
  - sha256: A6308B2A8D21C76E6D0D075787C4CE5303EDAAEC721DB25CF930173BDEA1BF65
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260203 -->

<!-- LM_EOD_20260204 -->
### EOD Snapshot — 20260204
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260204.md
  - sha256: E01FE1E93F3630593ABC9A34988C097BB5023C66A3E4C6466AF6B9C374482F4F
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260204 -->

<!-- LM_EOD_20260206 -->
### EOD Snapshot — 20260206
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260206.md
  - sha256: AE0FB90925F7E8EFB39AFF1C6B594D4C42D5C5F9306123F867915CBD14594B56
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260206 -->

<!-- LM_EOD_20260207 -->
### EOD Snapshot — 20260207
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260207.md
  - sha256: D0CBEDBD0A393D416042C02D21C8F079889B4258AA471188C5515DA134AA42DB
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260207 -->

<!-- LM_EOD_20260208 -->
### EOD Snapshot — 20260208
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260208.md
  - sha256: BD3B08A798F4A36BBFB965ED57C0C8D05C9133FEE73317CDD8F765D9457CE897
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260208 -->

<!-- LM_EOD_20260209 -->
### EOD Snapshot — 20260209
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260209.md
  - sha256: 0C2827491D23ECA705AD7F61E2B5CBE9DB33673F24755D5E1D5A970DAE1DCCDE
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260209 -->

<!-- LM_EOD_20260210 -->
### EOD Snapshot — 20260210
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260210.md
  - sha256: 6AE30DF0317661C194C846C8C58DFB4E6A4027709CAD58CF175F7F33BF06565D
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260210 -->

<!-- LM_EOD_20260211 -->
### EOD Snapshot — 20260211
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260211.md
  - sha256: 8C9A78E17617873CECD07C760DC078059C1653D4C14ABEAFEC779A9A6D5E40E6
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260211 -->

<!-- LM_EOD_20260212 -->
### EOD Snapshot — 20260212
- report: C:\ProgramData\LM\ops\eod\LM-EOD-Report.20260212.md
  - sha256: 5D31A8577CCE392ADCE0BEC3371CAEBA4D1401FE77D570F080C5781A40246143
- nowrite_present: True
- gate_present: False
<!-- /LM_EOD_20260212 -->
<!-- FILESPANEL_TEST 20260214T165313Z -->


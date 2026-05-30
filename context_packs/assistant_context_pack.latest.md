# 🧰 LedgerMind — assistant_context_pack (auto)
**Generated (UTC):** 2026-05-30T11:48:55.1417178Z
**Host:** DESKTOP-08IU10H

## Fast Status
- TruthGate nowrite lock: **True** (C:\ProgramData\LM\tx\exec\locks\tx_exec_truth_gate.nowrite)
- Bridge Task: TaskName:                             \LM-Bridge-TruthGate-1m; Next Run Time:                        N/A; Status:                               Disabled; Last Run Time:                        11/02/2026 07:05:01; Last Result:                          0; Scheduled Task State:                 Disabled; Start Time:                           10:19:00
- Open Run Once Task: TaskName:                             \LedgerMind\LM-Open-Run-Once-20260120-1620; Next Run Time:                        N/A; Status:                               Disabled; Last Run Time:                        20/01/2026 16:32:00; Last Result:                          0; Scheduled Task State:                 Disabled; Start Time:                           16:32:00
- DATAQ_MARKET: mw=CLOSED status=OFF allow_new_trades=False top_reason=openready_v2|market_window!=OPEN

## Fingerprints (SHA256)
- `C:\ledgermind\tools\lm-open.enable_pack.ps1` | sha256=`822E77F4CD3E7889F45662F1AA40367F0254A3207503CC11608247A2FAAE1AC3` | lastWriteUtc=2026-02-22T14:38:31.8399772Z | bytes=1816
- `C:\ledgermind\tools\lm-weekend.disable_pack.ps1` | sha256=`C4EA39ED53865F5ECC8E2D1806F7DF8736067656A13B6B258B29841694088899` | lastWriteUtc=2026-02-22T14:38:36.1774334Z | bytes=1354
- `C:\ledgermind\tools\lm-open.run.ps1` | sha256=`E51CD14A214E015ABE85661178B3EADF69D4137087F469D84C3D328F7FABE7D3` | lastWriteUtc=2026-01-30T16:42:42.8562381Z | bytes=3199
- `C:\ProgramData\LM\tasks\tr\LM-Open-Run-Once.ps1` | sha256=`95D5B49E881123AC8BC30217A61869861EB05BF34A52049325AEF3DF784FA66A` | lastWriteUtc=2026-02-02T07:55:05.0578012Z | bytes=2023
- `C:\ProgramData\LM\tasks\tr\LM-Bridge-TruthGate-1m.wrap.ps1` | sha256=`AD878B03ED352B8C35F7284D9259FC2DD693FD0E873F65BCAD67F91EA5A304C3` | lastWriteUtc=2026-03-20T22:15:04.3552134Z | bytes=2270
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




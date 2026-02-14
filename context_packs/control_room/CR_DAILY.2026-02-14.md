## CR_DAILY v1 — 2026-02-14 (UTC=2026-02-14T14:00:10.9120299Z)

### Core Status
- NOWRITE_PRESENT=True
- IBGW_RUNNING=False 4002_LISTEN=False (pid= session=)
- DATAQ: mw=CLOSED dhs=PASS allow=False reason=market_window=CLOSED
- PRICES: ok=True ageMin=3.94698320666667 miss=0
- TRUTH: allow_new_orders=False reduce_only=True reason=nowrite_lock
- RISK: ok=False status=fail reasons=insufficient_trade_days_30d:5<10 trade_days_30d=5
- MLR: ok=True

### System
- CPU=Intel(R) Core(TM) i7-14700F Cores=20 LP=28
- RAM_GB=128
- Realtek=1 Gbps Status=Up
- Power=Power Scheme GUID: 17bccb59-a850-4fd0-b768-b162ef063876  (Ultimate Performance (LM))

### Volumes
- C: freeGB=153.3 freePct=16.5 label=
- D: freeGB=4 freePct=99.4 label=BIOS
- E: freeGB=1583.5 freePct=85.1 label=LM-Data
- F: freeGB=926.8 freePct=99.9 label=DATA

### Tasks
- RunningCount=3
- Fails24hCount=1
  - FAIL task=LM-EOD-Orchestrator-Daily-2355 rc=1 lastRunUtc=2026-02-14T08:42:24.0000000Z

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


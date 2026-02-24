# INC 2026-02-22 IBGW Popups / UI Silence

## Summary
- Root cause: \LedgerMind\LM-IBGW-WindowControl-5m was Disabled; override path exits early before UI patch.
- Fix: UI_SILENCE_V3 injected before exit 0 inside override keep_open block.
- Hardening: lm-open.enable_pack ensures WindowControl enabled; weekend.disable_pack re-enables WindowControl (UI-only).
- Current mode: MINIMIZE (mode=6) ; silenceOff=False

## Evidence
### UI Silence log (tail)
    2026-02-22T14:33:01.3369618Z UI_SILENCE_V3 ok=True mode=6 pid=28920 windows=1
    2026-02-22T14:35:03.1046040Z UI_SILENCE_V3 ok=True mode=0 pid=28920 windows=1
    2026-02-22T14:38:25.7517499Z UI_SILENCE_V3 ok=True mode=0 pid=28920 windows=0
    2026-02-22T14:40:03.6922665Z UI_SILENCE_V3 ok=True mode=0 pid=28920 windows=0
    2026-02-22T14:40:19.1269199Z UI_SILENCE_V3 ok=True mode=0 pid=28920 windows=0
    2026-02-22T14:40:22.0275104Z UI_SILENCE_V3 ok=True mode=0 pid=28920 windows=0
    2026-02-22T14:43:57.1231204Z UI_SILENCE_V3 ok=True mode=6 pid=28920 windows=7
    2026-02-22T14:45:03.7890378Z UI_SILENCE_V3 ok=True mode=6 pid=24652 windows=1

### WindowControl log (tail)
    IBGW_WINDOW start ts_utc=2026-02-22T14:45:03.2749408Z
    OVERRIDE keep_open=True
    TARGET=START

### Task status
    LastTaskResult=0
    LastRunTime=02/22/2026 16:45:01
    NextRunTime=02/22/2026 16:50:00

### Fingerprints
- C:\ledgermind\tools\lm-ops.ibgw.window_control.override.ps1 | sha256=5CC9D0488C0C922FF3D1DC0EC7729B207A93A760BCD67CF54C162ADACA5C4270 | lwtUtc=2026-02-22T14:32:54.9703103Z
- C:\ledgermind\tools\lm-open.enable_pack.ps1 | sha256=822E77F4CD3E7889F45662F1AA40367F0254A3207503CC11608247A2FAAE1AC3 | lwtUtc=2026-02-22T14:38:31.8399772Z
- C:\ledgermind\tools\lm-weekend.disable_pack.ps1 | sha256=C4EA39ED53865F5ECC8E2D1806F7DF8736067656A13B6B258B29841694088899 | lwtUtc=2026-02-22T14:38:36.1774334Z

## Update 2026-02-22 (Spawn-Storm Fix)
- Confirmed stable: ibgateway_count=1 for 10+ minutes.
- Startup dedupe: kept 'LedgerMind Agent.lnk', moved duplicates to: C:\Users\97254\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\_disabled_20260222_183737

### Start-User log (tail)
    Sun 02/15/2026 11:43:05.61 START launching ibgateway
    Sun 02/22/2026 18:50:24.67 OK already running
    Sun 02/22/2026 18:50:25.80 OK already running
    Sun 02/22/2026 19:04:18.23 ALREADY_RUNNING skip
    Sun 02/22/2026 19:04:18.23 OK already running

### Fingerprints
- C:\ProgramData\LM\tasks\tr\LM-IBGW-Start-User.cmd | sha256=0CAA321FCAEE789AB409934C8EA32A3E7B9F51DFEEE7EC85DEFCA40A05224847 | lwtUtc=2026-02-22T16:50:32.1440235Z
- C:\ledgermind\tools\lm-ops.ibgw.window_control.override.ps1 | sha256=2C920B5ACD4C9D25531DFE10C490D5845EECA9D959922D133610CFC687C2A9E7 | lwtUtc=2026-02-22T15:05:02.7745226Z
- C:\ledgermind\tools\lm-open.enable_pack.ps1 | sha256=822E77F4CD3E7889F45662F1AA40367F0254A3207503CC11608247A2FAAE1AC3 | lwtUtc=2026-02-22T14:38:31.8399772Z
- C:\ledgermind\tools\lm-weekend.disable_pack.ps1 | sha256=C4EA39ED53865F5ECC8E2D1806F7DF8736067656A13B6B258B29841694088899 | lwtUtc=2026-02-22T14:38:36.1774334Z

## Update 2026-02-22 (Spawn-Storm Fix)
- Confirmed stable: ibgateway_count=1 for 10+ minutes.
- Startup dedupe: kept 'LedgerMind Agent.lnk', moved duplicates to: C:\Users\97254\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\_disabled_20260222_183737

### Start-User log (tail)
    Sun 02/15/2026 11:43:05.61 START launching ibgateway
    Sun 02/22/2026 18:50:24.67 OK already running
    Sun 02/22/2026 18:50:25.80 OK already running
    Sun 02/22/2026 19:04:18.23 ALREADY_RUNNING skip
    Sun 02/22/2026 19:04:18.23 OK already running

### Fingerprints
- C:\ProgramData\LM\tasks\tr\LM-IBGW-Start-User.cmd | sha256=0CAA321FCAEE789AB409934C8EA32A3E7B9F51DFEEE7EC85DEFCA40A05224847 | lwtUtc=2026-02-22T16:50:32.1440235Z
- C:\ledgermind\tools\lm-ops.ibgw.window_control.override.ps1 | sha256=2C920B5ACD4C9D25531DFE10C490D5845EECA9D959922D133610CFC687C2A9E7 | lwtUtc=2026-02-22T15:05:02.7745226Z
- C:\ledgermind\tools\lm-open.enable_pack.ps1 | sha256=822E77F4CD3E7889F45662F1AA40367F0254A3207503CC11608247A2FAAE1AC3 | lwtUtc=2026-02-22T14:38:31.8399772Z
- C:\ledgermind\tools\lm-weekend.disable_pack.ps1 | sha256=C4EA39ED53865F5ECC8E2D1806F7DF8736067656A13B6B258B29841694088899 | lwtUtc=2026-02-22T14:38:36.1774334Z

## Update 2026-02-24T08:21:42.3601677Z (UI Catcher V4)
- script: C:\ProgramData\LM\ops\ibgw_window\ibgw_ui_catcher.daemon.ps1 | sha256=3093D3DD5DDDF4599D867C6F9735EB27969C555B1B830DBEC9A15BD8200B8584
- log   : C:\ProgramData\LM\tasks\logs\LM-IBGW-UI-Catcher.log | sha256=18072856F04763321EE50C0A31210747662AED175EFD3A092C7D97E1676D0D0B
- flags: hide=False blank=False dbg=False
- task: \LedgerMind\LM-IBGW-UI-Catcher-OnLogon state=Disabled

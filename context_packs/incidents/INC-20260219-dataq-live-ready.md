## חדר בקרה — סיכום אירוע: DataQ Live Ready FAIL (`missing_symbols=SPY`) + תיקון Upstream

### תמונת מצב (סיום)

* `dataq_coverage_gate.latest.json` **PASS**
* `dataq_daily_report.latest.json` **PASS**
* `dataq_live_ready.latest.json` **PASS** (`allow_live_data=true`)
* `openclose_2026-02-18.csv` עכשיו מכיל: `AAPL, MSFT, SPY` (קנוני, בלי `.US`)
* `LM-Truth-Recon-ExportsLatest-1m` — אין `PermissionError` ב־400 שורות אחרונות; אירוע היסטורי מ־03/02/2026 בלבד

---

### P0 — מה קרה

**סימפטום:**

* `dataq_live_ready` נכשל עם `missing_symbols=SPY`
* `dataq_coverage_gate` נכשל עם `missing_symbols=SPY`
* `required_symbols.tx.txt` דרש `SPY`, אבל `openclose_2026-02-18.csv` הכיל `SPY.US` (מקור `stooq`) ⇒ mismatch

**Root Cause:**

* **Data Contract mismatch** בין סימבולים קנוניים (LM) לבין סימבולי ספק (stooq).
* כתוצאה: coverage הסתכל על tickers בקובץ openclose, לא מצא `SPY`, סימן FAIL ⇒ daily_report FAIL ⇒ live_ready FAIL.

---

### פעולות שבוצעו (Evidence + Fix)

#### 1) שחזור/תיקון Gate (Defense-in-depth)

* קובץ: `C:\ledgermind\tools\lm-dataq.coverage_gate.eval.ps1`
* הוחל Patch: **`LM_PATCH_CANON_TICKER_US_V3`**

  * נרמול `ticker` בזמן קריאה מ־CSV: `*.US` → ללא suffix
  * אחרי patch: `dataq_coverage_gate` עבר ל־PASS והציג `present=[AAPL,MSFT,SPY]`

#### 2) רענון Pipeline כדי ש-live_ready יזוז ל-PASS

* הרצה: `C:\ledgermind\tools\lm-dataq.daily_report.build.ps1`

  * עבר מ־FAIL ל־PASS (`DATAQ_DAILY status=pass reasons=0`)
* הרצה: `C:\ledgermind\tools\lm-dataq.live_ready.build.ps1`

  * עבר ל־PASS, `allow_live_data=true`

#### 3) תיקון Upstream אמיתי (Fix קבוע)

* קובץ writer: `C:\ledgermind\tools\lm-tx.micro_auto_prices.guard.run.ps1`
* הוחל Patch: **`LM_PATCH_OPENCL_CANON_TICKER_US_V2`**

  * לפני `outRows.Add(...)` נוצר `$tCanon = $t -replace '\.US$'`
  * ונכתב `ticker=$tCanon`
* אימות: openclose latest נכתב מחדש עם tickers קנוניים:

  * `AAPL,MSFT,SPY` ✅

---

### סטטוס סיכון / Gates

* **OPS:** תקין (Tasks רצו, קבצי gates עודכנו)
* **TRUTH:** כרגע אין PermissionError חדש ב־ExportsLatest; user=SYSTEM; ACL לקובץ gate נראה תקין
* **EXECUTION:** לא נבדק לעומק בסשן זה
* **EDGE:** לא רלוונטי לאירוע
* **Verdict:** **PASS ל-DataQ Live Ready** (אך להשאיר Fail-Closed למסחר לפי מדיניות LM עד שיתר gates/charter מאפשרים)

---

### נקודות לשיפור (P1/P2)

1. **Data Contract רשמי ל-openclose**
   * `ticker` תמיד קנוני
   * אופציונלי: להוסיף `provider_ticker` לצורכי תחקור
2. **ContractGate חדש**
   * מאמת שאין suffixes (`.US` וכו’) וש-required symbols מכוסים
3. **Hardening ל-Truth Recon**
   * לשנות אלרטים כך שיבדקו רק חלון לוג אחרון (כדי לא ליפול על אירוע היסטורי)

---

### קבצים/Hashes (עיקריים)

* `lm-dataq.coverage_gate.eval.ps1` — Patch V3 הוחל (hash השתנה בהתאם)
* `lm-tx.micro_auto_prices.guard.run.ps1` — Patch V2 הוחל (hash השתנה בהתאם)
* Gates:
  * `C:\ledgermind\data\lm\quality\gates\dataq_coverage_gate.latest.json` (PASS)
  * `C:\ledgermind\data\lm\quality\reports\dataq_daily_report.latest.json` (PASS)
  * `C:\ledgermind\data\lm\quality\gates\dataq_live_ready.latest.json` (PASS, allow=true)
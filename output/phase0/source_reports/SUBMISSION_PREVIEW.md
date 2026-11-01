> **หมายเหตุจาก repo การบ้าน (เพิ่มตอนคัดลอก 2026-11-01):** ไฟล์นี้คัดลอกจากรายงาน local `submission_preview/SUBMISSION_PREVIEW.md` ของ repo โค้ดแยก (ไม่เผยแพร่) โดยไม่แก้เนื้อหา คำว่า "the repository" ใน §7 หมายถึง repo local นั้น ซึ่ง **ไม่ได้อยู่ใน repo การบ้านนี้** · §7 เขียนว่า 281 tests เป็นค่าเก่า ค่าสุดท้ายคือ **284 tests** ตาม `CHECKPOINT_phase0_9.md` §3

# US Fundamental Inflection Radar — Phase 0 Technical Validation (submission preview)

**Status:** draft preview for coursework submission. Local only; not published.
**Snapshot:** SEC EDGAR bulk data of 2026-10-31.

> **Read this first.**
> - This is a technical validation of a data pipeline and an experimental screening model. It is not empirical
>   validation of model quality and not evidence of investment performance.
> - Every threshold is provisional, and none was tuned on the sample.
> - The 30 companies were chosen on purpose to cover edge cases, so no result describes the market.
> - The tool uses no prices and makes no buy or sell recommendation.

## 1. What the project does
The project screens US-listed companies, using SEC EDGAR filings only, for early improvement in their reported
fundamentals:
- **Margin path:** the operating margin is rising.
- **Acceleration path:** revenue growth is speeding up.
- **Cash path:** operating cash flow is improving faster than operating margin alone would explain.

Two specifications govern the work:
- **R2 (Revision 2), data engineering:** universe funnel, tag map, conversion of filed facts into quarters (with
  provenance), reconciliation and acceptance gates G0.1–G0.9.
- **R3 v0.1, the model:** a 16-row classification (P = profitable improver, G = cash-generating growth,
  T = turnaround, W = watch, U = short history), four debt states, a risk tier, evidence labels for each path, and
  the experimental *Threshold Multiple* ranking.
  - R3 supersedes R2 wherever the two differ.
  - R2's original weighted score is kept only as a labelled comparator.

## 2. Data and pipeline
- **Sources:**
  - `company_tickers_exchange.json`, submissions.zip (1.57 GB, 146 s download) and companyfacts.zip (1.41 GB, 123 s
    download).
  - Original 10-K/10-Q filings and 8-K earnings releases, used for manual reconciliation.
  - Requests are rate-limited to ≤ 5 per second, with a declared User-Agent kept outside version control.
- **Universe funnel (actual):** U0 8,023 listed → U1 6,036 NYSE/Nasdaq → U2 4,217 non-financial with an SIC code →
  U3 2,964 with at least 2 completed fiscal years and a recent 10-Q.
- **Fact → quarter conversion (R2 §5.2):**
  - Every filed version is preserved; for each fact the latest-filed version wins.
  - Periods are identified from their start and end dates only, never from the companyfacts `fy`/`fp` fields.
  - Quarterly values come from direct 3-month figures or from year-to-date differences.
  - Reconciliation checks: the four quarters sum to the fiscal year within max(0.5%, $100k), and direct quarters
    equal year-to-date differences within 0.5%.
  - A missing value stays null and is never treated as zero. Failed quarters never enter trailing-twelve-month (TTM)
    figures.
- **Approved corrections:**
  - D4: only 10-K, 10-K/A, 10-KT, 10-Q and 10-Q/A filings feed the conversion.
  - D5: a precision-aware tolerance, used only when the declared inline-XBRL decimals of every operand are verified.
  - D6: a debt fallback tag family, flagged as including finance leases.
  - D7: special handling for 10-KT fiscal-year transitions.

## 3. Validation results
| Gate | Result |
|---|---|
| G0.1 exchange labels · G0.2 submissions source | pass (zip by default, API as fallback; the API would take 27–53 minutes for the full universe) |
| G0.3 manual reconciliation | pass on arithmetic (240/240 values, 338/338 lineage links); independent validity reported separately below |
| G0.4 year-to-date cash flow · G0.6 no invented values · G0.7 restatements · G0.8 compatibility · G0.9 tag splice | pass |
| G0.5 fiscal calendar | pass: 8/8 standard cases plus the 10-KT transition case (5/5 conditions) |
| G0.10 classification · G0.11 debt states · G0.13 evidence exhaustive | pass |
| G0.12 ranking-key invariance | pass: 27 leave-one-out runs, each company ranked alone, 2 companies added, 5 input shuffles; no key or pairwise order changed |
| G0.14 coverage report | pass |

**Honest view of the 240 manually reconciled values.** Arithmetic agreement is not the same as independent validity:

| Category | Values |
|---|---|
| Presented 3-month figure for that exact period | 132 |
| Derived and corroborated by an earnings release | 24 |
| Valid by rule only (derived; no independent figure) | 68 |
| Rejected by the pipeline (mostly restated vintages) | 16 |

**Reconciliation failures:**
- Of 58 strict failures, 9 are tolerated as verified rounding.
- 47 exceed the rounding bound and stay failed; this includes every restatement case.
- 2 cannot be checked against the filing's declared precision.

## 4. Sample ranking (technical, 30-company purposive sample)
Ranking key: (Tier, best threshold multiple capped at 5×, credible paths, second credible multiple, CIK). There is
no weighted composite.

| Class | Rank | Company | Primary path (multiple) | Evidence | Tier | Risk |
|---|---|---|---|---|---|---|
| P | 1 | Murphy Oil | accel 5.00 (capped) | confirmed | A | no triggered flags — incomplete inputs |
| P | 2 | CrossAmerica Partners | accel 3.84 | confirmed | A | tier 1 (leverage) |
| P | 3 | AMCON Distributing | accel 2.58 | confirmed | A | no triggered flags — incomplete inputs |
| P | 4 | Federal Signal | cash 1.87 | confirmed | A | no triggered flags — complete inputs |
| P | 5 | Sirius XM | cash 1.30 | confirmed | A | no triggered flags — complete inputs |
| P | 6 | Laureate Education | margin 1.05 (2 paths) | early | A | no triggered flags — incomplete inputs |
| P | 7 | Acushnet | cash 1.48 | reversion risk | B | no triggered flags — complete inputs |
| G | 1 | Synaptics | margin 1.58 | confirmed | A | no triggered flags — complete inputs |
| G | 2 | Symbotic | margin 1.33 | confirmed | A | no triggered flags — incomplete inputs (debt unknown) |
| T | 1 | Alto Ingredients | margin 4.94 (3 paths) | confirmed | A | no triggered flags — complete inputs |
| T | 2 | Samsara | margin 4.93 (2 paths) | confirmed | A | no triggered flags — incomplete inputs (debt unknown) |

- "No triggered risk flags" does not mean the company is financially safe.
- **19 companies are searchable but unranked,** each with an explicit reason: no triggered path (for example
  Montauk Renewables), the Watch class, short history, insufficient history, or a negative control.
- **Supplemental technical cases (Intapp, Boeing)** are ranked separately and are not part of the sample.
- **Diagnostic variants:** path-count, R2-fixed and the faithful R2-original comparator (21-company pool; 6
  companies flagged `debt_missing` and 1 `cash_missing`, with that condition scored 0 as R2 specifies).
  - These are exploratory at this sample size and are not evidence that any model is better.
  - The R3 within-group percentile variant stays null below n = 30.

## 5. Coverage highlights
- Recent tag coverage across the U3 pool (2,960 companies): operating income 95.0%, revenue 93.2%, OCF 99.8%, any
  debt tag 67.5%.
- Special-item tags are present in the U3 pool for 29–43% of companies.
- In the sample core (21 companies):
  - the base-year special-items check is unknown for 15;
  - leverage is unknown for 8;
  - interest expense is missing at the ranking quarter for 18.
- In the sample core, 7 of the 15 companies with no triggered risk flags have complete risk inputs.

## 6. Limitations
1. The sample is purposive and small, so its group sizes (P 13, G 2, T 2) cannot support a model comparison.
2. Thresholds are provisional. Behavioural cases A–H check mechanics only.
3. Restated or recast filing vintages (LifeMD, Liberty Media, Gibraltar) are rejected rather than mixed, which
   shortens their usable history.
4. Debt and finance leases reported only with XBRL dimensions are unknown in companyfacts, so risk completeness is
   often partial.
5. NYSE American cannot be told apart from NYSE in the SEC exchange file.
6. There is no point-in-time price data. Any persistence study would be exploratory pseudo-point-in-time with
   survivorship bias.

## 7. Reproducibility
- The repository holds the code (Python 3.13, uv), the configuration, 281 tests, reconciliation fixtures and reports.
- SEC downloads, intermediate data and the credentials file are not included; they are regenerated locally.
- Run order: `universe` → `enrich-universe` → `fetch` → `extract` (plus `extract-companion`) → `quarterly` →
  `precision-check` → `quarterly` → `assess` → `score` → `summary-0.9`.

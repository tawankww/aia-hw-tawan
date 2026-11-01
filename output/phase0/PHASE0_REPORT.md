# US Fundamental Inflection Radar — Phase 0 final report (R3 v0.1)

**Date:** 2026-11-01 · **Data snapshot:** SEC EDGAR bulk data downloaded 2026-10-31 · **Status:** Phase 0 complete; Phase 1 not started.

> **สรุปภาษาไทย (อ่านก่อน)**
> - Phase 0 คือ **การตรวจสอบทางเทคนิค** ของ data pipeline และกลไกของโมเดล R3 บนข้อมูล SEC จริง **ไม่ใช่** การพิสูจน์ว่าโมเดลคัดหุ้นได้ดี และ **ไม่ใช่** หลักฐานผลตอบแทนการลงทุน
> - ตัวอย่าง 30 บริษัทเลือกแบบ purposive (เจาะจงเพื่อครอบคลุมกรณียาก) จึงใช้อ้างถึงทั้งตลาดหรืออ้างความแม่นในการพยากรณ์ไม่ได้
> - ทุก threshold เป็นค่าชั่วคราว (provisional) และไม่ได้ปรับจากผลของตัวอย่าง · ไม่ใช้ราคาหุ้น · ไม่มีคำแนะนำซื้อขาย
> - ตัวเลขทั้งหมดในไฟล์นี้มาจากรายงาน Phase 0.9 ของ repo โค้ด local ซึ่ง **ไม่ได้เผยแพร่** ผู้สอนอ่านรายงานได้ แต่ตรวจโค้ดหรือรันซ้ำจาก repo นี้ไม่ได้ (ดู §10)

---

## 1. Problem statement and objective

US-listed companies number in the thousands. An individual investor cannot read every filing to spot the few companies whose reported fundamentals have **just started** to improve. The goal is a monthly shortlist of about 20 companies (target mix 12 P / 5 G / 3 T) for further deep research, built **only from SEC EDGAR filings**, with transparent evidence strength and risk for each name.

It is a discovery screen. It is not a valuation tool, not a trading tool, and it uses no prices.

Phase 0 objective: before any market-wide run, prove that (a) real SEC data can be turned into trustworthy quarterly figures with full provenance, and (b) the R3 model's mechanics (classification, risk, evidence and ranking) behave exactly as specified.

## 2. Final R3 architecture and scope

Two specifications govern the work (full text: `output/design/04_phaseE_final_design.md` and `05_R3_final_spec_and_phase0_contract.md`):

| Layer | Spec | What it does |
|---|---|---|
| Data engineering | R2 | Universe funnel, tag map, fact → quarter conversion with provenance, reconciliation, gates G0.1–G0.9 |
| Model | R3 v0.1 | Classification → financial risk tier → three inflection paths → evidence labels → threshold-multiple ranking; gates G0.10–G0.14 |

R3 supersedes R2 wherever the two differ. R2's weighted score is kept only as a labelled diagnostic comparator.

R3 v0.1, as implemented:
1. **Signals** (TTM at quarter *t* against *t−4*; needs 8 consecutive valid quarters): revenue growth S_g, operating-margin change S_m, OCF-margin change S_c, and quarterly year-on-year acceleration S_a.
2. **Three inflection paths** (any one is enough): margin (S_m ≥ 2 pp), acceleration (S_a ≥ 5 pp with S_g > 0 and S_m ≥ −1 pp) and cash (S_c ≥ 2 pp with S_c − S_m ≤ 5 pp).
3. **Classification** with a 16-row truth table on the signs of operating income and OCF at *t−4* and *t*: P (profitable improver), G (cash-generating growth), T (turnaround), W (watch, never ranked) and U (short history, never ranked).
4. **Financial risk** is non-compensatory. There are four debt states (`known_debt`, `confirmed_zero`, `partial_zero`, `unknown`). Tier 2 applies only when high leverage **and** external-financing dependence are both true. Unknown inputs never change the tier and are shown as a separate completeness value.
5. **Evidence labels for each path:** breadth, comparison with the prior peak, and special items in the base year give `confirmed` / `early` / `single_period` / `reversion_risk`.
6. **Ranking key** within each class: (Tier A/B, best threshold multiple capped at 5×, number of credible paths, second credible multiple, CIK). There is no weighted composite. A company with no triggered path is searchable but has no rank number.

**Out of scope for v0.1:** prices and returns, valuation, sector scoring, LLM use at runtime, and a point-in-time universe. Phase 0 also excludes the dashboard, the threshold freeze and any validation of model quality.

## 3. Data sources and provenance strategy

All sources are SEC EDGAR (the Trust sources in `spec.md` §3):

| Source | URL | Use |
|---|---|---|
| Ticker/exchange list | https://www.sec.gov/files/company_tickers_exchange.json | Universe U0–U1 |
| submissions.zip (1,572,128,284 bytes) | https://www.sec.gov/Archives/edgar/daily-index/bulkdata/submissions.zip | SIC, fiscal calendar, filing list (U2–U3) |
| companyfacts.zip (1,414,913,699 bytes) | https://www.sec.gov/Archives/edgar/daily-index/xbrl/companyfacts.zip | XBRL facts for every company |
| XBRL companyfacts API | https://data.sec.gov/api/xbrl/ | Cross-check of extraction (30/30 agree) |
| Original 10-K / 10-Q, and 8-K earnings releases | https://www.sec.gov/cgi-bin/browse-edgar | Manual reconciliation; corroboration of Q4 |

All data were downloaded on 2026-10-31 (the snapshot date). Requests are limited to 5 per second, with a declared User-Agent. The User-Agent and contact details live only in a local `.env` and are not in this repository.

**How provenance works:**
- Every filed version of every fact is kept (39,640 rows for the sample). For each fact, the latest-filed version wins.
- Each derived quarterly value lists the source facts it came from (`source_fact_ids`). Each fact carries its CIK, accession number and filing date.
- Periods are identified only from their start and end dates, never from the companyfacts `fy`/`fp` fields.
- A missing value stays null and is never set to zero or carried forward. Quarters that fail reconciliation never enter TTM figures.
- Lineage at the level of individual facts and accession numbers lives in the local reports and fixtures, which are not published.

## 4. Methodology and technical validation

| Step | What was done |
|---|---|
| 0.2–0.3 | Built the local project skeleton and config. Measured the universe funnel and compared the submissions zip with the API (zip ≈ 146 s download plus 2.3 s parse for all of U0; API extrapolated at 27–53 minutes). |
| 0.4 | Extracted companyfacts for the sample (39,640 facts; 30/30 agreement with the SEC API). |
| 0.5 | Reconstructed quarters under R2 §5.2 and added the R3 `classify` / `risk` / `evidence` modules. |
| 0.6 | Manual reconciliation: 10 cases × 8 quarters × 3 metrics = 240 values compared with the printed filings, not with XBRL. |
| 0.7 | Owner-approved corrections D4–D7, plus automated tests (truth table, debt states, evidence property tests, A–H mechanism fixtures). |
| 0.8 | R3 sample ranking and the coverage report. |
| 0.9 | Corrected the comparators (D8) and produced the final Phase 0 summary with a sanitization scan. |

**Corrections approved during Phase 0:**
- **D4:** only 10-K, 10-K/A, 10-KT, 10-Q and 10-Q/A filings feed quarter conversion.
- **D5:** a rounding tolerance taken from the declared inline-XBRL decimals, used only when the decimals of every operand are verified.
- **D6:** a debt fallback tag family, flagged as including finance leases.
- **D7:** special handling for 10-KT fiscal-year transitions.
- **D8:** the R2-original comparator follows R2 exactly. The risk wording changed to "no triggered flags — complete/incomplete inputs". Tie-breaks count credible paths only.

**Sample design:** 30 companies chosen on purpose to cover clear P, G, T and W cases, short history, unknown or zero debt, special items, large acquisitions, high SBC and negative controls. Two extra cases, Intapp (G) and Boeing (T), are labelled supplemental and ranked separately.

## 5. Actual measured results

**Universe funnel (actual, full listed universe):** U0 8,023 listed → U1 6,036 NYSE/Nasdaq → U2 4,217 non-financial with an SIC code → U3 2,964 with at least 2 completed fiscal years and a recent 10-Q.

**Gates:**

| Gate | Result |
|---|---|
| G0.1 exchange labels | PASS (NYSE American cannot be separated from NYSE; documented) |
| G0.2 submissions source | PASS (zip by default, API as fallback) |
| G0.3 manual reconciliation | PASS **on arithmetic**: 240/240 values and 338/338 lineage links |
| G0.4 YTD cash flow | PASS (OCF R1: 119 pass, 0 fail) |
| G0.5 fiscal calendar | PASS: standard cases 8/8, plus the 10-KT transition case 5/5 |
| G0.6 no invented values · G0.7 restatements · G0.8 compatibility · G0.9 tag splice | PASS |
| G0.10 classification | PASS: 21 core + 3 provisional + 6 excluded = 30 |
| G0.11 debt states · G0.13 evidence exhaustive | PASS |
| G0.12 ranking-key invariance | PASS: 27 leave-one-out runs, each company ranked alone, supplemental cases added, 5 shuffles; 0 key changes and 0 pairwise-order changes |
| G0.14 coverage report | PASS |

Other results:
- **Automated tests:** 284 passed.
- **Reproducibility:** re-running gives byte-identical reports, and `facts_raw` is unchanged since Phase 0.4.

**Validity of the 240 manually reconciled values.** Agreeing in arithmetic is not the same as being independently valid:

| Category | Values |
|---|---|
| Presented 3-month figure for that exact period | 132 |
| Derived and independently corroborated (earnings release) | 24 |
| Valid by rule only (derived; no independent quarterly figure) | 68 |
| Rejected by the pipeline (mostly restated vintages) | 16 |

**Reconciliation failures:** there are 58 strict failures.
- 9 are tolerated as verified rounding (D5).
- 47 exceed the bound and stay failed; this includes every restatement case.
- 2 are `precision_unknown`.

**R3 sample Discovery Ranking (technical; purposive sample).** The ranking shows how the mechanics behave. It is not a recommendation:

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

What the ranking shows:
- 11 companies are ranked (P 7, G 2, T 2): 10 in Tier A and 1 in Tier B.
- 19 companies are searchable but unranked, each with an explicit reason: no triggered path (for example Montauk Renewables), Watch class, short history, insufficient history, or a negative control.
- "No triggered risk flags" does **not** mean financially safe.

**Coverage (tag presence):**
- **U3 pool (2,960 companies, actual):** operating income 95.0%, revenue 93.2%, OCF 99.8%, cash 98.9%, any debt tag 67.5%. Special-item tags: restructuring 29.4%, goodwill impairment 43.3%, asset impairment 38.2%.
- **Sample core (21, purposive):** risk completeness is 1.0 for 10 companies, 0.5 for 8 and 0.0 for 3. The base-year special-items check is unknown for 15, and interest expense is missing at *t* for 18.

**Run time on the local machine (27 companies):** reconstruction 21.3 s, assess 0.5 s, score with the G0.12 runs 2.0 s.

## 6. Key findings

1. **The pipeline is sound within its documented scope.** Extraction matches the SEC API, reconstruction is deterministic, and provenance is complete.
2. **Arithmetic agreement overstates validity.** Only 156 of the 240 values (132 + 24) have independent support, so the two must be reported separately.
3. **Strict compatibility rejects restated vintages.** LifeMD, Liberty Media and Gibraltar are rejected rather than mixed. This is correct, but it shortens their usable history.
4. **Missing data, not model logic, limits the risk guards.** Debt and finance leases reported only with XBRL dimensions are invisible in companyfacts. Interest expense and base-year special items are often missing.
5. **The R3 mechanics behave as specified:**
   - W, U and excluded companies are never ranked.
   - Reversion risk demotes a company to Tier B (Acushnet).
   - The 5× cap binds once (Murphy Oil).
   - A company's key does not depend on the other companies in the run.
6. **The comparators differ for structural reasons (exploratory only):**
   - R2-original scores Watch and no-path companies (Insmed, a W company, comes first).
   - R2-original ignores reversion risk (Acushnet ranks above Tier A companies).
   - The count variant shows the "2 modest beat 1 strong" pattern that R3 was designed to avoid.

## 7. Limitations

1. **Technical validation only.** There is no evidence of model quality, predictive power or investment performance.
2. **The sample is purposive and small.** Its groups (P 13, G 2, T 2) are far below n = 30, so no model comparison is possible. All R2-fixed and R2-original values are exploratory, and the R3 percentile variant stays null.
3. **Thresholds are provisional.** Behavioural cases A–H check mechanics only.
4. Risk inputs are incomplete for 11 of the 21 core companies, and 2 reconciliation items remain `precision_unknown` (Lantronix interest expense, 2020–21).
5. NYSE American cannot be told apart from NYSE.
6. There is no point-in-time price or survivorship-free data, so any later persistence study is exploratory.
7. Two interpretations await owner confirmation: I2 (the percentile reference population) and I5 (a separate group for provisional P/G/T).
8. The source code is **not** published, so the instructor sees reports, not code (§10).

## 8. R2 versus R3: design rationale

| Issue in the R2 score | R3 response |
|---|---|
| A weighted sum (20/20/25/25/10) of percentiles of one-period changes is compensatory: a large change can hide severe risk | Non-compensatory gates and a risk tier; improvement cannot buy back tier-2 risk |
| Low-base rebounds, acquisition-driven growth and base-year impairments rank above genuine improvers (mechanism cases A–H) | Classification by sign; evidence labels (prior peak, base-year special items, breadth); reversion risk demotes to Tier B |
| Growth carries 40% weight, although research says growth does not persist; a one-year ΔOCF is mostly working-capital noise | Growth is one path, not a weight; the cash path is checked against the margin change |
| A missing debt tag scored as 0 penalises companies with no debt | Four debt states; unknown never changes the tier; completeness shown separately |
| Percentile ranks depend on which other companies are in the pool | The key is computed from the company's own data, and G0.12 proves invariance |

What R3 keeps from R2: all of R2's data engineering (funnel, tag map, conversion, reconciliation and provenance). R2's score is retained only as a faithful, labelled comparator. **R3 is not claimed to be empirically better than R2;** that question is reserved for Phase 1 validation on groups with n ≥ 30. Full reasoning and citations are in `output/design/02_phaseB_research_phaseC_redteam.md` and `03_phaseD_candidates.md`.

## 9. Phase 1 roadmap (needs owner authorization)

1. **Run U4–U8 market-wide** on the 2,964 U3 companies. This takes about 40 minutes single-threaded from the local zips.
2. **Plan precision checks at scale.** D5 fetches original filings only for strict failures, which needs a bounded download plan within the rate limit.
3. **Produce the market-wide coverage report,** including group sizes P/G/T/W/U and attrition per rule. This is where n ≥ 30 groups make a variant comparison meaningful.
4. **Freeze the thresholds** from a design snapshot **before** any validation (R3 §2.8).
5. **Validate** as pre-registered in R3 §2.8:
   - an operational shortlist (RWC@20);
   - a blind owner review of the pooled top names from the five variants;
   - an exploratory persistence check against base rates;
   - sector concentration and threshold sensitivity.
6. **Build a 4-page dashboard** (Rankings, Company quick view with provenance, Coverage & Freshness, About-limitations), plus `shortlist.csv` and `run_meta.json`.

## 10. What the instructor can and cannot inspect

**Can inspect (this repository):**
- `spec.md`: goal, material questions, trust sources, outputs and success criteria, with the Phase 0 status.
- `output/design/01–05`: the complete design history. `05` is the approved R3 spec and the Phase 0 contract.
- `output/phase0/PHASE0_REPORT.md`: this file.
- `output/phase0/source_reports/`: unedited copies of the local Phase 0.9 checkpoint and the submission preview.
- `prompts.md`: the process log. `PROJECT_MEMORY.md`: problems, lessons and rules. `help.md`: questions for the instructor.

**Cannot inspect:**
- The source code, the 284 tests, the reconciliation fixtures and the machine-readable reports (`phase0_summary.json`, `coverage_report.json`). These live in a separate local repository that is not published.
- SEC downloads and intermediate data, which are regenerated locally and never committed.

The results above are therefore as reported by the local implementation. Publishing the code is an owner decision for later.

## 11. Questions for instructor feedback

See `help.md` (three focused questions). In brief:
1. Is a threshold-multiple ranking (no weighted composite, no percentiles) a defensible default for a discovery screen, or should a simple composite stay in the comparison set?
2. Is the honest-validity split (132 / 24 / 68 / 16) the right standard for data quality, or is arithmetic reconciliation enough at this stage?
3. Given incomplete risk inputs (debt unknown for many companies, interest expense mostly missing), should Phase 1 add guard data (for example dimensional XBRL debt) before the market-wide run, or run first and measure attrition?
4. For Phase 1 validation without prices, is a blind owner review plus exploratory persistence an acceptable success test?

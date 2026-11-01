> **หมายเหตุจาก repo การบ้าน (เพิ่มตอนคัดลอก 2026-11-01):** ไฟล์นี้คัดลอกจาก `reports/phase0_9/CHECKPOINT.md` ของ repo โค้ด local (ไม่เผยแพร่) โดยไม่แก้เนื้อหา ไฟล์ที่อ้างถึง (`phase0_summary.json`, `coverage_report.json`, รายงาน phase0_3–0_8, fixtures) อยู่ในเครื่องเท่านั้น ไม่ได้อยู่ใน repo นี้ · ประโยค "no changes to the homework repository" ใน §6 หมายถึงตอนจบ Phase 0.9 ก่อนรอบ 2 ของ repo นี้

# Phase 0.9 checkpoint — Phase 0 final technical validation summary

Date 2026-11-01 · SEC snapshot 2026-10-31 · local Git only (no remote, nothing pushed).
Machine-readable: `phase0_summary.json`, assembled by `radar summary-0.9` from the committed phase reports.

> **Scope.** Phase 0 validates the data pipeline and the mechanics of the R3 model. The ranking sample is a
> purposive 30 companies plus 2 separately labelled supplemental cases (Intapp, Boeing). This is **not empirical
> validation of model quality, not a market-wide estimate, and not evidence of investment performance.**
> Thresholds are provisional and were not tuned.

## 1. Phase 0.8 comparator corrections (owner review; record D8)
| # | Correction | Effect on the sample |
|---|---|---|
| 1 | **The R2 original follows R2 exactly** (eligibility from §4.1 U4–U7 and §5.4.1; 10-quarter M2, otherwise `provisional_no_accel` with ceiling 80; M5 debt condition = 0 with `debt_missing`, likewise `cash_missing`; weights 20/20/25/25/10; p1/p99 winsorizing and percentiles across R2 core; provisional companies on the same breakpoints; no minimum pool) | The R2 core pool grows from 14 (wrongly restricted) to **21**, all of them R3-core. 7 companies are scored with a 0 condition: 6 `debt_missing` (J&J, AMCON, Noble, RMR, Samsara, Symbotic) and 1 `cash_missing` (Atlas). |
| 2 | **Small samples:** the n ≥ 30 rule applies only to R3 `rank_variant_pct`. R2-fixed (n ≥ 2) and R2 original (any n) are computed with an EXPLORATORY warning | R2-fixed is computed for P (13), G (2) and T (2), and R2 original for 21. Every value is labelled "not evidence that any model outperforms another". |
| 3 | **Risk wording:** "No triggered risk flags — complete inputs" or "— incomplete inputs (…)"; field renamed `risk_no_flags_complete` | Of the 15 tier-0 core companies, 7 have complete inputs and 8 do not. |
| 4 | **Credible-path tie-breaks:** the key uses `n_credible_paths` and `second_credible_mult`; all triggered paths are kept for diagnostics | No change to the sample order: no company has a reversion-risk secondary path. A synthetic test proves the rule. |
| 5 | **Warnings:** `mixed_vintage` moved to provenance; material revisions and reconciliation conflicts (24 months to t) come first | Ranked companies show only informative warnings. The reconciliation conflicts appear on Gibraltar, LifeMD, Liberty Media and Montauk (one rounding item tolerated under D5). |
| 6 | **Report consistency:** the "six" M5 exclusions were actually 7 companies; the R2 pool is now documented | `reports/phase0_8/CHECKPOINT.md` §4 is corrected; the regression test pins the 6 + 1 list. |

The default R3 ranking is unchanged. G0.12, G0.13 and G0.14 pass again after the corrections. Re-running produces
byte-identical reports.

## 2. Final Phase 0 gates
| Gate | Final | Evidence |
|---|---|---|
| G0.1 exchange labels | PASS | Phase 0.3: distinct labels; NYSE American not separable (documented) |
| G0.2 submissions source | PASS | zip 1.57 GB in 146 s and parsed for all of U0 in 2.3 s (measured), against 27–53 minutes for the API (extrapolated) |
| G0.3 manual reconciliation | PASS on arithmetic | 240/240 values and 338/338 lineage links; independent validity in §3 |
| G0.4 year-to-date cash flow | PASS | OCF R1: 119 pass, 0 fail |
| G0.5 fiscal calendar | PASS | standard 8/8 plus the 10-KT transition (Lifecore 5/5) |
| G0.6 no invented values | PASS | every ok quarter traces to `facts_raw` ∪ companion; nulls stay null |
| G0.7 restatements | PASS | latest filed wins; all versions preserved |
| G0.8 compatibility · G0.9 tag splice | PASS | unit tests and real cases |
| G0.10 classification | PASS | truth table; 21 core + 3 provisional + 6 excluded = 30 |
| G0.11 debt states | PASS | fixtures and four real debt definitions |
| G0.12 ranking-key invariance | **PASS** | 27 leave-one-out runs, each company ranked alone, supplemental cases added, 5 shuffles: 0 key changes and 0 pairwise-order changes |
| G0.13 evidence exhaustive | PASS | property tests; on the sample, 0 companies with no path have a key |
| G0.14 coverage report | **PASS** | `reports/phase0_8/coverage_report.json` |

## 3. Confirmed findings
1. **Pipeline integrity.** `facts_raw` is unchanged from Phase 0.4 (39,640 rows, sha256 `1a7ec7d6…ca5d5`).
   - Re-running the reconstruction reproduces `quarterly.parquet` exactly.
   - `radar score` and `radar summary-0.9` are deterministic.
   - There are 284 tests.
2. **Reconciliation.** Of 58 strict failures, 9 are tolerated as verified rounding (decimals of every operand
   verified), 47 exceed the bound, and 2 are precision_unknown. All restatement cases stay rejected.
3. **Honest validity of the 240 manual values:**

   | Category | Values |
   |---|---|
   | Presented 3-month figure for that exact period | 132 |
   | Derived and corroborated by a release | 24 |
   | Valid by rule only | 68 |
   | Rejected by the pipeline | 16 |

4. **The R3 mechanics behave as specified.**
   - 11 sample companies are ranked: P 7, G 2, T 2.
   - 10 are Tier A and 1 is Tier B (Acushnet: no credible path).
   - 1 multiple is capped at 5× (Murphy).
   - The 6 core P companies with no path, including Montauk, are searchable but unranked.
   - W, U and excluded companies are never ranked.
5. **The comparators differ for structural reasons (exploratory, mechanics only):**
   - R2 original scores W and no-path companies; Insmed (W) comes first.
   - R2 original ignores reversion-risk demotion; Acushnet ranks above Tier A companies.
   - R2-fixed and R2 original reward even improvement across all signals (Laureate moves up).
   - The count variant shows the "2 modest beat 1 strong" pattern that R3 rejected.

## 4. Measured values replacing planning assumptions
| Item | Measured | Scope |
|---|---|---|
| submissions.zip | 1,572,128,284 bytes; 146 s download; 2.3 s parse for all of U0 | actual |
| companyfacts.zip | 1,414,913,699 bytes; 123 s download | actual |
| Funnel U0→U3 | 8,023 → 6,036 → 4,217 → 2,964 | actual listed universe |
| Recent coverage, U3 pool (2,960) | OI 95.0% · revenue 93.2% · OCF 99.8% · cash 98.9% · any debt tag 67.5% | tag presence |
| Special-item tags, U3 pool (recent) | restructuring 29.4% · goodwill impairment 43.3% · asset impairment 38.2% | tag presence |
| 12-quarter prior check (21 sample core) | unknown for margin 2, accel 1, cash 1 | purposive sample |
| Base-year special items (21 core) | unknown for 15 | purposive sample |
| G / T group sizes | 2 / 2 in the sample (plus 1 / 1 supplemental) | purposive; market size unknown until Phase 1 |
| Risk completeness (21 core) | 1.0 for 10 · 0.5 for 8 · 0.0 for 3; debt unknown for 3 | purposive sample |
| Pipeline time on 27 companies | reconstruction 21.3 s · assess 0.5 s · score (with G0.12 runs) 2.0 s | local machine |
| Local data | raw 3.0 GB · interim 5.7 MB · universe 1.9 MB (all gitignored) | local |

## 5. Unresolved limitations
1. **No model comparison is possible yet.** The sample groups (P 13, G 2, T 2) are far below n = 30. Every
   R2-fixed and R2-original value is exploratory, and `rank_variant_pct` is null.
2. **Missing data limits the guards.**
   - Interest expense is missing at t for 18 of 21 core companies.
   - The base-year special-items check is unknown for 15 of 21.
   - Risk inputs are incomplete for 11 of 21.
   - Dimension-only debt and finance-lease facts are unknown in companyfacts.
3. **Restated vintages** (LifeMD, Liberty Media, Gibraltar) and Lifecore's restart after its transition keep these
   companies out of core. This is correct under strict compatibility, but it shortens their history.
4. **Two reconciliation failures remain `precision_unknown`** (Lantronix interest expense, 2020–21).
5. **NYSE American is not separable.** There is no point-in-time price or survivorship-free data, so any later
   persistence study is exploratory.
6. **Interpretations I2 and I5** (percentile reference population; separate group for provisional P/G/T) stand
   pending owner confirmation.

## 6. Phase 1 readiness
**Technically ready, subject to owner authorization.** The pipeline, gates, fixtures and reports are in place.

What Phase 1 needs:
1. Run U4–U8 market-wide on the 2,964 U3 companies. The local zips already make this feasible: about 21 s of
   reconstruction per 27 companies, roughly 40 minutes in total, single-threaded.
2. Decide precision checks at scale. D5 fetches original filings only for strict failures, which needs a bounded
   download plan and adherence to the rate limit.
3. Produce the market-wide coverage report; this is where n ≥ 30 groups make the variant comparison meaningful.
4. Freeze thresholds from a design snapshot before any validation (R3 §2.8).
5. Make owner decisions on the submission preview (pyproject author address; whether to include the R2 reference
   document).

Not done, as instructed: no dashboard, no Phase 1, no push, no `/save`, no changes to the homework repository.

## 7. Submission preview (sanitized, not published)
- `submission_preview/SUBMISSION_PREVIEW.md` holds the write-up; `MANIFEST.md` lists the proposed contents and
  exclusions.
- The sanitization check in `phase0_summary.json` passes with 0 findings: no SEC User-Agent or contact, no e-mail, no
  local home path, no secret-like strings.
- `radar check-leaks` is clean on all committable files.

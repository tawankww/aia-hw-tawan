# Phase E — Final Design Proposal: R3 v0.1 (US Fundamental Inflection Radar)
วันที่ 2026-10-31 · สถานะ: **รอ final approval** · ไม่ implement · ไม่แก้ R2 plan · ไม่เริ่ม Phase 0

> หลักการ: smallest reliable discovery system · ทุก threshold เป็น config ที่ยังไม่ finalize · ทุกข้ออ้างแยก [E] evidence / [R] reasoning / [H] hypothesis · ไม่ claim predictive performance

---

## E1. Final Owner Mandate alignment (สรุปสิ่งที่ล็อกแล้ว)
| Mandate | การแปลงเป็นกฎใน R3 v0.1 |
|---|---|
| Fundamental Inflection Discovery ไม่ใช่ quality/growth screener | ไม่มี level-based ranking; rank ด้วย *การเปลี่ยนแปลง* หลัง gate ตาม sign เท่านั้น |
| Early discovery + evidence strength โปร่งใส | ฐาน = TTM yoy (8 ไตรมาส); evidence เป็น attributes + label แยก; confirmed = label/filter ไม่ใช่ rank advantage |
| 3 กลุ่ม P/G/T + W searchable | truth table E2 (exhaustive, mutually exclusive, tested) |
| Improvement ชดเชย severe risk ไม่ได้ | risk tier 2 = demote ใต้ทุกตัวใน view (ไม่ exclude) — กฎ non-compensatory ข้อเดียว |
| ไม่เอา 3 scoring models / ไม่เอา weighted composite ที่ไม่มีหลักฐาน | ชั้นร่วมเดียว + ranking key เดียว (E5) + diagnostic columns สำหรับ validation |
| R2 เป็น baseline | pipeline/provenance/recon ของ R2 คงไว้ทั้งหมด; score R2 คำนวณเป็น diagnostic |
| ไม่ใช้ราคาใน v0.1 | validation = persistence (exploratory) + blind review เท่านั้น |

---

## E2. Classification — truth table ที่แก้แล้ว (Correction A)
นิยาม (TTM ณ t และ t−4; ต้องมี 8 ไตรมาส status ok):
- `oi_pos(k)` = TTM_OI(k) > 0 · `ocf_pos(k)` = TTM_OCF(k) > 0
- **Upward crossing:** `oi_up` = ¬oi_pos(t−4) ∧ oi_pos(t) · `ocf_up` = ¬ocf_pos(t−4) ∧ ocf_pos(t)
- **Downward crossing:** `oi_down` = oi_pos(t−4) ∧ ¬oi_pos(t) · `ocf_down` = ocf_pos(t−4) ∧ ¬ocf_pos(t)

**Precedence (ตรวจตามลำดับ หยุดที่ข้อแรกที่จริง):**
1. `short_history` (5–7 ไตรมาส) → **U · Unclassified** (Provisional view; แสดง quarterly yoy ที่คำนวณได้)
2. ¬ocf_pos(t) ∨ oi_down ∨ ocf_down → **W · Watch** (ไม่ rank; reason = `ocf_negative` / `oi_turned_negative` / `ocf_turned_negative` / `mixed_crossing`)
3. oi_up ∨ ocf_up → **T · Credible Turnaround** (subtype: `double_cross` / `oi_cross` / `ocf_cross`)
4. oi_pos(t) ∧ ocf_pos(t) → **P · Profitable Improvers** (ณ จุดนี้รับประกันว่า oi_pos(t−4) ∧ ocf_pos(t−4) เพราะไม่มี crossing)
5. ¬oi_pos(t) ∧ ocf_pos(t) → **G-candidate** (รับประกัน ¬oi_pos(t−4) ∧ ocf_pos(t−4)) → ถ้า S_g > 0 → **G · Cash-Generating Growth**; ถ้า S_g ≤ 0 → **W** reason `loss_firm_no_growth`

| # | oi(t−4) | ocf(t−4) | oi(t) | ocf(t) | class | เหตุผล |
|---|---|---|---|---|---|---|
| 1 | − | − | − | − | W | ocf_negative |
| 2 | − | − | − | + | T | ocf_cross |
| 3 | − | − | + | − | W | ocf_negative |
| 4 | − | − | + | + | T | double_cross |
| 5 | − | + | − | − | W | ocf_turned_negative |
| 6 | − | + | − | + | G / W | S_g > 0 → G; else loss_firm_no_growth |
| 7 | − | + | + | − | W | ocf_turned_negative (+ oi_up ถูก override) |
| 8 | − | + | + | + | T | oi_cross |
| 9 | + | − | − | − | W | ocf_negative, oi_down |
| 10 | + | − | − | + | W | mixed_crossing (ocf_up แต่ oi_down) |
| 11 | + | − | + | − | W | ocf_negative |
| 12 | + | − | + | + | T | ocf_cross (บริษัทกำไรที่ OCF เพิ่งบวก) |
| 13 | + | + | − | − | W | oi_turned_negative, ocf_turned_negative |
| 14 | + | + | − | + | W | oi_turned_negative |
| 15 | + | + | + | − | W | ocf_turned_negative |
| 16 | + | + | + | + | P | — |
**Test บังคับ (G0.10):** ทุกบริษัท core อยู่ในคลาสเดียว; Σ P+G+T+W = core; Σ core + U + excluded = U0
**Ranked views:** P, G (เฉพาะที่มี ≥ 1 path triggered — ตาม decision 2), T · **Non-ranked:** W (sort ตาม S_m เพื่อ diagnostic ไม่มีเลขอันดับ), U, Excluded

---

## E3. Financial Risk — แก้ตาม Correction B, C
### E3.1 Debt state (ห้าม infer zero จาก tag ที่หาย)
| state | เงื่อนไข | leverage flag |
|---|---|---|
| `known_debt` | มี fact หนี้ ≥ 1 tag (`LongTermDebt` / `LongTermDebtNoncurrent`+`LongTermDebtCurrent` / `…Noncurrent`+`DebtCurrent` / `ShortTermBorrowings`) ที่ quarter end t ค่า > 0 | คำนวณได้ |
| `confirmed_zero` | มี fact หนี้ที่ t ค่า = 0 **อย่างชัดเจน** (บริษัท tag ค่า 0) | leverage_high = false |
| `unknown` | ไม่มี fact หนี้ใดที่ t | leverage_high = **unknown** (ไม่ true ไม่ false); `InterestExpense` ถ้ามี แสดงเป็น hint `interest_expense_present` เท่านั้น ไม่เปลี่ยน state |
### E3.2 Risk flags (ทุกกลุ่ม ranked มี OCF > 0 โดยนิยาม)
| flag | นิยาม | สถานะถ้าข้อมูลขาด |
|---|---|---|
| `leverage_high` | known_debt ∧ net_debt > 0 ∧ net_debt / TTM_OCF > L (default 5, **ไม่ finalize**) | unknown ถ้า debt unknown หรือ cash null |
| `fcf_negative` | TTM_OCF − TTM_capex < 0 | unknown ถ้า capex null — **informational เท่านั้น ไม่เข้า tier** (growth capex ≠ distress) |
| `runway_short` | fcf_negative ∧ cash(t) < R × |FCF ต่อไตรมาส| (default R = 4, ไม่ finalize) | unknown ถ้า capex/cash null — แยก growth capex ได้เพราะบริษัทที่มีเงินสดพอไม่ติด |
| `external_financing_dependence` | TTM financing CF > 0 ∧ fcf_negative | unknown ถ้า financing CF null — ไม่ใช้ threshold 5% revenue แล้ว (ตัดออก: proxy อ่อน) |
### E3.3 Risk tier (non-compensatory, demote เท่านั้น)
- **Tier 2 (severe):** `runway_short` = true **หรือ** (`leverage_high` = true ∧ `external_financing_dependence` = true) — ต้องเป็น signature ที่สอดคล้องกับ distress จริง ไม่ใช่สอง proxy อ่อนบวกกัน (Correction C)
- **Tier 1:** มี flag true อย่างน้อย 1 ใน {leverage_high, external_financing_dependence} แต่ไม่เข้า tier 2
- **Tier 0:** ทุก flag ที่คำนวณได้ = false
- **unknown ไม่เคยทำให้ tier สูงขึ้นหรือต่ำลง**; แสดง `risk_completeness` = จำนวน flag ที่คำนวณได้ / จำนวนที่ applicable และ `risk_tier_basis` = "complete" / "partial"
- ผลต่อลำดับ: tier 2 → Tier B ของ view (ใต้ทุกบริษัท Tier A); tier 0/1 ไม่ต่างกันในลำดับ (แสดงเป็น label)
- **Thresholds L, R ต้องผ่าน coverage + sensitivity ใน Phase 0/1 ก่อน finalize** (decision 4)

---

## E4. Inflection strength + Evidence attributes (Correction D)
### E4.1 Strength (3 เส้นทาง, TTM yoy ยกเว้น accel)
| path | สัญญาณ | trigger threshold T (config, ไม่ finalize) | เงื่อนไขประกอบ |
|---|---|---|---|
| margin | S_m = OPM(t) − OPM(t−4) pp | T_m = 2 pp | — |
| accel | S_a = mean[q_yoy(t), q_yoy(t−1)] − mean[q_yoy(t−2), q_yoy(t−3)] pp | T_a = 5 pp | S_g > 0 ∧ S_m ≥ −1 pp (growth ที่ margin เสื่อม > 1 pp ไม่นับ) |
| cash | S_c = OCFM(t) − OCFM(t−4) pp | T_c = 2 pp | AC = S_c − S_m ≤ 5 pp (ไม่งั้น `ocf_outruns_oi`, cash path ไม่ trigger) |
ไตรมาสที่ต้องการ: ทุก path 8 (t−7..t) — ตรวจแล้ว
### E4.2 Evidence attributes (แยกเป็น attribute ต่อ path ก่อน แล้วค่อยสรุป label)
| attribute | นิยาม | ค่า | ไตรมาส |
|---|---|---|---|
| `breadth_margin` | # k ∈ {t−3..t} ที่ OPM_q(k) > OPM_q(k−4) | 0–4 | 8 |
| `breadth_accel` | # k ∈ {t−2..t} ที่ q_yoy(k) > q_yoy(k−1) | 0–3 | 8 |
| `prior_margin` | OPM(t) vs max[OPM(t−4), OPM(t−8)] | above / not_above / **unknown** (< 12q) | 12 |
| `prior_growth` | S_g(t) vs S_g(t−4) | above / not_above / unknown | 12 |
| `base_special_items` | (Restructuring + GoodwillImpairment + AssetImpairment) ใน TTM(t−4) > 2% rev(t−4) | flagged / clear / **unknown** (ไม่มี tag) | 8 |
| `acq_warning` | asset growth − S_g > 20 pp ∨ Δgoodwill/assets(t−4) > 5% ∨ acquisitions TTM/assets > 5% | flagged / clear / unknown | 8 (**warning** ไม่ใช่หลักฐาน M&A) |
| `curtailment` | S_g < 0 ∧ S_m > 0 | flagged / clear | 8 |
| `sbc_dependent` (G, T) | SBC TTM ≥ OCF TTM | flagged / clear / unknown | 4 |
| pipeline flags | restated, tag_spliced, mixed_vintage, rev_jump, disc_ops, recon_fail | — | — |
**กฎ unknown:** unknown ไม่เคยเป็นบวกหรือลบ; แสดงเป็น "?" ใน dashboard; ไม่มี path ใดถูกปิดเพราะ unknown (decision 6)
### E4.3 Evidence label ต่อ path (deterministic)
สำหรับ path p ที่ trigger:
- `reversion_risk` ถ้า prior_p = not_above ∨ base_special_items = flagged (margin path) — accel path: prior_growth = not_above
- else `single_period` ถ้า breadth_margin ≤ 1 (margin) / breadth_accel = 0 (accel) / cash path: ใช้ breadth_margin (OCF รายไตรมาส noisy เกินไป — [R])
- else `confirmed` ถ้า prior_p = above ∧ breadth_margin ≥ 3 (margin) / breadth_accel ≥ 2 (accel)
- else `early` (รวมกรณี prior unknown)
### E4.4 Overall evidence label (exhaustive, mutually exclusive, precedence)
1. `insufficient` — class U (< 8q)
2. `none` — ไม่มี path trigger (บริษัทยังอยู่ใน P/G/T แต่ไม่อยู่ใน ranked list ของ G; ใน P/T อยู่ท้ายสุด)
3. `reversion_risk` — path ใดก็ตามที่ trigger มี label reversion_risk
4. `single_period` — ไม่มี reversion_risk และ **ทุก** path ที่ trigger เป็น single_period
5. `confirmed` — ไม่มีข้อ 3–4 และ path ที่ใช้เป็น primary (E5) เป็น confirmed
6. `early` — อื่น ๆ ทั้งหมด
ผลต่อลำดับ: ข้อ 3 → Tier B; ข้อ 4 → cap magnitude (E5); ข้อ 5–6 **ไม่ต่างกัน** (decision 5)

---

## E5. Ranking mechanics — R3 v0.1 (Correction E: ไม่ใช่ path-count ล้วน ไม่ใช่ max-percentile ล้วน, ไม่มีน้ำหนักเพิ่ม, ไม่มี winsorize)
### E5.1 Threshold multiple (ไม่มี parameter ใหม่)
สำหรับ path p ที่ trigger: `mult_p` = S_p / T_p (เช่น margin +6 pp / 2 pp = 3.0×; accel +10 pp / 5 pp = 2.0×) — ใช้ threshold ตัวเดียวกับ trigger จึง **ไม่เพิ่ม parameter**; ความหมายอ่านง่าย: "ดีขึ้น 3 เท่าของขั้นต่ำที่ถือว่ามีนัย"
- **cap:** mult_p ≤ C_max (default 5×) เพื่อไม่ให้ observation สุดโต่งเดียวครอง (แทน winsorize/percentile)
- **single_period cap:** ถ้า path p มี label single_period → mult_p ≤ 1.0 (trigger แต่ไม่ได้รับเครดิตขนาด — จำกัด noise ไตรมาสเดียว ไม่ใช่การให้รางวัล confirmed)
### E5.2 Ranking key (lexicographic ภายในกลุ่ม P / G / T แยกกัน)
1. **Tier:** A = risk tier ≤ 1 ∧ evidence ≠ reversion_risk; B = อื่น ๆ (ใน T เพิ่ม curtailment = flagged → B)
2. **best_mult** = max_p mult_p (primary path = p ที่ให้ค่านี้) — "เส้นทางเดียวที่แรงและน่าเชื่อถือ" ชนะ "สองเส้นทางพอประมาณ" ได้ (decision 1) แต่ไม่ชนะเสมอ: 2 paths ที่ 2.5× ชนะ 1 path ที่ 2.0×
3. **n_paths** (จำนวน path trigger) — tie-break แรก
4. **second_mult** — tie-break สอง
5. cik
ไม่มี sum, ไม่มีน้ำหนัก, ไม่มี percentile ในลำดับหลัก → ลำดับของบริษัทหนึ่งไม่ขึ้นกับบริษัทอื่น (repeatability ตรงไปตรงมา)
### E5.3 Diagnostic columns (คำนวณเสมอ ไม่ใช้จัดลำดับ default; ใช้ใน validation E8)
- `rank_variant_count` = ลำดับตาม (Tier, n_paths, best_mult) — คือ R3-min เดิม
- `rank_variant_pct` = ลำดับตาม (Tier, max_p pct_within_group(S_p | เงื่อนไข)) — คือ R3-full **โดยไม่ winsorize** (percentile ไม่เปลี่ยนเมื่อ winsorize → ตัดออกตามข้อสังเกตเจ้าของ); ถ้ากลุ่ม n < 30 → null
- `score_r2fixed` = 0.25·Σ pct(M1..M4) ภายในกลุ่ม — baseline control
- `rank_in_sector` = ลำดับตาม key E5.2 ภายใน SIC-2 (ข้อมูลประกอบ)
สลับ default ได้ด้วย config `ranking.variant ∈ {multiple, count, pct}` — ไม่ใช่ระบบแยก
### E5.4 Behavioral sanity checks (A–H) — **sanity ของกลไก ไม่ใช่หลักฐานเชิงประจักษ์** (Correction F)
| | class | tier | best_mult (คาด) | หมายเหตุ |
|---|---|---|---|---|
| A | W | — | — | ไม่ rank ✔ |
| B | P | A | margin 4/2 = 2.0×, cash 2.0× → n_paths 2 | |
| C | G | A | cash 7/2 = 3.5× (AC = 3 ≤ 5), margin 2.0× | top ของ G ✔ |
| D | P | **B** (reversion_risk ถ้ามี 12q หรือ special-items tag) | 3.0× | ใต้ Tier A ทั้งหมด ✔; ถ้า 8q + ไม่มี tag → early ปน (ข้อจำกัดประกาศแล้ว) |
| E | P | A | margin 2/2 = 1.0× (breadth 4, confirmed) | อยู่ใน list แต่ไม่ต้น — ตาม decision 5; filter "confirmed" ดึงขึ้นมาได้ |
| F | P | A | margin 5/2 = 2.5×, cash 2.5× → n_paths 2 | |
| G | P | A | accel 10/5 = 2.0× (ถ้า half-year ≈ 10 pp), n_paths 1 | G vs F: F 2.5× > G 2.0× → **F ยังเหนือ G ด้วยขนาด ไม่ใช่ด้วยจำนวน path**; ถ้า accel ของ G = +15 pp → 3.0× → G เหนือ F — ตอนนี้ตัดสินด้วย magnitude ที่เทียบกับ threshold ไม่ใช่กฎโครงสร้าง (ตอบ decision 1) |
| H | P | **B** (leverage_high ∧ external_financing_dependence ถ้า FCF < 0; มิฉะนั้น tier 1 + acq_warning → Tier A แต่ accel ไม่ trigger เพราะ S_m = −3 < −1 → n_paths 0 → ท้ายสุด) | — | ไม่ติด top ทั้งสองกรณี ✔ |

---

## E6. Candidate comparison (final) — ความต่างทั้งหมดอยู่ที่ E5 เท่านั้น ชั้น E2–E4 เหมือนกัน
| มิติ | R2 เดิม | R2-fixed (diagnostic) | R3 count (R3-min เดิม) | R3 pct (R3-full ไม่ winsorize) | **R3 multiple (เสนอ)** |
|---|---|---|---|---|---|
| โครงสร้าง | weighted sum 5 ตัว | equal-weight sum 4 ตัว ใน tier | lexicographic count | max percentile ใน tier | max threshold-multiple ใน tier |
| Compensatory? | ใช่ | ใช่ (ใน tier) | ไม่ | ไม่ | ไม่ |
| "2 modest > 1 strong"? | — | — | **ใช่ (ปัญหา)** | ไม่ | ไม่ (ตัดสินด้วยขนาดเทียบ threshold) |
| Outlier เดียวครอง top? | winsorize ช่วยบางส่วน | เหมือน | ไม่ (count) | **ใช่ (ปัญหา)** | จำกัดด้วย cap 5× + single_period cap |
| ขึ้นกับ universe | ใช่ | ใช่ | ไม่ | ใช่ (+ n<30 ปัญหา) | **ไม่** |
| Parameters เพิ่มจาก trigger | weights 5 + winsor | weights | 0 | 0 | 1 (C_max) |
| Interpretability | ต่ำ | กลาง | สูง | กลาง | **สูง** ("margin +6 pp = 3× ขั้นต่ำ") |
| Implementation | กลาง | กลาง | ต่ำ | กลาง | ต่ำ |
ข้อสังเกตที่เจ้าของแก้ไว้: ความต่างของ ranking variant **ยังเป็นสมมติฐาน**; ตัวอย่าง A–H ไม่พอตัดสิน → ตัดสินด้วย E8 บนข้อมูลจริง

---

## E7. SEC data feasibility & unresolved coverage
| สิ่งที่ต้องการ | tag | สถานะ | ถ้าขาด |
|---|---|---|---|
| revenue, OI, OCF, cash | ตาม R2 5.1 | tag มาตรฐาน; **coverage OperatingIncomeLoss ไม่มีสถิติสาธารณะ → วัด Phase 0** | OI ขาด → excluded `missing_tag` (คงเดิม); fallback ยังไม่อนุมัติ |
| debt | ตาม E3.1 + `ShortTermBorrowings` | state 3 ค่า | unknown |
| financing CF | `NetCashProvidedByUsedInFinancingActivities` | มาตรฐาน | flag unknown |
| capex | `PaymentsToAcquirePropertyPlantAndEquipment` | มาตรฐาน | fcf/runway unknown |
| assets, goodwill, acquisitions | `Assets`, `Goodwill`, `PaymentsToAcquireBusinessesNetOfCashAcquired` | มาตรฐาน; goodwill/acq ไม่ tag ≠ ไม่มี | acq_warning unknown/clear ตามกฎ: ถ้า `Assets` มีแต่ goodwill/acq ไม่มี → ใช้เฉพาะเกณฑ์ asset growth |
| special items | `RestructuringCharges`, `GoodwillImpairmentLoss`, `AssetImpairmentCharges` | มาตรฐาน; coverage ต่ำน่าจะเป็นปกติ (มีเมื่อมีรายการ) | unknown |
| SBC | `ShareBasedCompensation` | มาตรฐาน | unknown |
| disc ops | `IncomeLossFromDiscontinuedOperationsNetOfTax` | มาตรฐาน | ไม่ flag |
| interest | `InterestExpense` | hint เท่านั้น | — |
**ยังไม่รู้และต้องวัดก่อน finalize:** (1) % core ที่มี OI tag (2) % ที่ debt = unknown (3) % ที่มี 12 ไตรมาส (prior check ใช้ได้กี่ %) (4) % ที่ special-items tags มี (5) ขนาดกลุ่ม G และ T จริง (ถ้า < 30 → pct variant null) (6) การกระจายของ S_m/S_a/S_c เพื่อตั้ง T_m/T_a/T_c ให้ "มีนัย" (ตั้งจาก design snapshot เท่านั้น ห้ามแตะหลังเห็น selection snapshot)
**ข้อจำกัดเครือข่าย:** Phase 0 ต้องรันบน Mac mini (sec.gov ถูกบล็อกใน cloud session) — คงเดิมจาก R2

---

## E8. Pre-registered validation protocol (Correction F)
### E8.1 สองการประเมินที่แยกกัน
- **(i) Operational shortlist evaluation:** รายการที่ผู้ใช้จะเห็นจริง = top-12 P + top-5 G + top-3 T ของ **default variant** (quota ปรับได้ ไม่บังคับครบ) → วัด RWC@20 ของรายการนี้ชุดเดียว = "ระบบใช้งานได้ไหม"
- **(ii) Blind-review candidate pool:** union ของ top-30 P + top-10 G + top-5 T จาก 4 variant (multiple / count / pct / r2fixed) บน selection snapshot → ตัดซ้ำ สุ่ม ซ่อน variant และซ่อน label → เจ้าของให้ worthy yes/no + เหตุผลจาก list → คำนวณต่อ variant: RWC@20, @30, overlap (Jaccard), unique discoveries + yes-rate ของ unique, rejection histogram
- A–H = **mechanism sanity checks** ในชุดทดสอบอัตโนมัติ (ใช้ fixture ที่สร้างจากกลไก) ไม่รายงานเป็นหลักฐาน
### E8.2 Fundamental persistence (exploratory retrospective)
- Pseudo-PIT: T0 = snapshot − 24 เดือน; ใช้เฉพาะ facts ที่ `filed ≤ T0`; วัดที่ t+4 ด้วย facts filed ∈ (T0, snapshot]
- นิยาม sustained (เจ้าของ): P: OPM(t+4) − OPM(t−4) ≥ 0.5·S_m ∧ oi_pos ∧ ocf_pos ที่ t+4; growth path: S_g(t+4) ≥ 0.5·S_g(t) > 0; G: ocf_pos(t+4) ∧ (S_m(t+4) ≥ 0 ∨ oi_pos(t+4)); T: ocf_pos(t+4) ∧ S_g(t+4) ≥ 0
- **Base rate บังคับ:** เทียบกับบริษัทกลุ่มเดียวกันที่ไม่ติด top-N และ random sample ของกลุ่ม; รายงานความต่าง + bootstrap CI; รายงานแยกตาม evidence label (ให้ข้อมูลเปรียบเทียบ mean reversion ที่ guardrail 1 ต้องการ)
- Sensitivity: T_m, T_a, T_c, L, R, C_max แต่ละตัว ×0.5 / ×2 → Jaccard ของ top-20 และ persistence; ผลพลิก = ไม่เสถียร ต้องรายงาน
- **ข้อจำกัดที่ประกาศล่วงหน้า (ห้ามอ้างเกิน):** (a) ไม่ได้สร้าง historical universe ที่สมบูรณ์ — บริษัทที่ deregister/delist ระหว่าง T0..snapshot หายจาก companyfacts (b) vintage ที่ถูกแทนที่หรือลบออกจาก companyfacts อาจไม่กลับมา → pseudo-PIT ไม่เท่ากับ as-reported จริง (c) survivorship bias **ไม่เท่ากันข้าม variant และข้ามกลุ่ม**: กลุ่ม T/G มี failure rate สูงกว่า → persistence ของ T/G และของ variant ที่เลือกบริษัทเหล่านี้มากกว่าถูกเป่าให้สูงเกินจริงมากกว่า — ต้องรายงานสัดส่วน P/G/T ใน top-N ของแต่ละ variant ควบคู่ (d) ไม่มีการทดสอบผลตอบแทน
- Prospective: บันทึก snapshot ทุกเดือนหลัง Phase 1 (scores + run_id) เพื่อวัด persistence จริงใน 12 เดือน
### E8.3 Pre-registered decision rule สำหรับ default variant
- default เริ่มต้น = **multiple**; เปลี่ยนเป็น variant อื่นได้เฉพาะเมื่อใน (ii) RWC@20 สูงกว่า ≥ 4 ตัว **และ** persistence (E8.2) ไม่แย่กว่า **และ** ไม่ได้มาจาก variant ที่เอียงไป T/G มากกว่าอย่างมีนัย (ข้อ c)
- ถ้าไม่มี variant ใดผ่าน → คง multiple; ถ้า count เท่ากับ multiple → เลือก count (ง่ายกว่า)
### E8.4 Stability / concentration / coverage / failure analysis
Spearman + Jaccard top-50 ระหว่าง 2 snapshot ห่าง 1 ไตรมาส; HHI SIC-2 ของ top-50 vs กลุ่ม; funnel U0→core→P/G/T/W/U; negative controls (bank/REIT/20-F excluded; 10-K/A → restated; ≥ 3 บริษัท M&A ใหญ่ใน 30 ตัว → acq_warning); manual inspection top-20 P + ทุก T → บันทึก failure ทุกตัว (pipeline / data / model)

---

## E9. Decision table R2 → R3 v0.1
| หัวข้อ | R2 | R3 v0.1 | เหตุผล | หลักฐาน | trade-off |
|---|---|---|---|---|---|
| โครงสร้าง | weighted score 20/20/25/25/10 | gates (sign) → classification → non-compensatory tier → ranking ด้วย max threshold-multiple | mandate เป็น non-compensatory; น้ำหนักไม่มีหลักฐาน | [R] Einhorn 1970; Dawes 1979; Novy-Marx 2016; F/G-score เป็น gated binary [E] | เสีย "score ตัวเดียว" ที่ดูแม่น; ได้ลำดับที่อธิบายได้และไม่ขึ้นกับ universe |
| M1 growth 20% | ranking factor | S_g เป็นเงื่อนไขของ accel path + gate ของ G; ไม่ rank ด้วย level ของ growth | growth level ไม่ persist และเป็น glamour | [E] Chan et al. 2003; LSV 1994; FF 2006 | บริษัทโตเร็วคงที่ไม่ได้ priority (ตาม mandate) |
| M2 accel TTM 10q | g(t)−g(t−2) | half-year quarterly yoy change (8q) | หลักฐานใช้ quarterly yoy; TTM second-diff ไม่มี paper และต้องการ 10q | [E] He & N 2020 (earnings); [H] revenue | รับ noise ไตรมาสมากขึ้นเล็กน้อย (half-year ลดลง) |
| M3 margin 25% | Δ จุดเดียว percentile | S_m + evidence attributes (breadth, prior-level, special items) | reversion ครอบงำ Δ จุดเดียว | [E] FF 2000; Burgstahler 2002; Akbas 2017 (trend เพิ่มข้อมูล) | ต้องการ 12q สำหรับ prior check (optional) |
| M4 OCF 25% | Δ จุดเดียว percentile | OCF > 0 เป็น gate/classification; S_c เป็น path เมื่อ AC ≤ 5 pp | ΔOCF ปีเดียว = working capital noise; level เป็นหลักฐานคุณภาพ | [E] Dechow 1994; Ball et al. 2016; Kapons & Veenman 2024 | บริษัทที่ OCF ดีขึ้นเร็วกว่า OI มากไม่ได้เครดิต (ตั้งใจ) |
| M5 stability 10% | 3 เงื่อนไขใน sum | classification (sign) + risk tier demote; debt 3 states | gate ไม่ใช่ 10% ของ sum; null ≠ 0 ≠ low-risk | [R]; BRS 2006 [E] สำหรับ financing | thresholds L, R ยังไม่ finalize |
| Flags | ไม่แตะ score | reversion_risk/curtailment(T)/risk tier 2 → Tier B; อื่นเป็น label | FP mechanisms ต้องมีผลต่อลำดับ | [R] จาก B1/B4/B8 | FP บางแบบยังไม่ติด (8q ไม่มี tag) |
| Provisional เพราะ M2 | 10q | ตัดเงื่อนไข; 8q = core; 5–7q = U | ลดการตัดบริษัทอายุน้อยโดยไม่จำเป็น | นับไตรมาส [R] | — |
| Sector | score เดียว + rank_in_sector | เหมือนเดิม | หลักฐานไม่รองรับ sector scoring; pp เทียบได้พอ | [E] Novy-Marx 2011 (within-industry) [R] | threshold pp สัมบูรณ์ bias ต่อ high-margin sector → ดู HHI ใน validation |
| Winsorize | p1/p99 ก่อน percentile | ตัดออก (percentile invariant); cap 5× ใน multiple | ซ้ำซ้อน | เจ้าของชี้ | — |
| Validation | ไม่มี | persistence (exploratory) + blind review + decision rule pre-registered | — | — | ไม่พิสูจน์ predictive validity |

---

## E10. Minimal Phase 0 implementation specification (เพิ่มจาก R2 Phase 0 — ไม่ลงมือตอนนี้)
1. **config.yaml เพิ่ม:** tag map ใหม่ (E7), thresholds {T_m 2, T_a 5, T_c 2, AC_max 5, L 5, R 4, C_max 5, base_special_pct 2, acq: 20pp/5%/5%}, `ranking.variant: multiple`, quotas {P 12, G 5, T 3}
2. **โมดูลใหม่ 4 ตัว (pure functions, deterministic):** `classify(oi_t4, ocf_t4, oi_t, ocf_t, s_g, n_quarters)` → class+reason · `risk(debt_state, net_debt, ocf, capex, cash, fin_cf)` → flags+tier+completeness · `evidence(series)` → attributes+labels · `rank(group_df, variant)` → rank columns
3. **ตาราง scores.parquet เพิ่มคอลัมน์:** class, class_reason, crossing_type, risk_tier, risk_completeness, debt_state, flags[], paths[], mult_*, best_mult, primary_path, n_paths, evidence_label, evidence_* attributes, rank_default, rank_variant_count, rank_variant_pct, score_r2fixed, rank_in_sector
4. **Tests เพิ่ม:** G0.10 truth table 16 แถว + exclusivity บน 30 บริษัท · debt 3 states (fixture: มี tag=0 / มี tag>0 / ไม่มี tag + มี interest) · unknown ไม่เปลี่ยน tier · evidence label exhaustive (property test: ทุก combination ของ attributes ให้ label เดียว) · ranking invariance (เพิ่ม/ลบบริษัท → rank_default ของตัวอื่นไม่เปลี่ยน) · A–H mechanism fixtures
5. **coverage_report เพิ่ม:** % core ต่อ tag ใน E7, ขนาด P/G/T/W/U, % มี 12q, distribution ของ S_m/S_a/S_c (สำหรับเลือก threshold จาก design snapshot)
6. **Gate ใหม่ก่อน Phase 1:** G0.10 classification, G0.11 debt states, G0.12 ranking invariance, G0.13 coverage ของ guard tags รายงานครบ
7. ลำดับงาน Phase 0 ของ R2 คงเดิม (0.0–0.9); งานข้อ 1–6 แทรกที่ 0.2 (config/skeleton), 0.5 (โมดูล), 0.7 (tests), 0.8 (score+coverage)

## E11. สิ่งที่อยู่นอก v0.1
ราคา/return validation (Phase 2 diagnostic), ΔATO signal, gross profitability, sector-specific thresholds, OI fallback tags, PIT universe จริง, LLM ใด ๆ, Alpha export, scheduler

## E12. Unresolved assumptions ที่ต้อง approve
1. ranking default = **multiple** พร้อม cap 5× และ single_period cap 1× (ยอมรับว่า single_period cap มีผลต่อลำดับแม้ confirmed ไม่มี)
2. แถว 10 (ocf_up ∧ oi_down) และ 7 เป็น W ไม่ใช่ T — conservative
3. แถว 12 (บริษัทกำไรที่ OCF เพิ่งบวก) เป็น T ไม่ใช่ P
4. G ranked ต้องมี ≥ 1 path; G ที่ไม่มี path อยู่ใน view G แบบไม่มีเลขอันดับ (searchable)
5. T: curtailment → Tier B; P: curtailment = label เท่านั้น
6. `fcf_negative` ไม่เข้า tier; tier 2 = runway_short ∨ (leverage_high ∧ external_financing_dependence)
7. T_m/T_a/T_c ตั้งจาก design snapshot แล้วล็อกก่อน selection snapshot
8. ยอมรับว่า D-type จับได้เฉพาะเมื่อมี 12q หรือ special-items tag

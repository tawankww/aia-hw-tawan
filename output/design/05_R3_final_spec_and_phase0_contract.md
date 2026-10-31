# R3 v0.1 — Final Specification (consolidated with approved R2 data-engineering spec) + Phase 0 Execution Contract
วันที่ 2026-10-31 · สถานะ: **แก้ตาม Final Design Review แล้ว รอ execution authorization** · ไม่ implement

## 0. Change log (Phase E → Final)
| # | เปลี่ยน | จาก | เป็น |
|---|---|---|---|
| C1 | debt `confirmed_zero` | zero tag เดียวพอ | ต้องมี explicit zero ครบ noncurrent + current + (short-term zero หรือ interest ≈ 0); ไม่ครบ = `partial_zero` → ปฏิบัติเหมือน unknown |
| C2 | `runway_short` | trigger tier 2 ได้เอง | **diagnostic** เท่านั้น; tier 2 = `leverage_high` ∧ `external_financing_dependence` เท่านั้น |
| C3 | evidence demotion | reversion ใน path ใดก็ได้ → Tier B | primary path เลือกจาก path ที่ **ไม่มี** reversion_risk; Tier B เฉพาะเมื่อไม่มี path credible เหลือ; cash path มี evidence ของตัวเอง (breadth_cash, prior_cash) ไม่ยืมของ margin |
| C4 | ranking invariance test | เทียบเลขอันดับ | เทียบ **ranking keys ต่อบริษัท** และ **pairwise order** ของบริษัทที่คงอยู่; เลขอันดับเปลี่ยนได้ |
| C5 | min-one-path | ใช้กับ G เท่านั้น | ใช้กับ P, G, T ทั้ง ranked view และ operational shortlist; 0 path = searchable section ไม่มีเลขอันดับ |
| V1 | comparator | R2-fixed อย่างเดียว | เพิ่ม `score_r2_original` บน subset ที่ R2 เดิมคำนวณได้ (10q + M5) ติดป้ายแยก |
| V2 | attrition report | ไม่มี | นับบริษัทที่หาย/ถูก demote ต่อ gate, evidence rule, risk rule ทุก run |
| V3 | A–H | sanity | ระบุใน README/About ว่าไม่ใช่ empirical validation |
| — | caps 5×/1×, curtailment→Tier B ใน T, thresholds | — | **provisional** ทุกตัว; freeze จาก design snapshot ก่อน validation |

---

## 1. Consolidation map — ส่วนไหนของ R2 คงเดิม ส่วนไหนแทนที่
| R2 section | สถานะใน R3 v0.1 |
|---|---|
| Context, §1 Executive rec, §2 Feasibility, §3 Architecture (3.1–3.4), §3.3 zip vs API | **คงเดิมทั้งหมด** |
| §4.1 Funnel U0–U4, U6, U7, U8 | คงเดิม; **U5 แก้:** core = ≥ 8q ok (ตัดเงื่อนไข 10q); 5–7q = U |
| §4.2 Coverage report | คงเดิม + เพิ่ม: coverage ของ guard tags, ขนาด P/G/T/W/U, attrition ต่อกฎ (V2), distribution S_m/S_a/S_c |
| §5.1 Tag map | คงเดิม + **เพิ่ม tags** (ตาราง §2.6 ด้านล่าง); debt rule เปลี่ยนเป็น 4 states |
| §5.2 Fact → quarter (dedupe, compat, derive, recon, provenance, TTM) | **คงเดิมทั้งหมด** (ห้ามใช้ `fy`/`fp` เป็น key — ระบุชัด) |
| §5.3 Metrics M1–M5 | **แทนที่** ด้วย §2.1–2.4 (S_g, S_a, S_m, S_c, classification, risk) — M-values ยังคำนวณเป็น diagnostic สำหรับ score_r2_original / r2fixed |
| §5.4 Score | **แทนที่** ด้วย §2.5 ranking |
| §5.5 Views | ปรับ: Profitable Improvers (default) / Cash-Generating Growth / Credible Turnaround / Watch / Provisional / Excluded; ทุก ranked view มี section "unranked (no triggered path)" |
| §5.6 False-positive table | รวมเข้า §2.3 evidence attributes (บางตัวมีผลต่อ tier ตามกฎ) |
| §6 MVP scope | คงเดิม + 4 โมดูลใหม่ |
| §7 Roadmap Phase 0 | คงเดิม + แทรกงาน R3 (ดู §4 contract) |
| §8 Gates G0.1–G0.9, G1.1–G1.7 | คงเดิม; G1.2 แก้ตาม C4; เพิ่ม G0.10–G0.14 |
| §9–10, A–E | คงเดิม; decisions ค้าง 1–4 ของ R2 ถูกแทนด้วย provisional thresholds ของ R3 |

---

## 2. R3 v0.1 — specification สุดท้าย (สรุปเฉพาะส่วนที่แทนที่/เพิ่ม)

### 2.1 Signals (TTM ณ t; ทุกตัวต้องการ 8 ไตรมาส status ok)
S_g = TTM_rev(t)/TTM_rev(t−4) − 1 · S_m = OPM(t) − OPM(t−4) · S_c = OCFM(t) − OCFM(t−4) · S_a = mean[q_yoy(t), q_yoy(t−1)] − mean[q_yoy(t−2), q_yoy(t−3)] · AC = S_c − S_m
Path trigger (thresholds provisional, config): margin: S_m ≥ T_m (2 pp) · accel: S_a ≥ T_a (5 pp) ∧ S_g > 0 ∧ S_m ≥ −1 pp · cash: S_c ≥ T_c (2 pp) ∧ AC ≤ 5 pp

### 2.2 Classification (ไม่เปลี่ยนจาก Phase E — truth table 16 แถว, precedence U → W → T → P → G/W) — approved

### 2.3 Evidence attributes (C3 แก้)
ต่อ path แยกกันทั้งสาม:
| path | breadth | prior (12q, ไม่มี = unknown) | reversion_risk ถ้า | single_period ถ้า | confirmed ถ้า |
|---|---|---|---|---|---|
| margin | # k∈{t−3..t}: OPM_q(k) > OPM_q(k−4) (0–4) | OPM(t) vs max[OPM(t−4), OPM(t−8)] | prior = not_above ∨ base_special_items = flagged | breadth ≤ 1 | prior = above ∧ breadth ≥ 3 |
| accel | # k∈{t−2..t}: q_yoy(k) > q_yoy(k−1) (0–3) | S_g(t) vs S_g(t−4) | prior = not_above | breadth = 0 | prior = above ∧ breadth ≥ 2 |
| cash | # k∈{t−3..t}: OCFM_q(k) > OCFM_q(k−4) (0–4) **[H: quarterly OCF noisy — รายงาน distribution ใน Phase 0]** | OCFM(t) vs max[OCFM(t−4), OCFM(t−8)] | prior = not_above | breadth ≤ 1 | prior = above ∧ breadth ≥ 3 |
else `early` (รวม unknown). Warnings ไม่มีผลต่อ tier: `acq_warning`, `curtailment` (ยกเว้น T, provisional), `sbc_dependent`, `secondary_reversion_warning`, pipeline flags
**Overall label (precedence):** insufficient → none (0 path) → reversion_risk (**ทุก** path ที่ trigger มี reversion_risk) → single_period (ทุก path ที่ credible เป็น single_period) → confirmed (primary path confirmed) → early

### 2.4 Risk (C1, C2 แก้)
**Debt state:**
- `known_debt`: fact หนี้ > 0 ที่ t จาก tag ใด
- `confirmed_zero`: explicit 0 สำหรับ noncurrent (`LongTermDebtNoncurrent` หรือ `LongTermDebt`) **และ** explicit 0 สำหรับ current (`LongTermDebtCurrent` หรือ `DebtCurrent`) **และ** (explicit 0 `ShortTermBorrowings` **หรือ** `InterestExpense` TTM ≤ 0.1% revenue)
- `partial_zero`: มี zero บางส่วนแต่ไม่ครบ → ปฏิบัติเหมือน unknown ใน tier, แสดงแยก
- `unknown`: ไม่มี fact หนี้
**Flags:** `leverage_high` (known_debt ∧ net_debt > 0 ∧ net_debt/TTM_OCF > L=5) · `external_financing_dependence` (fin CF > 0 ∧ FCF < 0) · **diagnostic เท่านั้น:** `fcf_negative`, `runway_short` (cash < R=4 × |FCF/4|) · ทุก flag มีค่า true/false/**unknown**
**Tier:** 2 = leverage_high ∧ external_financing_dependence (ทั้งคู่ true) · 1 = ตัวใดตัวหนึ่ง true · 0 = ทั้งคู่ false · unknown ไม่เปลี่ยน tier; `risk_completeness` = n computed / 2; `risk_basis` = complete/partial

### 2.5 Ranking (variant `multiple` = default experimental)
1. ต่อ path ที่ trigger: mult_p = min(S_p / T_p, C_max=5); ถ้า path นั้น single_period → mult_p = min(mult_p, 1.0)
2. **Primary path** = argmax mult_p ในหมู่ path ที่ **ไม่ใช่** reversion_risk; ถ้าไม่มี → primary = argmax ทั้งหมด และบริษัท Tier B (label reversion_risk)
3. **Tier:** A = risk_tier ≤ 1 ∧ primary credible (ไม่ใช่ reversion_risk) [∧ ใน T: curtailment = clear — provisional]; B = อื่น
4. **Key** = (Tier, best_mult = mult_primary, n_paths, second_mult, cik) — lexicographic ภายในกลุ่ม
5. **0 path → ไม่มี key, ไม่มีเลขอันดับ** อยู่ section "searchable" ของ view (C5 — P, G, T เหมือนกัน); operational shortlist ดึงเฉพาะบริษัทที่มี key
6. Diagnostics (คำนวณเสมอ): `rank_variant_count`, `rank_variant_pct` (ภายในกลุ่ม, ไม่ winsorize, null ถ้า n<30), `score_r2fixed` (equal-weight 4 pct ใน tier), **`score_r2_original`** (สูตร R2 20/20/25/25/10 + winsor p1/p99 บน subset ที่มี 10q และ M5 คำนวณได้ — ป้าย "R2 original comparator"), `rank_in_sector`

### 2.6 Tags เพิ่มจาก R2 §5.1
`NetCashProvidedByUsedInFinancingActivities`, `PaymentsToAcquirePropertyPlantAndEquipment`, `Assets`, `Goodwill`, `PaymentsToAcquireBusinessesNetOfCashAcquired` (+`…Gross`), `ShareBasedCompensation`, `RestructuringCharges`, `GoodwillImpairmentLoss`, `AssetImpairmentCharges`, `InterestExpense`, `IncomeLossFromDiscontinuedOperationsNetOfTax`, `ShortTermBorrowings`, `DebtCurrent`, `LongTermDebtCurrent`, `LongTermDebtNoncurrent` — ทุกตัว: ไม่มี fact = unknown (ยกเว้น goodwill/acq ที่ใช้เฉพาะเกณฑ์ที่คำนวณได้)

### 2.7 Missing-data policy (รวม)
null จริงเสมอ · unknown ไม่บวกไม่ลบ · แสดง "?" + completeness · ไม่มีค่าใดถูกแทน · prior/12q optional · ทุกแถว derived อ้าง source_fact_ids (R2 G0.6 คงเดิม)

### 2.8 Validation (pre-registered; สรุป) 
(i) operational shortlist RWC@20 (12/5/3, เฉพาะบริษัทที่มี key) · (ii) blind pool: union top-30/10/5 จาก 5 ตัวเทียบ {multiple, count, pct, r2fixed, **r2_original**} ซ่อน variant/label · persistence exploratory pseudo-PIT + base rate + CI + แยกตาม evidence label + สัดส่วน P/G/T ต่อ variant (survivorship ไม่เท่ากัน) · **sector concentration** (HHI SIC-2 top-50 vs group) · **threshold sensitivity** (T_m, T_a, T_c, L, C_max ×0.5/×2 → Jaccard top-20) · **attrition report** ต่อกฎ (V2) · decision rule: เปลี่ยน default เฉพาะ RWC@20 ≥ +4 ∧ persistence ไม่แย่กว่า ∧ ไม่เอียง T/G · A–H = mechanism sanity ไม่ใช่ validation (V3) · ข้อจำกัด PIT/survivorship ประกาศใน About

---

## 3. Acceptance gates เพิ่ม/แก้
| Gate | เกณฑ์ |
|---|---|
| G0.10 Classification | truth table 16 แถว fixture ผ่าน; บน 30 บริษัท: ทุก core อยู่คลาสเดียว, ΣP+G+T+W = core, Σcore+U+excluded = U0 |
| G0.11 Debt states | fixtures: zero ครบ → confirmed_zero; zero ตัวเดียว → partial_zero (tier เหมือน unknown); มี fact > 0 → known; ไม่มี → unknown; unknown/partial ไม่เปลี่ยน tier; completeness ถูก |
| G0.12 Ranking invariance (C4) | ลบ/เพิ่มบริษัทอื่นใน universe → ranking key (Tier, best_mult, n_paths, second_mult) ของทุกบริษัทที่คงอยู่ **ไม่เปลี่ยน** และ pairwise order ของทุกคู่ที่คงอยู่เหมือนเดิม (เลขอันดับเปลี่ยนได้); variant pct: ทดสอบเฉพาะ reproducibility บน universe เดิม |
| G0.13 Evidence exhaustive | property test: ทุก combination ของ attributes → label เดียว; secondary reversion ไม่เปลี่ยน Tier เมื่อ primary credible; 0 path → ไม่มี key |
| G0.14 Coverage of guard tags | coverage_report มี % ต่อ tag ใน §2.6 + ขนาด P/G/T/W/U + attrition ต่อกฎ + distribution S_m/S_a/S_c |
| G1.2 (แก้) | ตาม C4 แทนข้อความเดิม "rank core ไม่เปลี่ยน" |
| G1.8 (ใหม่) | About/README ระบุ: thresholds provisional, A–H ไม่ใช่ validation, pseudo-PIT exploratory + survivorship ไม่เท่ากัน, r2_original เป็น comparator |

---

## 4. Phase 0 Execution Contract (รอ authorization)

### 4.1 สิ่งที่เจ้าของต้องทำก่อน (blockers จาก R2 §E คงเดิม)
1. สร้าง repo `fundamental-inflection-radar` (local หรือ GitHub) และอนุญาตให้ผม init โครง
2. `.env` บน Mac mini: `SEC_USER_AGENT="<ชื่อ/องค์กร> <อีเมลติดต่อ>"` (ไม่ commit)
3. เชื่อม Remote Control ไปยังโฟลเดอร์ repo บน Mac mini (cloud session เข้า sec.gov ไม่ได้)
4. ยืนยันฉบับตรวจ 4 พ.ย. = Phase 0 + coverage report ตัวอย่าง (R2 §9) 
5. อนุมัติให้เขียน `spec.md` ของ repo การบ้านจาก R2 + R3

### 4.2 ขั้นตอน (ลำดับ R2 §7 คงเดิม; **ตัวหนา** = งาน R3 แทรก)
| ขั้น | งาน | ผลลัพธ์ | gate |
|---|---|---|---|
| 0.1 | spec.md repo การบ้าน (goal, 5 material questions, trust sources = SEC endpoints เท่านั้น, output, success criteria) + prompts.md entry รอบนี้ **+ PROJECT_MEMORY: กฎ "ไม่ใช้ fy/fp เป็น key", "null ≠ 0", "unknown ไม่บวกไม่ลบ"** | ไฟล์ใน aia-hw-tawan (push เมื่อสั่ง /save) | — |
| 0.2 | skeleton repo ใหม่, pyproject (uv), **config.yaml: tag map R2+§2.6, thresholds provisional, ranking.variant=multiple, quotas 12/5/3**, tests/ | skeleton | — |
| 0.3 | `radar universe` 30 CIK: distinct exchange → mapping; วัด submissions zip vs API | universe.parquet, ผลวัด | G0.1, G0.2 |
| 0.4 | `radar fetch` + `radar extract` (companyfacts.zip, --cik-list) **เก็บ tags §2.6 ด้วย** | facts_raw | — |
| 0.5 | `radar quarterly` ครบ R2 §5.2 **+ โมดูล `classify`, `risk`, `evidence` (pure functions)** | quarterly, recon, data_quality | G0.4, G0.5, G0.7, G0.8, G0.9 |
| 0.6 | manual recon 8–10 บริษัท (fixtures yaml: cik, accn, URL, วันที่, ค่า) | fixtures + test | G0.3 |
| 0.7 | automated tests 30 บริษัท **+ truth table, debt states, evidence property test, invariance, A–H mechanism fixtures** | pytest ผ่าน | G0.6, G0.10–G0.13 |
| 0.8 | `radar score` (variant multiple + diagnostics) + coverage_report **+ attrition + distribution S_m/S_a/S_c** | scores (30), report | G0.14 |
| 0.9 | บันทึกค่าจริงแทนข้อสมมติ (coverage OI/debt/12q/special-items, ขนาด G/T, เวลา/ขนาดไฟล์) ลง PROJECT_MEMORY.md; `/save` | — | — |
**ไม่มี dashboard, ไม่มี threshold freeze, ไม่มี validation ใน Phase 0** (ต้องการ universe เต็ม → Phase 1)

### 4.3 ตัวอย่าง 30 บริษัท (คงเกณฑ์ R2) + เพิ่มเงื่อนไขให้ครอบคลุม R3
ต้องมีอย่างน้อย: P ชัด ×6, G (OI<0, OCF>0, โต) ×3, T (sign crossing ภายใน 4q) ×3, W ×3 (รวม A-type), U/short history ×2, debt confirmed_zero ×1, debt unknown ×1, special items ปีฐาน ×1, acquisition ใหญ่ ×2, SBC สูง ×1 — เลือกจากข้อมูลจริงหลัง 0.4 ไม่เดาล่วงหน้า บันทึกเหตุผลที่เลือกแต่ละตัว

### 4.4 First implementation steps (ทันทีหลัง authorization)
1. 0.1: เขียน spec.md + prompts.md + PROJECT_MEMORY ใน aia-hw-tawan (ไม่ push จนกว่า /save)
2. 0.2: init repo ใหม่บน Mac mini ผ่าน Remote Control: `uv init`, โครง `radar/{universe,fetch,extract,quarterly,classify,risk,evidence,rank,score,export}.py`, `config.yaml`, `tests/fixtures/`
3. 0.3: รัน universe จริง 30 CIK + พิมพ์ distinct exchange → หยุดรายงานผลให้เจ้าของดูก่อนเขียน mapping

### 4.5 สิ่งที่ผมจะไม่ทำใน Phase 0
ไม่ freeze thresholds · ไม่รัน validation · ไม่ดึงราคา · ไม่ใช้ LLM ใน pipeline · ไม่ push โดยไม่สั่ง · ไม่แก้ R2 plan file · ไม่ claim coverage/performance

### 4.6 Decision points ที่จะกลับมาถามระหว่าง Phase 0
(a) exchange mapping หลังเห็น distinct values (b) zip vs API หลังวัด (c) ถ้า coverage `OperatingIncomeLoss` ใน 30 ตัว < 80% → เสนอ fallback ก่อนเดินต่อ (d) รายชื่อ 30 บริษัทสุดท้าย (e) ก่อน /save ทุกครั้ง

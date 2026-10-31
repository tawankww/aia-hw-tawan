# Phase D — Candidate Model Design & Adversarial Comparison
วันที่ 2026-10-31 · สถานะ: เสนอเพื่อ design review · ไม่ใช่ final · ไม่แก้ R2 plan · ไม่ implement

> ทุกค่า threshold ในเอกสารนี้เป็น **config default ที่ตั้งโดยเหตุผล ไม่ใช่ค่าที่พิสูจน์แล้ว** และต้องผ่าน sensitivity test ใน validation; ทุก "ผลคาดการณ์" ของ behavioral test เป็นการไล่กลไกด้วยมือ ไม่ใช่ผลรัน

## D0. Guardrails ที่ยึด (จากเจ้าของ)
1. Mean reversion = ความเสี่ยงที่มีหลักฐาน ไม่อ้างว่าใหญ่กว่า inflection โดยไม่มีข้อมูลเปรียบเทียบ → ใน validation จะวัด base rate จริง
2. Profitability trend (Akbas: gross profit/assets) ≠ operating margin trend → ใช้เป็นแรงบันดาลใจของ "evidence breadth" เท่านั้น ติดป้าย [H]
3. จำนวนไตรมาสตรวจแล้ว (ตาราง D1.4)
4. Asset growth / Δgoodwill / acquisition cash = **warning** ไม่ใช่หลักฐาน M&A
5. OCF − SBC = supplementary diagnostic; GAAP OCF ยังเป็นตัวหลัก
6. งานวิจัย composite ไม่ได้พิสูจน์ gate+rank > weighted ใน task นี้ → ทั้งสามแบบได้ validation เท่าเทียม

---

## D1. ชั้นร่วม (ใช้เหมือนกันทั้ง 3 candidate) — แยกหน้าที่ 4 ชั้นตามที่สั่ง

### D1.1 Hard Eligibility (เหมือน R2 U1–U6 ยกเว้นที่ระบุ) — หน้าที่: "คำนวณได้และเชื่อถือได้" เท่านั้น ไม่ตัดสินคุณภาพ
- U1–U4, U6 ($100M TTM revenue, config) คงเดิม
- U5 ปรับ: **core = ≥ 8 ไตรมาส status ok** (ทุก metric หลักต้องการ 8); 5–7 ไตรมาส = `short_history` (คำนวณ quarterly yoy ได้ แสดงใน Provisional); < 5 = `insufficient_history`
- U7 freshness คงเดิม (135/150 วัน → provisional)
- **ไม่มี gate เรื่อง growth, margin, หนี้ ในชั้นนี้**
- Debt-missing fix: ถ้าไม่มี tag หนี้ใด ๆ (`LongTermDebt`, `LongTermDebtNoncurrent`, `LongTermDebtCurrent`, `DebtCurrent`, `ShortTermBorrowings`) **และ** `InterestExpense` null หรือ < 0.5% revenue → ถือว่า `debt_none_inferred` (net debt = −cash) + flag; ถ้ามี interest expense แต่ไม่มี debt tag → `debt_missing` (null, ไม่ลงโทษ ไม่ให้รางวัล: risk tier = unknown)

### D1.2 Classification (จาก sign ของ TTM OI และ TTM OCF ที่ t และ t−4) — หน้าที่: "บริษัทอยู่ในสถานะไหน"
| กลุ่ม | เงื่อนไข | หมายเหตุ |
|---|---|---|
| **P · Profitable Improvers** | OI(t) > 0, OCF(t) > 0, OI(t−4) > 0, OCF(t−4) > 0 | default view |
| **G · Cash-Generating Growth** | OI(t) ≤ 0, OCF(t) > 0, OCF(t−4) > 0, g_rev(t) > 0 | g_rev ≤ 0 → `loss_firm_shrinking` (searchable, ไม่ rank) — เป็น design hypothesis ตาม mandate ข้อ 1 |
| **T · Credible Turnaround** | OCF(t) > 0 **และ** (OI(t−4) ≤ 0 หรือ OCF(t−4) ≤ 0) — คือ "มีอย่างน้อยหนึ่ง sign ข้ามจาก ≤0 เป็น >0 ภายใน 4 ไตรมาส และเงินสดจากการดำเนินงานบวกตอนนี้" | OI(t) อาจยังติดลบ; ผู้ที่ OI(t) > 0 ด้วย = "double crossing" แสดงเป็น label |
| **W · Watch (ไม่ rank)** | OCF(t) ≤ 0 | "ขาดทุนน้อยลง" โดยไม่ข้ามศูนย์ = ไม่มีเหตุการณ์ (B8); ค้นได้ใน Excluded/Watch พร้อม metric |
การแบ่งนี้ใช้ boolean 4 ตัว ไม่มี threshold ใด ๆ; บริษัทหนึ่งอยู่ได้กลุ่มเดียว; ΣP+G+T+W = core

### D1.3 Financial Risk (non-compensatory; demote ไม่ exclude) — หน้าที่: "ความเสี่ยงที่ improvement ชดเชยไม่ได้"
| flag | นิยาม (TTM) | tag ที่ใช้ | ใช้กับ |
|---|---|---|---|
| `leverage_high` | net_debt > 0 และ net_debt / OCF > 5 (OCF > 0) | debt tags, cash, OCF | P, G, T |
| `net_capital_raiser` | financing cash flow > 0 และ > 5% revenue | `NetCashProvidedByUsedInFinancingActivities` | ทุกกลุ่ม |
| `runway_short` | FCF = OCF − capex < 0 และ cash / (−FCF ต่อไตรมาส) < 4 ไตรมาส | `PaymentsToAcquirePropertyPlantAndEquipment`, cash | G, T |
| `risk_unknown` | debt_missing | — | แสดง ไม่ demote |
**Risk tier:** 0 = ไม่มี flag · 1 = 1 flag · 2 (severe) = ≥ 2 flags หรือ (`leverage_high` + `net_capital_raiser`) → **tier 2 ไม่มีสิทธิ์อยู่ใน Top tier ของ view ใด** (ถูกจัดใน "Flagged" ใต้บริษัท tier 0–1 ทั้งหมด) — นี่คือกฎ non-compensatory ข้อเดียวของระบบ
ค่า 5×, 5%, 4 ไตรมาส = default ไม่มีหลักฐาน → sensitivity test บังคับ

### D1.4 Inflection Strength — หน้าที่: "เปลี่ยนแรงแค่ไหน ในเส้นทางไหน" (3 เส้นทาง ไม่บังคับครบ)
| เส้นทาง | สัญญาณ | นิยาม | ไตรมาสที่ต้องการ (ตรวจแล้ว) |
|---|---|---|---|
| Margin | S_m | OPM(t) − OPM(t−4) pp, OPM = TTM_OI / TTM_rev | 8 (t−7..t) |
| Growth / Acceleration | S_g, S_a | S_g = TTM_rev(t)/TTM_rev(t−4) − 1; S_a = mean[q_yoy(t), q_yoy(t−1)] − mean[q_yoy(t−2), q_yoy(t−3)], q_yoy(k) = Q_rev(k)/Q_rev(k−4) − 1 | S_g: 8; S_a (half-year): 8 (t−7..t); single-quarter a(t) = q_yoy(t) − q_yoy(t−1) ต้องการ 6 (t−5..t) — ใช้ half-year เพื่อลด noise ไตรมาสเดียว; R2 M2 แบบ TTM ต้องการ 10 |
| Cash | S_c | OCFM(t) − OCFM(t−4) pp | 8 |
| Accrual consistency (ไม่ใช่เส้นทาง เป็นตัวกำกับ S_c) | AC | S_c − S_m; ถ้า > +5 pp → `ocf_outruns_oi` และ S_c ไม่นับเป็นเส้นทาง (นับเป็น supporting เท่านั้น) | 8 |
ไม่มี ΔATO ใน v0.1 (Phase 2 candidate)

### D1.5 Evidence Credibility — หน้าที่: "หลักฐานว่าเป็น inflection ไม่ใช่ noise/reversion" — ติด label, ห้ามเป็น gate, ห้ามลงโทษประวัติสั้น
| องค์ประกอบ | นิยาม | ไตรมาส |
|---|---|---|
| **Breadth** n_up | จำนวนไตรมาส k ∈ {t−3..t} ที่ OPM_q(k) > OPM_q(k−4) (0–4) — seasonal-safe, ไม่ต้อง regression | 8 |
| **Prior-level check** | OPM(t) เทียบ max[OPM(t−4), OPM(t−8)] | 12; ถ้ามีแค่ 8 → `prior_unknown` (ไม่ลงโทษ) |
| **Base-period special items** | (RestructuringCharges + GoodwillImpairmentLoss + AssetImpairmentCharges) ใน TTM(t−4) > 2% revenue(t−4) → `base_special_items` | 8 + tag coverage (ไม่มี tag = ไม่รู้ ไม่ flag) |
| **Anomaly warnings** (จาก R2 + ใหม่) | `acq_warning`: asset growth yoy − S_g > 20 pp **หรือ** Δgoodwill/assets(t−4) > 5% **หรือ** acquisitions payment TTM / assets > 5% · `curtailment`: S_g < 0 และ S_m > 0 · `sbc_dependent` (G/T): ShareBasedCompensation TTM ≥ OCF TTM · `disc_ops`: discontinued ops tag มีค่า > 5% OI · `restated`, `tag_spliced`, `mixed_vintage`, `rev_jump` จาก pipeline | 8 |
**Evidence state (label):**
- `confirmed` : S_m > 0, n_up ≥ 3, prior-level ผ่าน (OPM(t) > max prior), ไม่มี base_special_items
- `early` : S_m > 0 หรือ S_a > 0, แต่ prior_unknown (8–11 ไตรมาส) หรือ n_up = 2
- `possible_mean_reversion` : S_m > 0 แต่ OPM(t) ≤ max prior **หรือ** base_special_items
- `mixed` : n_up ≤ 1 ทั้งที่ S_m > 0 (ดีขึ้นจากไตรมาสเดียว)
- `short_history` : 5–7 ไตรมาส
Evidence state มีผลต่อลำดับ **เพียงข้อเดียว**: `possible_mean_reversion` ถูกจัดใต้ state อื่นภายใน tier เดียวกัน (D ปรากฏ แต่ต่ำกว่า E); `confirmed` กับ `early` **ไม่ต่างกันในลำดับ** (ไม่ลงโทษบริษัทอายุน้อย) ต่างกันแค่ label

---

## D2. Candidate 1 — **R3-min** (ไม่มี percentile, ไม่มีน้ำหนัก, ลำดับไม่ขึ้นกับบริษัทอื่น)
**แนวคิด:** นับ "เส้นทาง inflection ที่ผ่าน threshold สัมบูรณ์" (0–3) แล้วเรียงด้วยกฎ lexicographic
1. Eligibility → Classification (D1.2) → Risk tier (D1.3) → Evidence (D1.5)
2. **Path count** C ∈ {0,1,2,3}: Margin path ถ้า S_m ≥ +2 pp · Accel path ถ้า S_a ≥ +5 pp และ S_g > 0 และ S_m ≥ −1 pp (growth ที่มาพร้อม margin เสื่อมไม่นับ) · Cash path ถ้า S_c ≥ +2 pp และ AC ≤ +5 pp
3. **ลำดับภายในกลุ่ม:** (a) tier: risk 0–1 และ evidence ≠ possible_mean_reversion → Tier A; อื่น → Tier B · (b) C มากก่อน · (c) ภายใน C เท่ากัน: ค่าสูงสุดของ (S_m, S_a/2) desc (ไม่ใช่ sum; "/2" แค่ทำหน่วย pp ให้ใกล้กัน — arbitrary ระบุไว้) · (d) tie: cik
4. Output: `group`, `path_count`, `paths[]`, `evidence_state`, `risk_tier`, `flags[]`, `rank_in_group`, `rank_in_sector` (ข้อมูลประกอบ)
5. **ไม่มี** score ตัวเลขรวม; dashboard แสดง C/3 + labels
**ข้อดี:** ลำดับของบริษัทหนึ่งไม่เปลี่ยนเมื่อ universe เปลี่ยน (G1.2 ผ่านโดยโครงสร้าง); อธิบายได้ในประโยคเดียว ("ผ่าน 2 ใน 3 เส้นทาง, หลักฐาน early, ความเสี่ยงต่ำ"); config 3 threshold
**ข้อเสีย:** threshold สัมบูรณ์ข้ามอุตสาหกรรม (2 pp ใน grocery ≠ software); tie เยอะใน C เท่ากัน; G (growth path เดียว) จะอยู่ใต้ F (2 paths) เสมอ

## D3. Candidate 2 — **R3-full** (gate + classification + ranking หลายมิติด้วย percentile ภายในกลุ่ม)
1–ชั้นร่วมเหมือน R3-min
2. **Strength percentile ภายในกลุ่ม** (P, G, T คำนวณแยก): R_m = pct(S_m), R_a = pct(S_a | S_g > 0 และ S_m ≥ −1 pp; ไม่ผ่านเงื่อนไข → R_a = 0), R_c = pct(S_c | AC ≤ 5 pp; ไม่ผ่าน → 0)
3. **Strength = max(R_m, R_a)**; R_c เป็น tie-break และ support label (`cash_confirms` ถ้า R_c ≥ 50) — ใช้ max ไม่ใช่ sum เพื่อให้ "เด่นเส้นทางเดียว" เท่ากับ "ปานกลางสองเส้นทาง" ไม่ได้บวกกันเป็น compensatory
4. **ลำดับ:** Tier A/B (เหมือน R3-min) → Strength desc → R_c desc → cik
5. Winsorize ที่ p1/p99 ภายในกลุ่มก่อน percentile (เหมือน R2)
**ข้อดี:** ลด sector/scale bias ของ threshold สัมบูรณ์; G สามารถชนะ F ได้ถ้า accel ของ G อยู่ top percentile ขณะ margin ของ F อยู่ percentile ต่ำกว่า
**ข้อเสีย:** ลำดับขึ้นกับ universe (เพิ่ม/ลบบริษัทเปลี่ยน percentile); G และ T เป็นกลุ่มเล็ก → percentile ไม่เสถียร (ต้องมี n ≥ 30 ไม่งั้น fallback เป็น R3-min ordering); อธิบายยากกว่าเล็กน้อย ("อยู่ percentile 92 ของ margin improvement ในกลุ่ม Profitable")

## D4. Candidate 3 — **R2-fixed** (baseline ที่ fair: คง weighted score แต่แก้ข้อบกพร่องที่ไม่เกี่ยวกับโครงสร้าง)
1. ชั้นร่วม D1.1–D1.3, D1.5 เหมือนกัน (classification + risk + evidence ใช้เป็น tier/label)
2. M1 = S_g, **M2 = S_a (half-year quarterly yoy, 8 ไตรมาส แทน TTM 10)**, M3 = S_m, M4 = S_c (ถ้า AC > 5 pp → M4 = 0), **M5 ตัดออก** (ย้ายไปเป็น classification + risk tier)
3. Score = 0.25·R(M1) + 0.25·R(M2) + 0.25·R(M3) + 0.25·R(M4) (equal weight ตาม Dawes; percentile ภายในกลุ่ม)
4. ลำดับ: Tier A/B → Score desc
**บทบาท:** เป็น control ที่บอกว่า "ความต่างมาจากโครงสร้าง (sum vs max/count) หรือจาก gate/guard ที่ทุกแบบได้เหมือนกัน" — ถ้า R2-fixed ให้ผลเท่า R3 ใน validation แปลว่าสิ่งที่มีค่าคือชั้นร่วม ไม่ใช่วิธี rank

---

## D5. Behavioral Preference Tests (A–H) — ไล่กลไกด้วยมือ ไม่ใช่ผลรัน ไม่ใช่ predictive validity
สมมติ: ทุกตัวมี 12 ไตรมาส, ตัวเลขตามที่เจ้าของให้; "✔" = ตรง preference, "✘" = ไม่ตรง, "~" = ตรงบางส่วน

| บริษัท | Class | Risk | Evidence | R3-min (C, ลำดับใน P) | R3-full | R2-fixed | preference | ผล |
|---|---|---|---|---|---|---|---|---|
| A ก้นเหว ต้องเพิ่มทุน OCF ยังลบ | **W** (OCF ≤ 0) | 2 | — | ไม่ rank, searchable | เหมือน | เหมือน | ไม่ใช่ priority | ✔ ทั้งสาม (ผลมาจากชั้นร่วม) |
| B โต 6%, OPM 12→16, OCF 10→14, หนี้ลด | P | 0 | confirmed (ถ้า n_up ≥ 3) | C = 2 (margin, cash) | R_m สูง-กลาง | score กลาง-สูง | Primary ต้น | ✔ / ✔ / ~ |
| C โต 35%, OPM −5→−1, OCF +8→+15, เงินสดสุทธิ | **G** | 0 | early/confirmed | C = 2 (margin +4, cash +7 แต่ AC = +3 ≤ 5 → นับ); accel ไม่ทราบ | top ของ G | top ของ G | Growth group แยก | ✔ ทั้งสาม |
| D impairment ปีฐาน 14→8→14 | P | 0 | **possible_mean_reversion** (OPM(t) = OPM(t−8), base_special_items ถ้ามี tag) | Tier B (ใต้ทุกตัว Tier A), C = 2 | Tier B | Tier B | ปรากฏพร้อม flag ต่ำกว่า E | ✔ ทั้งสาม — **แต่ต้องมี 12 ไตรมาส หรือ special-items tag**; ถ้ามีแค่ 8 ไตรมาสและไม่มี tag → D = `early` ปน E → ✘ (ข้อจำกัดที่รู้ล่วงหน้า) |
| E 10→12→14→16 | P | 0 | confirmed (n_up = 4, เหนือ prior) | C = 1 (margin +2 pp ถึง threshold พอดี; cash สมมติ +2) → C = 2 | R_m กลาง (+2 pp ไม่สูงใน percentile) | score กลาง | Primary ต้น, > D | ~ / ~ / ~ : **E ชนะ D ทุกแบบ** (D อยู่ Tier B) แต่ E ไม่ได้อยู่ "ต้น ๆ" ของ P เพราะ +2 pp เล็ก — เป็นจุดที่ "ขนาด" กับ "ความสม่ำเสมอ" ขัดกัน; R3 ไม่ให้รางวัล confirmed ในลำดับตามคำสั่งเจ้าของ |
| F โต 1%, OPM 10→15, OCF 11→16 | P | 0 | early/confirmed | C = 2 | R_m สูง | score สูง (M3, M4) | Primary อันดับ 2 | ✔ |
| G โต 15→25%, margin ทรงตัว | P | 0 | mixed/early (S_m ≈ 0) | **C = 1** (accel path เท่านั้น) | R_a สูง → Strength สูง | score กลาง-สูง (M1, M2) | Primary อันดับ 1 | **✘ R3-min (F > G)** / ✔ R3-full (G ≥ F ถ้า R_a(G) > R_m(F)) / ~ R2-fixed |
| H ซื้อกิจการ หนี้พุ่ง margin ลด | P | 1–2 (`leverage_high`, financing > 0 → 2) | `acq_warning` | Tier B; accel path ไม่นับเพราะ S_m = −3 < −1 → C = 0 | Tier B, R_a = 0 | Tier B, M1/M2 สูงแต่ tier B | ไม่ควรติด top | ✔ ทั้งสาม (ผลจาก risk tier + เงื่อนไข accel) |

**ข้อสรุปจาก behavioral tests:**
- 6 ใน 8 ผลลัพธ์ที่ "ถูก" มาจาก **ชั้นร่วม** (classification, risk tier, evidence) ไม่ใช่จากวิธี rank → สนับสนุนว่าคุณค่าอยู่ที่ gates/guards; วิธี rank เป็นเรื่องรอง
- ความต่างที่แท้จริงระหว่าง R3-min กับ R3-full คือกรณี G (เส้นทางเดียวแต่แรง) — R3-min ให้ "จำนวนเส้นทาง" ชนะ "ความแรง"; R3-full ให้ "ความแรงสูงสุด" ชนะ; เจ้าของ prefer G > F แต่ระบุว่าเป็น preference ไม่ใช่กฎ → ตัดสินด้วย blind review
- กรณี D ต้องการ 12 ไตรมาสหรือ special-items tag — ทั้งสาม candidate พลาดเหมือนกันถ้าไม่มี (ไม่มีวิธีอื่นจาก EDGAR)
- กรณี E เผยว่า "ไม่ให้รางวัล confirmed ในลำดับ" มีราคา: E ที่ดีขึ้นช้าแต่ชัวร์จะไม่อยู่ top 20 ของ P ถ้ามีบริษัท +8 pp จำนวนมาก; ทางเลือก (เจ้าของตัดสิน): (i) ยอมรับ (E อยู่ใน list แต่ไม่ต้น) (ii) ให้ view "Confirmed only" เป็น filter (iii) ให้ confirmed เป็น tie-break ก่อน cik — ผมเสนอ (ii) เพราะไม่แตะลำดับหลัก

---

## D6. Decision Matrix (คุณภาพเชิงเหตุผล ไม่ใช่ผลวัด; "—" = เท่ากันเพราะใช้ชั้นร่วม)
| มิติ | R2 (baseline เดิม) | R2-fixed | R3-min | R3-full |
|---|---|---|---|---|
| 1 Alignment กับ Mandate | ต่ำ: compensatory, growth 40%, A/H/D ขึ้นต้น | กลาง: gates แก้ A/H/D แต่ sum ยัง compensatory ภายใน Tier A | สูง | สูง |
| 2 Quality of signals | Δ จุดเดียว 4 ตัว + M2 TTM ไม่มีหลักฐาน | Δ จุดเดียว + S_a ที่มีหลักฐานกว่า | ชั้นร่วมเดียวกัน; threshold สัมบูรณ์ | ชั้นร่วมเดียวกัน; percentile ภายในกลุ่ม |
| 3 FP robustness | ต่ำ (flags ไม่แตะ rank) | — (ชั้นร่วม) | — | — |
| 4 Breadth of coverage | M2 ต้อง 10q; OI tag hard | S_a 8q; W/short_history ยังค้นได้ | เหมือน R2-fixed | เหมือน |
| 5 SEC availability | ดี | ต้องเพิ่ม tags: financing CF, capex, Assets, Goodwill, acquisitions, SBC, special items, interest expense (ทั้งหมด standard us-gaap; coverage ต้องวัด) | เหมือน | เหมือน |
| 6 Interpretability | "score 71.3" อธิบายด้วย flags | "score 68 ใน P, Tier A" | **"P · 2/3 paths · early · risk 0"** ดีที่สุด | "P · strength pct 92 (accel) · early · risk 0" |
| 7 Reproducibility | percentile ขึ้นกับ universe | เหมือน | **ไม่ขึ้นกับ universe** | ขึ้นกับ universe + กลุ่มเล็กไม่เสถียร |
| 8 Implementation complexity | กลาง | กลาง (+ชั้นร่วม) | **ต่ำสุด** (+ชั้นร่วม, ไม่มี percentile) | กลาง (+percentile ต่อกลุ่ม + fallback n<30) |
| 9 Expected shortlist usefulness | น่าจะต่ำ (statistical extremes) | กลาง | สูง แต่ tie เยอะ, G-type อยู่ต่ำ | สูง, ยืดหยุ่นกว่าใน G-type |
| Missing-data burden | debt null ลงโทษ | ชั้นร่วมแก้: debt_none_inferred/unknown | — | — |
| ความเสี่ยง overfit | น้ำหนัก 5 ตัว + winsorize | น้ำหนักเท่ากัน (ไม่จูน) | 3 threshold (sensitivity test) | winsorize + percentile (ไม่มีน้ำหนัก) |
| ต้นทุนประมวลผล | ต่ำ | ต่ำ | ต่ำ | ต่ำ (percentile ต่อกลุ่มไม่แพง) — ไม่มี candidate ใดต้องการเวลารันเพิ่มอย่างมีนัย |

## D7. Failure modes เฉพาะของแต่ละแบบ
- **R3-min:** (1) threshold 2 pp/5 pp ตัดบริษัท margin สูงที่ดีขึ้น 1.5 pp (ปริมาณเงินมาก) → ทางลด: เพิ่ม path เมื่อ S_m ≥ 2 pp **หรือ** ΔOI/TTM_rev(t−4) ≥ 2 pp (ใกล้เคียง) — ยังเป็น absolute (2) tie จำนวนมากใน C = 1 → ลำดับขึ้นกับ max(S_m, S_a/2) ซึ่งเป็น "ขนาด" อีกครั้ง (3) G-type อยู่ใต้ F-type เสมอ
- **R3-full:** (1) กลุ่ม G/T เล็ก (อาจ < 30 หลัง gate) → percentile ไร้ความหมาย → ต้อง fallback (2) เพิ่มบริษัท 100 ตัวเข้า P เปลี่ยน rank ของทุกตัว → ยากต่อ repeatability test ข้าม snapshot (3) max(R_m, R_a) ให้ outlier เส้นทางเดียวครอง top (บริษัท accel จาก base effect ไตรมาสเดียว แม้ half-year จะช่วย)
- **R2-fixed:** (1) sum ภายใน Tier A ยัง compensatory: margin +12 pp ชดเชย growth −5% ได้ (2) 4 metric Δ จุดเดียว correlate → percentile sum เน้น "เปลี่ยนทุกอย่างพร้อมกัน" = บริษัทผันผวน (3) ยังมีตัวเลข score ที่ชวนให้เชื่อความแม่นยำ
- **ทุกแบบ:** (1) D-type กับ 8 ไตรมาสและไม่มี special-items tag หลุดเป็น early (2) curtailment ที่ revenue ลดเพียง −0.5% ไม่ติด flag (3) `acq_warning` ให้ false alarm กับบริษัทที่ลงทุน capex หนัก (asset growth สูงแต่ organic) — เป็น warning ตาม guardrail 4 (4) OI tag coverage ไม่รู้ (5) ค่า threshold ของ risk ไม่มีหลักฐาน

## D8. Data requirements (เพิ่มจาก R2) — ทุกตัว us-gaap standard; **coverage = วัดใน Phase 0 ไม่เดา**
| concept | tag หลัก (fallback) | ใช้ใน | ถ้าไม่มี |
|---|---|---|---|
| financing CF | `NetCashProvidedByUsedInFinancingActivities` | risk | `net_capital_raiser` = unknown |
| capex | `PaymentsToAcquirePropertyPlantAndEquipment` | runway | runway = unknown (G/T) |
| total assets | `Assets` | acq_warning | ไม่มี warning |
| goodwill | `Goodwill` | acq_warning | ถือว่าไม่มี goodwill (ระวัง: ไม่ tag ≠ ไม่มี) |
| acquisitions | `PaymentsToAcquireBusinessesNetOfCashAcquired` (`…Gross`) | acq_warning | ถือว่าไม่มีรายการ |
| SBC | `ShareBasedCompensation` | sbc_dependent | ไม่ flag (ระบุ unknown) |
| special items | `RestructuringCharges`, `GoodwillImpairmentLoss`, `AssetImpairmentCharges` | base_special_items | ไม่ flag (unknown) |
| interest expense | `InterestExpense` | debt_none_inferred | debt_missing |
| discontinued ops | `IncomeLossFromDiscontinuedOperationsNetOfTax` | disc_ops | ไม่ flag |
ไตรมาสขั้นต่ำ: core 8; evidence prior-level 12 (optional); short_history 5–7
**งานเพิ่มใน Phase 0:** นับ coverage ของทุก tag ข้างบน + `OperatingIncomeLoss` ใน 30 บริษัท และ (Phase 1) ทั้ง universe ก่อนตัดสินใจว่า guard ใดใช้ได้ broad

## D9. Validation Protocol (pre-registered; เขียนก่อนเห็นผล)
### D9.1 แยกข้อมูลออกแบบกับข้อมูลตัดสิน
- **Design set:** 30 บริษัท Phase 0 + snapshot S_design (companyfacts ณ วันที่ X) → ใช้ตั้ง threshold, debug, ตรวจ behavioral tests
- **Selection set:** snapshot S_select = ข้อมูล ณ วันที่ Y ≥ X + 1 ไตรมาส และ **pseudo point-in-time cutoff T0 = Y − 24 เดือน**: สร้าง ranking โดยใช้เฉพาะ facts ที่ `filed ≤ T0` (facts_raw มีทุก version พร้อม filed → ทำได้) แล้ววัด persistence ด้วย facts ที่ filed ใน (T0, Y]
- ห้ามแก้ threshold/กฎหลังเห็นผล selection set; ถ้าแก้ ต้องรอ snapshot ใหม่
- ข้อจำกัดที่ประกาศล่วงหน้า: companyfacts มีเฉพาะบริษัทที่ยังอยู่ใน EDGAR → **survivorship bias** (บริษัทที่ delist/ล้มระหว่าง T0..Y หายไป → persistence สูงเกินจริงทุก model เท่า ๆ กัน); restated facts ที่ filed หลัง T0 ถูกตัดออกแล้ว แต่ facts ที่ถูก *ลบ* จาก companyfacts ไม่กลับมา → label ผลว่า **Exploratory Retrospective Validation** ตามที่เจ้าของสั่ง; เริ่มเก็บ **prospective snapshots รายเดือน** ทันทีที่ Phase 1 รัน
### D9.2 Fundamental Persistence (วัดที่ t+4 ไตรมาสหลัง T0) — นิยาม operational
| กลุ่ม | เกณฑ์ "sustained" (เจ้าของ) | นิยามวัด | base rate เปรียบเทียบ |
|---|---|---|---|
| P | รักษา ≥ 50% ของ improvement | OPM(t+4) − OPM(t−4) ≥ 0.5 × [OPM(t) − OPM(t−4)] **และ** OI(t+4) > 0, OCF(t+4) > 0 | สัดส่วนเดียวกันในบริษัท P ที่ **ไม่ติด** top-N (และใน random sample ของ P) |
| P (growth path) | ยังโต + ≥ 50% ของ growth rate | S_g(t+4) ≥ 0.5 × S_g(t) และ > 0 | เหมือนกัน |
| G | OCF ยัง > 0 และ (S_m(t+4) ≥ 0 หรือ OI(t+4) > 0) | — | G ที่ไม่ติด top |
| T | OCF(t+4) > 0 (sign ไม่กลับ) และ revenue ไม่หด | — | T ที่ไม่ติด top |
- รายงานสำหรับ top-20 / top-50 ของแต่ละ model: persistence rate, base rate, ความต่าง, 95% CI (bootstrap) — **ขนาดตัวอย่างเล็กจะทำให้ CI กว้าง ต้องรายงานตรง ๆ**
- Sensitivity: threshold 2 pp/5 pp/5×/5%/4q แต่ละตัว ×0.5 และ ×2 → รายงานว่า top-20 เปลี่ยนกี่ตัว (Jaccard) และ persistence เปลี่ยนเท่าไร; ถ้าผลพลิกเมื่อขยับ threshold = ไม่เสถียร ต้องบอก
- แยก mean reversion: ภายใน top-N นับสัดส่วน `possible_mean_reversion`/`early`/`confirmed` และ persistence ของแต่ละ state แยกกัน → นี่คือข้อมูลเปรียบเทียบที่ guardrail 1 ต้องการ
### D9.3 Blind Owner Review — Research-worthy Candidates @20
- Pool: top-20 ของ P จากทั้ง 3 model (+ top-5 G, top-3 T) บน S_select; รวม, ตัดซ้ำ, สุ่มลำดับ, ซ่อน model
- Card มาตรฐานต่อบริษัท: ชื่อ/SIC/revenue TTM, 8 ไตรมาสของ rev/OI/OCF (ตาราง), S_m/S_g/S_a/S_c, flags ทั้งหมด, risk flags, **ไม่แสดง group/rank/evidence state** (เพื่อไม่ชี้นำ) — หรือแสดงแบบสุ่มครึ่งหนึ่ง (A/B) เพื่อวัดว่า label ช่วยไหม
- เจ้าของตอบต่อบริษัท: worthy yes/no + เหตุผลสั้น (เลือกจาก list: ปลอม/base effect/M&A/ไม่น่าสนใจ/น่าสนใจเพราะ…)
- Metrics ต่อ model: RWC@20 (จำนวน yes ใน top-20), Jaccard overlap ระหว่าง model, จำนวนบริษัทที่ model นั้นหาได้เพียงเจ้าเดียว ("unique discoveries") และ yes-rate ของ unique, rejection-reason histogram
- Pre-registered decision rule: เลือก model ที่ซับซ้อนกว่าได้ **เฉพาะเมื่อ** RWC@20 สูงกว่า ≥ 4 ตัว **และ** persistence ไม่แย่กว่า; ไม่เช่นนั้นเลือก **R3-min**
### D9.4 Stability, concentration, coverage, failure analysis
- Rank stability: Spearman ของ ordering ภายใน P ระหว่าง S_design และ S_select (1 ไตรมาสห่าง) และ Jaccard top-50
- Sector concentration: HHI ของ SIC-2 ใน top-50 เทียบ HHI ของ P ทั้งหมด; รายงาน sector ที่ over-represented
- Coverage funnel: U0→core→P/G/T/W + tag coverage ของทุก guard
- Negative controls: bank/REIT/20-F ต้องอยู่ excluded; บริษัทที่มี 10-K/A → restated flag; 3 บริษัทที่รู้ว่าเป็น M&A ใหญ่ (เลือกจาก 30 ตัว Phase 0) → acq_warning ต้องติด
- Manual inspection: top-20 ของ P + ทุกตัวใน T → เปิด filing ต้นฉบับ ตรวจว่า inflection "จริง" (อย่างน้อยไม่ใช่ error ของ pipeline) → รายงาน failure ทุกตัวพร้อมสาเหตุ (pipeline / data / model)
### D9.5 สิ่งที่พิสูจน์ไม่ได้ใน v0.1 (ประกาศล่วงหน้า)
- predictive validity ต่อผลตอบแทน (ไม่มีราคา)
- persistence ที่ปราศจาก survivorship bias (ไม่มี PIT universe)
- ว่า threshold ที่เลือก optimal (มีแค่ sensitivity)
- ว่า G > F หรือ F > G "ถูก" — เป็น preference

## D10. ข้อเสนอ R3 สำหรับ v0.1 (ก่อน design review)
**เสนอ R3-min เป็น default ของ v0.1** ด้วยเหตุผล: (1) ผล behavioral tests ส่วนใหญ่มาจากชั้นร่วมที่ทั้งสามแบบใช้เหมือนกัน → ความต่างของวิธี rank เป็นเรื่องรอง → เลือกแบบที่เรียบง่ายและ reproducible ที่สุดตาม complexity constraint (2) ลำดับไม่ขึ้นกับ universe ทำให้ repeatability/consistency gates ง่ายและ shortlist ไม่สั่นเมื่อ universe เปลี่ยน (3) อธิบายได้ประโยคเดียว
**พร้อมเงื่อนไข:** (a) คำนวณ `strength_pct` แบบ R3-full และ `score_r2fixed` เป็น **คอลัมน์ diagnostic** (ต้นทุนต่ำ) เพื่อใช้ใน validation D9 โดยไม่ต้องสร้างระบบแยก (b) ถ้า blind review ให้ R3-full ชนะตาม decision rule D9.3 ค่อยสลับ default — สลับได้ด้วย config เพราะชั้นร่วมเหมือนกัน (c) เพิ่ม view "Confirmed only" เป็น filter สำหรับกรณี E

## D11. Unresolved — ต้องการคำตอบเจ้าของใน design review
1. R3-min ให้ F > G (2 paths > 1 path) ยอมรับไหม หรือต้องการ R3-full เป็น default ตั้งแต่ต้น
2. กลุ่ม G ต้องการ g_rev > 0 เป็นเงื่อนไขเข้ากลุ่ม (ตามชื่อ Growth) หรือให้ loss firm ที่ OCF > 0 ทุกตัวเข้าได้
3. กลุ่ม T นิยามด้วย "sign crossing ภายใน 4 ไตรมาส + OCF(t) > 0" ตัดบริษัทที่ขาดทุนน้อยลงแต่ยังไม่ข้ามศูนย์ออกจาก ranked views → ตรงกับ trade-off ข้อ 1 ที่เจ้าของยอมรับใน Phase A ใช่ไหม
4. Risk tier 2 = demote (ไม่ exclude) ด้วย threshold 5×/5%/4q — ยอมรับเป็น default ที่ต้อง sensitivity test ไหม
5. Evidence `confirmed` ไม่มีผลต่อลำดับ (มีแค่ label + filter) — ยืนยัน
6. 12 ไตรมาสสำหรับ prior-level check เป็น optional (ไม่มี = prior_unknown ไม่ลงโทษ) — ยืนยัน
7. Validation ใช้ pseudo-PIT จาก filed dates ใน facts_raw ซึ่งเป็น Exploratory เท่านั้น — ยอมรับ

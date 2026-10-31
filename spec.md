---
title: US Fundamental Inflection Radar
owner: Tawan
status: approved-for-phase-0
updated: 2026-10-31
---

# Spec

> เครื่องมือคัดกรองหุ้นสหรัฐทั้งตลาดจาก SEC EDGAR เพื่อหา "บริษัทที่พื้นฐานกำลังเปลี่ยนไปในทางที่ดีขึ้น" (fundamental inflection) แล้วส่งต่อไปทำ deep research ที่อื่น ไม่ใช่เครื่องมือ valuation ไม่ใช่เครื่องมือเทรด ไม่ใช้ราคาหุ้น
> แผนฉบับเต็ม: R2 (data engineering) + R3 v0.1 (scoring) — สรุปกฎไว้ในภาคผนวกท้ายไฟล์นี้; โค้ดอยู่ใน repo แยก `fundamental-inflection-radar` (local, Mac mini)

## 1. Goal · อยากตัดสินใจเรื่องอะไร

เปิด dashboard แล้วได้ shortlist ราว 20 บริษัทต่อเดือน (ประมาณ 12 Profitable Improvers · 5 Cash-Generating Growth · 3 Credible Turnaround ปรับตามคุณภาพจริง) ที่งบการเงินแสดงหลักฐานว่า **เพิ่งเริ่ม** ดีขึ้น พร้อมบอกได้โปร่งใสว่าหลักฐานแข็งแรงแค่ไหนและมีความเสี่ยงอะไร เพื่อเลือกว่าบริษัทไหนคุ้มค่าเอาไปศึกษาต่อ

## 2. Material questions · คำถามที่เปลี่ยนการตัดสินใจ

1. บริษัทอยู่ในสถานะไหน: กำไรและกระแสเงินสดจากการดำเนินงานเป็นบวกทั้งคู่ (P), ยังขาดทุนตามบัญชีแต่เงินสดบวกและรายได้โต (G), หรือเพิ่งข้ามจากลบเป็นบวก (T) — ถ้าเงินสดจากการดำเนินงานยังติดลบ ไม่จัดอันดับ
2. การเปลี่ยนแปลงเกิดในเส้นทางไหนและแรงแค่ไหน: operating margin ดีขึ้น, การเติบโตของรายได้เร่งขึ้น, หรือ cash flow margin ดีขึ้น (วัดเป็น percentage point ต่อปี เทียบกับขั้นต่ำที่ถือว่ามีนัย)
3. หลักฐานน่าเชื่อถือแค่ไหน: ดีขึ้นกี่ไตรมาสใน 4 ไตรมาสล่าสุด, เกินระดับเดิมก่อนปีฐานหรือแค่ฟื้นกลับ (mean reversion), ปีฐานมี impairment/restructuring หรือไม่, มีสัญญาณซื้อกิจการ หรือหดธุรกิจหรือไม่
4. ความเสี่ยงทางการเงินสูงจน improvement ชดเชยไม่ได้หรือไม่: หนี้สูงเทียบกระแสเงินสด **และ** ยังต้องพึ่งเงินทุนจากภายนอก — ถ้าข้อมูลหนี้ไม่ครบ ต้องบอกว่า "ไม่ทราบ" ไม่ใช่ "ไม่มีหนี้"
5. ตัวเลขทุกตัวย้อนกลับไปถึง fact ใน filing ต้นฉบับได้หรือไม่ และกระทบยอด (Σ4 ไตรมาส = ปีงบ) ผ่านหรือไม่

## 3. Trust sources · แหล่งข้อมูลที่อนุญาต (SEC เท่านั้น)

| แหล่ง | URL | ใช้ตอบคำถามข้อ |
|---|---|---|
| SEC EDGAR · companyfacts.zip (XBRL company facts ทุกบริษัท, อัปเดตทุกคืน) | https://www.sec.gov/Archives/edgar/daily-index/xbrl/companyfacts.zip | 1, 2, 3, 4, 5 |
| SEC EDGAR · company_tickers_exchange.json (รายชื่อ/ตลาด) | https://www.sec.gov/files/company_tickers_exchange.json | universe |
| SEC EDGAR · submissions (zip หรือ API ต่อ CIK: SIC, fiscal year end, รายการ filing) | https://www.sec.gov/Archives/edgar/daily-index/bulkdata/submissions.zip · https://data.sec.gov/submissions/CIK##########.json | universe, freshness |
| SEC EDGAR · XBRL APIs (companyfacts / companyconcept / frames) สำหรับตรวจสอบ | https://data.sec.gov/api/xbrl/ | 5 (cross-check) |
| SEC EDGAR · ไฟล์ 10-K / 10-Q ต้นฉบับ (manual reconciliation 8–10 บริษัท) | https://www.sec.gov/cgi-bin/browse-edgar | 5 |

กติกา: ทุกตัวเลขต้องมี cik, accession number, URL ของ filing และวันที่ดึง หาไม่ได้ให้เว้นว่าง (null) และบอก ห้ามเดา ห้ามแทนค่า ห้ามใช้ราคาหุ้น/ข้อมูลโบรกเกอร์/ข้อมูลที่ซื้อมา · การเข้าถึงต้องตั้ง User-Agent ที่มีอีเมลติดต่อ (เก็บใน `.env` บน Mac mini ไม่ commit) และไม่เกิน 5 req/s · งานวิจัยวิชาการที่ใช้ออกแบบกฎ (อ้างใน prompts.md) เป็นเหตุผลของการออกแบบ ไม่ใช่แหล่งข้อมูลตัวเลข

## 4. Output · หน้าตาผลงานที่ต้องการ

- **Phase 0 (ฉบับตรวจ 4 พ.ย.):** pipeline จริงบน 30 บริษัทตัวอย่าง (purposive sample ไม่ใช่ตัวแทนตลาด), manual reconciliation 8–10 บริษัทเทียบ filing ต้นฉบับ, ตาราง scores ตัวอย่าง, coverage report ตัวอย่างที่ติดป้ายชัดว่าเป็น sample ไม่ใช่ตลาด
- **Phase 1 (MVP):** dashboard Streamlit 4 หน้า (Rankings มี views P / G / T / Watch / Provisional / Excluded · Company quick view พร้อม provenance ถึง fact ดิบ · Coverage & Freshness · About-limitations) + `shortlist.csv` + `run_meta.json`
- ไฟล์ใน repo การบ้าน: `spec.md`, `prompts.md`, `PROJECT_MEMORY.md`, `output/` (ภาพหน้าจอ, coverage report ตัวอย่าง, shortlist ตัวอย่าง) · โค้ดอยู่ repo แยก

## 5. Success criteria · รู้ได้อย่างไรว่างานผ่าน

- [ ] ทุกค่า revenue / operating income / OCF รายไตรมาส 8 ไตรมาสล่าสุดของ 8–10 บริษัทตรงกับ filing ต้นฉบับภายใน $1k และ Σ4 ไตรมาส = ปีงบ (recon ผ่าน) หรือถูกติดสถานะ recon_fail และไม่ถูกจัดอันดับ
- [ ] ทุกแถวที่ derive มี `source_fact_ids` ชี้กลับไปยัง fact ดิบได้ ไม่มีค่าที่แต่งขึ้นหรือ carry-forward
- [ ] ทุกบริษัทที่ผ่าน eligibility อยู่ในคลาสเดียวเท่านั้น (P / G / T / W) และผลรวมเท่ากับ universe ที่ตรวจ (truth table 16 แถวผ่าน)
- [ ] ข้อมูลที่หาย (หนี้, special items, ประวัติ 12 ไตรมาส) แสดงเป็น "ไม่ทราบ" ไม่ถูกตีความเป็นบวกหรือลบ และมีคอลัมน์ completeness
- [ ] ลำดับของบริษัทหนึ่ง (ranking key) ไม่เปลี่ยนเมื่อเพิ่ม/ลบบริษัทอื่นใน universe; รัน 2 ครั้งจากข้อมูลดิบเดียวกันได้ผลเหมือนกัน
- [ ] coverage report แสดง funnel ทุกขั้น, coverage ของทุก tag ที่ใช้, และจำนวนบริษัทที่หาย/ถูกลดอันดับต่อกฎ; ไม่มีคำว่า full-market coverage / point-in-time / backtest ในเอกสาร
- [ ] (Phase 1) blind review ของ shortlist 20 ตัว: เจ้าของให้ "ควรศึกษาต่อ" ได้โดยอ่าน card มาตรฐาน และ persistence ถูกรายงานเทียบ base rate พร้อมข้อจำกัด (exploratory เท่านั้น)

---

## ภาคผนวก · สรุปกฎ R3 v0.1 (experimental baseline — ทุก threshold เป็นสมมติฐานจนกว่าจะวัดจากข้อมูลจริง)

**สัญญาณ (TTM ณ ไตรมาส t เทียบ t−4; ต้องการ 8 ไตรมาสต่อเนื่องที่ status ok)**
- S_g = TTM_rev(t)/TTM_rev(t−4) − 1 · S_m = OPM(t) − OPM(t−4) [pp] · S_c = OCFM(t) − OCFM(t−4) [pp] · S_a = ค่าเฉลี่ย yoy growth รายไตรมาส 2 ไตรมาสล่าสุด − 2 ไตรมาสก่อนหน้า [pp]
- เส้นทาง inflection (ไม่บังคับครบ): margin: S_m ≥ 2 pp · accel: S_a ≥ 5 pp และ S_g > 0 และ S_m ≥ −1 pp · cash: S_c ≥ 2 pp และ (S_c − S_m) ≤ 5 pp

**Classification (ตรวจตามลำดับ หยุดที่ข้อแรกที่จริง)** 1) ประวัติ 5–7 ไตรมาส → U · 2) OCF(t) ≤ 0 หรือมี downward crossing (OI/OCF จากบวกเป็นไม่บวก) → W (ไม่จัดอันดับ) · 3) มี upward crossing (OI/OCF จากไม่บวกเป็นบวก) → T · 4) OI และ OCF บวก → P · 5) OI ไม่บวก, OCF บวก → G ถ้า S_g > 0 มิฉะนั้น W

**Evidence (ต่อเส้นทาง; 12 ไตรมาสเป็น optional ไม่มี = unknown)** breadth = จำนวนไตรมาสที่ดีขึ้น yoy ใน 4 (accel: ใน 3) · prior = เกินระดับสูงสุดของ t−4/t−8 หรือไม่ · base_special_items (restructuring/impairment ในปีฐาน > 2% revenue) · label ต่อเส้นทาง: reversion_risk / single_period / confirmed / early · label รวม: insufficient → none → reversion_risk (ทุกเส้นทาง) → single_period → confirmed → early · confirmed ไม่มีผลต่อลำดับ (filter เท่านั้น) · warnings: acq_warning (asset growth − S_g > 20 pp, Δgoodwill/assets > 5%, acquisitions/assets > 5%), curtailment (S_g < 0 และ S_m > 0), sbc_dependent (SBC ≥ OCF), disc_ops, restated, tag_spliced, mixed_vintage

**Risk (non-compensatory, demote ไม่ exclude)** debt state: known / confirmed_zero (ต้องมี explicit 0 ครบ noncurrent + current + short-term หรือ interest ≈ 0) / partial_zero (= unknown) / unknown · flags: leverage_high (net debt/TTM OCF > 5), external_financing_dependence (financing CF > 0 และ FCF < 0) · diagnostic เท่านั้น: fcf_negative, runway_short · tier 2 = ทั้งสอง flag เป็นจริง → อยู่ใต้ทุกบริษัทใน view · unknown ไม่เปลี่ยน tier และแสดง completeness แยก

**Ranking (variant "multiple")** mult_p = min(S_p / threshold_p, 5); ถ้าเส้นทางนั้น single_period → ≤ 1 · primary = เส้นทางที่ mult สูงสุดในหมู่เส้นทางที่ไม่ใช่ reversion_risk (ไม่มี → Tier B) · key = (Tier A/B, best_mult, จำนวนเส้นทาง, mult ที่สอง, cik) ภายในกลุ่ม P/G/T · ไม่มีเส้นทาง → ค้นได้ ไม่มีเลขอันดับ · คอลัมน์ diagnostic: count, percentile (ไม่ winsorize), score_r2fixed, score_r2_original (บน subset ที่ R2 เดิมคำนวณได้), rank_in_sector

**สิ่งที่ไม่อยู่ใน v0.1** ราคา/ผลตอบแทน, valuation, sector scoring, ΔATO, OI fallback tags, PIT universe จริง, LLM, Alpha export, scheduler

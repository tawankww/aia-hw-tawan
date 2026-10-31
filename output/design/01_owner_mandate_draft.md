# Owner Discovery Mandate — DRAFT v1 (รอเจ้าของยืนยัน) · 2026-10-31

## 1. Purpose
Fundamental Inflection Discovery Engine ไม่ใช่ quality screener / growth screener / return predictor.
Output = shortlist ส่งต่อ Deep Research (Alpha) ~20 บริษัท/เดือน

## 2. Three discovery groups (classification จาก OI>0, OCF>0 + evidence)
1. Profitable Improvers (Primary, default view, ~12/20): OI>0 AND OCF>0 + evidence of inflection
2. Cash-Generating Growth (~5/20): OI<=0, OCF>0, growth, improving OPM; guard against SBC/WC-driven OCF
3. Credible Turnaround (Secondary, ~3/20): financially weak, credible recovery, not just low-base
- quotas เป็นแนวทาง ไม่บังคับให้ครบ
- ไม่ต้องการ 3 scoring models โดยอัตโนมัติ → วิจัยว่า score เดียว + classification ใช้ได้ไหม

## 3. What is an inflection
- 3 เส้นทาง: revenue acceleration, margin expansion, cash flow improvement — ไม่บังคับต้องมีครบ
- growth = ranking factor ไม่ใช่ hard gate; acceleration ยังไม่ตัด จนกว่าวิจัยจะชี้
- owner preference (ไม่ใช่ evidence): G > F > H; E > D; B > A; C ต้องไม่หายไป
- ไม่ให้ improvement ขนาดใหญ่ชดเชย financial deterioration รุนแรงได้ไม่จำกัด

## 4. Early vs confirmed
- เลือก early discovery (single TTM yoy comparison เป็นฐาน) + evidence strength classification
- states: Early Inflection / Confirmed Improvement / Possible Mean Reversion / Insufficient Historical Evidence
- temporal confirmation = supporting evidence ไม่ใช่ hard gate
- 16Q requirement ไม่ยอมรับเป็น gate; วิจัยวิธีที่ใช้ประวัติสั้น; rolling TTM ซ้อนกันไม่ใช่หลักฐานอิสระ
- top-20 ต้อง rank ด้วย credibility ไม่ใช่ raw improvement score

## 5. Accepted trade-offs
- ยอม FP มากขึ้นในการสแกน แลก early signal; แต่ top-20 ต้องมี credibility ordering
- ยอมพลาด: quality compounders ไม่มี inflection; turnaround ที่ยังไม่มีหลักฐาน; growth ที่มาพร้อม deterioration รุนแรง
- ยอมเพิ่ม classification complexity เล็กน้อย; ไม่ยอมเพิ่ม scoring models
- OCF>0 อย่างเดียวไม่ใช่หลักฐาน quality

## 6. Validation (v0.1)
- Primary: (1) Fundamental Persistence = sustained ไม่ใช่ continuous
  - margin: t+12m รักษา >= 50% ของ improvement ที่ตรวจพบ (เกณฑ์ตั้งต้น ทดสอบ sensitivity)
  - growth: ยังโต + ทดสอบ >= 50% ของ growth rate เดิม
  - primary ต้องคง OI>0, OCF>0 ไม่ deteriorate รุนแรง
  - turnaround ใช้เกณฑ์ของตัวเอง
  - แยก persistence ออกจาก mean reversion
- (3) Blind owner review: Research-worthy Candidates @20; รายงาน overlap, diversity, rejection reasons
- ห้ามใช้ price/forward return เลือกหรือจูน model ใน v0.1; Phase 2 อนุญาตเป็น independent diagnostic เท่านั้น ห้ามจูน weights จาก return โดยไม่มี OOS
- ต้องจัดการ/ระบุ: look-ahead (restated), survivorship, filing date vs period end, overlapping TTM, same-data design+eval
- ถ้าไม่มี PIT universe → label "Exploratory Retrospective Validation"; เก็บ prospective snapshots หลังใช้งานจริง
- เปรียบเทียบ R2 vs candidates บน: persistence, RWC@20, breadth, FP/FN patterns, coverage, simplicity, implementation cost
- R3 ไม่ต้องชนะทุกข้อ; ระบุความไม่แน่นอนตามจริง

## 7. Open research questions for Phase B
- acceleration มี incremental info เหนือ growth ไหม; TTM vs quarterly yoy acceleration
- ลด acquisition-driven growth โดยไม่อ่าน M&A รายบริษัท
- margin expansion vs revenue acceleration ประเมินอย่างไร
- quality gate + inflection ranking vs weighted score
- improvement 2 มิติ ดีกว่า 1 มิติจริงไหม (หรือแค่กฎที่ดูสมเหตุสมผล)
- evidence strength จากประวัติสั้นมีค่าจริงไหม
- SBC/WC guard สำหรับ Cash-Generating Growth จาก XBRL (ShareBasedCompensation tag ฯลฯ)
- 3 กลุ่มจำเป็นจริงไหม

## CONFIRMED 2026-10-31 with clarifications
1. Cash-Generating Growth: OI<=0, OCF>0 + inflection อย่างน้อย 1 มิติ (rev accel / OPM improve / credible OCF improve); ไม่บังคับ OPM ดีขึ้น; ต้องไม่มี severe deterioration; growth สูงแต่ไม่มี change ไม่ได้ priority แต่ยังค้นได้ — ยังเป็น design hypothesis
2. Top-20 priority = Strength of Inflection + Evidence Credibility + Financial Risk ร่วมกัน; ไม่ล็อกน้ำหนัก; evidence strength ห้ามเป็น hard gate ที่ลงโทษบริษัทอายุน้อย; D ยังปรากฏพร้อม Possible Mean Reversion flag
3. Sector-relative ranking: Phase B ตัดสินจากหลักฐาน; อาจเป็นแค่ข้อมูลประกอบ
Key owner question for Phase B: weighted composite vs quality gates + inflection ranking — answer with evidence

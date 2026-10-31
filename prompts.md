# บันทึกขั้นตอน (process log)

> เพิ่ม 1 entry ทุกครั้งก่อน commit ต่อท้ายไฟล์ ห้ามลบของเก่า
> ผู้สอนอ่านไฟล์นี้เพื่อดูวิธีคิดของคุณ ไม่ใช่แค่ผลงานสุดท้าย
> ถ้าใช้ AI ช่วยทำงาน AI จะเขียน entry ให้ตาม AGENTS.md ถ้า commit เอง ให้เขียนเองตามแบบด้านล่าง

---

## รอบ 1 · 2026-10-31

- **ทำอะไร:** ออกแบบเชิงแนวคิดของโปรเจกต์ US Fundamental Inflection Radar จนจบ (Phase A–E) ก่อนเขียนโค้ด: สัมภาษณ์เจ้าของแบบ grill-me 5 คำถามเพื่อสกัด Owner Discovery Mandate → วิจัยงานวิชาการด้าน financial statement analysis → red-team score เดิม (R2) → ออกแบบ candidate 3 แบบ → สรุปเป็น R3 v0.1 ที่อนุมัติแล้ว → เขียน spec.md ฉบับนี้ (แทน template) และบันทึก PROJECT_MEMORY.md → เก็บเอกสารออกแบบฉบับเต็ม 5 ไฟล์ไว้ที่ `output/design/` (01 mandate · 02 research + red team · 03 candidates · 04 final design · 05 R3 spec + Phase 0 contract) · ยังไม่มีโค้ด ยังไม่ดาวน์โหลดข้อมูล
- **เครื่องมือ / model:** Claude Code (cloud session บน claude.ai/code) ใช้ skill grill-me + WebSearch · ผู้เรียนตั้งชื่อ AI ในบทบาทนี้ว่า "Fable" (research & design architect)
- **Prompt ที่ใช้:** (คัดมา 3 อันหลักจากทั้งหมด 9 รอบสนทนา ตัดทอนส่วนยาวด้วย […])

```text
# US Fundamental Inflection Radar
# Fable — Owner Grill-Me + Adversarial Scoring Research
You are Fable, the independent research and design architect for this project.
We have completed Revision 2 of the planning document […]
Before implementation, I want to challenge the INVESTMENT RESEARCH LOGIC behind the system, especially its screening philosophy, financial metrics, scoring methodology, and discovery effectiveness.
This is NOT another software architecture review.
Your mission is to determine whether our proposed scoring system is actually fit for discovering financially improving US companies worth further investigation.
You have permission to fundamentally challenge and redesign the scoring philosophy.
Do not defend your previous design simply because you created it.
Do not assume that adding more metrics or increasing complexity produces a better system.
## PHASE A — UNDERSTAND AND GRILL ME […] Ask ONE question at a time […] Use up to 5 high-impact questions […]
## PHASE B — INDEPENDENT RESEARCH […] Provide verifiable citations and distinguish empirical evidence from hypotheses […]
## PHASE C — RED TEAM THE R2 SCORE […] M1 Revenue Growth 20% · M2 Revenue Growth Acceleration 20% · M3 Operating Margin Improvement 25% · M4 Operating Cash Flow Improvement 25% · M5 Financial Stability 10% […]
## PHASE D — REDESIGN AND CHALLENGE […] At least one must be a minimal, robust model […]
## PHASE E — FINAL DESIGN PROPOSAL […] Do not claim that R3 is empirically superior until it has been tested.
## NON-NEGOTIABLE CONSTRAINTS — No implementation […] SEC EDGAR remains the primary financial data source. No LLM-dependent scoring at runtime […] No stock-price or technical-data dependency for v0.1 […]
IMPORTANT: Your objective is not to build the most sophisticated scoring model. Your objective is to discover companies that are genuinely worth investigating, using the smallest reliable set of financial signals.
```

```text
Phase D Authorization — Approved with Research Guardrails
[…] Mean Reversion เป็นความเสี่ยงที่มีหลักฐานสนับสนุน แต่ไม่ควรอ้างว่ามีขนาดใหญ่กว่า Inflection ส่วนใหญ่โดยไม่มีข้อมูลเปรียบเทียบ […]
ออกแบบ Candidate 3 แบบตามเสนอ: R3-min, R3 Gate + Classification + Ranking และ R2-fixed […]
เน้นแยก Hard Eligibility, Financial Risk, Inflection Strength และ Evidence Credibility ตามหน้าที่ของแต่ละส่วน อย่าสร้าง Hard Gate เพิ่มเพียงเพราะตัวแปรนั้นมีความสัมพันธ์กับผลตอบแทนหุ้นในงานวิจัย […]
ผมต้องการ Smallest Reliable Discovery System ไม่ใช่ Quantitative Factor Platform ขนาดใหญ่
```

```text
R3 Final Design — Approved for Phase 0 Execution
I approve the consolidated R2 data-engineering specification and R3 v0.1 Threshold Multiple model as the experimental baseline.
The conceptual design is now frozen for Phase 0. Do not start another research or scoring-design cycle.
[…] Resolve the 30-company selection dependency […] This is a purposive validation sample, not a representative estimate of market-wide coverage.
[…] Do not infer complete zero debt from low interest expense […]
[…] Do not extrapolate data coverage or model effectiveness from the 30-company sample to the US-listed universe […]
Execution sequence: Proceed with steps 0.1–0.3 first.
```

- **ผลที่ได้:**
  - Owner Discovery Mandate ที่ยืนยันแล้ว: ระบบนี้คือ Fundamental Inflection Discovery Engine (ไม่ใช่ quality/growth screener) หา 3 กลุ่ม Profitable Improvers (หลัก) / Cash-Generating Growth / Credible Turnaround โดยเน้น early signal + บอก evidence strength, ไม่ใช้ราคาใน v0.1, validate ด้วย fundamental persistence (exploratory) + blind owner review
  - ผลวิจัย (อ้าง abstract/secondary เพราะ session เข้าเว็บ paper ไม่ได้): mean reversion ของ profitability ~38%/ปี และเร็วสุดเมื่ออยู่ต่ำ (Fama & French 2000); trend 8 ไตรมาสมีข้อมูลเพิ่มจาก level (Akbas, Jiang & Koch 2017); acceleration มีหลักฐานสำหรับ earnings แบบ quarterly yoy ไม่ใช่ TTM และไม่พบ paper สำหรับ revenue (He & Narayanamoorthy 2020); growth สูงไม่ persist และ asset growth จาก M&A พยากรณ์ผลแย่ (Chan et al. 2003; Cooper et al. 2008); ΔOCF ปีเดียวคือ working-capital noise ขณะที่ OCF level เป็นหลักฐานคุณภาพ (Dechow 1994; Ball et al. 2016); composite ที่สำเร็จเป็น gated equal-weight binary และ weighted composite เสี่ยง overfit (Piotroski 2000; Dawes 1979; Einhorn 1970; Novy-Marx 2016) — ไม่มี head-to-head test ว่า gate+rank ดีกว่า weighted score
  - Red team R2: score เดิมเป็น compensatory sum ของ "ขนาดการเปลี่ยนแปลงจุดเดียว" จึงจัดอันดับบริษัทฟื้นจากฐานต่ำ / ซื้อกิจการ / impairment ปีฐาน ไว้เหนือบริษัทที่ดีขึ้นจริง และลงโทษบริษัทไม่มีหนี้ที่ไม่มี tag หนี้ · pipeline/provenance/recon ของ R2 ดีและคงไว้ทั้งหมด
  - R3 v0.1: eligibility (sign) → classification 16-row truth table → risk tier non-compensatory → 3 เส้นทาง inflection → evidence attributes → ranking ด้วย threshold multiple (ไม่มี weighted sum, ไม่มี percentile ในลำดับหลัก) + diagnostic columns เพื่อเทียบ variants ใน validation
  - ยังขาด: ข้อมูลจริงทุกอย่าง — coverage ของ tag, ขนาดกลุ่ม G/T, การกระจายของสัญญาณสำหรับตั้ง threshold; เอกสารออกแบบฉบับเต็ม (Phase B–E) เก็บไว้ใน `output/design/` ของ repo นี้แล้ว (จะคัดลอกเข้า repo โค้ดอีกชุดเมื่อสร้าง)
- **ตัดสินใจอะไร และเพราะอะไร:**
  - เลือกบริษัทแบบ "ดีอยู่แล้ว ดีขึ้นอีก" เป็นกลุ่มหลัก (75%) เพราะเป้าหมายคือ inflection ที่ยั่งยืนพอส่งต่อไป deep research ไม่ใช่อัตราการเปลี่ยนแปลงสูงสุด; ยอมพลาด turnaround ที่ยังไม่มีหลักฐาน ดีกว่าเสียเวลากับบริษัทขาดทุนที่คะแนนสูงจาก low-base effect
  - ตั๋วเข้ากลุ่มหลักต้องมีทั้ง operating income และ OCF บวก และเพิ่มกลุ่ม Cash-Generating Growth เพื่อไม่ให้บริษัทโตเร็วที่ยังไม่กำไร GAAP แต่เงินสดบวกหายไปจากระบบ; OCF บวกอย่างเดียวไม่ถือเป็นหลักฐานคุณภาพ
  - เลือก early discovery (เทียบ TTM จุดเดียว) + evidence label แทนการบังคับประวัติ 2–3 ปี เพราะไม่อยากค้นพบช้าและไม่อยากตัด IPO/spin-off; ประวัติสั้น = "ไม่ทราบ" ไม่ใช่ลบ
  - growth เป็น ranking factor ไม่ใช่ hard gate; acceleration ยังไม่ตัดแต่เปลี่ยนนิยามเป็น quarterly yoy เพราะหลักฐานใช้แบบนั้นและต้องการแค่ 8 ไตรมาส
  - ไม่ใช้ราคาหุ้นใน v0.1 แม้ตอน validate เพราะเสี่ยง overfit และระบบต้องเป็น EDGAR-only; เปิดเป็น independent diagnostic ใน Phase 2
  - เลือก gates + ranking แทน weighted score เพราะ preference ของเจ้าของเป็น non-compensatory (improvement ชดเชย severe risk ไม่ได้) และน้ำหนักไม่มีหลักฐานรองรับ; ยอมรับว่ายังไม่ได้พิสูจน์เชิงประจักษ์ จึงคำนวณ R2 เดิมและ variants อื่นเป็น diagnostic เพื่อเทียบ
  - ตัวอย่าง 30 บริษัทเป็น purposive sample สำหรับ validate pipeline ไม่ใช่ตัวแทนตลาด; coverage ของตลาดต้องวัดใน Phase 1
  - ฉบับตรวจ 4 พ.ย. = Phase 0 + coverage report ตัวอย่าง (ติดป้ายชัด) เพราะ Phase 1 ไม่น่าทันและไม่ลด mission
- **ขั้นต่อไป:** ขั้น 0.2–0.3 บน Mac mini ผ่าน Remote Control: init repo `fundamental-inflection-radar` ใต้ Projects, config.yaml, ตรวจ SEC endpoints + User-Agent, พิมพ์ distinct exchange labels, วัด submissions zip vs API บน 30 CIK แล้วรายงานผลก่อน normalization (session cloud เข้า sec.gov ไม่ได้ จึงต้องรันบนเครื่อง)

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

---

## รอบ 2 · 2026-11-01

- **ทำอะไร:** รวมผล Phase 0 (ขั้น 0.2–0.9) ที่ทำเสร็จบน Mac mini ใน repo โค้ด local แยก เข้ามาเป็นชุดส่งตรวจ Phase 0 ฉบับสุดท้ายใน repo การบ้าน ส่วนงานโค้ดทั้งหมดของ Phase 0.2–0.9 ทำนอก repo นี้ prompt ของ session local เหล่านั้นไม่ได้ส่งมาให้ session นี้ จึงไม่ได้คัดลอกมาไว้ที่นี่ รอบนี้ทำ 7 อย่าง:
  1. fast-forward branch รอบนี้ไปที่ commit รอบ 1 เพื่อเอา `output/design/` และบันทึกรอบ 1 กลับมา (ดู PROJECT_MEMORY รอบ 2)
  2. เขียน `output/phase0/PHASE0_REPORT.md` เป็นรายงานหลักสำหรับผู้สอน ครอบคลุมโจทย์, สถาปัตยกรรม R3, แหล่งข้อมูลและ provenance, วิธีตรวจสอบ, ผลที่วัดได้จริง, ข้อค้นพบ, ข้อจำกัด, เหตุผล R2 กับ R3, แผน Phase 1, สิ่งที่ผู้สอนตรวจได้และตรวจไม่ได้, และคำถามถึงผู้สอน
  3. คัดลอกรายงาน local ที่ sanitize แล้ว 2 ไฟล์ (checkpoint 0.9 และ submission preview) ไว้ที่ `output/phase0/source_reports/` โดยไม่แก้เนื้อหา แค่เติมหมายเหตุไว้หัวไฟล์
  4. รวม memory entries Phase 0.3–0.9 ที่ผู้เรียนเตรียมไว้เข้า `PROJECT_MEMORY.md` และเพิ่มกฎใหม่ 12 ข้อโดยไม่ซ้ำกฎเดิม พร้อมตารางค่าจริงที่วัดได้
  5. อัปเดต `spec.md`: สถานะ, แหล่ง 8-K, ไฟล์ output, ติ๊ก success criteria พร้อมผล, และเพิ่มการแก้ไข D4–D8 ในภาคผนวกโดยไม่แตะกฎ R3 เดิม
  6. เขียน `help.md` 3 คำถามถึงผู้สอน
  7. อัปเดต `output/README.md` และตารางใน `README.md`
- **เครื่องมือ / model:** Claude Code (cloud session บน claude.ai/code) ใช้ model ตามการตั้งค่าของ session (นโยบายของ environment ไม่ให้ใส่ model ID ลงไฟล์ที่ push) · งาน Phase 0.2–0.9 ทำใน repo local บน Mac mini
- **Prompt ที่ใช้:** prompt หลักของรอบนี้ ตัดทอนด้วย […] และตัด path ในเครื่องออกเพื่อความเป็นส่วนตัว

```text
AIA Coursework — Final Phase 0 Submission
We have completed the local implementation and technical validation of US Fundamental Inflection Radar, R3 v0.1, on my Mac mini.
This Claude Cloud session is the coursework and instructor-review workspace, NOT the primary development environment.
The real application is maintained separately in a local-only repository: […]
Do not assume you have access to that local repository. Use only the course repository and the Phase 0 reports / sanitized submission preview that I have explicitly provided to this cloud session.
[…]
Step 3 — Update the coursework deliverables
Produce a clear, concise and instructor-reviewable final Phase 0 package.
[…] Incorporate the supplied Phase 0 memory entries into the existing memory document, preserving meaningful prior content and avoiding duplicates. […]
Make clear that this is technical validation, not investment-performance validation. The 30-company sample is purposive and cannot establish market-wide coverage or predictive performance.
Do not claim the local source code is available for inspection unless it has actually been supplied or published.
Step 4 — Public-content and privacy review […] Use the sanitized preview's manifest as the starting point. Do not publish the entire local development history.
Do not overwrite the approved R3 design with old R2 scoring rules.
Step 5 — Save according to course requirements […] This is my authorization to publish the finalized coursework update through that prescribed workflow. […]
If essential source documents are missing, stop before publishing and identify exactly what I must provide. Do not invent missing results.
```

- **ผลที่ได้:**
  - ได้ชุดส่งตรวจ Phase 0 ที่ผู้สอนอ่านจบได้ในไฟล์เดียว (`output/phase0/PHASE0_REPORT.md`) โดยตัวเลขทั้งหมดมาจาก checkpoint 0.9 และ submission preview ตามจริง ได้แก่:
    - funnel 8,023 → 6,036 → 4,217 → 2,964
    - facts 39,640 แถว, ตรงกับ SEC API 30/30
    - reconciliation 240/240 และ lineage 338/338
    - ความถูกต้องอิสระ 132 / 24 / 68 / 16
    - ranking 11 บริษัท (P 7, G 2, T 2)
    - gates G0.1–G0.14 ผ่าน
    - 284 tests
  - ดีตรงไหน: แยกความตรงทางคณิตศาสตร์ออกจากความถูกต้องอิสระชัดเจน ประกาศขอบเขตไว้ตั้งแต่ต้นรายงาน (technical validation, purposive sample, threshold provisional, ไม่ใช้ราคา) และบอกตรง ๆ ว่าผู้สอนตรวจโค้ดไม่ได้
  - ยังขาด:
    - โค้ด, tests, fixtures และ `coverage_report.json` / `phase0_summary.json` ฉบับเต็มยังอยู่ในเครื่อง ผู้สอนจึงตรวจซ้ำไม่ได้
    - ยังไม่มีภาพหน้าจอหรือ dashboard (เป็นงาน Phase 1)
    - เลขกลุ่มใน README ยังไม่ระบุ
    - `prompts.md` มี 2 รอบ แต่ checklist ของคอร์สกำหนดอย่างน้อย 3 รอบ
- **ตัดสินใจอะไร และเพราะอะไร:**
  - **เผยแพร่เฉพาะรายงานที่ sanitize แล้ว ไม่เผยแพร่โค้ด:** ผู้เรียนกำหนดให้ repo โค้ดเป็น local-only และสั่งไม่ให้เผยแพร่ประวัติการพัฒนาทั้งหมด ส่วน repo การบ้านเป็น Public จึงใช้ MANIFEST ของ preview เป็นจุดเริ่ม แต่เอาเฉพาะเอกสาร ไม่เอาโค้ด
  - **ไม่ต้องตัดสินเรื่องอีเมลใน `pyproject.toml` และเอกสาร R2 ต้นฉบับ:** MANIFEST ให้เจ้าของเป็นผู้ตัดสินสองเรื่องนี้ แต่ทั้งสองไฟล์ไม่ได้เข้า repo นี้ จึงยังไม่ต้องตัดสินในรอบนี้
  - **fast-forward ไปที่ commit รอบ 1:** ผู้เรียนอ้างถึงเอกสารออกแบบ R3 ใน `output/design/` ซึ่งมีอยู่เฉพาะบน branch ของรอบ 1 ที่ยังไม่ merge การ fast-forward เก็บประวัติรอบ 1 ไว้ครบโดยไม่เขียนทับ และไม่ทำให้ design R3 ถูกแทนด้วยกฎ R2
  - **เพิ่ม 8-K earnings release ใน Trust sources:** Phase 0.6 ใช้ 8-K ยืนยัน Q4 จริง และ 8-K อยู่บน SEC EDGAR จึงยังอยู่ในกติกา "SEC เท่านั้น" แต่ spec ต้องบอกให้ตรงกับสิ่งที่ทำจริง
  - **ใช้ 284 tests ไม่ใช้ 281:** checkpoint 0.9 และผู้เรียนระบุ 284 ส่วน 281 ใน preview §7 เป็นค่าเก่า จึงใส่หมายเหตุไว้แทนการแก้ต้นฉบับ
  - **เขียนรายงานหลักเป็นภาษาอังกฤษพร้อมสรุปไทย** (AI เลือก): รายงานต้นทางเป็นภาษาอังกฤษ การคงศัพท์และตัวเลขตามต้นฉบับช่วยลดความเสี่ยงที่ตัวเลขหรือความหมายจะคลาดเคลื่อนตอนแปล
  - **ติ๊ก success criteria 6 ข้อจาก 7:** gate ที่ตรงกันผ่านตามรายงาน local แต่ติดป้ายว่าผ่านเฉพาะในขอบเขต purposive sample ส่วนข้อ Phase 1 (blind review) ยังไม่ทำ
  - **คง R2 ไว้เป็น comparator เท่านั้น:** ตามที่อนุมัติในรอบ 1 เพราะ weighted score ของ R2 เป็นแบบ compensatory และไม่มีหลักฐานรองรับน้ำหนัก แต่ยังไม่มีข้อมูลพอจะพิสูจน์ว่า R3 ดีกว่า
- **ขั้นต่อไป:**
  1. รอคอมเมนต์ผู้สอนใน Issues (5–6 พ.ย.) และ Repair Clinic (7 พ.ย.)
  2. เติมเลขกลุ่มใน README
  3. เจ้าของตัดสินว่าจะเผยแพร่โค้ดหรือรายงาน JSON หรือไม่
  4. ขอ authorization สำหรับ Phase 1: รัน U4–U8 ทั้งตลาด 2,964 บริษัท, วางแผน precision check, ทำ coverage report ทั้งตลาด, freeze thresholds ก่อน validation แล้วค่อยทำ dashboard

---

## รอบ 3 · 2026-11-04

- **ทำอะไร:** เตรียมชุดส่งตรวจฉบับสุดท้ายจากระบบที่ทำงานจริงใน repo local โดยไม่ดาวน์โหลดข้อมูลใหม่และไม่แก้โมเดล
  1. รวมงานที่กระจายอยู่ 3 branch: fast-forward `main` → รอบ 1 → รอบ 2 (สายเดียวกัน ไม่มี merge commit ไม่เขียนประวัติใหม่)
  2. สร้าง `output/submission/sample30_evidence.csv` ด้วยสคริปต์จาก run R3 v0.1 baseline ที่ freeze แล้ว (SEC snapshot 2026-10-31) สำหรับ 30 บริษัทของ sample Phase 0.4 เดิม แล้ว cross-check กับผล Phase 0.8 และผล manual reconciliation Phase 0.6
  3. ถ่ายภาพหน้าจอจริง 3 ภาพจาก dashboard ในเครื่องที่เปิดบน run baseline (Discovery Board, Company Profile ของ Federal Signal พร้อม reconciliation และรายการ filing, Coverage & freshness) ติดป้ายว่าเป็นแอปในเครื่องและข้อมูลย้อนหลัง
  4. เขียน `METHODOLOGY.md`, `COVERAGE_SUMMARY.md` และ `VERIFICATION.md` · อัปเดต `README.md` (อ่านจบใน 30–60 วินาที), `spec.md` (สถานะ Phase 1 ตามจริง, คง 5 หัวข้อ), `PROJECT_MEMORY.md` (บทเรียน Phase 1 และรอบนี้), `help.md`
- **เครื่องมือ / model:** Claude Code (Claude desktop app, แท็บ Code บน Mac mini) model Claude Opus 5.5 · Python/pandas อ่าน Parquet ของ run · headless Chrome ถ่ายภาพหน้าจอ · งาน Phase 1 ก่อนหน้า (2026-11-01 ถึง 11-04) ทำใน session local หลายรอบ ซึ่งสรุปจาก checkpoint และ decision log ของ repo local ไว้ใน `PROJECT_MEMORY.md` ไม่ได้คัดลอก prompt ของ session เหล่านั้นมา
- **Prompt ที่ใช้:** prompt หลักของรอบนี้ ตัดทอนด้วย […]

```text
Fundamental Inflection Radar — Prepare AI Investor Alpha Homework Submission
[…]
New priority: prepare the coursework submission due November 4, 2026, 21:00 Asia/Bangkok.
[…]
Objective
Create a compact, professional submission that demonstrates a working, reproducible US Fundamental Inflection Radar using real SEC EDGAR data.
[…]
3. Use the existing verified SEC financial snapshot, preferably the frozen October 31, 2026 baseline. Do not trigger a new SEC download, refresh, promotion or financial-model change.
4. Select approximately 30 real companies using a documented, deterministic and defensible selection method.
5. Include examples of P, G, T, W and missing/uncertain-data cases when present. Do not artificially assign categories or cherry-pick companies to make the model look successful.
[…]
7. Report which examples have independently reconciled SEC filing data. Do not imply that all 30 companies were manually reconciled if that has not happened.
[…]
Do not create fake financial numbers, simulated research findings or misleading AI-generated dashboard images.
[…]
Stop before pushing to the public repository and request my approval of the final public diff.
```

- **ผลที่ได้:**
  - CSV 30 แถว: P 13 (ได้อันดับ 7) · G 2 · T 2 · W 4 · ประวัติสั้น 3 · excluded 6 (negative controls 3 ถูกตัดถูกเหตุผล)
  - 25 บริษัทที่อยู่ใน U3 ให้ผลใน run ทั้งตลาดตรงกับ sample run ของ Phase 0.8 ทุกตัว (class, primary path, multiple, evidence, tier) · อีก 5 อยู่นอก U3 โดยการออกแบบ
  - ระบุชัดว่า reconcile ด้วยมือแค่ 10 จาก 30 บริษัท และใน 7 บริษัท P ที่ได้อันดับ มีแค่ FSS ที่ reconcile แล้ว
  - ตัวเลขในภาพหน้าจอของ Federal Signal (ตำแหน่ง 242 จาก 474, strength 1.9×, revenue TTM +24%) ตรงกับแถวใน CSV
  - เติมเลขกลุ่มใน README เป็น Group 2 (g02) ตามที่เจ้าของแจ้ง (ชื่อ repo `aia-hw-tawan` คงเดิม)
  - ยังขาด: ส่งหลัง 21:00
- **ตัดสินใจอะไร และเพราะอะไร:**
  - **ใช้ sample 30 บริษัทเดิมของ Phase 0.4 ไม่คัดใหม่:** sample นี้เลือกด้วยกฎ deterministic และ seed คงที่ก่อนเห็นผล ranking การคัดใหม่ตอนนี้จะเลือกได้จากผลที่เห็นแล้ว ซึ่งเสี่ยง cherry-pick ตามที่ prompt ห้าม
  - **ใช้ run baseline ที่ freeze แล้ว ไม่ใช้ run `cal16v1` ที่ dashboard ใช้เป็นค่า default:** prompt ระบุให้ใช้ baseline 31 ต.ค. และ baseline คือ reference ที่ไม่เปลี่ยน ภาพหน้าจอจึงเปิด dashboard ด้วย run เดียวกัน เพื่อให้ตัวเลขในภาพกับ CSV ตรงกัน
  - **รายงานอันดับเป็น "ตำแหน่งในคลาสของทั้ง U3":** อันดับภายใน 30 บริษัทไม่มีความหมายสำหรับผู้ใช้จริง ส่วนตำแหน่งในทั้ง U3 คือสิ่งที่ dashboard แสดง
  - **ไม่ใช้ภาพหน้า Update Center:** หน้านั้นแสดงชื่อไฟล์ config ในเครื่องและสถานะ backup ภายใน จึงใช้หน้า Data & Methods ซึ่งแสดง coverage และ freshness แทน
  - **ไม่ทำ visual walkthrough หน้าเดียว (optional):** เลย deadline แล้ว จึงใส่ลำดับการทำงานเป็นข้อความใน README แทน
- **ขั้นต่อไป:**
  1. รอคอมเมนต์ผู้สอนใน Issues และ Repair Clinic (7 พ.ย.)

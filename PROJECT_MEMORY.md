# PROJECT_MEMORY.md

> ความจำระยะยาวของโปรเจกต์ AI อ่านไฟล์นี้ก่อนเริ่มงานทุกครั้ง ผู้สอนอ่านเพื่อดูว่าคุณเจออะไรมาและเรียนรู้อะไร
> ห้ามวาง error ยาว ๆ ที่อาจมี key หรือข้อมูลส่วนตัว ให้สรุปสั้น ๆ แทน

## กฎที่ได้จากบทเรียน

> กฎ 1 บรรทัด ที่ AI ต้องทำตามจากนี้ไป

- งานที่ต้องเข้า sec.gov / data.sec.gov ให้รันบน Mac mini ผ่าน Remote Control เพราะ cloud session ถูก network policy บล็อก ห้ามอ้างว่าทำเสร็จถ้ายังเข้าไม่ได้
- ห้ามใช้ฟิลด์ `fy` / `fp` ของ companyfacts เป็น key จับคู่ไตรมาส (มันอธิบาย filing ไม่ใช่ period) ให้ใช้ `start`/`end`/`filed` เสมอ
- ค่าที่หายต้องเป็น null จริง ห้ามแทนด้วย 0 หรือ carry-forward; "ไม่มี tag หนี้" ≠ "ไม่มีหนี้" ต้องมี explicit zero ครบนิยามจึงเป็น confirmed_zero
- สถานะ unknown ต้องไม่ทำให้คะแนน/tier ดีขึ้นหรือแย่ลง และต้องแสดง completeness แยกจาก severity
- ทุก threshold ของ R3 เป็นสมมติฐานจนกว่าจะวัดจากข้อมูลจริง; ตั้งจาก design snapshot แล้ว freeze ก่อน validation ห้ามปรับหลังเห็นผล
- ตัวอย่าง 30 บริษัทเป็น purposive sample ห้ามนำ coverage หรือผลของมันไปอ้างแทนทั้งตลาด
- ตัวอย่าง A–H (บริษัทสมมติ) เป็น mechanism sanity check เท่านั้น ห้ามรายงานเป็นหลักฐานเชิงประจักษ์
- flag ที่ตรวจจับ false positive ได้ต้องมีผลต่อ eligibility หรือ tier อย่างน้อยบางตัว ไม่ใช่แค่ติดป้ายแล้วปล่อยให้ขึ้นอันดับต้น (บทเรียนจาก R2)
- ผู้เรียน push ขึ้น GitHub เมื่อสั่ง /save เท่านั้น; ห้ามใส่อีเมลติดต่อ SEC หรือค่าใน `.env` ลงไฟล์ใด
- (Phase 0.4–0.5) ค่าที่อ่านจาก parquet/pandas ว่างเป็น NaN ไม่ใช่ None: เทียบ null ด้วย `pd.isna` และแปลง NaN กลับเป็น None ก่อนส่งเข้าโมดูล R3
- (Phase 0.4) ใน companyfacts.zip ให้เอา CIK จากชื่อไฟล์ เพราะบาง member ไม่มีฟิลด์ `cik`
- (D4) แปลงเป็นไตรมาสจาก 10-K, 10-K/A, 10-KT, 10-Q, 10-Q/A เท่านั้น ห้ามเอา fact จาก 8-K/40-F มาปน
- (Phase 0.5, D7) งวด transition ของ 10-KT ต้องหาจาก "งวดของ 10-KT เอง" ในทุกเวอร์ชันของ filing ไม่ใช่จากตัวเลขเปรียบเทียบปีก่อน และต้อง reject fiscal-year start ที่ซ้อนกับปีที่ยอมรับแล้ว
- (Phase 0.6–0.7) ความตรงทางคณิตศาสตร์ ≠ ความถูกต้องอิสระ: รายงานค่า presented / corroborated / valid-by-rule / rejected แยกกันเสมอ
- (Phase 0.6) Q4 ที่ derive จาก FY − 9M ต้องเทียบกับ Q4 ที่บริษัทเปิดเผยเอง (เช่น 8-K earnings release) เพราะถ้าวินเทจต่างกัน ค่าจะคำนวณได้แต่ผิด
- (D5) tolerance ปัดเศษต้องมาจาก decimals ที่ประกาศของทุก operand ที่จับคู่ fact ได้ตรง ไม่ใช่ค่าคงที่ ถ้ายืนยัน decimals ไม่ได้ = `precision_unknown` และยัง fail
- (Phase 0.8) ranking key ต้องคำนวณจากแถวของบริษัทนั้นเท่านั้น จึงทดสอบ invariance ได้ · สร้างคอลัมน์ก่อน filter เพราะ `assign` Series เข้า DataFrame ว่างจะรับ index มาทั้งก้อน
- (D8) comparator ต้องทำตามสเปกเดิมของมันตรงตัว (เช่น R2-original ให้ debt null = 0 + `debt_missing`) แล้วติดป้ายแยก ห้ามเอากฎ "ห้ามเติมค่า" ของ R3 ไปแก้ comparator
- (D8) ห้ามเขียนว่า "ความเสี่ยงต่ำ" ให้ใช้ "no triggered risk flags — complete inputs" หรือ "— incomplete inputs (…)" เท่านั้น
- (รอบ 2) งานที่ทำบน Mac mini ต้องนำเข้า repo การบ้านเฉพาะรายงานที่ sanitize แล้ว ห้ามอ้างว่าผู้สอนตรวจโค้ดได้ถ้าโค้ดยังไม่ได้เผยแพร่
- (รอบ 2) ก่อนเริ่มงานใน cloud session ให้รัน `git fetch` + `git branch -a` เพราะงานรอบก่อนอาจอยู่บน branch อื่นที่ยังไม่ merge
- (Phase 1) แก้โมเดลหลัง freeze ได้เฉพาะแบบ overlay ที่มีชื่อเวอร์ชันและ freeze file ของตัวเอง (เช่น `cal16v1`) ห้ามแก้ baseline ที่ freeze แล้วทับ และทุกตัวเลขที่ส่งตรวจต้องระบุ run id
- (Phase 1) ไฟล์ Parquet ต่างกลุ่ม (bucket) อาจมี schema ไม่เท่ากัน ให้อ่านคอลัมน์ที่ไม่บังคับผ่าน helper ที่คืนค่าว่างได้ ห้ามสมมติว่าทุกไฟล์มีทุกคอลัมน์
- (Phase 1) คำสั่งที่ "ตรวจสถานะ" ต้องอ่านอย่างเดียว ถ้าต้องบันทึกผลให้เขียนไฟล์ใหม่ ห้ามเขียนทับรายงานที่เจ้าของรับรองแล้ว
- (Phase 1) dashboard ห้ามมี composite score / percentile และห้ามเรียงตาม multiple ที่ไม่ cap; บริษัทที่ ranking input เท่ากันต้องแสดงเป็น "shared position" ไม่ใช่ลำดับที่แต่งขึ้น
- (Phase 1) การเรียก SEC แบบ live (แม้แค่ HEAD) และการดาวน์โหลดใหม่ต้องได้รับอนุญาตจากเจ้าของทุกครั้ง; ชุดส่งตรวจใช้ snapshot ที่ freeze แล้วเท่านั้น
- (รอบ 3) ตัวเลขในชุดส่งตรวจต้องสร้างด้วยสคริปต์จาก run ที่ freeze แล้ว และ cross-check กับรายงานเดิม ห้ามพิมพ์ตัวเลขเอง; ต้องบอกให้ชัดว่าบริษัทไหน reconcile ด้วยมือ บริษัทไหนไม่ได้ทำ
- (รอบ 3) ภาพหน้าจอที่ส่งต้องเป็นของจริงจากแอปและติดป้ายวันที่ข้อมูล ห้ามใช้หน้าที่มี path ในเครื่อง สถานะ backup หรือรายละเอียดเครือข่าย

## บันทึกปัญหา

> เพิ่มต่อท้าย ห้ามลบของเก่า

### 2026-10-31 · Cloud session เข้า SEC และเว็บงานวิจัยไม่ได้

- **เจออะไร:** `www.sec.gov`, `data.sec.gov`, `xbrl.us`, `fasb.org` และเว็บ paper หลายแห่ง (naaim.org, alphaarchitect.com, pages.stern.nyu.edu) resolve ไม่ได้ใน cloud session
- **สาเหตุ:** network policy ของ environment อนุญาตเฉพาะบาง domain
- **แก้อย่างไร:** Phase 0 ทั้งหมดที่แตะข้อมูลจริงย้ายไปรันบน Mac mini ผ่าน Remote Control; งานวิจัยอ้างจาก abstract และ secondary summary ที่ค้นผ่าน WebSearch ได้ และติดป้ายว่าตัวเลขต้องตรวจซ้ำกับต้นฉบับ
- **ลองวิธีไหนแล้วไม่เวิร์ก:** curl ตรงและ WebFetch — ทั้งคู่ DNS fail

### 2026-10-31 · Score ของแผน R2 จัดอันดับ "สถิติสุดโต่ง" ไม่ใช่ inflection

- **เจออะไร:** เมื่อไล่กลไกด้วยตัวอย่างสมมติ A–H บริษัทฟื้นจากฐานต่ำที่ยังต้องเพิ่มทุน, บริษัทที่โตด้วยการซื้อกิจการ และบริษัทที่ปีฐานมี impairment จะชนะบริษัทที่ดีขึ้นจริงอย่างต่อเนื่อง; บริษัทไม่มีหนี้ถูกหักคะแนน stability เพราะไม่มี tag หนี้
- **สาเหตุ:** score เป็น weighted sum (compensatory) ของ percentile ของ "ขนาดการเปลี่ยนแปลงจุดเดียว" 4 ตัว; growth มีน้ำหนัก 40% ทั้งที่งานวิจัยบอกว่าไม่ persist; ΔOCF ปีเดียวส่วนใหญ่เป็น working-capital noise; flags ไม่แตะ score; null หนี้ถูกตีเป็น 0 คะแนน
- **แก้อย่างไร:** ออกแบบ R3 v0.1: gate ตาม sign ของ OI/OCF → classification → risk tier แบบ non-compensatory → ranking ด้วย threshold multiple; คง pipeline/provenance/recon ของ R2 ทั้งหมด; คำนวณ score R2 เดิมเป็น diagnostic เพื่อเทียบใน validation
- **ลองวิธีไหนแล้วไม่เวิร์ก:** (1) classification ร่างแรกใน Phase D ให้บริษัท OI ลบ/OCF บวกทั้งสองช่วงเข้าได้ทั้ง G และ T — แก้ด้วย crossing events + precedence และ truth table 16 แถว (2) risk tier ร่างแรกใช้ runway_short และ financing inflow เป็น severe ได้เอง — เจ้าของชี้ว่าปน growth capex กับ distress จึงลดเป็น diagnostic (3) R3-min แบบนับเส้นทางให้ "2 เส้นทางพอประมาณ" ชนะ "1 เส้นทางแรง" เสมอ — แก้ด้วย threshold multiple โดยไม่เพิ่มน้ำหนัก

### 2026-10-31 · Phase 0.3: exchange labels, submissions source, filing eligibility

- **เจออะไร:** company_tickers_exchange.json มี label แค่ Nasdaq, NYSE, OTC, CBOE, TXSE, null; NYSE American ไม่แยก
  (ตัวอย่างที่รู้ว่าเป็น NYSE American ขึ้นเป็น "NYSE"). submissions.zip 1,572 MB โหลด 146 s + parse ทั้ง U0 2.3 s (วัดจริง);
  API 30 CIK 11.9 s (วัดจริง) → ทั้ง universe ≈ 27–53 นาที (ประมาณการ). zip กับ API ตรงกัน 30/30.
- **ตัดสิน (เจ้าของ):** U1 = NYSE + Nasdaq (ไม่อ้าง coverage NYSE American แยก); submissions.zip เป็นค่าหลัก API สำรอง;
  U3 = ≥ 2 ปีงบที่จบแล้วต่างกันจาก 10-K/10-KT ต้นฉบับ (ไม่นับ 10-K/A) ระบุปีด้วย reportDate ไม่ใช้ fy/fp.
- **ผล:** U0 8,023 → U1 6,036 → U2 4,217 → U3 2,964.

### 2026-11-01 · Phase 0.4: companyfacts extraction

- **วัดจริง:** companyfacts.zip 1.41 GB โหลด 123 s; facts_raw 30 บริษัท 39,640 แถว (ทุกเวอร์ชัน, ไม่มีค่าว่าง/เติมศูนย์),
  ตรงกับ companyfacts API 30/30. Coverage ล่าสุดใน U3 pool (2,960): OI 95.0%, revenue 93.2%, OCF 99.8%, debt ใด ๆ 67.5%.
- **บทเรียน:** parquet อ่านกลับ start ของ instant เป็น NaN ไม่ใช่ None → เทียบ null ต้องใช้ pd.isna;
  companyfacts บาง member ไม่มีฟิลด์ cik → ใช้ CIK จากชื่อไฟล์; facts มีจาก 8-K/40-F ด้วย ต้องตัดสิน form filter ก่อน 0.5.

### 2026-11-01 · Phase 0.5: quarterly reconstruction + R3 classification

- **ผล:** 27 บริษัท (ไม่รวม control 3 ตัว) · last-12q ok: revenue 95.0%, OI 93.0%, OCF 99.3% · class: P 12, G 2, T 2,
  W 5, U 3, excluded 3 · gates G0.4/0.6/0.7/0.8/0.9 ผ่าน, G0.5 ผ่าน 8/9 (Lifecore 10-KT) · controls ถูกตัดถูกเหตุผลทั้งหมด.
- **บทเรียน:** (1) 10-KT มีตัวเลขเปรียบเทียบปีก่อน → transition ต้องเป็น "งวดของ 10-KT เอง" จากทุกเวอร์ชัน ไม่ใช่จาก
  facts_current (2) ผู้ยื่นบางรายใส่ start ผิด 1–2 วัน → ต้อง reject fiscal-year start ที่ซ้อนปีที่ยอมรับแล้ว
  (3) ปีที่ถูก recast (disc ops) ทำให้ FY ล่าสุดกับ 9M ต่างวินเทจ → R1/R5 fail ตามที่ R2 ตั้งใจ (4) pandas เก็บ None เป็น NaN
  → ต้องแปลงกลับเป็น None ก่อนเข้า R3 (5) tag หนี้ LongTermDebtAndCapitalLeaseObligations ไม่อยู่ใน map → debt unknown.

### 2026-11-01 · Phase 0.6: manual reconciliation

- **ผล:** 10 กรณี × 8 ไตรมาส × 3 ตัวชี้วัด = 240/240 ตรงกับตัวเลขที่แสดงในงบต้นฉบับ (ไม่ใช่ XBRL), lineage 338/338;
  Q4 เทียบ earnings release: ผ่าน 27, ปัดเศษหลักล้าน 1, ไม่ผ่าน 4 (LifeMD — pipeline ติด recon_fail อยู่แล้ว), ไม่มีข้อมูล 8.
  เพิ่มกรณี supplemental Intapp (G) และ Boeing (T) โดยกฎ deterministic.
- **บทเรียน:** (1) Q4 = FY − 9M คำนวณตรงได้แต่ผิดจริงเมื่อวินเทจต่างกัน — ต้องเทียบกับ Q4 ที่บริษัทเปิดเผยเอง
  (2) layout งบต่างกันมาก (ปีเก่าก่อน, YTD ก่อน, หน่วยดอลลาร์เต็ม, ตาราง %) — reader ต้องอ่าน header ไม่ใช่เดาตำแหน่ง
  (3) Montauk ต่าง $1k เพราะ decimals=-3 (ปัดเศษ) (4) tag หนี้ของบางบริษัทอยู่ใน note แบบมี dimension จึงไม่อยู่ใน companyfacts.

### 2026-11-01 · Phase 0.7: corrections D5–D7 + automated validation

- **ผล:** gates G0.1–G0.11, G0.13 ผ่าน; G0.5 แยกเป็น standard (8/8) และ transition (Lifecore 5/5); G0.12 บางส่วน
  (ranking key อยู่ 0.8); G0.14 เลื่อนไป 0.8. Montauk เปลี่ยนเป็น core P จาก rounding ที่ยืนยันด้วย decimals;
  Federal Signal/Murphy/Sirius หนี้เป็น known (รวม finance lease). 240 ค่า manual: valid อิสระ 156, valid ตามกฎ 68, ถูกปฏิเสธ 16.
- **บทเรียน:** (1) ความตรงทางคณิตศาสตร์ ≠ ความถูกต้องอิสระ ต้องรายงานแยก (2) tolerance ต้องมาจาก decimals ของทุก operand
  ที่จับคู่ได้ตรง fact ไม่ใช่ floor (3) finance lease ของบางบริษัทมีแต่แบบ dimension → unknown ไม่ใช่ศูนย์.

### 2026-11-01 · Phase 0.8: R3 sample ranking + coverage report

- **ผล:** Threshold Multiple ranking — ranked 11 (P 7, G 2, T 2; Tier A 10, Tier B 1); Montauk core P แต่ 0 path → ไม่มีอันดับ;
  G0.12 ผ่าน (leave-one-out 27, เพิ่ม supplemental, shuffle — key และ pairwise order ไม่เปลี่ยน), G0.14 ผ่าน.
- **บทเรียน:** (1) key ต้องคำนวณจากแถวของบริษัทเองเท่านั้น จึงทดสอบ invariance ได้ตรง (2) `assign` Series เข้า DataFrame ว่าง
  จะรับ index ทั้งก้อน → ผลบวมเป็นพันแถว ต้องสร้างคอลัมน์ก่อน filter.

### 2026-11-01 · Phase 0.9: comparator corrections (D8) + Phase 0 summary

- **แก้ตามเจ้าของ:** R2 original ทำตาม R2 จริง (debt null → เงื่อนไข M5 = 0 + `debt_missing`, ไม่มี minimum n) → pool 21 ตัว
  (debt_missing 6, cash_missing 1); R2-fixed/R2 original บนกลุ่มเล็ก = exploratory พร้อมคำเตือน; n ≥ 30 ใช้กับ pct ของ R3 เท่านั้น;
  wording "No triggered risk flags — complete/incomplete inputs" แทน "verified low"; tie-break นับเฉพาะ credible paths;
  `mixed_vintage` ย้ายไป provenance.
- **วัดจริง:** reconstruction 27 บริษัท 21.3 s (≈ 40 นาทีสำหรับ U3 2,964), assess 0.5 s; facts_raw ไม่เปลี่ยน; 284 tests.
- **บทเรียน:** อย่าตีความ "ห้ามเติมค่า" ทับสเปกของ comparator — comparator ต้องซื่อตรงกับสเปกเดิมของมัน แล้วติดป้ายแยก.

### 2026-11-01 · ค่าจริงที่วัดได้ใน Phase 0 (แทนข้อสมมติในแผน — ตามขั้น 0.9 ของ Phase 0 contract)

> ที่มา: `output/phase0/source_reports/CHECKPOINT_phase0_9.md` §4 · ข้อมูล SEC EDGAR snapshot 2026-10-31 (URL ใน `spec.md` §3)

| รายการ | ค่าที่วัดได้ | ขอบเขต |
|---|---|---|
| submissions.zip | 1,572,128,284 bytes; โหลด 146 s; parse U0 ทั้งหมด 2.3 s | จริง |
| companyfacts.zip | 1,414,913,699 bytes; โหลด 123 s | จริง |
| Funnel U0→U3 | 8,023 → 6,036 → 4,217 → 2,964 | universe จริง |
| Coverage ล่าสุด U3 pool (2,960) | OI 95.0% · revenue 93.2% · OCF 99.8% · cash 98.9% · debt tag ใด ๆ 67.5% | tag presence |
| Special-item tags U3 pool | restructuring 29.4% · goodwill impairment 43.3% · asset impairment 38.2% | tag presence |
| Prior check 12 ไตรมาส (21 core) | unknown: margin 2, accel 1, cash 1 | purposive sample |
| Special items ปีฐาน (21 core) | unknown 15 | purposive sample |
| ขนาดกลุ่ม G / T | 2 / 2 ในตัวอย่าง (+ supplemental 1 / 1) | purposive; ขนาดจริงในตลาดรู้ใน Phase 1 |
| Risk completeness (21 core) | 1.0 = 10 · 0.5 = 8 · 0.0 = 3; debt unknown 3 | purposive sample |
| เวลารัน 27 บริษัท | reconstruction 21.3 s · assess 0.5 s · score (รวม G0.12) 2.0 s | เครื่อง local |

### 2026-11-01 · รอบ 2: งานรอบ 1 ไม่อยู่บน branch ที่ได้รับมอบหมาย และรายงาน local มีตัวเลขไม่ตรงกัน 1 จุด

- **เจออะไร:** (1) cloud session นี้เริ่มบน branch ที่ยังเป็น Initial commit ไม่มี `output/design/` และ spec ยังเป็น template ทั้งที่รอบ 1 บันทึกไว้แล้ว (2) `SUBMISSION_PREVIEW.md` §7 เขียน 281 tests แต่ checkpoint 0.9 เขียน 284
- **สาเหตุ:** (1) commit รอบ 1 ถูก push ไปบน branch อื่น (`claude/clever-cerf-d0jn24`) และยังไม่ merge เข้า main (2) preview §7 ไม่ได้อัปเดตหลังเพิ่ม test ใน 0.9
- **แก้อย่างไร:** (1) fast-forward branch ของรอบนี้ไปที่ commit รอบ 1 (เป็น parent ตรง จึงไม่มี conflict และไม่เสียประวัติ) แล้วทำงานต่อจากนั้น (2) ใช้ 284 ตาม checkpoint และผู้ใช้ และใส่หมายเหตุไว้หัวไฟล์สำเนา preview โดยไม่แก้เนื้อหาต้นฉบับ
- **ลองวิธีไหนแล้วไม่เวิร์ก:** —

### 2026-11-01 ถึง 11-04 · Phase 1 ใน repo local: freeze → รันทั้ง U3 → dashboard (สรุปจาก checkpoint และ decision log ของ repo local)

- **ทำอะไร:** freeze threshold ของ R3 v0.1 (2026-11-01T09:20Z) แล้วรันทั้ง U3 2,964 บริษัทบน snapshot 2026-10-31 (ประมาณ 6 นาที 10 workers, ไม่มี pipeline error) · ตรวจ sensitivity / tie / ความสม่ำเสมอของข้อมูล · ออกแบบ blind review protocol · ทำ dashboard Streamlit แบบ read-only หลายรอบ · ออกแบบการ refresh และ backup โดยยังไม่เปิดใช้
- **เจออะไร / แก้อย่างไร:**
  1. **ปฏิทินไตรมาส 16 สัปดาห์:** บางบริษัทใช้ไตรมาสแรกยาว 16 สัปดาห์ ทำให้ไตรมาสคู่เทียบผิดยาว → แก้เป็น overlay เวอร์ชัน `cal16v1` (6 บริษัทเข้าเงื่อนไข, ranking key ไม่เปลี่ยน 0 ตัว) โดยไม่แตะ baseline ที่ freeze แล้ว
  2. **หน้า Company Detail crash:** ไฟล์ recon Parquet มีคอลัมน์ precision แค่ใน 8 จาก 32 bucket → อ่านคอลัมน์ที่ไม่บังคับผ่าน helper
  3. **คำสั่ง integrity-status เขียนทับรายงานที่รับรองแล้ว:** → เปลี่ยนเป็นตรวจอย่างเดียว ถ้าต้องบันทึกใช้ `--record` เขียนไฟล์ใหม่
  4. **เจ้าของไม่ผ่านหน้าตา dashboard v0.2:** → สร้าง row ด้วย component มาตรฐานของ Streamlit ตาม prototype ที่เจ้าของเลือก, ตัด strength bar ออกเพราะแถวต้น ๆ ติด cap 5× เท่ากันหมด จึงแสดงเป็น "shared position" แทน
  5. **window_start ขยับตามวัน snapshot:** การเลือก tag และการเข้าเงื่อนไข 16 สัปดาห์ขึ้นกับหน้าต่างข้อมูลทั้งหมด → snapshot ใหม่อาจเปลี่ยนผลแม้ข้อมูลบริษัทไม่เปลี่ยน จึงต้องมี gate ตรวจก่อน promote ข้อมูลใหม่ (ยังเป็นแค่ข้อเสนอ ไม่ได้เปิดใช้)
- **ลองวิธีไหนแล้วไม่เวิร์ก:** แสดง strength bar ต่อแถว (ทุกแถวบนสุดเต็มเท่ากัน ไม่ช่วยตัดสินใจ) · ให้คำสั่งตรวจสถานะเขียนไฟล์รายงานเอง
- **สิ่งที่ยังเสี่ยง:** ยังไม่มี scheduled refresh (ข้อมูลค้างที่ 2026-10-31) · backup production ยังไม่เปิดใช้ · thresholds ยัง provisional · ยังไม่มีหลักฐานคุณภาพโมเดล · หนี้ไม่ทราบ 384 จาก 1,571 core

### 2026-11-04 · รอบ 3: เตรียมชุดส่งตรวจ — งานอยู่กระจาย 3 branch และ deadline ผ่านไประหว่างทำ

- **เจออะไร:** (1) `main` บน GitHub ยังเป็น template ส่วนงานรอบ 1 อยู่บน `claude/clever-cerf-d0jn24` และรอบ 2 อยู่บน `claude/great-hypatia-94h3c1` (ผู้สอนเปิด repo จะเห็นแค่ template) (2) ภาพหน้าจอแบบ full-page ของ Streamlit ได้แค่ความสูงหน้าจอ (3) ตอนเริ่มเตรียมภาพหน้าจอ เวลาเลย 21:00 ของวันส่งไปแล้ว
- **สาเหตุ:** (1) cloud session แต่ละครั้ง push ไป branch ใหม่และไม่มีใคร merge (2) Streamlit เลื่อนหน้าใน container ภายใน ไม่ใช่ตัวเอกสาร (3) งานเตรียมชุดส่งเริ่มในวันส่ง หลังงาน Phase 1 ด้าน backup (pilot ด้วยข้อมูลสังเคราะห์) จบ
- **แก้อย่างไร:** (1) fast-forward `main` ไปที่รอบ 2 (เป็นสายเดียวกัน ไม่ต้อง merge commit) แล้วทำรอบ 3 ต่อบน `main` (2) เปิด headless Chrome ด้วย viewport สูงตั้งแต่แรก แล้ว crop (3) ส่งเฉพาะสิ่งที่จำเป็น ไม่เพิ่ม feature
- **ลองวิธีไหนแล้วไม่เวิร์ก:** ขยาย viewport หลังโหลดหน้า (Streamlit ไม่ render ส่วนที่เพิ่ม)

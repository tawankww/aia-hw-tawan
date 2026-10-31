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

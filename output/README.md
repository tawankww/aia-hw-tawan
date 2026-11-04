# output/

ผลงานของโปรเจกต์ **US Fundamental Inflection Radar** (การบ้าน AI Investor Alpha)

ห้ามใส่: ไฟล์ข้อมูลดิบที่ซื้อมา, บทวิเคราะห์โบรกเกอร์, ข้อมูลพอร์ตจริง, ไฟล์ SEC bulk ที่ดาวน์โหลดมา, `.env`

## ลิงก์ผลงาน (เริ่มอ่านที่ข้อ 1)

1. [`submission/`](submission/): **ชุดส่งตรวจ 4 พ.ย.** (ข้อมูล SEC snapshot 2026-10-31, run R3 v0.1 baseline ที่ freeze แล้ว)
   - [`sample30_evidence.csv`](submission/sample30_evidence.csv): 30 บริษัทพร้อมคลาส สัญญาณ ตัวเลข TTM risk วันที่ และ accession/URL ของ SEC
   - [`METHODOLOGY.md`](submission/METHODOLOGY.md): วิธีเลือก, ผล, สิ่งที่ตรวจแล้วและยังไม่ตรวจ, คำอธิบายคอลัมน์
   - [`COVERAGE_SUMMARY.md`](submission/COVERAGE_SUMMARY.md): funnel, class × eligibility, ความครบของข้อมูล risk, freshness
   - [`VERIFICATION.md`](submission/VERIFICATION.md): ผลการตรวจความถูกต้องและความปลอดภัยก่อนเผยแพร่
   - [`screenshots/`](submission/screenshots/): ภาพหน้าจอจริงจาก dashboard ในเครื่อง 3 ภาพ
2. [`phase0/PHASE0_REPORT.md`](phase0/PHASE0_REPORT.md): รายงานสรุป Phase 0 (เขียน 2026-11-01 ก่อนมี dashboard)
3. [`phase0/source_reports/`](phase0/source_reports/): สำเนารายงาน local ที่ sanitize แล้ว (ไม่ได้แก้เนื้อหา)
4. [`design/`](design/): เอกสารออกแบบรอบ 1 (mandate, research + red team, candidates, final design, R3 spec + Phase 0 contract)

## ขอบเขตที่ต้องรู้

- เป็นการตรวจสอบทางเทคนิค (technical validation) ไม่ใช่การพิสูจน์ผลการลงทุน · ตัวอย่าง 30 บริษัทเป็น purposive sample
- โค้ด, tests และข้อมูลดิบอยู่ใน repo local แยก **ไม่ได้เผยแพร่** ผู้สอนตรวจผลและตรวจตัวเลขกับ sec.gov ได้ แต่รันซ้ำจาก repo นี้ไม่ได้
- dashboard รันในเครื่องเท่านั้น (ไม่มีลิงก์ hosted) จึงส่งเป็นภาพหน้าจอ

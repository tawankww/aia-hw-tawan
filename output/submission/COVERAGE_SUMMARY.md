# Coverage & freshness summary · R3 v0.1 baseline · SEC data of 2026-10-31

> ตัวเลขทั้งหมดมาจาก run frozen baseline เดียวกับ `sample30_evidence.csv` (ดู `METHODOLOGY.md` §1) และตรงกับหน้า Data & Methods ใน `screenshots/03_coverage_and_freshness.png`
> นี่คือ coverage ของ **ข้อมูลในเครื่องของผู้เรียน ณ snapshot เดียว** ไม่ใช่ point-in-time และไม่ใช่ "full-market coverage" ของตลาดสหรัฐ

## Universe funnel

| Gate | ขั้น | บริษัท |
|---|---|---|
| U0 | อยู่ใน company_tickers_exchange.json | 8,023 |
| U1 | SEC exchange label = NYSE หรือ Nasdaq (แยก NYSE American ไม่ได้) | 6,036 |
| U2 | SIC ไม่ใช่การเงิน (ตัด 6000–6799) | 4,217 |
| U3 | ผู้ยื่น 10-K/10-Q ในประเทศ, ≥ 2 ปีงบ, มี 10-Q ล่าสุด | 2,964 |
| U4 | มี us-gaap revenue เป็น USD | 2,773 |
| U8 | core (กลุ่มที่จัดอันดับได้) | 1,571 |
| U8 | provisional (ประวัติสั้น 112 · filing ล่าช้า 67) | 179 |
| U8 | excluded หลัง U3 | 1,214 |

สาเหตุหลักที่ถูกตัดหลัง U3: TTM revenue ต่ำกว่าขั้นต่ำ 458 · ประวัติไม่พอ 432 · ไม่มี us-gaap USD revenue 183 · ไม่มี tag operating income 122

## Class × eligibility

| class | core | provisional | excluded |
|---|---:|---:|---:|
| P | 1,009 | 49 | 43 |
| G | 79 | 2 | 9 |
| T | 180 | 7 | 32 |
| W | 303 | 9 | 374 |
| U | 0 | 112 | 0 |
| (ไม่มีคลาส) | 0 | 0 | 756 |

ได้อันดับ (มีอย่างน้อย 1 เส้นทางผ่านเกณฑ์) 705 บริษัท: core P 474 · core T 155 · core G 55 · provisional 21 · Tier B 127
evidence ของกลุ่มที่ได้อันดับ: confirmed 510 · early 103 · reversion_risk 88 · single_period 4

## ความครบของข้อมูล risk (core 1,571)

| รายการ | ผล |
|---|---|
| debt state | known 1,162 · unknown 384 · partial_zero 23 (นับเป็น unknown) · confirmed_zero 2 |
| risk inputs ครบ 2/2 | 677 · ครบ 1/2: 764 · 0/2: 130 |
| special items ปีฐาน ครบทุก tag | 5 บริษัทเท่านั้น (ไม่ครบเลย 1,100) |

ข้อจำกัดสำคัญ: หนี้และ finance lease ที่รายงานแบบมี XBRL dimension มองไม่เห็นใน companyfacts → ระบบแสดง "ไม่ทราบ" ไม่ตีความเป็นศูนย์

## Freshness (core + provisional, นับถึงวัน snapshot 2026-10-31 ไม่ใช่วันนี้)

| percentile | วันนับจากสิ้นไตรมาส | วันนับจากวันยื่น |
|---|---:|---:|
| p10 | 31 | 2 |
| p50 | 123 | 84 |
| p90 | 123 | 89 |

ไตรมาสล่าสุดใน run จบ 2026-10-04 · filing ล่าสุด 2026-10-30 · ข้อมูลนี้ **ไม่ได้อัปเดตอัตโนมัติ** (automatic refresh ยังไม่เปิดใช้)

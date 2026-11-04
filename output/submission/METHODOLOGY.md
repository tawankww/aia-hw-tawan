# หลักฐานและวิธีการ (Evidence & methodology) · ชุดส่งตรวจ 4 พ.ย. 2026

> **สรุป 3 บรรทัด**
> - ทุกตัวเลขในโฟลเดอร์นี้มาจาก **run ที่ freeze แล้ว** ของ Radar บนข้อมูล SEC EDGAR snapshot **2026-10-31** (ไม่ได้ดาวน์โหลดใหม่ ไม่ได้ refresh ไม่ได้แก้โมเดล)
> - 30 บริษัทคือ **purposive sample ของ Phase 0.4** ที่เลือกไว้ล่วงหน้าเมื่อ 2026-11-01 ด้วยกฎ deterministic (seed 20261031) ก่อนเห็นผล ranking ไม่ได้คัดใหม่เพื่อให้โมเดลดูดี
> - เป็น **technical validation** ของ pipeline และกลไกโมเดล ไม่ใช่หลักฐานว่าโมเดลคัดหุ้นได้ดี ไม่ใช้ราคา ไม่ใช่คำแนะนำซื้อขาย

## 1. แหล่งข้อมูลที่ใช้ (frozen source evidence)

| รายการ | ค่า |
|---|---|
| SEC snapshot | companyfacts.zip, submissions.zip, company_tickers_exchange.json ดาวน์โหลด 2026-10-31 (ICT) |
| Run | `20261101T093108Z_cf20261031_sub20261031_tix20261031_r3f0ac4ad8_full` (R3 v0.1 baseline, frozen reference) |
| Model / thresholds | R3 v0.1 · `r3_config_sha256` ขึ้นต้น `f0ac4ad8` · freeze record ลงเวลา 2026-11-01T09:20:37Z (ก่อน run เริ่ม 09:31Z) |
| ขอบเขต run | U3 = 2,964 บริษัท (NYSE/Nasdaq, non-financial, ยื่น 10-K/10-Q ในประเทศ, ≥ 2 ปีงบ) |
| ความถูกต้องของไฟล์ | dashboard ตรวจ sha256 ของ 131 ไฟล์ตอนเปิด run → freeze record matched |

หมายเหตุตามจริง: run นี้ถูกสร้างขณะ git working tree "dirty" (มีโค้ดที่ยังไม่ commit) ซึ่ง dashboard แสดงไว้ตรง ๆ ความถูกต้องของไฟล์ผลลัพธ์ตรวจแยกด้วย hash

## 2. วิธีเลือก 30 บริษัท (เลือกไว้ก่อนเห็นผล)

ใช้ตัวอย่าง Phase 0.4 เดิมทั้งชุด ไม่เพิ่ม ไม่ตัด:

1. **Negative controls 3 ตัว** (ต้องถูกตัดออกจาก universe): ธนาคาร 1 (SIC 6020–6036), REIT 1 (SIC 6798), ผู้ยื่น 20-F/40-F 1
2. **Short history 2 ตัว** (IPO / spin-off ที่ประวัติไม่พอ)
3. **Positives 25 ตัว** จาก U3 pool: revenue ≥ $100M และมี operating income + OCF; เติมโควตากรณียากก่อน (fiscal year ไม่ตรงปฏิทิน, ปี 52/53 สัปดาห์, 10-KT, restatement, tag เปลี่ยน, turnaround, ซื้อกิจการ, หนี้ไม่ทราบ ฯลฯ) แล้วเติมที่เหลือแบบสุ่ม seed คงที่ (sha256(seed:cik), seed 20261031)

ตัวแปรที่ใช้เลือกเป็น **proxy จากงบรายปี** ไม่ใช่ผล classification ของ R3 ดังนั้นการกระจายคลาสในตารางคือผลที่ออกมาจริง ไม่ได้บังคับ

**Bias ที่ต้องรู้:** positives ต้องมี `OperatingIncomeLoss` จึงทำให้ coverage ใน sample สูงเกินจริงโดยการออกแบบ ห้ามใช้ sample นี้อ้าง coverage ของตลาด (ใช้ตัวเลขทั้ง U3 ใน `COVERAGE_SUMMARY.md` แทน)

## 3. ผลจริงของ 30 บริษัท (จาก engine ไม่ได้แต่งเพิ่ม)

| สถานะ | จำนวน | บริษัท (ticker) |
|---|---|---|
| **P** Profitable improver · ได้อันดับ | 7 | MUR, CAPL, DIT, FSS, SIRI, LAUR, GOLF (Tier B: reversion risk) |
| **P** · ไม่มีเส้นทางผ่านเกณฑ์ (ค้นได้ ไม่มีอันดับ) | 6 | RMR, JJSF, NE, MNTK, GOOGL, CTOS |
| **G** Cash-generating growth · ได้อันดับ | 2 | SYNA, SYM |
| **T** Turnaround · ได้อันดับ | 2 | ALTO, IOT |
| **W** Watch (ไม่จัดอันดับโดยกฎ) | 4 | LTRX, AESI, MBC, INSM |
| **U / provisional** ประวัติสั้น | 3 | NAVN, NXB, ROCK |
| **Excluded** | 6 | CBNA, VICI, B (negative controls) · LFCR, LFMD, FWONA (`insufficient_history`: LFMD และ FWONA มี filing ที่ถูก recast/restate ซึ่ง pipeline ปฏิเสธแทนการผสม vintage · LFCR มีการเปลี่ยนปีงบด้วย 10-KT) |

- อันดับใน CSV (`rank_within_class_u3`) คือ **ตำแหน่งในคลาสเดียวกันของทั้ง U3** (เช่น FSS = 242 จาก 474 ใน P) ไม่ใช่อันดับภายใน 30 บริษัท
- "ไม่มี risk flag ที่ trigger" **ไม่ได้แปลว่าปลอดภัย** ให้ดู `risk_display` ว่า input ครบหรือไม่
- `mixed_vintage` แยกไว้ในคอลัมน์ `provenance_mixed_vintage` (เป็นข้อมูล provenance ไม่ใช่ warning) — เกิดกับเกือบทุกบริษัทเพราะไตรมาสที่ derive ใช้ตัวเลขจากหลาย filing ตามกฎ latest-filed

## 4. การตรวจสอบที่ทำ (และที่ยังไม่ได้ทำ)

| การตรวจ | ผล |
|---|---|
| ผลของ engine ใน CSV ตรงกับ run Phase 0.8 (sample run) หรือไม่ | 25/25 บริษัทที่อยู่ใน U3 ตรงทั้ง class, primary path, multiple, evidence, tier · อีก 5 อยู่นอก U3 โดยการออกแบบ (controls 3 + short history 2) |
| **Manual reconciliation กับ filing ต้นฉบับ (Phase 0.6)** | **10 จาก 30 บริษัทเท่านั้น:** JJSF, GOOGL, LFMD, ROCK, MNTK, LFCR, FSS, LTRX, SYNA, ALTO · แต่ละบริษัท 8 ไตรมาส × 3 ตัว (revenue, operating income, OCF) = 240 ค่า ตรงกับงบที่พิมพ์ใน filing 240/240 |
| ความถูกต้องอิสระของ 240 ค่านั้น | 132 ค่าตรงกับตัวเลข 3 เดือนที่บริษัทแสดงเอง · 24 ค่า derive แล้วมี earnings release (8-K) ยืนยัน · 68 ค่าถูกตามกฎแต่ไม่มีตัวเลขอิสระ · 16 ค่า pipeline ปฏิเสธเอง |
| อีก 20 บริษัท | **ไม่ได้ reconcile ด้วยมือ** ใช้ automated reconciliation ของ pipeline เท่านั้น (Σ4 ไตรมาส = ปีงบ, direct = YTD diff) |

ใน 7 บริษัทที่ได้อันดับและอยู่ใน recon มีแค่ FSS, SYNA, ALTO ส่วน MUR, CAPL, DIT, SIRI, LAUR, GOLF, SYM, IOT ยังไม่ได้ reconcile ด้วยมือ

## 5. วิธีอ่าน `sample30_evidence.csv`

| คอลัมน์ | ความหมาย |
|---|---|
| `selection_pool`, `selected_for` | เหตุผลที่บริษัทอยู่ใน sample (กฎ Phase 0.4) |
| `eligibility`, `r3_class` | core / provisional / excluded และคลาส P G T W U |
| `ranked`, `rank_within_class_u3`, `ranked_in_class_u3_n` | ได้อันดับไหม · ตำแหน่งในคลาสของทั้ง U3 · จำนวนที่ได้อันดับในคลาสนั้น |
| `primary_path`, `best_multiple_capped5` | เส้นทางที่แรงที่สุด (margin / accel / cash) และกี่เท่าของ threshold (สูงสุด 5×) |
| `S_g_…pct`, `S_m_…pp`, `S_c_…pp`, `S_a_…pp` | สัญญาณ TTM ณ ไตรมาส t เทียบ t−4 (S_g เป็น %, ที่เหลือเป็น percentage point) |
| `ttm_revenue_usd`, `ttm_ocf_usd`, `op_margin_ttm` | ตัวเลข TTM ณ `latest_quarter_end` (op_margin เป็นสัดส่วน 0.162 = 16.2%) |
| `debt_state`, `risk_display`, `warnings` | สถานะหนี้ 4 แบบ, คำอธิบาย risk พร้อม completeness, คำเตือน evidence |
| `revenue_t_*` | revenue ของไตรมาส t, วิธี derive, accession number และ URL ของ filing index บน sec.gov |
| `companyfacts_api_url` | URL ของ SEC XBRL API สำหรับตรวจซ้ำ |
| `manual_recon_phase0_6`, `manual_recon_result` | เคย reconcile ด้วยมือหรือไม่ และผล |
| `phase0_8_sample_result`, `matches_phase0_8` | ผลของ sample run เดิม และตรงกับ baseline หรือไม่ |

เซลล์ว่าง = ไม่มีค่า (null) ไม่ใช่ศูนย์

## 6. สิ่งที่ห้ามอ่านจากไฟล์นี้

- ห้ามอ่านเป็นคำแนะนำลงทุนหรือการคาดการณ์ราคา (ระบบไม่ใช้ราคาหุ้นเลย)
- ห้ามอ่านเป็นข้อมูล live: ข้อมูลหยุดที่ 2026-10-31 (ไตรมาสล่าสุดใน run จบ 2026-10-04, filing ล่าสุด 2026-10-30)
- ห้ามอ่านเป็น backtest หรือหลักฐานว่า R3 ดีกว่าโมเดลอื่น — ยังไม่มีการ validate คุณภาพโมเดล (blind review ทางการกำหนดไว้บน snapshot ≥ 2027-02-01)
- threshold ทุกตัวเป็นค่าชั่วคราว (provisional) ที่ freeze ก่อน run และไม่ได้ปรับตามผล

# WBTH IT Asset Inventory — Hardware

เว็บแอปหน้าเดียวสำหรับดูและจัดการชีต **Hardware** ในไฟล์ IT Asset Inventory (.xlsx)
ค้นหาว่าใครถืออุปกรณ์อะไร ส่งมอบ รับคืน เพิ่มเครื่องใหม่ด้วยการสแกนบาร์โค้ด/QR/OCR แล้วดาวน์โหลดกลับเป็นไฟล์ Excel เดิม

**เปิดใช้งาน:** https://anochaakimbo.github.io/wbth-asset-inventory/

> A single-file web app for the Hardware sheet of an IT asset inventory workbook: look up devices by owner,
> check devices in and out, add devices by scanning barcode/QR or reading the label with OCR,
> and write the changes back into the original .xlsx. Everything runs in the browser.

## ข้อมูลอยู่ที่ไหน

- ไฟล์ Excel ถูกอ่านและเขียนในเบราว์เซอร์เท่านั้น ไม่มีการอัปโหลดไปเซิร์ฟเวอร์ใด
- เบราว์เซอร์จำไฟล์ล่าสุดและรายการที่ยังไม่ได้ดาวน์โหลดไว้ใน localStorage ของเครื่องนั้น ใช้เครื่องส่วนกลางให้กด "เปิดไฟล์อื่น" ทับ หรือล้างข้อมูลเว็บไซต์ก่อนลุก
- repo นี้มีแค่โค้ดกับข้อมูลตัวอย่างปลอม **ห้าม commit ไฟล์ Excel/CSV ที่มีข้อมูลจริง** (`.gitignore` กันไว้แล้ว)
- OCR บน GitHub Pages ใช้ Tesseract.js ประมวลผลในเครื่อง ส่วนในหน้า claude.ai จะส่งรูปป้ายให้ Claude อ่าน

## หน้าหลัก

| หน้า | ทำอะไรได้ |
|---|---|
| ภาพรวม | ตัวเลขสรุปตามสถานะ, แยกตามประเภท/แผนก, ประกันที่จะหมดรายเดือน 12 เดือน, รายการที่ต้องดูแล (หมดประกันแต่ยังใช้งาน, ไม่มี Asset tag, tag/serial ซ้ำ ฯลฯ) |
| อุปกรณ์ | แท็บสถานะ, ค้นหา, ตัวกรอง (Type, Department, Location, ประกัน, OS, AutoPilot, License, Disposal), เลือกคอลัมน์, ส่งออก CSV |
| ผู้ถือครอง | รายชื่อทุกคน, อุปกรณ์ที่แต่ละคนถือ, ส่งมอบจากคลัง (เลือกหลายชิ้น), รับคืน/รับคืนทั้งหมด, คัดลอกรายการไปทำใบส่งมอบ |

ค้นหาได้จากทุกหน้าด้วย `Ctrl K` (หรือ `/`) และปุ่ม **สแกน** บนแถบด้านบนใช้หาเครื่องจากบาร์โค้ด

## สแกน S/N

- **กล้องสด** ใช้ได้เมื่อเปิดผ่าน HTTPS (GitHub Pages) หรือ localhost ถ้าเปิดกล้องไม่ได้ ใช้ปุ่ม "ถ่ายรูป / เลือกรูป" แทน (บนมือถือจะเปิดกล้องถ่ายรูปให้)
- อ่านได้: QR, Code 128, Code 39/93, EAN/UPC, ITF, Data Matrix, PDF417 (ใช้ `BarcodeDetector` ของเบราว์เซอร์ถ้ามี ไม่งั้นใช้ ZXing)
- กรอกให้อัตโนมัติเมื่อมั่นใจว่าเป็น S/N:
  - บาร์โค้ด Lenovo รูปแบบ `1S` + รุ่น 10 ตัว + S/N 8 ตัว → ตัด S/N ออกมาให้
  - QR ที่มีคำว่า `S/N`, `Serial`, `Service Tag` หรือเป็นลิงก์ที่มีพารามิเตอร์ `sn=`
  - ค่าที่ตรงกับเครื่องที่มีอยู่แล้วในไฟล์
  - รูปที่มีโค้ดน่าจะเป็น S/N แค่อันเดียว
- ถ้าป้ายมีหลายโค้ด จะแสดงให้เลือกเอง EAN/UPC (รหัสสินค้า) จะถูกจัดไว้ท้ายสุด
- **OCR** สำหรับป้ายที่ไม่มีบาร์โค้ด: ถ่ายรูปแล้วกด "อ่านตัวหนังสือ" ตรวจค่าที่ได้กับป้ายอีกครั้งเสมอ (O กับ 0, I กับ 1 สับสนได้)

## สถานะคำนวณจากอะไร

ไฟล์ไม่มีคอลัมน์สถานะ แอปจึงดูจาก `Asset owner` และ `Disposal`

| สถานะ | เงื่อนไข |
|---|---|
| คืนแล้ว | Asset owner = `Returned` หรือ Disposal = `Returned` |
| ชำรุด | Asset owner มีคำว่า broken / ชำรุด / เสีย / ซ่อม |
| ในคลัง | Asset owner ว่าง, `-`, `N/A`, `IT Inventory`, `To be Active` |
| ใช้งานอยู่ | นอกเหนือจากนั้น (คน ทีม หรือห้องประชุม) |

รับคืนจะเขียน Asset owner เป็น `IT Inventory`, `Returned` หรือ `IT Inventory - Broken` และเปลี่ยน Department ตามที่เลือก

## เขียนกลับลง Excel อย่างไร

- การเพิ่ม/แก้ไข/ส่งมอบ/รับคืน จะค้างอยู่ในหน้าจนกว่าจะกด **ดาวน์โหลด Excel**
- ไฟล์ที่ได้คือไฟล์เดิมทั้งเล่ม: แถวใหม่ต่อท้ายชีต Hardware โดยก๊อปรูปแบบจากแถวสุดท้ายและขยาย AutoFilter, แถวที่แก้จะเขียนเฉพาะเซลล์ที่เปลี่ยน, ชีตอื่นไม่ถูกแตะ
- หลังดาวน์โหลด ให้ใช้ไฟล์ใหม่ (`*_updated_YYYYMMDD-HHMM.xlsx`) เป็นไฟล์หลักต่อไป

## โครงสร้าง

ทั้งหมดอยู่ใน `index.html` ไฟล์เดียว ไม่ต้อง build ไลบรารีโหลดจาก jsDelivr ตามต้องใช้

| ไลบรารี | ใช้ทำอะไร | License |
|---|---|---|
| [ExcelJS](https://github.com/exceljs/exceljs) 4.4.0 | อ่าน/เขียน .xlsx โดยคงรูปแบบเดิม | MIT |
| [ZXing-js](https://github.com/zxing-js/library) 0.21.3 | อ่านบาร์โค้ด/QR | Apache-2.0 |
| [Tesseract.js](https://github.com/naptha/tesseract.js) 5.1.1 | OCR | Apache-2.0 |
| [Lucide](https://lucide.dev) icons (ฝังในไฟล์) | ไอคอน | ISC |
| [Sarabun](https://fonts.google.com/specimen/Sarabun) | ฟอนต์ (ใช้ TH SarabunPSK ถ้าเครื่องมี) | OFL |

รันในเครื่อง: `npx serve .` หรือ `python3 -m http.server` แล้วเปิด http://localhost:8000 (ต้องเป็น localhost หรือ HTTPS กล้องสดถึงจะใช้ได้)

# QAHubs — Setup Guide

QAHubs คือ Test Management Platform ที่ทำงานบน Google Sheets + Google Apps Script (GAS) เป็น Backend และไฟล์ `index.html` เป็น Frontend แบบ Single-Page App ไม่ต้องใช้ Server หรือ Database เพิ่มเติม

---

## สิ่งที่ต้องมีก่อน

- Google Account (Gmail)
- เข้าถึง [Google Drive](https://drive.google.com) และ [Google Sheets](https://sheets.google.com)
- เข้าถึง [Google Apps Script](https://script.google.com)

---

## ขั้นตอนที่ 1 — สร้าง Google Spreadsheet

1. ไปที่ [Google Sheets](https://sheets.google.com) แล้วสร้าง Spreadsheet ใหม่ (ตั้งชื่อว่าอะไรก็ได้ เช่น `QAHubs Data`)
2. คัดลอก **Spreadsheet ID** จาก URL ของ Spreadsheet

   ```
   https://docs.google.com/spreadsheets/d/  ← ID อยู่ตรงนี้  /edit
   ```

   ตัวอย่าง ID: `1x-WEHf1Q1wF_A7E8ngx1Nu7i7oCWZ74Ys3sgqHtW0lk`

---

## ขั้นตอนที่ 2 — ตั้งค่า Google Apps Script

1. เปิด [Google Apps Script](https://script.google.com) แล้วสร้าง Project ใหม่ (New Project)
2. ลบโค้ดเดิมออกทั้งหมด แล้ว **วางโค้ดจาก `Code.gs`** ลงไปแทน
3. แก้ไขค่าตัวแปรในบรรทัดต้นของไฟล์:

   ```javascript
   const SS_ID = 'YOUR_SPREADSHEET_ID_HERE'; // ← ใส่ ID จากขั้นตอนที่ 1
   ```

4. (ถ้าต้องการ) เปลี่ยน Encryption Key ให้ปลอดภัยยิ่งขึ้น:

   ```javascript
   var ENCRYPT_KEY = 'YOUR_CUSTOM_SECRET_KEY'; // ← ต้องตรงกับ index.html ด้วย
   ```

   > ⚠️ ถ้าเปลี่ยน `ENCRYPT_KEY` ต้องอัปเดตทั้ง `Code.gs` **และ** `index.html` พร้อมกัน และ re-seed ข้อมูลเดิมใน Sheet ทั้งหมด

---

## ขั้นตอนที่ 3 — Initialize Sheets

1. ใน Apps Script Editor กด **Run** แล้วเลือก function `initSheets`
2. ครั้งแรกจะมีหน้าต่างขอ Permission — กด **Review Permissions** → เลือก Google Account → **Allow**
3. ตรวจสอบใน Google Sheets ว่ามี Sheet ต่อไปนี้ถูกสร้างขึ้นแล้ว:

   | Sheet | คำอธิบาย |
   |---|---|
   | Users | ข้อมูลผู้ใช้งาน |
   | Teams | ข้อมูลทีม |
   | Members | สมาชิกในทีม |
   | Projects | โปรเจกต์ |
   | Tasks | งานในโปรเจกต์ |
   | TestCases | Test Case |
   | Defects | Defect / Bug |
   | Logs | Activity Log |
   | LoginLogs | ประวัติการล็อกอิน |
   | PendingMembers | คำขอเข้าร่วมทีม |

---

## ขั้นตอนที่ 4 — Deploy เป็น Web App

1. ใน Apps Script Editor ไปที่ **Deploy → New Deployment**
2. เลือก Type: **Web app**
3. ตั้งค่าดังนี้:

   | ตัวเลือก | ค่าที่ต้องใช้ |
   |---|---|
   | Execute as | **Me** (บัญชีของคุณ) |
   | Who has access | **Anyone** (ไม่ต้อง Sign-in) |

4. กด **Deploy** แล้วคัดลอก **Web App URL** ที่ได้ เช่น:

   ```
   https://script.google.com/macros/s/AKfycb.../exec
   ```

5. ทดสอบ URL โดยเพิ่ม `?action=ping` ต่อท้าย — ควรได้ response:

   ```json
   { "ok": true, "data": { "pong": true, "time": "..." } }
   ```

---

## ขั้นตอนที่ 5 — ตั้งค่า Frontend (index.html)

1. เปิดไฟล์ `index.html` ด้วย Text Editor
2. ค้นหาบรรทัด `GAS_URL` แล้วแก้ไขให้ตรงกับ Web App URL ที่ Deploy ไว้:

   ```javascript
   const GAS_URL = 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec';
   ```

3. ตรวจสอบให้แน่ใจว่า `ENCRYPT_KEY` ใน `index.html` ตรงกับใน `Code.gs`:

   ```javascript
   const ENCRYPT_KEY = 'YOUR_CUSTOM_SECRET_KEY'; // ← ต้องเหมือนกันทุกอักขระ
   ```

---

## ขั้นตอนที่ 6 — เปิดใช้งาน

เปิดไฟล์ `index.html` ในเบราว์เซอร์ได้เลย (ไม่ต้องใช้ Web Server) หรืออัปโหลดไปฝากบน:

- **GitHub Pages** — ฟรี, เหมาะสำหรับ Static files
- **Google Drive** (Shared Link)
- Web Hosting ทั่วไป

---

## การอัปเดต Web App (Re-deploy)

เมื่อแก้ไข `Code.gs` แล้ว ต้อง Deploy เวอร์ชันใหม่ทุกครั้ง:

1. **Deploy → Manage Deployments**
2. กดแก้ไข (✏️) บน Deployment ที่มีอยู่
3. เปลี่ยน Version เป็น **New Version**
4. กด **Deploy**

> ⚠️ URL ของ Web App จะไม่เปลี่ยน ไม่ต้องอัปเดต `index.html`

---

## ตรวจสอบ / Debug

| วิธี | รายละเอียด |
|---|---|
| `?action=ping` | ทดสอบว่า GAS ทำงานอยู่ |
| `?action=getAll&sheet=users` | ดึงข้อมูล Users ทั้งหมด |
| `?action=init` | สร้าง Sheet ใหม่ (ถ้ายังไม่มี) |
| Apps Script → Executions | ดู Log และ Error ของ GAS |
| `testPayload()` | รัน function ใน Editor เพื่อทดสอบ encode/decode |

---

## โครงสร้างไฟล์

```
qahubs/
├── Code.gs        # Google Apps Script Backend
└── index.html     # Frontend (Single-Page App, React via CDN)
```

---

## หมายเหตุด้านความปลอดภัย

- **ENCRYPT_KEY** ใช้ XOR cipher เพื่อ obfuscate ข้อมูล `email`, `password` และ `inviteCode` ใน Sheet — ไม่ใช่การเข้ารหัสระดับ Production
- ไม่ควรแชร์ `ENCRYPT_KEY` หรือ `SS_ID` ให้บุคคลอื่น
- ถ้าต้องการความปลอดภัยสูงกว่านี้ ควรย้ายไปใช้ Backend ที่รองรับ Authentication มาตรฐาน (เช่น Firebase, Supabase)

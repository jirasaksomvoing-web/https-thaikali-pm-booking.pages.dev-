THAIKALI PM BOOKING — Vercel Frontend + Google Apps Script Backend

ไฟล์ในชุดนี้
- index.html / styles.css / app.js : เว็บไซต์ Frontend สำหรับ deploy บน Vercel
- api/booking.js : Vercel Serverless Function ทำหน้าที่เป็น proxy ไปยัง Apps Script
- google-apps-script/Code.gs : Backend โดยใช้โค้ดระบบจอง PM เดิม เพิ่ม doPost API ที่ตรวจ BOOKING_SECRET
- vercel.json : การตั้งค่า Vercel
- .env.example : ตัวอย่าง environment variables (ห้ามใช้ secret ตัวอย่างจริง)

ภาพรวม
พนักงาน -> เว็บไซต์ Vercel -> /api/booking -> Google Apps Script Web App -> Google Sheets
ผู้ใช้งานเข้าลิงก์ Vercel ไม่จำเป็นต้องเปิดหน้า /exec โดยตรง

ส่วนที่ 1 — Google Apps Script
1. เปิด Google Spreadsheet ที่ใช้ระบบ PM เดิม > Extensions > Apps Script
2. สำรองโค้ดเดิมก่อน แล้วนำ Code.gs ในโฟลเดอร์ google-apps-script ไปใช้แทน/รวมกับ Code.gs เดิม โดยต้องเก็บ spreadsheet binding และข้อมูล CONFIG ให้ถูกต้อง
3. ไปที่ Project Settings > Script Properties > Add script property:
   Name: BOOKING_SECRET
   Value: สร้าง secret แบบสุ่มยาวอย่างน้อย 32 ตัวอักษร (อย่าใช้ค่าตัวอย่าง)
4. Run setupSystem() จาก Apps Script Editor หนึ่งครั้ง และอนุญาตสิทธิ์ที่ระบบร้องขอ
5. Deploy > New deployment > Web app
   - Execute as: Me (เจ้าของสคริปต์)
   - Who has access: Anyone (ถ้านโยบายองค์กรอนุญาต)
6. คัดลอก URL ที่ลงท้าย /exec

ส่วนที่ 2 — Vercel
1. สร้าง Project ใหม่ใน Vercel แล้ว Import Git Repository ที่มีไฟล์ทั้งหมดในโฟลเดอร์นี้ หรือใช้ Vercel CLI จากโฟลเดอร์นี้
2. ใน Project > Settings > Environment Variables เพิ่ม:
   APPS_SCRIPT_URL = URL /exec จาก Apps Script
   BOOKING_SECRET = ค่าเดียวกันกับ Script Property BOOKING_SECRET
3. กำหนด Environment ให้ Production (และ Preview หากจำเป็น) แล้ว Redeploy
4. เปิด URL ของ Vercel เพื่อทดสอบหน้าเว็บ

ข้อควรตรวจสอบใน Google Sheets
- Users: ต้องมี FullName/Name และ Department/แผนก; แนะนำ Email และ Active ด้วย
- รายการคอมพิวเตอร์_พร้อมแผน: LOCATION_NAME ต้องตรงกับ Department ใน Users และต้องมี Asset Item ที่ไม่ซ้ำ
- Departments: ต้องมี Department และ HeadEmail โดย Department ต้องตรงกับ LOCATION_NAME
- ผู้บริหาร: ต้องมีคอลัมน์ Email/อีเมลผู้บริหาร
- PM_Bookings และ Audit_Log ถูกสร้าง/ปรับโดย setupSystem เมื่อรัน

ข้อจำกัด/ความปลอดภัยสำคัญ
- BOOKING_SECRET อยู่ใน Vercel server-side environment และ Script Properties เท่านั้น ห้ามใส่ใน index.html/app.js
- API นี้ไม่เปิดเผย action processApproval เพราะการอนุมัติต้องตรวจสอบตัวตนผู้อนุมัติจริง; คงขั้นตอนอนุมัติผ่าน Google Apps Script/ช่องทางที่ยืนยันตัวตนได้
- หน้าสาธารณะให้เลือกชื่อจาก Users แต่การเลือกชื่อไม่ใช่การยืนยันตัวตน ผู้ใช้สามารถเลือกชื่อผู้อื่นได้ หากต้องการป้องกันการแอบอ้าง ต้องเพิ่ม Google OAuth/SSO หรือระบบยืนยันตัวตนก่อนสร้างคำขอ
- API ชุดนี้ไม่เปิดการยกเลิกคำขอจากหน้าเว็บสาธารณะ เพื่อป้องกันการแอบอ้างชื่อผู้จอง หากต้องการยกเลิกควรเพิ่มระบบยืนยันตัวตนหรือ cancellation token ที่เดาไม่ได้
- การนำ Vercel มาเป็น Frontend จะทำให้ผู้ใช้ทั่วไปไม่ต้องเปิด /exec โดยตรง แต่ไม่ได้ลบหน้าความปลอดภัยที่ Google อาจแสดงเมื่อมีคนเปิด URL /exec เอง
- ก่อนใช้งานจริง ทดสอบการจับคู่ Department กับ LOCATION_NAME, จำนวนเครื่อง, วันจองย้อนหลัง, อีเมลหัวหน้าแผนก และการแจ้งเตือน

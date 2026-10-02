# Skillfolio

หน้าเว็บ Skillfolio (HTML ล้วน) รันบน GitHub Pages และใช้ Google Apps Script Web App เป็นเซิร์ฟเวอร์หลังบ้าน ข้อมูลเก็บใน Google Sheets

- `docs/index.html` หน้าผู้ใช้
- `docs/admin.html` หน้าแอดมิน

## วิธี deploy

1. **Apps Script** (โค้ดฝั่งเซิร์ฟเวอร์ไม่ได้อยู่ใน repo นี้)
   Deploy → Manage deployments → แก้ deployment เดิม → Version: New version
   ตั้ง Execute as = Me, Who has access = Anyone แล้วคัดลอก URL ที่ลงท้ายด้วย `/exec`
2. **ตั้งค่าหน้าเว็บ** แก้ 2 บรรทัดบนสุดของ `docs/index.html` และ `docs/admin.html`
   ```js
   const API_URL = 'https://script.google.com/macros/s/…/exec';
   const APP_URL = 'https://<user>.github.io/skillfolio/';
   ```
3. **GitHub Pages** Settings → Pages → Deploy from a branch → `main` / `docs`

หน้าเว็บเรียกเซิร์ฟเวอร์ด้วย `fetch` แบบ POST และ header `Content-Type: text/plain;charset=utf-8`
(ห้ามใช้ `application/json` เพราะจะติด CORS)

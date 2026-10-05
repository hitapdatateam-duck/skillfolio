# Skillfolio

Skillfolio เป็นเว็บสำหรับรวบรวมทักษะ ประสบการณ์ และความสนใจของคนในทีม
แต่ละคนสร้างโปรไฟล์ทักษะของตัวเองได้ ส่วนผู้ดูแลจะเห็นภาพรวมทักษะของทั้งทีม

- **หน้าผู้ใช้:** https://hitapdatateam-duck.github.io/skillfolio/
- **หน้าแอดมิน:** https://hitapdatateam-duck.github.io/skillfolio/admin.html

---

## สารบัญ

1. [ภาพรวมระบบ](#ภาพรวมระบบ)
2. [โครงสร้างไฟล์](#โครงสร้างไฟล์)
3. [หน้าเว็บคุยกับเซิร์ฟเวอร์อย่างไร](#หน้าเว็บคุยกับเซิร์ฟเวอร์อย่างไร)
4. [ฐานข้อมูล (Google Sheets)](#ฐานข้อมูล-google-sheets)
5. [ติดตั้งครั้งแรก](#ติดตั้งครั้งแรก)
6. [อัปเดตระบบ](#อัปเดตระบบ)
7. [ความเร็วและแคช](#ความเร็วและแคช)
8. [ความปลอดภัย](#ความปลอดภัย)
9. [แก้ปัญหาที่พบบ่อย](#แก้ปัญหาที่พบบ่อย)
10. [รายการตรวจหลัง deploy](#รายการตรวจหลัง-deploy)

---

## ภาพรวมระบบ

```
เบราว์เซอร์ผู้ใช้
   │  1. โหลดหน้าเว็บ (HTML/CSS/JS ล้วน)
   ▼
GitHub Pages  ── docs/index.html, docs/admin.html
   │
   │  2. fetch POST {fn, args, id}
   ▼
Google Apps Script Web App (/exec)  ── code.gs (ไม่ได้อยู่ใน repo นี้)
   │  3. อ่าน/เขียนข้อมูล
   ▼
Google Sheets  ── Profiles, Users, Sessions, Logs
```

- **หน้าเว็บ** เป็นไฟล์ HTML ล้วน โฮสต์บน GitHub Pages ฟรี
- **เซิร์ฟเวอร์** คือ Google Apps Script ที่ deploy เป็น Web App ทำหน้าที่เป็น API ให้หน้าเว็บ
- **ข้อมูล** ทั้งหมดเก็บใน Google Sheets ของบัญชีที่ deploy สคริปต์

**หน้าตา:** ธีมขาว เหลือง ทอง แนว minimal หน้าแรกมีพื้นหลังเคลื่อนไหวแบบเรียบ (แสงทอง เส้นไหม ประกาย และเนินเขาที่เลื่อนช้าๆ)
ถ้าเครื่องผู้ใช้ตั้งค่า "ลดภาพเคลื่อนไหว" ไว้ หน้าจะแสดงแบบนิ่ง สไตล์ทั้งหมดอยู่ใน `<style id="theme-gold">` ซึ่งซ้อนทับสไตล์เดิม

ลิงก์ Apps Script เดิม (`/exec` แบบเปิดตรง) ยังใช้งานได้ เพราะ `doGet` เดิมยังอยู่
หน้าเว็บบน GitHub Pages กับลิงก์เดิมจึงใช้ข้อมูลชุดเดียวกัน

---

## โครงสร้างไฟล์

| ไฟล์ | อยู่ที่ | หน้าที่ |
|---|---|---|
| `docs/index.html` | repo นี้ | หน้าผู้ใช้: สำรวจโปรไฟล์ สร้างและแก้โปรไฟล์ สมัครสมาชิก ล็อกอิน |
| `docs/admin.html` | repo นี้ | หน้าแอดมิน: ภาพรวมทีม จัดการผู้ใช้ บันทึกการใช้งาน |
| `README.md` | repo นี้ | เอกสารนี้ |
| `code.gs`, `index.html`, `admin.html` | โปรเจกต์ Apps Script เท่านั้น | โค้ดเซิร์ฟเวอร์ และหน้าเว็บเวอร์ชันที่เปิดผ่านลิงก์ Apps Script เดิม |

> **ห้ามอัปโหลด `code.gs` หรือไฟล์สำรองใดๆ ขึ้น repo นี้**
> repo นี้เป็นสาธารณะ ใครก็อ่านได้ ควรเก็บเฉพาะไฟล์ที่ไม่มีรหัสผ่าน ID ภายใน หรือข้อมูลส่วนตัว

### ค่าตั้งต้นในหน้าเว็บ

ทั้ง `docs/index.html` และ `docs/admin.html` มีค่าตั้งต้น 2 บรรทัดอยู่ด้านบนสุดของไฟล์

```js
const API_URL = 'https://script.google.com/macros/s/…/exec'; // Apps Script Web App (/exec)
const APP_URL = 'https://hitapdatateam-duck.github.io/skillfolio/'; // GitHub Pages (ลงท้ายด้วย /)
```

- `API_URL` คือ URL ของ Apps Script Web App ที่ลงท้ายด้วย `/exec`
- `APP_URL` คือ URL ของ GitHub Pages ใช้สร้างลิงก์แชร์โปรไฟล์และลิงก์ไปหน้าแอดมิน **ต้องลงท้ายด้วย `/`**

ถ้าเปลี่ยน URL ใด ต้องแก้ให้ตรงกันทั้ง 2 ไฟล์

---

## หน้าเว็บคุยกับเซิร์ฟเวอร์อย่างไร

### รูปแบบคำขอและคำตอบ

หน้าเว็บส่งคำขอแบบ `POST` ไปที่ `API_URL` ด้วยฟังก์ชัน `rpc(fn, ...args)`

```http
POST /macros/s/…/exec
Content-Type: text/plain;charset=utf-8

{"fn": "getProfile", "args": ["<profile-id>", {"token": "", "admin": "", "user": "<session>"}], "id": "<random-uuid>"}
```

เซิร์ฟเวอร์ (`doPost` ใน `code.gs`) ตอบกลับเป็น JSON หนึ่งในสองแบบ

```json
{"ok": true,  "result": { ... }}
{"ok": false, "error": "ข้อความ error เดิมจากเซิร์ฟเวอร์"}
```

ถ้าได้ `ok:false` หน้าเว็บจะ `throw new Error(error)` ข้อความ error ที่หน้าเว็บใช้แยกกรณีจึงทำงานเหมือนเดิม เช่น

- `ADMIN_SESSION_EXPIRED` หน้าแอดมินจะล็อกและขอรหัสผ่านใหม่
- `USER_SESSION_EXPIRED` หน้าผู้ใช้จะออกจากระบบและแจ้งให้ล็อกอินใหม่

### ทำไมต้องใช้ `text/plain` ไม่ใช่ `application/json`

ถ้าใช้ `application/json` เบราว์เซอร์จะส่งคำขอ `OPTIONS` (CORS preflight) ไปถามก่อน
Apps Script ไม่ตอบคำขอ `OPTIONS` คำขอจึงถูกบล็อกด้วย error เรื่อง CORS
`text/plain` จัดเป็น "simple request" จึงไม่มี preflight และเซิร์ฟเวอร์ยังอ่าน JSON จาก `e.postData.contents` ได้ตามปกติ

### คำสั่งที่เรียกได้ (whitelist)

`doPost` เรียกได้เฉพาะฟังก์ชันในรายการนี้ ตรวจด้วย `Object.prototype.hasOwnProperty`
ชื่ออื่นจะได้ `{"ok":false,"error":"ไม่รู้จักคำสั่ง: …"}`

| กลุ่ม | ฟังก์ชัน |
|---|---|
| ตั้งค่า | `getConfig` คืนค่า `{ emailDomains }` |
| โปรไฟล์ | `listProfiles`, `getProfile`, `saveProfile`, `deleteProfile`, `setProfileVisibility`, `claimProfile` |
| สมาชิก | `registerUser`, `loginUser`, `logoutUser`, `getMe`, `changePassword`, `requestPasswordReset`, `resetPassword` |
| แอดมิน | `loginAdmin`, `logoutAdmin`, `adminListUsers`, `adminListLogs`, `adminUpdateUser`, `adminSetUserLocked`, `adminSetPassword`, `adminRevokeSessions`, `adminDeleteUser` |

ฟังก์ชันภายใน เช่น `setup` หรือฟังก์ชันที่ลงท้ายด้วย `_` เรียกจากภายนอกไม่ได้

### คำตอบหายระหว่างทาง (HTTP 404) และการส่งซ้ำ

Apps Script ไม่ได้ส่งคำตอบกลับมาตรงๆ แต่ redirect (302) ไปลิงก์ชั่วคราวที่ `script.googleusercontent.com`
แล้วเบราว์เซอร์ค่อยไปรับคำตอบจากลิงก์นั้น บางครั้ง Google ตอบ **404 "ไม่พบเพจ ไดรฟ์"** ทั้งที่สคริปต์ทำงานเสร็จแล้ว
ในการทดสอบเกิดประมาณ 1 ใน 10 ครั้ง

ระบบรับมือแบบนี้

1. ทุกคำขอมี `id` สุ่มใหม่ (UUID)
2. `doPost` เก็บคำตอบของแต่ละ `id` ไว้ใน `CacheService` 10 นาที
3. ถ้าได้ 404 หรือการเชื่อมต่อล้ม หน้าเว็บจะส่งคำขอเดิมซ้ำด้วย `id` เดิม ส่งได้สูงสุด 3 ครั้ง แต่ละครั้งเว้นช่วงสั้นๆ
4. ถ้าเซิร์ฟเวอร์เคยทำงานกับ `id` นี้แล้ว จะส่งคำตอบเดิมกลับไป **ไม่ทำงานซ้ำ**
   การบันทึก การสมัครสมาชิก หรือการเปลี่ยนรหัสผ่านจึงไม่เกิดซ้ำ

> การส่งซ้ำทุกคำสั่งปลอดภัยเมื่อ `code.gs` เป็นเวอร์ชันที่ `doPost` เก็บคำตอบตาม `id` แล้วเท่านั้น
> ถ้าย้อนเซิร์ฟเวอร์ไปเวอร์ชันเก่า ต้องย้อนหน้าเว็บให้ส่งซ้ำเฉพาะคำสั่งที่อ่านข้อมูลด้วย

---

## ฐานข้อมูล (Google Sheets)

สเปรดชีตชื่อ **Skillfolio • Profiles** สร้างอัตโนมัติตอนรัน `setup()`
ID ของสเปรดชีตเก็บใน Script Property `SPREADSHEET_ID` ไม่ได้อยู่ในโค้ด

| แท็บ | คอลัมน์ | เก็บอะไร |
|---|---|---|
| `Profiles` | id, tokenHash, updatedAt, profileJSON, ownerId | ข้อมูลโปรไฟล์ทั้งหมดในรูป JSON และเจ้าของโปรไฟล์ |
| `Users` | id, email, name, passwordHash, createdAt, lastLoginAt, status, lastSeenAt | บัญชีสมาชิก รหัสผ่านเก็บเป็น salted hash (HMAC-SHA256 400 รอบ) |
| `Sessions` | tokenHash, userId, expiresAt | session ของผู้ใช้ เก็บเฉพาะ hash ของ token อายุ 30 วัน |
| `Logs` | time, event, userId, email, detail | บันทึกการใช้งาน เก็บล่าสุดประมาณ 6,000 แถว |

**เหตุการณ์ในแท็บ Logs:** `register`, `login`, `login_fail`, `login_blocked`, `logout`, `visit`, `password_change`,
`reset_request`, `reset`, `profile_create`, `profile_delete`, `admin_login`, `admin_login_fail`, `admin_edit`,
`admin_lock`, `admin_unlock`, `admin_password`, `admin_revoke`, `admin_delete`

> แก้ข้อมูลในชีตโดยตรงได้ แต่หน้าเว็บอาจแสดงข้อมูลเก่าได้นานถึง 10 นาที ดู [ความเร็วและแคช](#ความเร็วและแคช)

---

## ติดตั้งครั้งแรก

ทำตามลำดับนี้ถ้าจะติดตั้งระบบใหม่ทั้งหมด หรือย้ายไปบัญชีอื่น

### 1. เตรียม Apps Script

1. ไปที่ https://script.google.com แล้วสร้างโปรเจกต์ใหม่
2. สร้างไฟล์ `code.gs`, `index.html`, `admin.html` แล้ววางโค้ดลงไป
   ไฟล์ทั้งหมดอยู่กับผู้ดูแลระบบ ไม่ได้อยู่ใน repo นี้
3. ไปที่ **Project Settings → Script Properties** แล้วเพิ่ม property
   - `ADMIN_PASSWORD` คือรหัสผ่านหน้าแอดมิน (จำเป็น)
4. ในหน้า editor เลือกฟังก์ชัน `setup` แล้วกด **Run** แล้วอนุญาตสิทธิ์ที่ขอ
   - สคริปต์จะสร้างสเปรดชีตให้ และตั้งค่า `SPREADSHEET_ID` เอง
   - ถ้ามีสเปรดชีตเดิมอยู่แล้ว ให้ใส่ `SPREADSHEET_ID` เองก่อนกด Run

ค่าตั้งต้นที่แก้ได้อยู่ด้านบนสุดของ `code.gs`

| ค่า | ค่าเริ่มต้น | ความหมาย |
|---|---|---|
| `ALLOWED_EMAIL_DOMAINS` | `['hitap.net']` | โดเมนอีเมลที่สมัครสมาชิกได้ `[]` คือรับทุกโดเมน หน้าเว็บอ่านค่านี้ผ่าน `getConfig` |
| `REQUIRE_LOGIN_TO_CREATE` | `true` | ต้องล็อกอินก่อนสร้างโปรไฟล์ |
| `SESSION_DAYS` | `30` | อายุ session ของผู้ใช้ |
| `PASSWORD_MIN` | `8` | ความยาวรหัสผ่านขั้นต่ำ |
| `ALLOW_ADMIN_EMBED` | `false` | ให้หน้าแอดมินบนลิงก์ Apps Script ฝังในเว็บอื่นได้หรือไม่ |
| `PROFILE_CACHE_SECONDS` | `600` | อายุแคชข้อมูลโปรไฟล์ (วินาที) |
| `REPLY_CACHE_SECONDS` | `600` | อายุคำตอบที่เก็บไว้สำหรับคำขอที่ส่งซ้ำ (วินาที) |

### 2. Deploy Apps Script เป็น Web App

1. **Deploy → New deployment** แล้วเลือกชนิด **Web app**
2. ตั้งค่า
   - **Execute as:** Me
   - **Who has access:** Anyone
3. กด **Deploy** แล้วคัดลอก URL ที่ลงท้ายด้วย `/exec`

### 3. ตั้งค่าหน้าเว็บ

แก้ 2 บรรทัดบนสุดของ `docs/index.html` และ `docs/admin.html`

```js
const API_URL = 'https://script.google.com/macros/s/<deployment-id>/exec';
const APP_URL = 'https://<github-user>.github.io/skillfolio/';
```

### 4. เปิด GitHub Pages

1. อัปโหลด `docs/` และ `README.md` ขึ้น branch `main`
2. ไปที่ **Settings → Pages**
   - **Source:** Deploy from a branch
   - **Branch:** `main` และโฟลเดอร์ `/docs`
3. กด **Save** แล้วรอ 1–2 นาที
4. ถ้า `APP_URL` ยังไม่ตรงกับ URL ของ Pages ให้แก้แล้วอัปโหลดใหม่

---

## อัปเดตระบบ

### อัปเดตโค้ดเซิร์ฟเวอร์ (`code.gs`)

1. แก้โค้ดใน editor ของ Apps Script แล้วกดบันทึก
2. ไปที่ **Deploy → Manage deployments** แล้วกด ✏️ ที่ deployment เดิม
3. **Version:** เลือก **New version** แล้วกด **Deploy**

> **ห้ามกด New deployment**
> ถ้าสร้าง deployment ใหม่ จะได้ URL `/exec` ใหม่ และหน้าเว็บจะเรียกเซิร์ฟเวอร์ไม่ได้จนกว่าจะแก้ `API_URL`
> การแก้ deployment เดิมทำให้ URL ไม่เปลี่ยน

### อัปเดตหน้าเว็บ (`docs/`)

1. แก้ไฟล์ใน `docs/` (ผ่านเว็บ GitHub หรือ `git push`)
2. GitHub Pages จะ build ใหม่เองภายใน 1–2 นาที
3. ผู้ใช้ที่เปิดหน้าค้างไว้ ให้กด **Cmd+Shift+R** (Mac) หรือ **Ctrl+Shift+R** (Windows) เพื่อโหลดไฟล์ใหม่

### ลำดับเมื่อแก้ทั้งสองฝั่ง

ถ้าหน้าเว็บใหม่ต้องใช้ความสามารถใหม่ของเซิร์ฟเวอร์ ให้ **deploy เซิร์ฟเวอร์ก่อน แล้วค่อยอัปโหลดหน้าเว็บ**
`doPost` ไม่สนใจ field ที่ไม่รู้จัก หน้าเว็บเวอร์ชันเก่าจึงยังใช้กับเซิร์ฟเวอร์ใหม่ได้

---

## ความเร็วและแคช

Apps Script ใช้เวลาประมาณ 1 วินาทีต่อคำขอเสมอ บวกขั้น redirect อีกประมาณ 0.5 วินาที
ระบบจึงลดจำนวนครั้งที่ต้องเรียกและลดงานต่อครั้งด้วยวิธีต่อไปนี้

| จุด | ทำอะไร | ผล |
|---|---|---|
| แคชข้อมูลโปรไฟล์ (เซิร์ฟเวอร์) | ฟังก์ชันที่อ่านอย่างเดียว เช่น `listProfiles`, `getProfile`, `getMe` และหน้าผู้ใช้ของแอดมิน อ่านแท็บ `Profiles` จาก `CacheService` แทนการเปิด Sheets | จาก 2–3 วินาที เหลือประมาณ 1.3 วินาทีต่อครั้ง |
| ล้างแคชเมื่อมีการเขียน | ทุกครั้งที่บันทึก ลบ เปลี่ยนสถานะ เชื่อมโปรไฟล์ หรือลบผู้ใช้ จะเปลี่ยนเวอร์ชันแคช | หน้าเว็บเห็นข้อมูลใหม่ทันที |
| ฟังก์ชันที่เขียนข้อมูล | ส่งชีตเข้าไปเองและอ่านจากชีตโดยตรงเสมอ ไม่ใช้แคช | ไม่มีโอกาสเขียนทับด้วยข้อมูลเก่า |
| เก็บรายชื่อไว้ในเบราว์เซอร์ (หน้าผู้ใช้) | เก็บรายชื่อโปรไฟล์สาธารณะไว้ใน `localStorage` เปิดหน้าครั้งถัดไปจะแสดงทันที แล้วค่อยอัปเดต | รายชื่อขึ้นทันทีเมื่อเคยเปิดมาก่อน |
| ลิงก์ `?id=` | โหลดโปรไฟล์พร้อมกับรายชื่อ ไม่ต้องรอ | เห็นโปรไฟล์เร็วขึ้น 2–3 วินาที |

**ข้อควรรู้**
- ถ้าแก้ข้อมูลใน Google Sheets โดยตรง หน้าเว็บอาจแสดงข้อมูลเก่าได้นานถึง 10 นาที (`PROFILE_CACHE_SECONDS`)
- ถ้าแคชมีปัญหา ระบบจะกลับไปอ่านจากชีตเอง ไม่แสดง error
- หน้าแอดมินไม่เก็บรายชื่อไว้ในเบราว์เซอร์ เพราะมีโปรไฟล์ที่ไม่เปิดเผยปนอยู่

---

## ความปลอดภัย

- **Whitelist:** `doPost` เรียกได้เฉพาะฟังก์ชันในรายการ ฟังก์ชันภายในเรียกไม่ได้
- **สิทธิ์ตรวจที่เซิร์ฟเวอร์ทุกครั้ง:** สิทธิ์แอดมิน เจ้าของโปรไฟล์ และโดเมนอีเมล ตรวจใน `code.gs` ไม่ได้พึ่งหน้าเว็บ
  ถ้าหน้าเว็บปล่อยอีเมลโดเมนอื่นผ่านมา เซิร์ฟเวอร์ก็ยังปฏิเสธ
- **รหัสผ่านแอดมิน:** อยู่ใน Script Properties (`ADMIN_PASSWORD`) ไม่ได้อยู่ในโค้ดหรือ repo
  session แอดมินมีอายุ 6 ชั่วโมง
- **รหัสผ่านผู้ใช้:** เก็บเป็น salted hash ถ้าใส่ผิด 5 ครั้งจะล็อก 10 นาที
- **กันการฝังหน้าแอดมิน (clickjacking):** GitHub Pages ตั้ง header `X-Frame-Options` ไม่ได้
  `admin.html` จึงตรวจด้วย JavaScript ถ้าถูกเปิดใน iframe จะซ่อนเนื้อหาทั้งหมด และแสดงลิงก์ให้เปิดในแท็บใหม่แทน
- **คำตอบที่เก็บไว้สำหรับส่งซ้ำ:** อาจมี session token อยู่ด้วย เก็บ 10 นาที โดยใช้ hash ของ `id` สุ่มเป็นกุญแจ
  ต้องรู้ `id` นั้นจึงจะดึงคำตอบได้
- **Repo สาธารณะ:** ห้ามเพิ่มไฟล์ที่มีรหัสผ่าน ID สเปรดชีต หรือข้อมูลส่วนตัว
  `API_URL` อยู่ในหน้าเว็บได้ เพราะเป็น URL สาธารณะอยู่แล้ว

---

## แก้ปัญหาที่พบบ่อย

| อาการ | สาเหตุ | วิธีแก้ |
|---|---|---|
| `เซิร์ฟเวอร์ตอบกลับไม่ถูกต้อง (HTTP 405)` | `API_URL` ยังไม่ได้ตั้ง หรือผิด หน้าเว็บจึงส่ง POST ไปที่ GitHub Pages | ตั้ง `API_URL` เป็น URL `/exec` ให้ถูกต้องทั้ง 2 ไฟล์ |
| `เซิร์ฟเวอร์ตอบกลับไม่ถูกต้อง (HTTP 404)` | Google ทำคำตอบหายระหว่างทาง ระบบส่งซ้ำให้เองแล้วแต่ยังไม่สำเร็จ หรือ deployment ถูกลบหรือเก็บถาวร | กดลองใหม่ ถ้าเกิดตลอด ให้ตรวจว่า URL `/exec` ยังใช้งานได้ใน Manage deployments |
| Console ขึ้น error เรื่อง CORS | `Content-Type` ไม่ใช่ `text/plain` หรือ Who has access ไม่ใช่ Anyone | ใช้ `text/plain;charset=utf-8` และตั้ง Who has access = Anyone |
| `ไม่รู้จักคำสั่ง: …` | เรียกฟังก์ชันที่ไม่อยู่ใน whitelist หรือเซิร์ฟเวอร์ยังเป็นเวอร์ชันเก่า | เพิ่มชื่อใน `API_` ใน `code.gs` แล้ว deploy New version |
| `ยังไม่ได้ตั้งรหัสผ่านแอดมิน` | ไม่มี Script Property `ADMIN_PASSWORD` | เพิ่มใน Project Settings → Script Properties |
| `เปิดฐานข้อมูลไม่ได้ …` | `SPREADSHEET_ID` ผิด หรือบัญชีที่ deploy ไม่มีสิทธิ์เข้าสเปรดชีต | ตรวจ `SPREADSHEET_ID` แล้วรัน `setup()` |
| แก้โค้ดแล้วเว็บยังทำงานแบบเดิม | ยังไม่ได้ deploy New version หรือเบราว์เซอร์ใช้ไฟล์เก่า | deploy New version แล้วกด Cmd/Ctrl+Shift+R |
| แก้ข้อมูลในชีตแล้วเว็บยังไม่เปลี่ยน | แคชโปรไฟล์ยังไม่หมดอายุ | รอไม่เกิน 10 นาที หรือแก้ผ่านหน้าเว็บแทน |
| เว็บช้าเป็นบางครั้ง (5–20 วินาที) | Apps Script ช้าเป็นช่วงๆ โดยเฉพาะครั้งแรกหลังไม่มีคนใช้นาน | เป็นข้อจำกัดของ Apps Script ส่วนใหญ่หายเองในคำขอถัดไป |

**ดูว่าเซิร์ฟเวอร์ทำงานหรือไม่:** เปิดโปรเจกต์ Apps Script แล้วไปที่ **Executions**
จะเห็นทุกครั้งที่ `doPost` ถูกเรียก พร้อมระยะเวลาและสถานะ

**ทดสอบ API จาก Terminal:**

```bash
curl -sL -X POST -H 'Content-Type: text/plain;charset=utf-8' -d '{"fn":"getConfig","args":[]}' 'https://script.google.com/macros/s/<deployment-id>/exec'
```

ควรได้ `{"ok":true,"result":{"emailDomains":["hitap.net"]}}`

---

## รายการตรวจหลัง deploy

- [ ] หน้าแรกโหลดรายชื่อโปรไฟล์ได้
- [ ] หน้าสมัครสมาชิกปฏิเสธอีเมลที่ไม่ใช่ `@hitap.net` และขึ้นข้อความ "ใช้ได้เฉพาะอีเมล @hitap.net เท่านั้น"
- [ ] ลิงก์ `?id=<profile-id>` ของโปรไฟล์สาธารณะเปิดได้
- [ ] `admin.html` แสดงหน้าใส่รหัสผ่าน และล็อกอินแอดมินได้
- [ ] หน้าแอดมินแสดงรายชื่อผู้ใช้และบันทึกการใช้งานได้
- [ ] Console ของเบราว์เซอร์ไม่มี error เรื่อง CORS
- [ ] ลิงก์ Apps Script เดิมยังเปิดได้

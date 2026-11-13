# SMART TRUCK : SAFETY RUN ⭐
### ภารกิจล่าดาว พิชิตเซฟตี้

เกม 3D Endless Runner สำหรับพนักงานขับรถบรรทุก TMT STEEL ส่งเสริมการสวม PPE 100% และ ZERO ACCIDENT
เล่นผ่านเว็บได้ทั้งมือถือและคอมพิวเตอร์ เก็บคะแนนด้วย Firebase (Anonymous Auth + Firestore) และ Deploy ด้วย Firebase Hosting

---

## โครงสร้างโปรเจกต์

```
public/
  index.html           ตัวเกมทั้งหมด (HTML + CSS + JavaScript ไฟล์เดียว)
  firebase-config.js   ค่าเชื่อมต่อ Firebase (ต้องแก้ใส่ของโปรเจกต์จริง)
firestore.rules        กฎความปลอดภัยของฐานข้อมูล
firestore.indexes.json
firebase.json          ตั้งค่า Hosting + Firestore
.firebaserc            ชื่อโปรเจกต์ Firebase (ต้องแก้)
.github/workflows/firebase-deploy.yml   Deploy อัตโนมัติเมื่อ push ขึ้น main
```

## วิธีใช้งาน (สำหรับผู้เล่น / เจ้าหน้าที่)

- **พนักงานขับรถ**: เปิดลิงก์เกม → กด START GAME → กรอกทะเบียนรถ → เล่น คะแนนบันทึกอัตโนมัติ ดูคะแนนตัวเองได้ที่ปุ่ม **คะแนนของฉัน**
- **เจ้าหน้าที่**: แตะ **โลโก้ SMART TRUCK** หน้าแรก 5 ครั้งติดกัน → กรอกรหัส → เข้า **หลังบ้าน**
  ดูสรุปรายสัปดาห์ (เริ่มวันจันทร์ 00:00 น. เวลาไทย), อันดับ 1–3, ประวัติการเล่น, ค้นหา และคัดลอก CSV ไปวางใน Excel

> รหัสเจ้าหน้าที่ **ไม่ได้อยู่ในโค้ด** — ตรวจสอบฝั่งเซิร์ฟเวอร์โดย Firestore Rules เทียบกับเอกสาร `config/staff` ฟิลด์ `pin`
> เปลี่ยนรหัสได้ทุกเมื่อใน Firestore Console โดยไม่ต้องแก้โค้ด

---

## ตั้งค่าครั้งแรก (ทำครั้งเดียว)

### 1) สร้างโปรเจกต์ Firebase
1. เข้า https://console.firebase.google.com → **Add project** ตั้งชื่อ เช่น `smart-truck-safety-run`
2. เมนู **Build → Authentication → Get started → Sign-in method** เปิด **Anonymous**
3. เมนู **Build → Firestore Database → Create database** เลือก Location `asia-southeast1 (Singapore)` แบบ **Production mode**
4. **Project settings (รูปเฟือง) → Your apps → เพิ่ม Web app (`</>`)** แล้วคัดลอกค่า `firebaseConfig`

### 2) ใส่ค่าในโปรเจกต์
- แก้ `public/firebase-config.js` ใส่ค่า `firebaseConfig` ที่คัดลอกมา
- แก้ `.firebaserc` เปลี่ยน `YOUR_PROJECT_ID` เป็น Project ID จริง

### 3) ตั้งรหัสเจ้าหน้าที่
ใน Firestore Console → **Start collection** ชื่อ `config` → Document ID `staff` → เพิ่มฟิลด์
`pin` (string) = `6776`

### 4) Deploy ครั้งแรก (จากเครื่องคอมพิวเตอร์)
ต้องมี Node.js 18+ แล้วรัน
```bash
npm install -g firebase-tools
firebase login
firebase deploy --only hosting,firestore:rules
```
จะได้ลิงก์เกม เช่น `https://<project-id>.web.app` ส่งลิงก์นี้ให้พนักงานในกลุ่ม LINE ได้เลย (ไม่ต้องมีบัญชีใด ๆ)

### 5) (ไม่บังคับ) Deploy อัตโนมัติจาก GitHub
1. Google Cloud Console ของโปรเจกต์ → **IAM & Admin → Service Accounts → Create** ให้สิทธิ์ **Firebase Admin**
   → แท็บ Keys → **Add key → JSON** ดาวน์โหลดไฟล์ (ห้าม commit ไฟล์นี้ลง repo)
2. GitHub repo → **Settings → Secrets and variables → Actions → New repository secret**
   ชื่อ `FIREBASE_SERVICE_ACCOUNT` วางเนื้อหาไฟล์ JSON ทั้งไฟล์
3. ต่อจากนี้ทุกครั้งที่ push ขึ้น `main` ระบบจะ Deploy Hosting + Rules ให้อัตโนมัติ

---

## ความปลอดภัยของข้อมูล (สรุป Firestore Rules)

| ใคร | ทำอะไรได้ |
|---|---|
| ผู้เล่นทุกคน (Anonymous) | บันทึกผลการเล่นใหม่ (ตรวจชนิดข้อมูล/ขนาด) · อ่านได้เฉพาะผลของตัวเอง · แก้/ลบไม่ได้ |
| เจ้าหน้าที่ (กรอก PIN ถูก) | อ่านผลของทุกคน · ลบผลได้ |
| `config/*` | ไม่มีใครอ่าน/เขียนจากเกม (แก้ได้เฉพาะใน Console) |

ข้อควรทราบ
- เกมทำงานฝั่งผู้เล่น ผู้มีความรู้ด้านเว็บอาจปลอมคะแนนได้ ควรใช้ผลเพื่อสร้างแรงจูงใจ และตรวจสอบรอบที่คะแนนสูงผิดปกติก่อนให้รางวัล
- รหัส 4 หลักเดาได้ถ้าตั้งใจ แนะนำเปลี่ยนเป็นรหัสที่ยาวขึ้นใน `config/staff`
- ข้อมูลผูกกับ "อุปกรณ์" (Anonymous) + "ทะเบียนที่กรอก" ถ้าเปลี่ยนเครื่อง/ล้างข้อมูลเบราว์เซอร์ หน้า "คะแนนของฉัน" จะเริ่มใหม่ แต่ข้อมูลในหลังบ้านยังอยู่ครบ

## ทดสอบในเครื่อง
```bash
cd public && python3 -m http.server 8080
# เปิด http://localhost:8080  (ถ้ายังไม่ใส่ firebase-config.js เกมยังเล่นได้ แต่คะแนนเก็บในเครื่อง)
```

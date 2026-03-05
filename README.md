# Lab 11 — Express + TypeScript + MongoDB Atlas + JWT

## 📁 โครงสร้างไฟล์
```
lab11/
├── src/
│   ├── app.ts                 ← จุดเริ่มต้นของ app
│   ├── db.ts                  ← เชื่อมต่อ MongoDB
│   ├── models/User.ts         ← Mongoose Schema
│   ├── middleware/auth.ts     ← ตรวจสอบ JWT
│   ├── routes/authRoutes.ts   ← /register /login /logout
│   ├── routes/pageRoutes.ts   ← / /login(GET) /profile
│   ├── types/express.d.ts     ← TypeScript type สำหรับ req.user
│   └── views/
│       ├── home.ejs
│       ├── login.ejs
│       └── profile.ejs
├── .env                       ← ⚠️ ห้าม commit!
├── .env.example               ← template สำหรับ .env
├── .gitignore
├── package.json
└── tsconfig.json
```

---

## 🚀 วิธีติดตั้งและรันในเครื่อง

### ขั้นที่ 1: ติดตั้ง Dependencies
```bash
npm install
```

### ขั้นที่ 2: สร้างไฟล์ .env
```bash
cp .env.example .env
```
แล้วแก้ไข `.env` ใส่ค่าจริง:
```
PORT=3000
MONGODB_URI=mongodb+srv://username:password@cluster.xxxxx.mongodb.net/lab11_studentID?retryWrites=true&w=majority
JWT_SECRET=my_super_secret_key_at_least_32_chars_long
NODE_ENV=development
```

### ขั้นที่ 3: ตั้งค่า MongoDB Atlas
1. ไปที่ https://cloud.mongodb.com/
2. สร้าง Free Cluster
3. Network Access → Add IP → `0.0.0.0/0`
4. Database Access → Add User (username + password)
5. Connect → Drivers → คัดลอก URI มาใส่ใน `.env`
6. Browse Collections → Add My Own Data → ตั้งชื่อ DB: `lab11_studentID`, Collection: `users`

### ขั้นที่ 4: รัน Dev Server
```bash
npm run dev
```
เปิด http://localhost:3000

---

## 🧪 ทดสอบ
1. ไปที่ http://localhost:3000/login
2. กรอก email/password แล้วกด **Register**
3. Login ด้วย email/password เดิม → ควรไปหน้า `/profile`
4. กด **Logout** → ควรกลับหน้า home
5. ลองเข้า http://localhost:3000/profile ตรงๆ → ควรได้ 401

---

## ☁️ Deploy บน Render

### ขั้นที่ 1: Push ขึ้น GitHub
```bash
git init
git add .
git commit -m "init: Lab11 express ts atlas jwt"
git branch -M main
git remote add origin <YOUR_GITHUB_REPO_URL>
git push -u origin main
```

### ขั้นที่ 2: สร้าง Web Service บน Render
1. ไปที่ https://render.com/ → New → Web Service
2. เชื่อม GitHub repo
3. ตั้งค่า:
   - **Name**: `lab11-express-ts-atlas-jwt-<studentID>`
   - **Branch**: `main`
   - **Build Command**: `npm install && npm run build`
   - **Start Command**: `npm run start`
4. Environment Variables → เพิ่ม:
   - `MONGODB_URI` = (URI จาก Atlas)
   - `JWT_SECRET` = (random string ยาวๆ)
   - `NODE_ENV` = `production`
5. Deploy!

---

## 🔑 อธิบายการทำงาน JWT + Cookie

```
[User กรอก email/password]
       ↓
[Server ตรวจสอบกับ DB + bcrypt]
       ↓
[สร้าง JWT token: { userId, email } signed ด้วย JWT_SECRET]
       ↓
[ใส่ token ลงใน httpOnly Cookie]
       ↓
[Browser เก็บ cookie ไว้ ส่งกลับมาทุก request อัตโนมัติ]
       ↓
[requireAuth middleware ดึง token จาก cookie → verify → ใส่ req.user]
       ↓
[Route handler ใช้ req.user.email แสดงผล]
```

## ⚠️ Security Notes
- `httpOnly: true` → JavaScript ฝั่ง client อ่าน cookie ไม่ได้ (ป้องกัน XSS)
- `secure: true` (production) → ส่งผ่าน HTTPS เท่านั้น
- `sameSite: "lax"` → ป้องกัน CSRF บางส่วน
- `bcrypt` → hash password ก่อนเก็บ (ห้ามเก็บ plain text เด็ดขาด)
- `.env` ไม่ commit → JWT_SECRET และ DB URI ต้องเป็นความลับ

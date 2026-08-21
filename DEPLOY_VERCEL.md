# Hướng Dẫn Deploy Lên Vercel (Tự Động & Nhanh Chóng)

Dự án đã được cấu hình tối ưu sẵn 100% để chạy trên **Vercel** với kiến trúc **Fullstack (Frontend React Vite + Backend Serverless Express API)**.

---

## 🚀 Cách 1: Deploy Trực Tiếp Qua Vercel CLI (Nhanh Nhất, 1 Phút)

Nếu anh đã tải code về máy tính:

1. **Cài đặt Vercel CLI (nếu chưa có):**
   ```bash
   npm i -g vercel
   ```

2. **Chạy lệnh Deploy:**
   ```bash
   vercel
   ```
   *Chọn các thiết lập mặc định (nhấn Enter liên tục).*

3. **Cấu hình biến môi trường Gemini API Key:**
   ```bash
   vercel env add GEMINI_API_KEY
   ```
   *(Dán API key của anh vào và chọn `Production`, `Preview`, `Development`)*

4. **Phát hành lên Production:**
   ```bash
   vercel --prod
   ```

---

## 🌐 Cách 2: Kết Nối GitHub (Tự Động Deploy Khi Có Code Mới)

1. **Đẩy code lên GitHub:**
   ```bash
   git init
   git add .
   git commit -m "Initial commit for Vercel deploy"
   git branch -M main
   git remote add origin https://github.com/<tai-khoan-cua-anh>/<ten-repo>.git
   git push -u origin main
   ```

2. **Import vào Vercel:**
   - Vào [https://vercel.com/new](https://vercel.com/new)
   - Chọn repository vừa tạo và bấm **Import**
   - Tại mục **Environment Variables**, thêm:
     - **Name:** `GEMINI_API_KEY`
     - **Value:** `<API_KEY_CUA_ANH>`
   - Nhấn nút **Deploy**!

---

## ⚙️ Các Cấu Hình Đã Thiết Lập Sẵn Trong Dự Án
- `vercel.json`: Định tuyến `/api/*` về Serverless function `api/index.ts` và các trang còn lại về `dist/index.html`.
- `api/index.ts`: Cổng tiếp nhận Serverless của Express app.
- `.env.example`: Danh mục biến môi trường cần thiết.

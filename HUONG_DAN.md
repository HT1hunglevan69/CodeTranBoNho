# Website lớp 9D1 - D1K51 (React + Tailwind + Supabase)

Giao diện, màu sắc, bố cục lấy theo website cá nhân `aaaa` (nền cyan, thẻ trắng bo góc, nút pill, font Recursive, màn hình "Click to enter...").
Nội dung và chức năng (ảnh lớp, bình luận, đăng nhập/đăng ký, hồ sơ, quản trị) lấy từ `d1k51-1site`.

## Cấu trúc thư mục
```
public/                 favicon, logo, manifest
src/
  App.js                màn hình chào + định tuyến
  config/app.js         ← BẠN ĐIỀN 3 giá trị công khai ở đây (Supabase URL, anon key, reCAPTCHA site key)
  api/supabase.js       kết nối Supabase + gọi Edge Function
  context/AuthContext   trạng thái đăng nhập dùng chung
  layouts/              Header (logo lớp, TikTok, nút tài khoản), Navbar, Footer
  pages/                About, Gallery, Guestbook, Login, Profile, Admin, 404
  pages/Gallery/galleryData.js   ← danh sách ảnh lớp (sửa ở đây)
  assets/imgs/          logo + ảnh lớp
sql/schema.sql                         ← chạy trong Supabase (giữ nguyên bản cũ)
supabase/functions/account/index.ts    ← Edge Function (giữ nguyên bản cũ)
vercel.json             cấu hình build + header bảo mật cho Vercel
```

## Chạy thử trên máy
Cần Node.js 18 trở lên.
```bash
npm install
npm start          # mở http://localhost:3000
```
Khi chưa cài Supabase, các trang vẫn xem được; đăng nhập và bình luận sẽ chưa hoạt động (đó là bình thường).

## Bước 1. Tạo project Supabase
1. Vào https://supabase.com → **Sign in** → **New project**.
2. Đặt tên (vd `d1k51`), đặt **Database password** (lưu lại), chọn region **Southeast Asia – Singapore**. Chờ ~2 phút.
3. **Project Settings → API**: chép **Project URL** và **anon public key**.

## Bước 2. Cấu hình Auth (quan trọng)
**Authentication → Sign In / Providers**:
- **TẮT** "Allow new users to sign up".
- Giữ **tắt** Anonymous sign-ins. Email provider để bật (hệ thống dùng email ảo nội bộ, không gửi mail).
- Password: đặt **Minimum password length = 10**.

## Bước 3. Tạo CSDL
**SQL Editor → New query** → dán toàn bộ `sql/schema.sql` → **Run**.
Nếu lỗi `schema "extensions" does not exist` hoặc `crypt`: chạy trước `create extension if not exists pgcrypto;` rồi sửa `extensions.crypt` thành `crypt` trong hàm `verify_password`.

## Bước 4. Tạo reCAPTCHA v2
1. https://www.google.com/recaptcha/admin/create
2. Label `D1K51`, loại **reCAPTCHA v2 → "Tôi không phải người máy" (checkbox)**.
3. Domains: thêm `localhost` (để thử trên máy) và, sau khi deploy, tên miền Vercel của bạn (vd `d1k51.vercel.app`).
4. Chép **Site key** (công khai) và **Secret key** (BÍ MẬT, chỉ dùng ở Bước 5).

## Bước 5. Triển khai Edge Function
Cần Supabase CLI (https://supabase.com/docs/guides/cli).
```bash
supabase login
supabase link --project-ref <MÃ-PROJECT>        # mã nằm trong URL: https://<MÃ>.supabase.co
supabase secrets set APP_SECRET="$(openssl rand -hex 32)" \
  RECAPTCHA_SECRET="<secret key reCAPTCHA>" \
  ALLOWED_ORIGINS="https://ten-ban.vercel.app,http://localhost:3000"
supabase functions deploy account --no-verify-jwt
```
- `ALLOWED_ORIGINS`: liệt kê chính xác các địa chỉ website (có `https://`, không có dấu `/` cuối), cách nhau dấu phẩy. Địa chỉ chạy thử trên máy là `http://localhost:3000`.
- ⚠️ **`APP_SECRET` không được đổi hay mất** sau khi đã có người đăng ký (đổi là mọi tài khoản không đăng nhập được). Hãy sao lưu nó.
- Không dùng được CLI? Vào **Edge Functions → Deploy a new function → Via Editor**, đặt tên `account`, dán nội dung `index.ts`, **tắt "Verify JWT"**, rồi thêm 3 secret ở **Edge Functions → Secrets**.

## Bước 6. Điền cấu hình web
1. Mở `src/config/app.js`, thay 3 giá trị `YOUR-...` bằng Project URL, anon key, reCAPTCHA Site key.
   (Hoặc đặt 3 biến môi trường `REACT_APP_SUPABASE_URL`, `REACT_APP_SUPABASE_ANON_KEY`, `REACT_APP_RECAPTCHA_SITE_KEY` trong Vercel.)
2. Mở `vercel.json`, thay `YOUR-PROJECT` trong `connect-src` bằng mã project Supabase.
   Nếu quên bước này, trình duyệt sẽ chặn mọi lệnh gọi Supabase khi chạy trên Vercel.

## Bước 7. Đưa lên Vercel
- Cách 1: đưa thư mục này lên GitHub → https://vercel.com/new → Import. Framework tự nhận **Create React App** (lệnh build đã nằm trong `vercel.json`).
- Cách 2: cài Vercel CLI rồi chạy `vercel --prod` trong thư mục dự án.

Sau khi có địa chỉ thật, quay lại **Bước 4** (thêm vào reCAPTCHA Domains) và **Bước 5** (chạy lại `supabase secrets set ALLOWED_ORIGINS=...`).

## Bước 8. Tạo admin đầu tiên
1. Mở `/login` → tab **Đăng ký** → tạo tài khoản của bạn, **lưu mã khôi phục**.
2. Trong Supabase **SQL Editor** chạy (đổi tên đăng nhập cho đúng):
```sql
update public.profiles set role = 'admin' where username = 'ten_dang_nhap_cua_ban';
```
3. Đăng xuất, đăng nhập lại → trên Header hiện nút **Quản trị**. Cô chủ nhiệm đăng ký xong thì làm lệnh tương tự.

## Thêm / sửa ảnh lớp
1. Bỏ file ảnh vào `src/assets/imgs/gallery/` (nên nén dưới ~300KB mỗi ảnh).
2. Mở `src/pages/Gallery/galleryData.js`, thêm một dòng `import` và một dòng vào `GALLERY`.
3. `category` là `hoctap` (Học tập), `dangoai` (Dã ngoại) hoặc `sukien` (Sự kiện). Hiện chưa có ảnh nào thuộc "Học tập"; mục đó sẽ hiện thông báo "Chưa có ảnh".

Tiêu đề (caption) các ảnh hiện tại do mình đặt tạm theo nội dung ảnh, bạn nên sửa lại cho đúng sự kiện thật.

## Danh sách kiểm tra sau khi cài
- [ ] Đăng ký → hiện hộp mã khôi phục → quay lại trang trước, bình luận được.
- [ ] Đăng nhập sai 5 lần liên tiếp → bị khóa tạm kèm số phút chờ.
- [ ] Quên mật khẩu → dùng mã khôi phục → đổi được, mã mới được cấp, mọi thiết bị bị đăng xuất.
- [ ] Hồ sơ → "Thiết bị đang đăng nhập": đăng nhập 2 trình duyệt, bấm "Đăng xuất" thiết bị kia → trình duyệt đó mất quyền ngay.
- [ ] Admin: khóa tài khoản thử, cấp lại mật khẩu (người đó bị buộc đổi khi đăng nhập), xóa bình luận.
- [ ] F12 → Network: không thấy mật khẩu gửi đi đâu ngoài `.../functions/v1/account`; không có secret nào trong mã nguồn.

## Lớp bảo mật và giới hạn
Giữ nguyên như bản `d1k51-1site` cũ: reCAPTCHA xác minh ở server, tắt đăng ký công khai của Supabase, mật khẩu băm bcrypt, khóa tăng dần khi đoán sai, mã khôi phục dùng một lần, thu hồi phiên từng thiết bị, RLS mọi bảng, nhật ký đăng nhập, CSP chặt (trong `vercel.json`).
Các giới hạn cần biết:
- Khóa tạm có thể bị lợi dụng để "chọc" khóa tài khoản người khác; đổi lại kẻ tấn công phải giải captcha mỗi lần thử.
- Mất cả mật khẩu lẫn mã khôi phục thì chỉ admin cấp lại được.
- Gói Supabase miễn phí **tự tạm dừng sau ~1 tuần không có truy cập**; vào dashboard bấm "Restore" khi cần.
- Nếu reCAPTCHA không hiện và Console báo vi phạm CSP, hãy chụp thông báo gửi lại để chỉnh `script-src` / `frame-src` trong `vercel.json`.

# Sổ Tòa Nhà

Dashboard quản lý tòa nhà 13 căn (100, 201, 202 … 700, 800). Có 3 phần chính:

1. **Điện nước.** Nhập số công tơ ngày 20 hằng tháng. App tự trừ số tháng trước, nhân đơn giá (điện 4.000đ/số), cộng nước sinh hoạt (một số căn được miễn) và nước Lavie (75.000đ/bình). Có nút copy tin nhắn Zalo để gửi hóa đơn cho từng khách.
2. **Tiền nhà.** Mỗi căn có giá thuê và chu kỳ trả riêng (1 tháng, 3 tháng, cả năm…). App tính căn đã trả đến tháng nào, căn nào sắp hết hạn, căn nào quá hạn.
3. **Thu chi.** Có danh sách chi cố định hằng tháng (bảo vệ, làm phòng, ông bà ngoại, lương Minh, anh Lãng, cô Phượng), chỉ cần tích vào ô khi đã trả. Ngoài ra ghi các khoản phát sinh: sửa chữa, vật tư, công an phường, bố mẹ gửi…

Có hai giao diện, tự đổi theo màn hình: **laptop** có menu bên trái, **điện thoại dọc** giống app thật với thanh tab dưới đáy. Trên điện thoại, chọn *Thêm vào màn hình chính* để mở như app.

---

## Cách hoạt động (và vì sao an toàn)

```
┌──────────────────────────┐        ┌──────────────────────────────┐
│ Repo CÔNG KHAI            │        │ Repo RIÊNG TƯ                 │
│ so-toa-nha                │        │ so-toa-nha-data               │
│ index.html, icon…         │ ─────▶ │ data.json  (toàn bộ dữ liệu)  │
│ → GitHub Pages (web)      │ token  │ chỉ ai có token mới đọc được  │
└──────────────────────────┘        └──────────────────────────────┘
         ▲                                     ▲
   Minh mở trên điện thoại            Claude cũng sửa data.json ở đây
```

- **Phần web** (code) để ở repo công khai, bật GitHub Pages miễn phí. Phần này không chứa số liệu nào.
- **Dữ liệu** (`data.json`) để ở repo **riêng tư**. App đọc và ghi qua GitHub API bằng một token cá nhân, mỗi máy nhập một lần.
- Mỗi lần sửa, app lưu ngay vào máy rồi **tự đẩy lên GitHub sau khoảng 2 giây**. Máy khác mở app sẽ thấy bản mới.
- Nếu GitHub có bản mới hơn (ví dụ Claude vừa cập nhật), app sẽ hỏi muốn dùng bản nào. Bản không chọn được giữ làm *bản dự phòng*.
- Mỗi lần lưu là một commit, nên GitHub giữ **lịch sử đầy đủ** và luôn quay lại được bản cũ.

## Cổng khách thuê (`khach.html`)

Mỗi khách đăng nhập bằng **số căn + mật khẩu** để xem hóa đơn điện nước, lịch sử các tháng, tình trạng tiền nhà của riêng căn mình, kèm số tài khoản để chuyển khoản.

- Trong app quản lý, vào **Cài đặt → Cổng khách thuê**, bấm **Tạo** ở từng căn. App sinh mật khẩu ngẫu nhiên (ví dụ `k7fq-m29x`) và copy sẵn tin nhắn Zalo gồm link, số căn, mật khẩu.
- Mỗi lần lưu lên GitHub, app mã hóa hóa đơn từng căn (AES-256-GCM, khóa sinh từ mật khẩu qua PBKDF2) rồi ghi vào `vault/<căn>.json` của repo công khai, gộp thành **1 commit**.
- Người ngoài tải `vault/301.json` về chỉ thấy chuỗi đã mã hóa. Không có mật khẩu thì không đọc được. Tên, số điện thoại, lương, thu chi **không bao giờ** lên repo công khai.
- Khách chuyển đi: bấm **Thu hồi**, file của căn đó bị xóa. Cấp lại cho khách mới: bấm **Tạo** hoặc **Đổi mật khẩu**.
- Khách quên mật khẩu: bấm nút **Gửi** để copy lại tin nhắn, hoặc đổi mật khẩu mới.

## Cài đặt (khoảng 10 phút, làm một lần)

1. **Tạo repo công khai** `so-toa-nha`, tải lên `index.html`, `khach.html`, `manifest.webmanifest`, `icon.svg`.
   Vào *Settings → Pages → Deploy from branch → main / root*. Vài phút sau có link dạng
   `https://<tài-khoản>.github.io/so-toa-nha/`.
2. **Tạo repo riêng tư** `so-toa-nha-data`, tải lên file `data.json`, kèm `CLAUDE.md` để Claude biết cách ghi dữ liệu.
3. **Tạo token:** *GitHub → Settings → Developer settings → Fine-grained tokens → Generate new token*
   - Repository access: **Only select repositories → so-toa-nha-data và so-toa-nha** (cả hai; repo công khai cần để đẩy hóa đơn mã hóa cho khách)
   - Permissions → Repository → **Contents: Read and write**
   - Đặt hạn dùng 1 năm, rồi copy token (dạng `github_pat_…`).
4. Mở link web → **Cài đặt → Đồng bộ GitHub**, điền tài khoản, `so-toa-nha-data`, `main`, `data.json` và token, rồi bấm **Kết nối & tải dữ liệu**.
5. Làm lại bước 4 trên điện thoại. Xong.

> Chưa muốn tạo repo? Cứ mở `index.html` là dùng được ngay. Dữ liệu khi đó chỉ nằm trên máy đang dùng. Vào **Cài đặt → Xem thử dữ liệu mẫu** để xem giao diện khi đã có số liệu.

> Token chỉ nằm trong trình duyệt của từng máy, không có trong code. Nếu mất điện thoại, vào GitHub xoá token đó là xong.

## Cập nhật hằng tháng: 3 cách, dùng cách nào tiện

| Việc | Tự nhập trên app | Nhờ Claude |
|---|---|---|
| Số công tơ ngày 20 | Mục **Điện nước**: gõ số mới, tiền tự tính | Gửi ảnh 13 công tơ kèm câu “cập nhật điện nước T11” |
| Lavie | Bấm + / − ở từng căn | Gửi ảnh tờ thống kê của bảo vệ |
| Gửi hóa đơn | Nút **Copy tin nhắn Zalo**, hoặc copy tất cả một lần | “Soạn tin hóa đơn T11 cho các căn” |
| Khách trả tiền nhà | **Tiền nhà → Ghi nhận**, app tự điền sẵn số tháng và số tiền | “Căn 301 vừa ck 27tr, trả 3 tháng từ T12” |
| Lương, chi cố định | **Thu chi**: tích vào ô (hoặc *Tích tất cả*) | “Tháng này đã trả lương hết” |
| Chi phát sinh | **Thu chi → Thêm**, gõ được `1,5tr`, `800k` | “Hôm nay chi 1tr công an phường” |

Claude làm việc trực tiếp trên repo `so-toa-nha-data` (qua Claude Code hoặc kết nối GitHub), làm theo hướng dẫn trong `CLAUDE.md`, rồi commit `data.json`.

## Các file

| File | Vai trò |
|---|---|
| `index.html` | App quản lý của Minh (giao diện + logic), không cần cài gì thêm |
| `khach.html` | Trang khách thuê đăng nhập xem hóa đơn (đọc `vault/*.json` đã mã hóa) |
| `data.json` | Dữ liệu ban đầu: 13 căn, đơn giá, danh sách nhân viên (số tiền để 0, Minh tự điền) |
| `CLAUDE.md` | Mô tả cấu trúc dữ liệu để Claude cập nhật đúng |
| `manifest.webmanifest`, `icon.svg` | Để cài lên màn hình chính điện thoại như app |

**Cần Minh xác nhận:** đơn giá nước sinh hoạt (đang để 15.000đ/m³), căn nào được miễn nước, và số tiền từng khoản lương. Tất cả sửa được trong **Cài đặt**.

# HƯỚNG DẪN SỬ DỤNG — STAFF / ADMIN / DEV / MODEL

Tài liệu này dành cho người sử dụng hệ thống hằng ngày. Mỗi role chỉ nhìn thấy hoặc thao tác được những phần mà backend cho phép.

## 1. Đăng nhập

1. Mở URL Web App `/exec`.
2. Nhập `Username`.
3. Nhập `Password`.
4. Bấm **Đăng nhập**.
5. Nếu đăng nhập thành công, hệ thống mở Dashboard.

Nếu báo `Sai tài khoản hoặc mật khẩu`, kiểm tra:

- đúng Username;
- tài khoản còn Active;
- mật khẩu có phân biệt hoa/thường.

Nếu tài khoản bị Owner vô hiệu hóa, tài khoản đó không thể đăng nhập.

---

# 2. STAFF

## 2.1 Dashboard

Staff dùng Dashboard để:

- xem trạng thái chấm công của mình;
- xem giờ làm trong tuần;
- xem điểm tuần;
- check-in;
- check-out;
- xác nhận đang làm việc khi có yêu cầu `WAITING`.

### Check-in

Khi bắt đầu ca:

1. vào Dashboard;
2. bấm **CHECK IN**;
3. chờ thông báo `Đã check-in`.

Không bấm nhiều lần. Hệ thống chỉ chấp nhận một check-in mỗi ngày.

### Check-out

Khi kết thúc ca:

1. bấm **CHECK OUT**;
2. hệ thống tính số phút đã làm;
3. cộng điểm hoàn thành ca theo Settings.

### Nếu thấy “Xác nhận bạn vẫn đang làm việc”

Bấm:

```text
TÔI VẪN ĐANG LÀM
```

trong 10 phút. Nếu không xác nhận, trigger có thể tự checkout ca.

## 2.2 Chấm công

Trang **Chấm công** của Staff/Model chỉ hiển thị record của chính tài khoản.

Có thể:

- xem ngày làm;
- check-in/out;
- xem trạng thái PRESENT/LATE;
- xem tổng phút;
- xem điểm;
- xuất CSV dữ liệu đang hiển thị.

## 2.3 Nghỉ phép

1. Vào **Nghỉ phép**.
2. Bấm **+ Tạo đơn nghỉ**.
3. Chọn `Từ ngày`.
4. Chọn `Đến ngày`.
5. Nhập lý do.
6. Bấm **Gửi đơn**.

Staff không tự duyệt đơn của mình. Đơn sẽ ở trạng thái `PENDING` cho tới khi Owner/Admin xử lý.

## 2.4 Điểm

Staff có thể xem log điểm của mình.

Điểm có thể đến từ:

- Có mặt;
- Đúng giờ;
- Đi muộn;
- Hoàn thành ca;
- Điều chỉnh thủ công bởi Admin/Owner.

## 2.5 Task

Vào **Task** để mở hệ thống task server.

Nếu iframe không tải, bấm **Mở Task**.

---

# 3. MODEL

Model sử dụng gần giống Staff:

- Dashboard;
- Chấm công của chính mình;
- Nghỉ phép;
- Xem điểm của mình;
- Task.

Model không có quyền:

- tạo/sửa user;
- cộng/trừ điểm cho người khác;
- xem Audit Log;
- sửa Settings;
- xem báo cáo tuần quản trị.

---

# 4. DEV

DEV dùng cho người phụ trách kỹ thuật/phát triển hệ thống.

DEV có:

- Dashboard;
- Chấm công;
- Nghỉ phép của chính mình;
- xem dữ liệu attendance toàn hệ thống trong trang Chấm công;
- xem Báo cáo tuần;
- xem Task.

DEV không có quyền:

- tạo/sửa user;
- vô hiệu hóa user;
- cộng/trừ điểm thủ công;
- xem Audit Log;
- sửa Settings.

### DEV thường dùng Báo cáo để kiểm tra

- tổng số nhân sự;
- số record có mặt;
- tổng giờ;
- tổng điểm;
- chi tiết theo từng nhân sự.

Nếu đang debug, DEV nên dùng `TROUBLESHOOTING.md` trước khi sửa code.

---

# 5. ADMIN

ADMIN là role quản trị vận hành hằng ngày.

## 5.1 Quản lý nhân sự

Vào **Nhân sự**.

### Tạo user

1. Bấm **+ Thêm nhân sự**.
2. Nhập Username.
3. Nhập tên hiển thị.
4. Chọn Role.
5. Nhập mật khẩu; có thể để trống để hệ thống sinh mật khẩu ngẫu nhiên.
6. Nhập ghi chú nếu cần.
7. Chọn Active.
8. Bấm **Lưu**.

Nếu để trống mật khẩu, hệ thống trả về mật khẩu được sinh trong thông báo.

### Sửa user

1. Bấm **Sửa** ở dòng tương ứng.
2. Chỉnh tên, role, trạng thái hoặc ghi chú.
3. Nếu cần đổi mật khẩu, nhập mật khẩu mới.
4. Bấm **Lưu**.

ADMIN không thể:

- sửa Owner;
- cấp role Owner;
- thay đổi trạng thái Active/Disabled của tài khoản.

Việc vô hiệu hóa hoặc kích hoạt tài khoản chỉ Owner thực hiện.

## 5.2 Quản lý điểm

Vào **Điểm**.

Bấm **+ Cộng/trừ điểm**.

Nhập:

- nhân sự;
- điểm số. Số âm dùng để trừ;
- lý do.

Ví dụ:

```text
+5   Hỗ trợ event
-2   Vi phạm quy định nội bộ
```

Luôn nhập lý do rõ ràng để dễ kiểm tra Audit Log.

## 5.3 Duyệt nghỉ phép

Vào **Nghỉ phép**.

Với đơn `PENDING`, ADMIN có thể:

- **Duyệt** → `APPROVED`;
- **Từ chối** → `REJECTED`.

Một đơn đã xử lý không thể duyệt lại.

## 5.4 Báo cáo

Vào **Báo cáo tuần** để xem báo cáo do trigger tạo.

## 5.5 Audit Log

ADMIN có thể xem log hành động:

- login;
- check-in/out;
- tạo/sửa user;
- cộng/trừ điểm;
- duyệt/từ chối nghỉ;
- thay đổi Settings;
- auto checkout;
- xác nhận đang làm việc.

Audit Log không phải nơi để sửa dữ liệu. Nó là lịch sử kiểm tra.

---

# 6. Ý nghĩa trạng thái

| Status | Ý nghĩa |
|---|---|
| `PRESENT` | Check-in đúng thời gian cho phép |
| `LATE` | Check-in sau giờ bắt đầu + grace |
| `PENDING` | Đơn nghỉ đang chờ duyệt |
| `APPROVED` | Đơn nghỉ đã duyệt |
| `REJECTED` | Đơn nghỉ bị từ chối |
| `AUTO_CHECKOUT` | Ca đã được đóng tự động |
| `WAITING` | Đang chờ xác nhận còn làm việc |
| `CONFIRMED` | Staff đã xác nhận đang làm việc |
| `EXPIRED` | Không xác nhận trong 10 phút, ca bị kết thúc |

---

# 7. Quy trình một ca làm bình thường

```text
Đăng nhập
   ↓
Dashboard
   ↓
CHECK IN
   ↓
Làm việc
   ↓
Nếu có WAITING → TÔI VẪN ĐANG LÀM
   ↓
CHECK OUT
   ↓
Hệ thống tính phút + điểm
```

# 8. Quy trình nghỉ phép

```text
Staff/Dev/Model tạo đơn
        ↓
     PENDING
        ↓
  Admin / Owner xem
      ↙     ↘
APPROVED   REJECTED
```

# 9. Quy tắc sử dụng quan trọng

- Không dùng chung tài khoản.
- Không gửi mật khẩu vào chat công khai.
- Không tự ý sửa Spreadsheet Database nếu chưa biết cấu trúc.
- Khi gặp lỗi, chụp nguyên thông báo lỗi để DEV kiểm tra.
- Không chạy `setup()` tùy tiện trên hệ thống đang vận hành; dù bản mới không xóa dữ liệu, vẫn nên chỉ chạy khi quản trị hệ thống yêu cầu.

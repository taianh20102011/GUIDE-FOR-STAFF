# HƯỚNG DẪN RIÊNG CHO OWNER

Owner là tài khoản có quyền cao nhất trong hệ thống. Tài liệu này tách riêng để tránh nhầm quyền với Staff/Admin/Dev/Model.

## 1. Việc Owner phải làm sau khi cài hệ thống

### Bước 1 — Lấy thông tin Owner đầu tiên

Sau khi chạy `setup()` lần đầu, xem Execution log.

Hệ thống trả về:

- Database ID;
- username `owner`;
- mật khẩu Owner ngẫu nhiên lần đầu.

**Không gửi mật khẩu này vào Discord/public chat.**

### Bước 2 — Đăng nhập

Mở Web App `/exec` và đăng nhập:

```text
Username: owner
Password: mật khẩu trong Execution log
```

### Bước 3 — Đổi mật khẩu Owner

Vào:

```text
Nhân sự → Sửa tài khoản owner
```

Nhập mật khẩu mới và bấm **Lưu**.

Nên dùng mật khẩu riêng cho Owner, không dùng lại mật khẩu Minecraft/Discord.

---

# 2. OWNER làm gì trong hệ thống?

Owner có quyền:

- xem toàn bộ Dashboard;
- xem attendance toàn hệ thống;
- quản lý user;
- tạo Admin/Dev/Staff/Model/Owner;
- chỉnh mật khẩu user;
- vô hiệu hóa tài khoản;
- cộng/trừ điểm;
- duyệt/từ chối nghỉ phép;
- xem Báo cáo tuần;
- xem Audit Log;
- sửa Settings;
- cấu hình Task URL;
- cấu hình Discord Webhook.

Owner được **miễn chấm công** và không cần tạo đơn nghỉ attendance.

---

# 3. Quản lý tài khoản

## Tạo Admin

Vào **Nhân sự → + Thêm nhân sự**.

Chọn:

```text
Role = ADMIN
```

Admin có thể vận hành hệ thống hằng ngày nhưng không được sửa Settings và không được cấp Owner.

## Tạo DEV

```text
Role = DEV
```

DEV phù hợp cho người phụ trách code/kiểm tra kỹ thuật.

## Tạo STAFF

```text
Role = STAFF
```

Đây là role nhân sự vận hành thông thường.

## Tạo MODEL

```text
Role = MODEL
```

Dùng cho thành viên/model cần chấm công nhưng không cần quyền quản trị.

## Tạo Owner phụ

Owner có thể cấp `OWNER`.

Chỉ cấp khi thật sự cần, vì Owner có toàn quyền vận hành.

---

# 4. Vô hiệu hóa tài khoản

Chọn user → **Vô hiệu**.

Hệ thống không xóa record cũ. Nó chỉ đổi:

```text
active = false
```

Do đó:

- lịch sử attendance vẫn còn;
- point logs vẫn còn;
- audit logs vẫn còn;
- tài khoản không thể đăng nhập.

Đây là chủ ý để bảo toàn lịch sử.

---

# 5. Cài đặt hệ thống

Vào **Settings**.

## Shift

### `SHIFT_START`

Giờ bắt đầu ca.

Mặc định:

```text
19:00
```

### `SHIFT_END`

Giờ kết thúc ca tham chiếu.

Mặc định:

```text
23:00
```

Bản hiện tại không ép checkout tại `SHIFT_END`; Staff vẫn có thể checkout sau đó.

### `LATE_GRACE_MINUTES`

Số phút được phép trễ trước khi trạng thái thành `LATE`.

Mặc định:

```text
10
```

Ví dụ:

```text
SHIFT_START = 19:00
GRACE = 10

19:10 → vẫn PRESENT
19:11 → LATE
```

## Auto checkout

### `AUTO_CHECKOUT`

Giờ tự động đóng ca chưa checkout.

Mặc định:

```text
04:00
```

Nếu ca bắt đầu 19:00 và chưa checkout, trigger có thể đóng ca vào 04:00 ngày hôm sau.

---

# 6. Cấu hình điểm

Các key:

```text
POINT_PRESENT
POINT_ON_TIME
POINT_COMPLETE_SHIFT
POINT_LATE
POINT_ABSENT
POINT_LEAVE
```

Mặc định:

```text
POINT_PRESENT = 5
POINT_ON_TIME = 2
POINT_COMPLETE_SHIFT = 2
POINT_LATE = -1
POINT_ABSENT = -5
POINT_LEAVE = 0
```

Lưu ý:

- `POINT_ABSENT` đang là giá trị cấu hình, chưa tự động trừ chỉ vì không có attendance record.
- Không chỉnh điểm khi chưa thống nhất quy định nội bộ.

---

# 7. Discord Webhook

Trong Settings nhập URL Webhook Discord.

Sau đó trigger có thể gửi:

### Work check

```text
KIỂM TRA CA LÀM VIỆC
```

### Weekly report

```text
STAFF WEEKLY REPORT
```

Nếu không dùng Discord, để trống.

---

# 8. Task Server

`TASK_URL` là URL app task.

Owner có thể sửa URL tại Settings.

Sau khi lưu:

```text
Task → Mở Task
```

Nếu iframe không hiển thị, đó thường là do website đích chặn embedding. Đây không nhất thiết là lỗi của hệ thống attendance.

---

# 9. Trigger và maintenance

Chạy một lần:

```text
createTrigger()
```

Trigger gọi:

```text
weeklyMaintenance()
```

mỗi giờ.

Maintenance xử lý:

1. work-check;
2. đóng work-check hết hạn;
3. auto-checkout;
4. weekly report;
5. Discord report.

Không tạo nhiều trigger giống nhau. `createTrigger()` đã tự xóa các trigger `weeklyMaintenance` cũ trước khi tạo lại một trigger mới.

---

# 10. Audit Log

Owner dùng Audit Log để kiểm tra hoạt động hệ thống.

Nên kiểm tra khi:

- user báo không đăng nhập được;
- điểm bị thay đổi;
- có người vô hiệu hóa tài khoản;
- Settings bị thay đổi;
- có auto-checkout;
- cần biết ai duyệt nghỉ.

Audit Log chỉ dùng để xem lịch sử; không sửa trực tiếp log.

---

# 11. Database Google Sheets

Các sheet chính:

```text
Users
Attendance
PointLogs
Leaves
AuditLog
WeeklyReports
Settings
```

### Không nên sửa tay

Không nên tự sửa:

- `passwordHash`;
- `id`;
- `createdAt`;
- `createdBy`;
- cấu trúc cột.

Nếu cần sửa dữ liệu sai, ưu tiên thao tác qua Web App hoặc để DEV xử lý.

---

# 12. Khôi phục khi lỗi

Nếu Web App báo lỗi:

1. vào Apps Script;
2. mở **Executions**;
3. xem execution bị lỗi;
4. copy message lỗi;
5. xem `TROUBLESHOOTING.md`.

Nếu nghi Database sai:

- không chạy `setup()` liên tục;
- không xóa sheet;
- không xóa `DB_ID` khỏi Script Properties;
- chụp màn hình cấu trúc sheet gửi DEV.

---

# 13. Quy tắc Owner

- Không dùng Owner cho nhân sự thông thường.
- Không chia sẻ mật khẩu Owner.
- Không đưa Spreadsheet Database thành public.
- Không cấp Owner hàng loạt.
- Không chạy code Apps Script lạ khi chưa kiểm tra quyền.
- Luôn kiểm tra Execution Log khi setup/deploy/trigger gặp lỗi.

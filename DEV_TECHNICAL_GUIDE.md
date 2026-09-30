# DEV TECHNICAL GUIDE — BẢO TRÌ VÀ PHÁT TRIỂN

Tài liệu này dành cho người phụ trách code/deploy hệ thống, không phải hướng dẫn thao tác attendance hằng ngày.

## 1. Stack

```text
Frontend: HTML + CSS + JavaScript
Backend: Google Apps Script
Database: Google Sheets
Auth: username/password + session token trong CacheService
Deployment: Google Apps Script Web App
Optional: Discord Webhook
```

Không có server Node/Python riêng trong project này.

## 2. File chính

```text
Index.html
    ├─ Login UI
    ├─ Dashboard
    ├─ Attendance
    ├─ Users
    ├─ Points
    ├─ Leaves
    ├─ Reports
    ├─ Audit
    ├─ Settings
    └─ Task iframe

Code.gs
    ├─ doGet / setup / trigger
    ├─ api router
    ├─ authentication/session
    ├─ permissions
    ├─ attendance
    ├─ users
    ├─ points
    ├─ leaves
    ├─ reports
    ├─ audit
    └─ spreadsheet helpers
```

## 3. Luồng request

Frontend không gọi trực tiếp function backend. Nó gọi:

```javascript
call('attendance', {...})
```

Sau đó `Index.html` gửi request tới:

```text
google.script.run.api(...)
```

Backend nhận tại:

```javascript
function api(action, payload) {
  switch (action) {
    ...
  }
}
```

Mỗi action sau đó gọi function nội bộ tương ứng.

## 4. Auth

Login:

```text
login()
  ↓
login_()
  ↓
check Users
  ↓
generate UUID token
  ↓
CacheService
  ↓
return token
```

Các request sau gửi:

```text
token
```

`currentUser_(token)` tìm session và user hiện tại.

Session hiện tại có TTL khoảng 6 giờ.

## 5. Permissions

Nguồn quyền nằm ở:

```javascript
permissions_(role)
```

Backend vẫn kiểm tra quyền trong từng API. Không được chỉ ẩn button ở frontend.

Ví dụ frontend có thể ẩn Settings với Staff, nhưng backend vẫn phải có:

```javascript
if (u.role !== 'OWNER') throw new Error(...)
```

Đây là nguyên tắc bảo mật quan trọng.

## 6. Database schema

### Users

```text
id
username
displayName
role
passwordHash
active
createdAt
createdBy
note
```

### Attendance

```text
id
userId
date
checkIn
checkOut
totalMinutes
status
points
note
workCheckStatus
workCheckSentAt
workCheckDeadline
workCheckResponseAt
```

### PointLogs

```text
id
userId
date
points
type
reason
createdAt
createdBy
```

### Leaves

```text
id
userId
start
end
reason
status
createdAt
reviewedAt
reviewedBy
```

### AuditLog

```text
id
time
actorId
actorName
action
targetId
details
```

### WeeklyReports

```text
id
weekStart
weekEnd
generatedAt
totalStaff
present
late
totalHours
totalPoints
detailsJson
```

### Settings

```text
key
value
```

## 7. Quy tắc code

### Không tạo hai function cùng tên

Đây là lỗi nghiêm trọng đã có ở bản cũ:

```javascript
function audit_(...) {}
function audit_(...) {}
```

JavaScript sẽ ghi đè declaration trước.

Bản fixed tách thành:

```text
audit_()       → ghi log
getAuditLogs_() → đọc log
```

### Luôn kiểm tra quyền trong backend

Không tin `state.permissions` từ client.

### Không trả secret trong bootstrap

`publicSettings_()` không trả:

```text
DISCORD_WEBHOOK
```

Secret chỉ trả trong `settings_()` dành cho Owner.

## 8. Khi thêm API mới

Ví dụ muốn thêm API `resetPassword`:

### Bước 1
Thêm case:

```javascript
case 'resetPassword': return resetPassword_(payload);
```

### Bước 2
Tạo function backend:

```javascript
function resetPassword_(p) {
  const actor = currentUser_(p.token);
  // kiểm tra quyền
  // tìm user
  // hash password
  // update sheet
  // audit
}
```

### Bước 3
Frontend:

```javascript
await call('resetPassword', {...});
```

### Bước 4
Cập nhật tài liệu API/action nếu cần.

## 9. Attendance logic

Check-in:

```text
current time
   ↓
SHIFT_START + GRACE
   ↓
PRESENT / LATE
   ↓
PointLogs
```

Mặc định:

```text
19:00 + 10 phút
19:10 → PRESENT
19:11 → LATE
```

Auto checkout:

```text
Attendance mở
   ↓
weeklyMaintenance()
   ↓
checkoutDeadline_()
   ↓
AUTO_CHECKOUT
```

## 10. Work-check logic

```text
Open attendance
      ↓
WAITING
      ↓
10 phút
   ↙       ↘
CONFIRMED  EXPIRED
```

`weeklyMaintenance()` chạy mỗi giờ.

Lưu ý Apps Script trigger có thể chạy lệch vài phút, nên code không dựa vào `minute === 0`.

## 11. Settings validation

`validateSettings_()` kiểm tra:

- time `HH:MM`;
- số điểm;
- weekday;
- URL http/https.

Không thêm setting mới vào frontend mà quên whitelist ở backend.

## 12. Deployment workflow

Sau khi sửa code:

1. Save project.
2. Chạy syntax check local nếu có thể.
3. Chạy một vài function test trong Apps Script.
4. Mở `Executions` kiểm tra lỗi.
5. Deploy version mới.
6. Test `/exec`.

### Deploy

```text
Deploy
→ Manage deployments
→ Edit
→ New version
→ Deploy
```

## 13. Test tối thiểu sau mỗi release

### Auth

- Owner login
- Staff login
- disabled user login phải fail

### Attendance

- check-in
- duplicate check-in phải fail
- check-out
- duplicate check-out phải fail
- late status

### Leave

- create leave
- invalid date phải fail
- admin approve
- approve twice phải fail

### Permission

- Staff không mở Users API
- Staff không mở Settings API
- Dev không cộng điểm
- Admin không cấp Owner
- Admin không đổi Active/Disabled
- Owner thao tác được tất cả chức năng được thiết kế

### Audit

- login có log
- check-in có log
- check-out có log
- user changes có log
- points có log
- leave review có log
- settings change có log

## 14. Static checks

Frontend:

```bash
# tách phần <script> của Index.html thành /tmp/index_script.js
node --check /tmp/index_script.js
```

Backend:

```bash
cp Code.gs /tmp/Code.gs.js
node --check /tmp/Code.gs.js
```

Lưu ý: `node --check` chỉ kiểm tra syntax; Google Apps Script APIs vẫn cần test trong môi trường Apps Script.

## 15. Những thứ không nên làm

Không:

- xóa `DB_ID` nếu chưa hiểu tác động;
- xóa sheet production;
- đổi tên cột tùy ý;
- copy `passwordHash` từ user này sang user khác;
- đưa Discord Webhook vào `publicSettings_()`;
- tin role do client gửi lên;
- dùng `innerHTML` với dữ liệu user mà không escape;
- tạo nhiều trigger `weeklyMaintenance`.

## 16. Khi người dùng báo lỗi

DEV nên lấy đủ:

```text
1. Role user
2. Tên màn hình
3. Thao tác vừa bấm
4. Error message
5. Thời gian xảy ra
6. Execution log
```

Sau đó đối chiếu `TROUBLESHOOTING.md`.

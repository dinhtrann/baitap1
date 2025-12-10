# Phân Tích PRD - Hệ Thống Quản Lý Thư Viện

## Tổng Quan
Hệ thống quản lý thư viện web-based với 3 vai trò chính:
- **Độc giả (Reader)**: Mượn sách, xem lịch sử, thanh toán phạt
- **Nhân viên thư viện (Librarian)**: Quản lý sách, xác nhận mượn/trả, theo dõi phạt
- **Quản lý viên (Admin)**: Quản lý tài khoản, báo cáo, cài đặt

---

## DANH SÁCH CÁC TÍNH NĂNG

### 2.1 QUẢN LÝ TÀI KHOẢN

#### 2.1.1 Đăng Ký
**Actor:** Mọi người  
**Mã số:** 2.1.1

**Primary Flow:**
1. User click "Đăng ký"
2. Nhập: Email, Tên, Mật khẩu, Confirm mật khẩu
3. Hệ thống validate dữ liệu
4. Tạo tài khoản với trạng thái "Chờ xác nhận"
5. Hiển thị thông báo thành công

**Alternative Flows:**
- User hủy bỏ form đăng ký

**Error/Edge Cases:**
- Email không đúng định dạng
- Email đã tồn tại trong hệ thống
- Tên để trống hoặc quá dài (>50 ký tự)
- Mật khẩu không đủ độ dài (<8 hoặc >16 ký tự)
- Confirm mật khẩu không khớp với mật khẩu
- Lỗi kết nối database khi tạo tài khoản

---

#### 2.1.2 Đăng Nhập
**Actor:** Mọi người  
**Mã số:** 2.1.2

**Primary Flow:**
1. User nhập Email & Mật khẩu
2. Hệ thống xác thực thông tin
3. Tạo JWT token (thời hạn 24h)
4. Chuyển hướng đến dashboard theo vai trò (Reader/Librarian/Admin)

**Alternative Flows:**
- User quên mật khẩu (chưa có trong PRD nhưng có thể có)

**Error/Edge Cases:**
- Email không đúng định dạng
- Email không tồn tại trong hệ thống
- Mật khẩu sai
- Tài khoản bị vô hiệu hóa
- Tài khoản chưa được xác nhận
- Lỗi tạo JWT token
- Lỗi kết nối database

---

#### 2.1.3 Hồ Sơ Cá Nhân
**Actor:** Độc giả  
**Mã số:** 2.1.3

**Primary Flow:**
1. Độc giả đăng nhập
2. Truy cập trang "Hồ sơ cá nhân"
3. Xem thông tin: Tên, Email, Số điện thoại, Địa chỉ, Ngày tham gia, Số lần mượn, Tổng số tiền phạt
4. Có thể cập nhật thông tin hoặc thay đổi mật khẩu

**Alternative Flows:**
- Cập nhật thông tin cá nhân
- Thay đổi mật khẩu

**Error/Edge Cases:**
- Chưa đăng nhập (redirect về trang login)
- Không phải vai trò độc giả (unauthorized)
- Tên để trống hoặc quá dài
- Số điện thoại không đúng định dạng
- Địa chỉ để trống hoặc quá dài (>255 ký tự)
- Mật khẩu cũ sai khi đổi mật khẩu
- Mật khẩu mới không đủ độ dài

---

### 2.2 QUẢN LÝ SÁCH

#### 2.2.1 Quản lý thể loại sách
**Actor:** Nhân viên thư viện  
**Mã số:** 2.2.1

**Primary Flow:**
1. Nhân viên đăng nhập với vai trò Librarian
2. Truy cập trang "Quản lý thể loại sách"
3. Xem danh sách thể loại ở dạng bảng
4. Có thể sửa trực tiếp trên bảng, thêm mới, hoặc xóa

**Alternative Flows:**
- Sửa thể loại trực tiếp trên bảng
- Thêm thể loại mới: Click "Thêm thể loại sách" → Nhập tên → Lưu
- Xóa thể loại: Click "Xóa" → Xác nhận → Xóa (nếu không có sách nào thuộc thể loại đó)

**Error/Edge Cases:**
- Chưa đăng nhập
- Không phải vai trò Librarian
- Tên thể loại để trống hoặc quá dài (>50 ký tự)
- Tên thể loại đã tồn tại
- Xóa thể loại đang có sách thuộc thể loại đó
- Lỗi kết nối database

---

#### 2.2.2 Thêm Sách Mới
**Actor:** Nhân viên thư viện  
**Mã số:** 2.2.2

**Primary Flow:**
1. Nhân viên click "Thêm Sách Mới"
2. Nhập: Tên sách, Tác giả, Năm xuất bản, ISBN, Thể loại, Mô tả, Số lượng bản sách
3. Hệ thống validate và lưu vào database với trạng thái "Có sẵn"
4. Hiển thị thông báo "Thêm sách thành công"

**Alternative Flows:**
- User hủy bỏ form

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Librarian
- Tên sách để trống hoặc quá dài (>100 ký tự)
- Tác giả để trống hoặc quá dài (>100 ký tự)
- ISBN không đúng định dạng (ISBN-10 hoặc ISBN-13)
- Năm xuất bản không hợp lệ (<1900 hoặc >năm hiện tại)
- Thể loại không tồn tại
- Mô tả để trống hoặc quá dài (>255 ký tự)
- Số lượng <= 0 hoặc không phải số
- Lỗi kết nối database

---

#### 2.2.3 Xem Danh Sách Sách
**Actor:** Tất cả người dùng (không cần login)  
**Mã số:** 2.2.3

**Primary Flow:**
1. User truy cập trang "Danh sách sách"
2. Xem danh sách sách với thông tin: Tên sách, Tác giả, Năm xuất bản, Thể loại, Số lượng có sẵn, Số lượng đang mượn
3. Có thể tìm kiếm, lọc, sắp xếp, phân trang

**Alternative Flows:**
- Tìm kiếm theo tên sách hoặc tác giả
- Lọc theo thể loại
- Sắp xếp: Tên (A-Z), Năm xuất bản (Mới nhất), Lượt mượn (Phổ biến nhất)
- Phân trang: 10 sách/trang

**Error/Edge Cases:**
- Không có sách nào trong hệ thống
- Kết quả tìm kiếm trống
- Lỗi kết nối database
- Lỗi phân trang (trang không tồn tại)

---

#### 2.2.4 Xem Chi Tiết Sách
**Actor:** Tất cả người dùng (không cần login)  
**Mã số:** 2.2.4

**Primary Flow:**
1. User click vào một cuốn sách từ danh sách
2. Xem chi tiết: Tên, Tác giả, ISBN, Năm xuất bản, Mô tả chi tiết, Số lượng bản đang có, đang mượn
3. Nếu là nhân viên thư viện: Xem lịch sử mượn (Độc giả, Ngày mượn, Ngày hết hạn)
4. Nếu là độc giả: Có thể nhấn "Mượn sách"

**Alternative Flows:**
- Độc giả click "Mượn sách" (chuyển đến flow 2.3.1)

**Error/Edge Cases:**
- Sách không tồn tại (404)
- Lỗi kết nối database
- Không có quyền xem lịch sử mượn (nếu không phải Librarian)

---

#### 2.2.5 Sửa & Xóa Sách
**Actor:** Nhân viên thư viện  
**Mã số:** 2.2.5

**Primary Flow (Sửa):**
1. Nhân viên xem chi tiết sách
2. Click "Sửa"
3. Chỉnh sửa thông tin
4. Click "Lưu"
5. Hệ thống cập nhật và hiển thị thông báo thành công

**Primary Flow (Xóa):**
1. Nhân viên xem chi tiết sách
2. Click "Xóa"
3. Xác nhận xóa trong modal
4. Hệ thống kiểm tra điều kiện (không có đơn mượn hoạt động)
5. Xóa sách và hiển thị thông báo thành công

**Alternative Flows:**
- Hủy bỏ form sửa
- Hủy bỏ xác nhận xóa

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Librarian
- Sách không tồn tại
- Validation errors khi sửa (tương tự như thêm sách)
- Xóa sách đang có đơn mượn hoạt động
- Lỗi kết nối database

---

### 2.3 QUẢN LÝ MƯỢN SÁCH

#### 2.3.1 Mượn Sách (Độc Giả)
**Actor:** Độc giả  
**Mã số:** 2.3.1

**Primary Flow:**
1. Độc giả đăng nhập
2. Xem chi tiết sách
3. Click "Mượn Sách"
4. Chọn thời hạn mượn (mặc định 14 ngày, tối đa 30 ngày)
5. Hệ thống kiểm tra điều kiện
6. Tạo đơn mượn ở trạng thái "Chờ xác nhận"
7. Hiển thị thông báo thành công

**Alternative Flows:**
- Hủy bỏ form mượn sách

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Reader
- Sách không còn sẵn (số lượng = 0)
- Độc giả đã mượn quá số sách tối đa (5 cuốn)
- Độc giả có khoản phạt chưa thanh toán
- Thời hạn mượn không hợp lệ (<1 hoặc >30 ngày)
- Lỗi kết nối database

---

#### 2.3.2 Mượn Sách (Nhân Viên Thư Viện)
**Actor:** Nhân viên thư viện  
**Mã số:** 2.3.2

**Primary Flow:**
1. Nhân viên đăng nhập với vai trò Librarian
2. Truy cập trang "Quản lý mượn trả" → Tab "Chờ xác nhận"
3. Xem danh sách đơn mượn chờ xác nhận
4. Chọn một đơn mượn và có 2 lựa chọn:
   - **Xác nhận:** Click "Xác nhận" → Cập nhật trạng thái thành "Đã mượn", giảm số lượng sách có sẵn
   - **Từ chối:** Click "Từ chối" → Nhập lý do từ chối → Cập nhật trạng thái thành "Bị từ chối"

**Alternative Flows:**
- Hủy bỏ modal từ chối

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Librarian
- Đơn mượn không tồn tại
- Đơn mượn không ở trạng thái "Chờ xác nhận"
- Sách đã hết (khi xác nhận)
- Lý do từ chối để trống (khi từ chối)
- Lỗi cập nhật database

---

#### 2.3.3 Xem Lịch Sử Mượn Sách
**Actor:** Độc giả  
**Mã số:** 2.3.3

**Primary Flow:**
1. Độc giả đăng nhập
2. Truy cập trang "Lịch sử mượn sách"
3. Xem các tab:
   - Sách đang mượn: Tên, Tác giả, Ngày mượn, Hạn trả, Số ngày còn lại
   - Sách đã trả: Tên, Ngày mượn, Ngày trả
   - Sách bị từ chối: Tên, Tác giả, Lý do từ chối
4. Có thể lọc theo trạng thái, gia hạn sách, hoặc tạo yêu cầu trả sách

**Alternative Flows:**
- Lọc theo trạng thái
- Xem lý do từ chối
- Gia hạn sách: Click "Gia hạn" (+7 ngày, tối đa 1 lần, chỉ khi chưa hết hạn)
- Tạo yêu cầu trả sách (chuyển đến flow 2.4.1)

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Reader
- Không có lịch sử mượn
- Sách đã hết hạn không thể gia hạn
- Đã gia hạn rồi (chỉ được gia hạn 1 lần)
- Lỗi kết nối database

---

### 2.4 TRẢ SÁCH

#### 2.4.1 Yêu Cầu Trả Sách
**Actor:** Độc giả  
**Mã số:** 2.4.1

**Primary Flow:**
1. Độc giả đăng nhập
2. Truy cập trang "Lịch sử mượn sách"
3. Xem danh sách sách đang mượn
4. Click "Xin trả sách" trên sách muốn trả
5. Xác nhận trong modal
6. Hệ thống tạo yêu cầu trả sách ở trạng thái "Chờ xác nhận"

**Alternative Flows:**
- Hủy bỏ modal xác nhận

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Reader
- Sách không ở trạng thái "Đang mượn"
- Đã có yêu cầu trả sách ở trạng thái "Chờ xác nhận" cho đơn mượn này
- Lỗi kết nối database

---

#### 2.4.2 Xác Nhận Trả Sách
**Actor:** Nhân viên thư viện  
**Mã số:** 2.4.2

**Primary Flow:**
1. Nhân viên đăng nhập với vai trò Librarian
2. Truy cập trang "Quản lý mượn trả" → Tab "Chờ xác nhận trả"
3. Xem danh sách yêu cầu trả sách chờ xác nhận
4. Nhận sách vật lý từ độc giả
5. Click "Xác nhận trả" và chọn tình trạng sách:
   - **Bình thường:** Xác nhận → Cập nhật đơn mượn thành "Đã trả", tăng sách có sẵn, giảm sách đang mượn
   - **Hư hỏng:** Chọn mức phạt, nhập ghi chú → Kiểm tra trả muộn → Tạo phiếu phạt hư hỏng và trả muộn (nếu có) → Cập nhật đơn thành "Đã trả"
   - **Mất:** Chọn mức phạt, nhập ghi chú → Tạo phiếu phạt → Cập nhật đơn thành "Đã trả"

**Alternative Flows:**
- Hủy bỏ modal xác nhận

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Librarian
- Yêu cầu trả sách không tồn tại
- Yêu cầu không ở trạng thái "Chờ xác nhận"
- Tình trạng sách chưa được chọn
- Mức phạt chưa được chọn (khi hư hỏng hoặc mất)
- Ghi chú để trống hoặc quá dài (>500 ký tự) (khi hư hỏng hoặc mất)
- Mức phạt không tồn tại
- Lỗi tạo phiếu phạt
- Lỗi cập nhật database

---

### 2.5 QUẢN LÝ NỢ & PHẠT

#### 2.5.1 Quản lý mức phạt
**Actor:** Quản lý viên  
**Mã số:** 2.5.1

**Primary Flow:**
1. Quản lý viên đăng nhập với vai trò Admin
2. Truy cập trang "Quản lý mức phạt"
3. Xem danh sách mức phạt ở dạng bảng
4. Có thể sửa trực tiếp trên bảng, thêm mới, hoặc xóa

**Alternative Flows:**
- Sửa mức phạt trực tiếp trên bảng
- Thêm mức phạt mới: Click "Thêm mức phạt" → Nhập thông tin (Tên, Số tiền, Ngày phạt) → Lưu
- Xóa mức phạt: Click "Xóa" → Xác nhận → Xóa

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Admin
- Tên mức phạt để trống hoặc quá dài (>25 ký tự)
- Số tiền <= 0 hoặc không phải số
- Ngày phạt không hợp lệ
- Mức phạt đang được sử dụng (có thể không cho xóa)
- Lỗi kết nối database

---

#### 2.5.2 Xem & Thanh Toán Khoản Phạt (Độc Giả)
**Actor:** Độc giả  
**Mã số:** 2.5.2

**Primary Flow:**
1. Độc giả đăng nhập
2. Truy cập trang "Khoản phạt của tôi"
3. Xem danh sách khoản phạt chưa thanh toán: Nguyên nhân phạt, Số tiền, Ngày phạt, Trạng thái
4. Chọn phiếu phạt ở trạng thái "Chưa thanh toán"
5. Thanh toán bằng chuyển khoản qua ngân hàng
6. Click "Đã thanh toán"
7. Phiếu phạt chuyển sang trạng thái "Chờ xác nhận"

**Alternative Flows:**
- Không có khoản phạt nào

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Reader
- Phiếu phạt không tồn tại
- Phiếu phạt không ở trạng thái "Chưa thanh toán"
- Lỗi cập nhật trạng thái

---

#### 2.5.3 Xem & Thanh Toán Khoản Phạt (Nhân Viên Thư Viện)
**Actor:** Nhân viên thư viện  
**Mã số:** 2.5.3

**Primary Flow:**
1. Nhân viên đăng nhập với vai trò Librarian
2. Truy cập trang "Quản lý khoản phạt"
3. Xem danh sách khoản phạt chưa thanh toán
4. Xem chi tiết khoản phạt
5. Có 2 lựa chọn:
   - **Xác nhận thanh toán:** Click "Đã thanh toán" → Cập nhật trạng thái thành "Đã thanh toán"
   - **Từ chối:** Click "Từ chối" → Nhập lý do từ chối → Cập nhật trạng thái thành "Từ chối"

**Alternative Flows:**
- Hủy bỏ modal từ chối

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Librarian
- Phiếu phạt không tồn tại
- Phiếu phạt không ở trạng thái "Chờ xác nhận"
- Lý do từ chối để trống (khi từ chối)
- Lỗi cập nối database

---

### 2.6 QUẢN LÝ NGƯỜI DÙNG

#### 2.6.1 Danh Sách Người Dùng
**Actor:** Quản lý viên  
**Mã số:** 2.6.1

**Primary Flow:**
1. Quản lý viên đăng nhập với vai trò Admin
2. Truy cập trang "Quản lý người dùng"
3. Xem danh sách: Email, Tên, Vai trò (Reader/Librarian/Admin), Ngày tham gia, Trạng thái (Kích hoạt/Vô hiệu hóa)
4. Có thể tìm kiếm, lọc, vô hiệu hóa/kích hoạt tài khoản

**Alternative Flows:**
- Tìm kiếm theo email/tên
- Lọc theo vai trò
- Vô hiệu hóa/Kích hoạt tài khoản: Click nút tương ứng → Xác nhận → Cập nhật trạng thái

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Admin
- User không tồn tại
- Không thể vô hiệu hóa chính mình
- Lỗi cập nhật trạng thái

---

#### 2.6.2 Gán Vai Trò
**Actor:** Quản lý viên  
**Mã số:** 2.6.2

**Primary Flow:**
1. Quản lý viên đăng nhập với vai trò Admin
2. Truy cập trang "Quản lý người dùng"
3. Chọn một user từ danh sách
4. Click "Gán vai trò" hoặc "Sửa vai trò"
5. Chọn vai trò mới: Reader, Librarian, hoặc Admin
6. Xác nhận
7. Hệ thống cập nhật vai trò của user

**Alternative Flows:**
- Hủy bỏ modal gán vai trò

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không phải Admin
- User không tồn tại
- Vai trò không hợp lệ
- Không thể thay đổi vai trò của chính mình (có thể)
- Lỗi cập nhật database

---

### 2.7 BÁO CÁO & THỐNG KÊ

#### 2.7.1 Báo Cáo Tổng Quan (Dashboard)
**Actor:** Quản lý viên, Nhân viên thư viện  
**Mã số:** 2.7.1

**Primary Flow:**
1. User đăng nhập với vai trò Admin hoặc Librarian
2. Truy cập trang Dashboard
3. Xem các thống kê:
   - Tổng số sách: Có sẵn / Đang mượn / Bị mất / Hư hỏng
   - Tổng số độc giả: Hoạt động / Vô hiệu hóa
   - Tổng đơn mượn hôm nay
   - Top 5 sách phổ biến nhất
   - Danh sách độc giả nợ quá hạn

**Alternative Flows:**
- Click vào từng thống kê để xem chi tiết

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không có quyền (không phải Admin/Librarian)
- Lỗi truy vấn database
- Không có dữ liệu để hiển thị

---

#### 2.7.2 Báo Cáo Chi Tiết
**Actor:** Quản lý viên, Nhân viên thư viện  
**Mã số:** 2.7.2

**Primary Flow:**
1. User đăng nhập với vai trò Admin hoặc Librarian
2. Truy cập trang "Báo cáo chi tiết"
3. Chọn loại báo cáo:
   - Báo cáo Sách: Tổng số sách, tình trạng, số lần mượn
   - Báo cáo Mượn Trả: Số lần mượn/trả theo ngày/tháng/quý
   - Báo cáo Phạt: Tổng doanh thu phạt, người nợ ngoài hạn
   - Báo cáo Sách Mất/Hư: Danh sách sách cần thay thế
4. Chọn khoảng thời gian (Ngày, Tuần, Tháng, Quý, Năm)
5. Xem báo cáo
6. Có thể xuất ra CSV

**Alternative Flows:**
- Xuất báo cáo ra CSV: Click "Xuất CSV" → Tải file

**Error/Edge Cases:**
- Chưa đăng nhập hoặc không có quyền
- Khoảng thời gian không hợp lệ
- Không có dữ liệu trong khoảng thời gian đã chọn
- Lỗi truy vấn database
- Lỗi xuất file CSV

---

## TỔNG KẾT

**Tổng số tính năng:** 20 tính năng

**Phân loại theo vai trò:**
- **Không cần đăng nhập:** 2 tính năng (2.2.3, 2.2.4)
- **Độc giả:** 7 tính năng (2.1.3, 2.3.1, 2.3.3, 2.4.1, 2.5.2)
- **Nhân viên thư viện:** 8 tính năng (2.2.1, 2.2.2, 2.2.5, 2.3.2, 2.4.2, 2.5.3, 2.7.1, 2.7.2)
- **Quản lý viên:** 5 tính năng (2.5.1, 2.6.1, 2.6.2, 2.7.1, 2.7.2)

**Giai đoạn triển khai:** Đã được xác định trong PRD (6 giai đoạn)


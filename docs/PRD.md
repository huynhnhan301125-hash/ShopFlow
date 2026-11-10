# ShopFlow — Product Requirements Document (PRD)

**Loại tài liệu:** PRD đồ án học tập
**Phiên bản:** 1.0
**Mô hình:** Ứng dụng mua sắm B2C, một cửa hàng duy nhất

## 1. Tổng quan

### 1.1. Bài toán

Mua sắm trực tiếp có thể tốn thời gian tìm cửa hàng, di chuyển, tìm sản phẩm và chờ thanh toán. Trải nghiệm còn phụ thuộc vào giao thông, thời tiết, giờ mở cửa và tình trạng tồn kho. Khách hàng cũng có thể quên mang tiền hoặc chưa quyết định mua ngay một sản phẩm.

### 1.2. Giải pháp và giá trị

ShopFlow cho phép khách hàng xem, tìm kiếm và chọn sản phẩm trực tuyến; quản lý giỏ hàng; đặt hàng và thanh toán theo các phương thức được hỗ trợ. Thông tin sản phẩm và đánh giá giúp khách hàng tham khảo trước khi mua. Giao diện hướng đến sự trực quan, dễ dùng và phù hợp với ứng dụng di động.

### 1.3. Giả định và giới hạn của đồ án

- **Mô hình kinh doanh:** Một cửa hàng duy nhất (Single Merchant); không phải sàn thương mại điện tử và không có người bán bên thứ ba.
- **Tiền tệ và vận chuyển:** Sử dụng Việt Nam đồng (VNĐ); miễn phí vận chuyển trong phạm vi đồ án.
- **Ứng dụng cửa hàng:** Bản v1 tập trung vào ứng dụng khách hàng, không xây dựng giao diện quản trị. Các trạng thái vận hành của cửa hàng được mô phỏng.
- **Nguồn dữ liệu:** Danh mục sản phẩm và dữ liệu đánh giá đọc từ DummyJSON. Đăng ký/đăng nhập, giỏ hàng, đơn hàng, ví giả lập và tồn kho được xử lý bằng dịch vụ hoặc dữ liệu mô phỏng của đồ án.
- **Đánh giá:** Đánh giá và nhận xét có sẵn chỉ để xem. Viết đánh giá mới là tính năng Should Have, chưa thuộc bản v1.
- **Thanh toán:** COD và Ví giả lập chỉ phục vụ mô phỏng; không xử lý giao dịch tiền thật.

## 2. Phạm vi sản phẩm

| Must Have | Should Have | Could Have | Won’t Have |
| --- | --- | --- | --- |
| Tài khoản: đăng ký, đăng nhập, duy trì phiên và đăng xuất. | Bộ lọc theo danh mục, giá, hãng, đánh giá và mức độ bán chạy. | Chuyển đổi giao diện sáng/tối. | Thanh toán bằng thẻ quốc tế như Visa hoặc Mastercard. |
| Khám phá sản phẩm: danh sách phân trang, tìm kiếm và trang chi tiết. | Người dùng viết đánh giá sản phẩm. | Chuyển đổi ngôn ngữ. | Liên hệ trực tiếp với người bán, đơn vị giao hàng hoặc shipper. |
| Giỏ hàng: thêm, sửa số lượng, xóa và chuyển sang đặt hàng. |  |  | Định vị GPS. |
| Đặt hàng và thanh toán bằng COD hoặc Ví giả lập. |  |  | Nhiều người bán hoặc mô hình sàn thương mại điện tử. |
| Chống tạo đơn trùng bằng Idempotency Token và trạng thái xử lý trên giao diện. |  |  |  |
| Xem lịch sử, chi tiết và trạng thái đơn hàng; hủy đơn theo điều kiện. |  |  |  |

## 3. Quy tắc nghiệp vụ

### 3.1. Trạng thái đơn hàng và thanh toán

Trạng thái đơn hàng và trạng thái thanh toán là hai thông tin độc lập. Vì vậy, đơn COD có thể ở trạng thái **Chờ xác nhận** nhưng vẫn **Chưa thanh toán**.

**Trạng thái đơn hàng**

| Trạng thái | Ý nghĩa |
| --- | --- |
| Chờ thanh toán | Đơn Ví giả lập đã tạo và đang chờ thanh toán trong thời hạn 10 phút. |
| Chờ xác nhận | Đơn đã được tạo hoặc thanh toán thành công, đang chờ xử lý. COD vẫn chưa thanh toán ở trạng thái này. |
| Chuẩn bị đơn hàng | Đơn đang được chuẩn bị để giao. |
| Đang giao hàng | Đơn đã bàn giao cho đơn vị giao hàng mô phỏng. |
| Hoàn thành | Khách đã nhận hàng; COD được ghi nhận thanh toán khi thu tiền thành công. |
| Đã hủy | Đơn đã bị hủy bởi khách hoặc do hết thời hạn thanh toán. |

**Trạng thái thanh toán:** Chưa thanh toán (Unpaid), Đã thanh toán (Paid), Đã hoàn tiền (Refunded).

| Phương thức và sự kiện | Trạng thái đơn hàng | Trạng thái thanh toán | Xử lý tồn kho |
| --- | --- | --- | --- |
| COD — tạo đơn thành công | Chờ xác nhận | Chưa thanh toán | Giữ số lượng đã đặt. |
| COD — bàn giao vận chuyển | Đang giao hàng | Chưa thanh toán | Chốt số lượng đã giữ; giảm tồn kho thực tế một lần. |
| COD — giao hàng và thu tiền thành công | Hoàn thành | Đã thanh toán | Không trừ tồn kho lần nữa. |
| COD — hủy trước khi bàn giao | Đã hủy | Chưa thanh toán | Giải phóng số lượng đã giữ. |
| Ví giả lập — tạo đơn thành công | Chờ thanh toán | Chưa thanh toán | Giữ số lượng đã đặt tối đa 10 phút. |
| Ví giả lập — thanh toán thành công | Chờ xác nhận | Đã thanh toán | Tiếp tục giữ số lượng đến khi bàn giao vận chuyển. |
| Ví giả lập — hết hạn hoặc hủy trước khi trả tiền | Đã hủy | Chưa thanh toán | Giải phóng số lượng đã giữ. |
| Ví giả lập — hủy sau khi trả tiền, trước khi chuẩn bị đơn | Đã hủy | Đã hoàn tiền | Hoàn tiền vào ví và giải phóng số lượng đã giữ. |

### 3.2. Quy tắc tồn kho

- **Tồn kho thực tế (Physical Stock):** Số lượng hàng hiện có trong kho.
- **Số lượng tạm giữ (Reserved Stock):** Số lượng đã dành cho các đơn chưa bàn giao vận chuyển.
- **Số lượng khả dụng (Available Stock):** `Available Stock = Physical Stock - Reserved Stock`.
- Khi tạo đơn, hệ thống kiểm tra số lượng khả dụng và giữ số lượng mua cho cả COD và Ví giả lập. Không tạo đơn nếu số lượng yêu cầu vượt quá lượng khả dụng.
- Khi bàn giao đơn cho vận chuyển, hệ thống giảm tồn kho thực tế và số lượng tạm giữ đúng một lần.
- Khi đơn bị hủy trước khi bàn giao, hệ thống chỉ giải phóng số lượng tạm giữ; không cộng thêm vào tồn kho thực tế.
- Các thao tác giữ, chốt hoặc giải phóng tồn kho phải được xử lý nhất quán để tránh bán vượt tồn hoặc cập nhật kho hai lần.

## 4. Yêu cầu chức năng và tiêu chí nghiệm thu

### 4.1. Tài khoản và xác thực

#### 4.1.1. Đăng nhập thành công

**User story:** Là khách hàng, tôi muốn đăng nhập bằng tài khoản và mật khẩu để mua hàng, truy cập giỏ hàng và xem lịch sử đơn.

- **Given:** Người dùng đã có tài khoản và đang ở màn hình đăng nhập.
- **When:** Người dùng nhập đúng tài khoản, mật khẩu và chọn **Đăng nhập**.
- **Then:** Nút chuyển sang trạng thái đang tải và bị vô hiệu hóa trong lúc gửi yêu cầu. Khi đăng nhập thành công, phiên được lưu an toàn và người dùng được chuyển đến trang chủ.

#### 4.1.2. Đăng nhập thất bại

- **Thiếu thông tin:** Nếu tài khoản hoặc mật khẩu để trống, hiển thị cảnh báo tại trường tương ứng và không gửi yêu cầu lên máy chủ.
- **Sai thông tin:** Hiển thị thông báo “Tài khoản hoặc mật khẩu không chính xác. Vui lòng thử lại!”. Nút đăng nhập hiển thị trạng thái đang tải và bị vô hiệu hóa trong lúc chờ phản hồi.
- **Mất kết nối:** Hiển thị “Không thể kết nối đến máy chủ. Vui lòng kiểm tra lại đường truyền mạng”.
- **Chưa có tài khoản:** Chọn **Đăng ký** sẽ mở màn hình đăng ký.

#### 4.1.3. Đăng ký

**User story:** Là khách hàng, tôi muốn tạo tài khoản để sử dụng ứng dụng mua sắm.

- **Given:** Người dùng đang ở màn hình đăng ký.
- **When:** Người dùng điền tên đăng nhập, mật khẩu và xác nhận mật khẩu hợp lệ rồi chọn **Đăng ký**.
- **Then:** Hệ thống kiểm tra dữ liệu; mật khẩu được băm một chiều trước khi lưu. Nếu thành công, người dùng được chuyển về màn hình đăng nhập. Nếu thất bại, hiển thị lỗi phù hợp để người dùng sửa thông tin.

#### 4.1.4. Đăng xuất

**User story:** Là người dùng, tôi muốn đăng xuất khỏi tài khoản hiện tại.

- **Given:** Người dùng đang ở phần cài đặt.
- **When:** Người dùng chọn **Đăng xuất**.
- **Then:** Ứng dụng xóa token và dữ liệu phiên trên thiết bị, sau đó chuyển về màn hình đăng nhập.

### 4.2. Khám phá và tìm kiếm sản phẩm

#### 4.2.1. Danh sách sản phẩm và phân trang

**User story:** Là khách hàng, tôi muốn xem sản phẩm dạng lưới và tải thêm sản phẩm khi cuộn để duyệt danh sách mượt mà.

- **Given:** Người dùng mở trang chủ hoặc cuộn gần cuối danh sách.
- **When:** Ứng dụng tải trang dữ liệu đầu tiên hoặc trang tiếp theo.
- **Then:** Hiển thị trạng thái đang tải phù hợp. Khi tải xong, mỗi sản phẩm có ảnh, tên, giá, số sao và số lượt mua.

#### 4.2.2. Tìm kiếm theo từ khóa

**User story:** Là khách hàng, tôi muốn tìm sản phẩm bằng tên hoặc từ khóa.

- **Given:** Người dùng đang ở thanh tìm kiếm.
- **When:** Người dùng nhập từ khóa; ứng dụng hiển thị gợi ý phù hợp. Người dùng chọn gợi ý hoặc gửi từ khóa để xem kết quả.
- **Then:** Hiển thị danh sách sản phẩm phù hợp. Nếu không có kết quả, hiển thị “Sản phẩm bạn tìm kiếm không tồn tại”.

#### 4.2.3. Chi tiết sản phẩm

**User story:** Là khách hàng, tôi muốn xem thông tin chi tiết trước khi quyết định mua.

- **Given:** Người dùng đang xem danh sách sản phẩm.
- **When:** Người dùng chọn một sản phẩm.
- **Then:** Hiển thị tên, ảnh, giá, số lượt mua, điểm đánh giá, mô tả và đánh giá có sẵn; cung cấp nút **Thêm vào giỏ hàng** và **Mua ngay**.

### 4.3. Giỏ hàng và đặt hàng

#### 4.3.1. Thao tác với giỏ hàng

**User story:** Là khách hàng, tôi muốn thêm, sửa số lượng và xóa sản phẩm trong giỏ hàng.

- Khi tăng hoặc giảm số lượng, ứng dụng kiểm tra tồn kho khả dụng và cập nhật số lượng cùng tổng tiền nếu hợp lệ.
- Nếu sản phẩm không còn đủ hàng, hiển thị “Sản phẩm đã được bán hết” và không cập nhật số lượng vượt tồn.
- Nếu giỏ hàng trống, hiển thị thông báo “Giỏ hàng của bạn đang trống” cùng nút **Mua sắm ngay** để về trang chủ.
- Chọn mua hàng trong giỏ sẽ mở màn hình xác nhận đơn hàng.

#### 4.3.2. Xác nhận và tạo đơn

**User story:** Là khách hàng, tôi muốn đặt hàng từ giỏ hàng hoặc mua ngay.

- **Given:** Khách hàng chọn mua từ giỏ hàng hoặc màn hình chi tiết sản phẩm.
- **When:** Người dùng xác nhận họ tên, số điện thoại, địa chỉ giao hàng; chọn COD hoặc Ví giả lập; rồi chọn **Đặt hàng**.
- **Then:**
    - Kiểm tra dữ liệu giao hàng và tồn kho khả dụng trước khi tạo đơn.
    - Khóa nút gửi trong lúc xử lý. Tạo một Idempotency Token duy nhất cho lần đặt hàng; nếu yêu cầu được gửi lại do lỗi mạng, hệ thống trả về kết quả của lần gửi trước thay vì tạo đơn trùng.
    - Lưu ảnh, tên, giá và số lượng sản phẩm tại thời điểm đặt vào chi tiết đơn (`order_items`) để lịch sử đơn không đổi nếu thông tin sản phẩm thay đổi.
    - Giữ số lượng hàng tương ứng theo quy tắc tồn kho ở mục 3.2.

**COD**

- Tạo đơn với trạng thái **Chờ xác nhận** và thanh toán **Chưa thanh toán**.
- Sau khi tạo đơn thành công, xóa các sản phẩm đã đặt khỏi giỏ hàng và hiển thị màn hình đặt hàng thành công.
- Khi đơn được giao và thu tiền thành công, cập nhật đơn **Hoàn thành** và thanh toán **Đã thanh toán**.

**Ví giả lập**

- Tạo đơn với trạng thái **Chờ thanh toán**, thanh toán **Chưa thanh toán** và bắt đầu thời hạn giữ hàng 10 phút.
- Nếu thanh toán thành công trong thời hạn, cập nhật thanh toán **Đã thanh toán**, đơn **Chờ xác nhận**, rồi xóa sản phẩm đã đặt khỏi giỏ hàng.
- Nếu người dùng hủy hoặc hết 10 phút mà chưa thanh toán, cập nhật đơn **Đã hủy**, giải phóng số lượng tạm giữ và không ghi nhận thanh toán.
- Nếu thanh toán thành công nhưng đơn bị hủy trước khi chuẩn bị, hoàn đúng số tiền về Ví giả lập và cập nhật thanh toán **Đã hoàn tiền**.
- Mỗi tài khoản thử nghiệm có số dư giả lập ban đầu 5.000.000 VNĐ. Nếu số dư không đủ, hiển thị “Số dư ví không đủ để thanh toán”; đơn vẫn ở trạng thái **Chờ thanh toán** trong thời hạn còn lại. Bản v1 không bao gồm chức năng nạp tiền.
- Nếu mất mạng khi thanh toán, hiển thị lỗi kết nối. Người dùng có thể kiểm tra trạng thái đơn và thử thanh toán lại trong thời hạn; không ghi nhận thanh toán hai lần.

#### 4.3.3. Mặt hàng hết

- **Given:** Sản phẩm trong giỏ hoặc trên trang chi tiết không còn đủ số lượng khả dụng.
- **When:** Người dùng chọn **Mua ngay** hoặc **Đặt hàng**.
- **Then:** Hiển thị “Mặt hàng đã hết”, vô hiệu hóa thao tác đặt mua và không tạo đơn.

### 4.4. Hủy đơn hàng

**User story:** Là khách hàng, tôi muốn hủy đơn khi không còn nhu cầu mua.

- Nút **Hủy đơn** chỉ hiển thị khi đơn ở trạng thái **Chờ thanh toán** hoặc **Chờ xác nhận**. Không cho khách tự hủy sau khi đơn chuyển sang **Chuẩn bị đơn hàng** hoặc **Đang giao hàng**.
- Khi chọn hủy, hiển thị lý do như “Không còn nhu cầu”, “Có giá tốt hơn”, “Thay đổi thông tin giao hàng” hoặc “Khác”, cùng lựa chọn xác nhận hay quay lại.
- Đơn COD chưa thanh toán khi bị hủy sẽ giữ trạng thái thanh toán **Chưa thanh toán** và giải phóng lượng hàng đã giữ.
- Đơn Ví giả lập đã thanh toán khi bị hủy trước khi chuẩn bị sẽ được hoàn tiền, chuyển sang **Đã hoàn tiền** và giải phóng lượng hàng đã giữ.
- Đơn Ví giả lập chưa thanh toán bị hủy hoặc hết hạn sẽ không phát sinh hoàn tiền.

### 4.5. Lịch sử và chi tiết đơn hàng

**User story:** Là khách hàng đã đăng nhập, tôi muốn xem lại đơn hàng, trạng thái giao hàng và chi tiết sản phẩm đã đặt.

- Danh sách đơn có các nhóm: **Tất cả**, **Chờ thanh toán**, **Đang xử lý** (Chờ xác nhận/Chuẩn bị), **Đang giao**, **Hoàn thành** và **Đã hủy**.
- Chi tiết đơn hiển thị mã đơn, ngày đặt, sản phẩm, ảnh, số lượng, giá tại thời điểm đặt, phí vận chuyển, tổng tiền, phương thức và trạng thái thanh toán.
- Với đơn **Chờ thanh toán**, hiển thị thời gian còn lại và nút **Thanh toán ngay**.
- Với đơn **Chờ thanh toán** hoặc **Chờ xác nhận**, hiển thị **Hủy đơn** theo quy tắc tại mục 4.4.

## 5. Yêu cầu phi chức năng

### 5.1. Hiệu năng và trải nghiệm

- Trong điều kiện Wi-Fi/4G ổn định, thời gian tải màn hình và trả kết quả tìm kiếm mục tiêu không quá 2 giây.
- Danh sách sản phẩm cuộn mượt, mục tiêu 60 FPS trên thiết bị di động cấu hình tiêu chuẩn dùng để kiểm thử đồ án.
- Các nút gửi yêu cầu hiển thị trạng thái đang xử lý và không cho thao tác lặp trong thời gian chờ.

### 5.2. Bảo mật

- Mật khẩu phải được băm một chiều bằng BCrypt hoặc Argon2; không lưu mật khẩu dạng bản rõ.
- Access Token và Refresh Token được lưu bằng Flutter Secure Storage, sử dụng Android Keystore trên Android và iOS Keychain trên iOS.
- Khi đăng xuất, xóa token và dữ liệu phiên khỏi thiết bị.
- Chỉ sử dụng dữ liệu và ví giả lập; không thu thập thông tin thẻ hoặc xử lý thanh toán tiền thật.

### 5.3. Ngoại tuyến và bộ nhớ đệm

- Khi mất Internet, người dùng vẫn có thể xem danh sách, chi tiết sản phẩm và giỏ hàng đã được tải trước đó từ bộ nhớ đệm.
- Các thao tác cần đồng bộ như đăng ký, đăng nhập, đặt hàng, thanh toán và thay đổi số lượng giỏ hàng bị chặn khi ngoại tuyến; hiển thị “Bạn đang ngoại tuyến. Vui lòng kiểm tra lại kết nối mạng”.
- Khi kết nối chưa ổn định sau thao tác đặt hàng hoặc thanh toán, ứng dụng phải cho phép kiểm tra trạng thái trước khi gửi lại để tránh tạo đơn hoặc thanh toán trùng.

## 6. Thuật ngữ

| Thuật ngữ | Ý nghĩa |
| --- | --- |
| COD | Thanh toán khi nhận hàng. |
| Ví giả lập (Mock Wallet) | Phương thức thanh toán mô phỏng, không dùng tiền thật. |
| Idempotency Token | Mã duy nhất giúp yêu cầu gửi lại không tạo thêm đơn hoặc thanh toán trùng. |
| Tồn kho tạm giữ (Reserved Stock) | Số lượng đã dành cho đơn hàng nhưng chưa bàn giao vận chuyển. |
| Tồn kho khả dụng (Available Stock) | Số lượng khách có thể đặt sau khi trừ lượng đang tạm giữ. |
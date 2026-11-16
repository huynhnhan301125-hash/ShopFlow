# ShopFlow — Từ điển Thuật ngữ Nghiệp vụ (GLOSSARY)

**Phiên bản:** 1.0

**Loại sản phẩm:** Hệ thống thương mại điện tử B2C — Một cửa hàng duy nhất (Single Merchant)

**Giai đoạn:** Phiên bản v1.0 (Nguyên mẫu kỹ thuật / Đồ án học tập)

Tài liệu này định nghĩa **Ngôn ngữ thống nhất (Ubiquitous Language)** dùng xuyên suốt đặc tả sản phẩm (`PRD.md`), thiết kế cơ sở dữ liệu (`DATABASE_DESIGN.md`) và mã nguồn của **ShopFlow**.

---

## 1. Quy ước đặt tên & Phạm vi kiến trúc v1

- **Quy ước định danh (Naming Convention):**
    - **Tài liệu nghiệp vụ:** `ShopFlow`.
    - **Repository, thư mục & cơ sở dữ liệu:** `shopflow`.
    - **Bảng và cột trong PostgreSQL:** `snake_case` viết thường toàn bộ (ví dụ: `order_items`, `reserved_stock`).
    - **Hằng số trạng thái & phân loại (Enum):** `UPPER_SNAKE_CASE` toàn bộ trên cả Database và API (ví dụ: `PENDING_PAYMENT`, `MOCK_WALLET`).
- **Kiến trúc bàn giao (Backend-first):** Phiên bản v1 ưu tiên hoàn thiện dịch vụ lõi (Backend API + PostgreSQL) độc lập trước khi tích hợp ứng dụng di động (Flutter Client).
- **Simulated Store & Courier Driver (Bộ kích hoạt vận hành mô phỏng):** Do v1 chưa xây dựng giao diện quản trị (Admin UI), các bước chuyển tiếp trạng thái phía cửa hàng và giao nhận (`AWAITING_CONFIRMATION` $\rightarrow$ `PROCESSING` $\rightarrow$ `SHIPPING` $\rightarrow$ `COMPLETED`) được kích hoạt thông qua nhóm API nội bộ phục vụ kiểm thử và demo.
- **Expiration Sweeper (Tác vụ quét đơn quá hạn):** Tiến trình chạy nền (Background Worker) quét định kỳ mỗi 30 giây dựa trên chỉ mục `idx_orders_expiration` để tìm các đơn `PENDING_PAYMENT` có `expires_at <= NOW()`, tự động chuyển sang `CANCELLED` và giải phóng `reserved_stock`.

---

## 2. Chủ thể & Xác thực (Actors & Identity)

- **Guest (Khách vãng lai):** Chủ thể truy cập hệ thống nhưng chưa xác thực danh tính; chỉ có quyền xem danh mục, tìm kiếm và đọc thông tin chi tiết sản phẩm.
- **User / Customer (Người dùng / Khách hàng):** Chủ thể đã đăng ký tài khoản và đăng nhập thành công (`users`), có quyền thao tác với giỏ hàng, đặt hàng và theo dõi đơn hàng. Trong phạm vi v1, mọi tài khoản đều là khách hàng và không phân tách vai trò quản trị viên (`role`).
- **Account (`users`):** Thực thể định danh và xác thực cốt lõi, gồm `username` duy nhất và mật khẩu đã băm một chiều (`password_hash` bằng BCrypt hoặc Argon2).
- **User Profile (`user_profiles`):** Thực thể lưu thông tin giao nhận cá nhân tách biệt khỏi tài khoản xác thực, gồm nhóm bắt buộc khi đặt hàng (`full_name`, `phone_number`, `address`) và nhóm tùy chọn (`email`, `date_of_birth`, `avatar_url`).
- **Lazy Profile Completion (`is_profile_completed`):** Cơ chế cho phép đăng ký nhanh chỉ với thông tin tài khoản (`is_profile_completed = FALSE`), nhưng chặn thao tác tạo đơn hàng cho đến khi người dùng bổ sung đủ 3 thông tin giao nhận bắt buộc (`is_profile_completed = TRUE`).
- **Session Tokens (`Access Token`, `Refresh Token`):** Cặp mã xác thực phiên làm việc do dịch vụ cấp phát khi đăng nhập, lưu an toàn tại phía client (Flutter Secure Storage / Keystore / Keychain), không lưu trong CSDL v1 và bị xóa cục bộ khi đăng xuất.

---

## 3. Sản phẩm & Quản lý Tồn kho (Catalog & Inventory)

- **Single Merchant (Một cửa hàng duy nhất):** Mô hình bán lẻ trực tiếp từ một đơn vị sở hữu duy nhất; loại trừ hoàn toàn người bán bên thứ ba (Third-party Seller) và các quy tắc chia đơn của sàn thương mại điện tử (Marketplace).
- **External Catalog Source (`DummyJSON` / `source_product_id`):** Nguồn dữ liệu ngoài dùng để nạp khởi tạo một lần (One-time Seed) thông tin gốc của sản phẩm vào bảng `products` và đọc trực tiếp danh sách đánh giá có sẵn theo mã ánh xạ 1-1 `source_product_id`. Dữ liệu nguồn không bao giờ ghi đè các trường tồn kho và lượt bán nội bộ.
- **Physical Stock (`physical_stock` — Tồn kho thực tế):** Tổng số lượng đơn vị hàng hóa vật lý hiện đang nằm trong kho của cửa hàng.
- **Reserved Stock (`reserved_stock` — Tồn kho tạm giữ):** Số lượng hàng hóa đã được khóa riêng cho các đơn hàng hợp lệ nhưng chưa bàn giao cho đơn vị vận chuyển (`0 <= reserved_stock <= physical_stock`).
- **Available Stock (Tồn kho khả dụng):** Số lượng hàng hóa thực sự còn có thể cho phép khách đặt mua mới:
    
    $$
    \text{Available Stock} = \text{physical\_stock} - \text{reserved\_stock}
    $$
    
- **Inventory Reservation / Soft Lock (Tạm giữ tồn kho):** Nghiệp vụ tăng `reserved_stock` ngay tại thời điểm tạo đơn thành công (áp dụng cho cả `COD` và `MOCK_WALLET`) nhằm ngăn chặn bán vượt tồn (Overselling). Với đơn `MOCK_WALLET`, thời hạn tạm giữ tối đa là **10 phút** (`expires_at`).
- **Inventory Commitment (Chốt tồn kho):** Nghiệp vụ giảm đồng thời `physical_stock` và `reserved_stock` đúng một lần khi đơn hàng chuyển sang trạng thái bàn giao vận chuyển (`SHIPPING`).
- **Inventory Release (Giải phóng tồn kho):** Nghiệp vụ giảm `reserved_stock` mà không làm thay đổi `physical_stock` khi đơn hàng bị hủy hợp lệ hoặc hết thời hạn thanh toán trước bước bàn giao vận chuyển.
- **Sold Count (`sold_count`):** Tổng số lượng đơn vị sản phẩm đã bàn giao cho đơn vị vận chuyển; chỉ ghi nhận tăng một lần tại thời điểm đơn chuyển sang `SHIPPING`.
- **Soft Delete (`deleted_at`):** Cơ chế ẩn sản phẩm ngừng kinh doanh khỏi danh mục bằng cách ghi nhận dấu thời gian thay vì xóa vật lý bản ghi, nhằm bảo toàn toàn vẹn tham chiếu cho lịch sử đơn hàng cũ.

---

## 4. Giỏ hàng, Đơn hàng & Toàn vẹn Giao dịch (Cart, Order & Transaction Integrity)

- **Cart (`carts` & `cart_items`):** Không gian lưu trữ tạm thời các sản phẩm dự định mua của một tài khoản (mỗi tài khoản có tối đa 1 giỏ hàng; mỗi sản phẩm xuất hiện tối đa 1 dòng trong giỏ theo `UNIQUE(cart_id, product_id)`). Thao tác trên giỏ hàng chỉ kiểm tra số lượng khả dụng và **không** tạm giữ tồn kho.
- **Order (`orders`) & Order Item (`order_items`):** `Order` là thực thể đại diện cho toàn bộ giao dịch đặt mua của khách hàng; `Order Item` là từng dòng mặt hàng cụ thể nằm trong đơn hàng đó.
- **Data Snapshot (Bản chụp dữ liệu tại thời điểm đặt):** Cơ chế sao chép cố định thông tin sản phẩm (`product_name`, `product_image_url`, `price`, `quantity` vào `order_items`) và thông tin người nhận (`recipient_name`, `recipient_phone`, `shipping_address` vào `orders`) lúc khởi tạo đơn để lịch sử giao dịch không bị biến đổi khi danh mục hoặc hồ sơ thay đổi về sau.
- **Idempotency Key (`idempotency_key`):** Chuỗi định danh duy nhất gắn với mỗi yêu cầu tạo đơn hoặc biến động số dư ví; đảm bảo các yêu cầu gửi lại (retry) do gián đoạn mạng chỉ trả về kết quả của giao dịch gốc thay vì tạo bản ghi trùng lặp.

---

## 5. Vòng đời Đơn hàng & Thanh toán (Order Lifecycle & Payment)

- **Order Status (`order_status` — Trạng thái xử lý đơn hàng):** Trục trạng thái phản ánh tiến độ xử lý và giao nhận vật lý của đơn hàng, vận hành độc lập với trạng thái thanh toán:
    - `PENDING_PAYMENT` *(Chờ thanh toán)*: Đơn `MOCK_WALLET` đã tạo, đang tạm giữ tồn kho trong giới hạn 10 phút chờ thanh toán.
    - `AWAITING_CONFIRMATION` *(Chờ xác nhận)*: Đơn `COD` vừa khởi tạo hoặc đơn `MOCK_WALLET` đã thanh toán thành công, đang chờ cửa hàng xử lý.
    - `PROCESSING` *(Chuẩn bị đơn hàng)*: Cửa hàng đang đóng gói; từ mốc trạng thái này khách hàng không được phép tự hủy đơn.
    - `SHIPPING` *(Đang giao hàng)*: Đã bàn giao cho đơn vị giao nhận mô phỏng; kích hoạt chốt tồn kho và cộng `sold_count`.
    - `COMPLETED` *(Hoàn thành)*: Khách hàng đã nhận hàng; đơn `COD` được ghi nhận thu tiền thành công tại bước này.
    - `CANCELLED` *(Đã hủy)*: Đơn bị hủy bởi khách hàng trong phạm vi cho phép hoặc tự động hủy do quá hạn 10 phút thanh toán.
- **Payment Status (`payment_status` — Trạng thái thanh toán):** Trục trạng thái phản ánh tình trạng thanh toán tiền tệ của đơn hàng:
    - `UNPAID` *(Chưa thanh toán)*: Chưa ghi nhận thanh toán thành công.
    - `PAID` *(Đã thanh toán)*: Đã khấu trừ thành công qua `MOCK_WALLET` hoặc đã thu tiền mặt khi hoàn thành đơn `COD`.
    - `REFUNDED` *(Đã hoàn tiền)*: Đã hoàn trả toàn bộ giá trị đơn về `MOCK_WALLET` khi đơn đã `PAID` bị hủy trước bước `PROCESSING`.
- **COD (`Cash on Delivery`):** Phương thức thanh toán tiền mặt khi nhận hàng; đơn khởi tạo ở cặp trạng thái (`AWAITING_CONFIRMATION`, `UNPAID`) và chỉ chuyển sang `PAID` khi đơn đạt `COMPLETED`.
- **Mock Wallet (`MOCK_WALLET` — Ví giả lập):** Phương thức thanh toán nội bộ mô phỏng bằng tiền Việt Nam đồng (VNĐ), không kết nối cổng thanh toán thật; mỗi tài khoản được cấp số dư thử nghiệm ban đầu **5.000.000 VNĐ** (`INITIAL_BALANCE`) và không hỗ trợ nạp tiền trong v1.
- **Wallet Ledger (`wallets` & `wallet_transactions`):** Hệ thống sổ cái bất biến ghi nhận mọi biến động số dư ví (`INITIAL_BALANCE`, `PAYMENT`, `REFUND`) kèm số dư trước (`balance_before`), số dư sau (`balance_after`) và `idempotency_key` để phục vụ đối soát.
- **Shipping Fee (`shipping_fee`) & Order Total (`total_amount`):** Phí vận chuyển cố định bằng `0` (miễn phí vận chuyển) trên toàn bộ đơn hàng v1; tổng tiền đơn hàng được tính bằng $\sum(\text{order\_items.quantity} \times \text{order\_items.price}) + \text{shipping\_fee}$.

---

## 6. Phân biệt các khái niệm dễ nhầm lẫn (Disambiguation Matrix)

| Cặp khái niệm | Điểm cốt lõi để phân biệt |
| --- | --- |
| **`order_status`** vs. **`payment_status`** | Hai trục trạng thái độc lập. Một đơn `COD` ở trạng thái `AWAITING_CONFIRMATION` vẫn có `payment_status = UNPAID`; một đơn `CANCELLED` có thể là `UNPAID` (hủy trước khi trả tiền) hoặc `REFUNDED` (đơn ví đã `PAID` sau đó hủy hợp lệ). |
| **`physical_stock`** vs. **`reserved_stock`** vs. **`Available Stock`** | `physical_stock` là hàng vật lý trong kho; `reserved_stock` là hàng đã khóa cho đơn chưa giao; `Available Stock` là phần chênh lệch còn đặt mua được. Tạo đơn chỉ tăng `reserved_stock`, chỉ khi chuyển sang `SHIPPING` mới trừ `physical_stock`. |
| **Thêm vào giỏ (`Cart`)** vs. **Tạo đơn (`Order`)** | Thao tác trên giỏ hàng chỉ lưu ý định mua và **không** khóa tồn kho; tạo đơn hàng mới kích hoạt cơ chế tạm giữ `reserved_stock`. |
| **Hủy đơn khi `UNPAID`** vs. **Hủy đơn khi `PAID`** | Cả hai đều chuyển `order_status` sang `CANCELLED` và giải phóng `reserved_stock`, nhưng chỉ đơn `MOCK_WALLET` đã `PAID` mới sinh giao dịch hoàn tiền `REFUND` và chuyển `payment_status` sang `REFUNDED`. |
| **`source_product_id`** vs. **`products.id`** | `source_product_id` là khóa định danh từ nguồn DummyJSON (dùng seed thông tin gốc và đọc đánh giá có sẵn), còn `products.id` là khóa chính nội bộ của ShopFlow dùng cho giỏ hàng, tồn kho và đơn hàng. |
| **Dữ liệu gốc (`products`, `user_profiles`)** vs. **Bản chụp (`order_items`, `orders.recipient_*`)** | Dữ liệu gốc phản ánh thông tin hiện tại và có thể thay đổi; dữ liệu bản chụp bị đóng băng tại thời điểm đặt hàng để giữ nguyên lịch sử giao dịch. |
| **`Guest`** vs. **`User`** vs. **`Admin`** | `Guest` chưa đăng nhập; `User` đã đăng nhập và là vai trò duy nhất trong v1; `Admin` chưa tồn tại dưới dạng tài khoản đăng nhập riêng ở v1 (vận hành qua API mô phỏng). |
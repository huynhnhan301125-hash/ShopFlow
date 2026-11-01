# ShopFlow - Product Requirements Document (PRD)

# 1. Tổng quan và Bài toán kinh doanh

## 1.1. Vấn đề thực tế (Problem Statement)

- Thời gian: Khách hàng phải mất thời gian tìm cửa hàng, di chuyển đến cửa hàng, tìm kiếm sản phẩm,chờ thanh toán tại quầy. Mọi việc đều phải trực tiếp làm dẫn đến mất nhiều thời gian.
- Không gian: Quá trình di chuyển phụ thuộc nhiều yếu tố như giao thông, thời tiết, tìm kiếm sản phẩm trong cửa hàng gây ra sự bất tiện khi mua hàng.
- Sản phẩm: Sản phẩm sẽ có đôi lúc hết hàng, cửa hàng đóng cửa khiến khách hàng không thể đáp ứng được nhu cầu hoặc phải đổi cửa hàng.
- Khác: Quên đem tiền hay điện thoại không thể thanh toán, không nhớ mặt hàng mình cần mua,…

## 1.2. Giải pháp và Giá trị cốt lõi (Value Proposition)

### ShopFlow mang đến các giải pháp giúp khách hàng hay người dùng giải quyết được các vấn đề trên, tối ưu hóa mọi thao tác/hành động giúp đáp ứng nhu cầu mua hàng.

- Người dùng có thể mua sắm trực tuyến, lựa chọn, tìm kiếm, thanh toán cho mặt hàng trực tuyến giúp giải quyết vấn đề thời gian và thanh toán.
- Người dùng có thể mua hàng mọi lúc, mọi nơi khi có kết nối internet giải quyết vấn đề không gian.
- Người dùng có thể có cái nhìn trực quan về mặt hàng mình định mua thông qua đánh giá của nhiều người dùng, thêm sản phẩm vào giỏ hàng cho các sản phẩm chưa cần mua ngay lập tức/ mua sau.
- Giao diện trực quan, thiết kế thân thiện với người dùng, tối ưu hóa trải nghiệm người dùng.

# 2. Phạm vi sản phẩm (Product Scope)

| Must Have | Should Have | Could Have | Won’t Have |
| --- | --- | --- | --- |
| Tài khoản người dùng: Đăng ký, Đăng nhập, Lưu phiên đăng nhập, Đăng xuất. | Bộ lọc theo danh mục ( giá, hãng, đánh giá của người dùng, bán chạy). | Chuyển đổi giao diện sáng/tối. | Thanh toán bằng thẻ quốc tế (Visa, Mastercard). |
| Khám phá hàng hóa: Danh sách sản phẩm phân trang, tìm kiếm sản phẩm theo từ khóa, trang chi tiết sản phẩm. | Viết đánh giá cho sản phẩm. | Chuyển đổi ngôn ngữ. | Liên hệ với bên bán hàng, bên giao hàng, shipper. |
| Giỏ hàng và đơn hàng: Thêm, sửa, xóa mặt hàng trong giỏ hàng, mua hàng ngay lập tức. | Vô hiệu hóa các nút bấm tránh người dùng spam. |  | Định vị GPS. |
| Cung cấp đa dạng phương thức thanh toán: COD (Thanh toán khi mua hàng), Mô phỏng chuyển khoản ngân hàng (QR Code / Mock Wallet). |  |  | Xử lý trùng đơn phía backend. |

# 3. Phân rã tính năng và Tiêu chí nghiệm thu (Feature Specifications)

## 3.1 Module Tài khoản và Xác thực

### 3.1.1. Story 1: Đăng nhập thành công

#### User Story: Là một khách hàng, tôi muốn đăng nhập bằng tài khoản và mật khẩu để có thể mua hàng, truy cập giỏ hàng, xem lịch sử giao hàng.

#### Kịch bản nghiệm thu:

- Given ( Điều kiện ban đầu): Người dùng đang ở màn hình đăng nhập, đã tạo tài khoản (có tài khoản và mật khẩu đầy đủ).
- When ( Hành động): Người dùng nhập đúng tài khoản, mật khẩu và bấm “Đăng nhập”. Khi bấm nút đăng nhập, nút bấm chuyển sang trạng thái Loading (xoay tròn) và bị vô hiệu hóa(disable) tránh người dùng bấm liên tục.
- Then ( Hệ thống phải làm gì): Thông báo đăng nhập thành công, lưu AccessToken và RefreshToken an toàn vào Secure Storage (Android Keystore/ iOS Keychain), chuyển hướng đến màn hình “Trang chủ”.

### 3.1.2. Story 2 : Đăng nhập thất bại

#### User Story: Là một khách hàng, tôi muốn đăng nhập bằng tài khoản và mật khẩu để có thể mua hàng, truy cập giỏ hàng, xem lịch sử giao hàng.

#### Kịch bản nghiệm thu 1: Người dùng để trống thông tin (Lỗi tại máy - Client Validation)

- Given: Người dùng ở màn hình đăng nhập.
- When: Để trống ô Tài khoản hoặc Mật khẩu và bấm nút “Đăng nhập”
- Then: Hệ thống hiển thị dòng chữ cảnh báo màu đỏ ngay dưới ô bị thiếu: “Vui lòng không để trống thông tin này”, và không gửi yêu cầu lên máy chủ.

#### Kịch bản nghiệm thu 2: Nhập sai thông tin đăng nhập (Lỗi từ máy chủ - Server Error)

- Given: Người dùng ở màn hình đăng nhập và điền đầy đủ hai ô nhưng thông tin không chính xác.
- When: Bấm nút “Đăng nhập”
- Then:
    - Hệ thống gửi yêu cầu lên máy chủ, sau đó hiển thị thông báo lỗi nổi: *“*Tài khoản hoặc mật khẩu không chính xác. Vui lòng thử lại!”.
    - Trong lúc chờ xác nhận từ máy chủ nút “Đăng nhập” sẽ bị vô hiệu hóa và hiển thị biểu tượng Loading (xoay tròn).

#### **Kịch bản nghiệm thu 3: Mất kết nối Internet (Lỗi mạng - Network Error)**

- Given: Điện thoại bị mất mạng (mất Wifi / 4G).
- When: Người dùng bấm nút “Đăng nhập”.
- Then: Hệ thống hiển thị thông báo: *“Không thể kết nối đến máy chủ. Vui lòng kiểm tra lại đường truyền mạng”*.

#### Kịch bản nghiệm thu 4: Người dùng chưa có tài khoản

- Given: Người dùng ở màn hình đăng nhập và chưa có tài khoản.
- When: Người dùng ấn “Đăng ký”.
- Then :
    - Hệ thống chuyển trang sang màn hình “Đăng ký”.

### 3.1.3. Story 3: Đăng ký tài khoản (Register)

#### User Story: Là một khách hàng, tôi muốn tạo tài khoản mua sắm cho mình.

#### Kịch bản nghiệm thu:

- Given: Người dùng ở trang đăng ký tài khoản.
- When: Người dùng nhập đúng các trường thông tin để tạo tài khoản (tên đăng nhập, mật khẩu và xác nhận mật khẩu) sau đó ấn đăng ký.
- Then: Hệ thống tiến hành kiểm tra và xác thực thông tin
    - Nếu thành công chuyển hướng người dùng về lại trang đăng nhập.
    - Nếu không thành công báo lỗi và yêu cầu người dùng đăng ký lại.

### 3.1.4. Story 4: Đăng xuất tài khoản (Logout)

#### User Story: Là một khách hàng, tôi muốn đăng xuất khỏi tài khoản hiện tại.

#### Kịch bản nghiệm thu:

- Given: Người dùng ở trang cài đặt của ứng dụng.
- When: Người dùng ấn “Đăng xuất”.
- Then: Hệ thống xóa Token trong máy và đưa người dùng về trang đăng nhập.

## 3.2 Module Khám phá hàng hóa và Tìm kiếm

### **3.2.1. Story 1: Xem danh sách sản phẩm & Cuộn trang (Pagination)**

#### **User Story:** Là một khách hàng, tôi muốn xem danh sách sản phẩm theo dạng lưới (Grid) và tự động tải thêm khi cuộn xuống dưới, để có thể xem nhiều sản phẩm mà không bị đơ giật ứng dụng.

### Kịch bản nghiệm thu:

- Given: Người dùng mở ứng dụng tại màn hình “Trang chủ”.
- When: Hệ thống đang tải dữ liệu (lần đầu) hoặc người dùng cuộn danh sách xuống gần cuối trang.
- Then: Hiển thị trạng thái Loading (xoay tròn) khi hệ thống tải dữ liệu và khi người dùng cuộn danh sách xuống mà dữ liệu chưa tải lên xong. Khi tải xong, hiển thị danh sách sản phẩm gồm: Hình ảnh, tên, giá bán, số sao, số lượt mua.

### **3.2.2. Story 2: Tìm kiếm sản phẩm theo từ khóa**

#### **User Story:** Là một khách hàng, tôi muốn gõ tên sản phẩm vào thanh tìm kiếm để tìm nhanh món hàng mình cần.

#### Kịch bản nghiệm thu:

- Given: Người dùng ở thanh tìm kiếm.
- When: Người dùng gõ từ khóa (ví dụ: “phone”) hệ thống đọc ký tự và gợi ý cho người dùng các sản phẩm theo từ khóa, sau đó người dùng ấn chọn gợi ý hoặc ấn enter khi gõ xong để đi đến trang danh mục sản phẩm mình muốn.
- Then:
    - Nếu có sản phẩm: Danh sách hiển thị các món phù hợp.
    - Trường hợp Edge Case (Không có hàng): Nếu gõ từ khóa không tồn tại (ví dụ: “xyz123”) thì màn hình hiển thị nội dung “Sản phẩm bạn tìm kiếm không tồn tại” .

### 3.2.3. Story 3: Xem chi tiết sản phẩm

#### User Story: Là một khách hàng, tôi muốn xem thông tin chi tiết của sản phẩm mình muốn mua.

#### Kịch bản nghiệm thu:

- Given: Người dùng ở trang danh mục sản phẩm và lựa chọn được sản phẩm mình muốn xem thông tin.
- When: Click vào sản phẩm cần mua.
- Then: Màn hình hiển thị các thông tin như tên mặt hàng, giá cả, số lượt mua, số sao, thông tin của sản phẩm, đánh giá sản phẩm. Hai nút bấm “Thêm vào giỏ hàng” với icon giỏ hàng và nút bấm “Mua ngay”.

## 3.3 Module giỏ hàng và Đặt hàng (Cart & Checkout)

### 3.3.1. Story 1: Thao tác với giỏ hàng

#### User Story: Là một khách hàng, tôi muốn thêm, sửa, xóa sản phẩm trong giỏ hàng của mình.

#### Kịch bản nghiệm thu:

- Given: Người dùng ở trang giỏ hàng thực hiện các thao tác chỉnh sửa trong giỏ hàng với các sản phẩm đã được thêm vào giỏ hàng.
- When: Người dùng chỉnh sửa các sản phẩm trong giỏ hàng (tăng/giảm số lượng, xóa mặt hàng), ấn mua các sản phẩm trong giỏ hàng.
- Then:
    - Với các thao tác tăng/giảm hay xóa mặt hàng hệ thống sẽ kiểm tra số lượng của sản phẩm tại cửa hàng bán chúng, nếu còn thì cập nhật lại số lượng, tổng tiền, không còn thì hiện thông báo “Sản phẩm đã được bán hết” giữ nguyên các trường thông tin.
    - Với thao tác mua hàng trong giỏ hàng thì hệ thống sẽ chuyển người dùng sang trang mua hàng.
    - Nếu giỏ hàng trống hiển thị dòng chữ “Giỏ hàng của bạn đang trống” kèm biểu tượng giỏ hàng trống và nút bấm “Mua sắm ngay” để chuyển đến trang chủ.

### 3.3.2. Story 2: Xác nhận đặt hàng

#### User Story: Là một khách hàng, tôi muốn mua hàng trong giỏ hàng hoặc mua ngay.

#### Kịch bản nghiệm thu 1: Mặt hàng còn

- Given: Khách hàng ở trang thông tin chi tiết sản phẩm hoặc trang giỏ hàng và quyết định mua mặt hàng mình cần.
- When:
    - Người dùng ấn nút “Mua hàng”, Điền đầy đủ các trường thông tin cần thiết như ( số điện thoại, địa chỉ, tên, email…), chọn phương thức thanh toán. Sau khi điền và chọn xong người nhận ấn “Đặt hàng”.
- Then:
    - Hệ thống tải dữ liệu mặt hàng, vô hiệu hóa nút “Mua hàng”.
    - Hệ thống tạo đơn hàng với trạng thái “Chờ thanh toán” và bắt đầu đếm ngược 10 phút.
    - Nếu hết 10 phút chưa thanh toán thành công:
        - Hệ thống sẽ tự động chuyển đổi trạng thái đơn hàng thành “Đã hủy”. Đồng thời hoàn lại tồn kho (+1) và mở lại nút thêm giỏ hàng và mua ngay cho người khác.
    - Nếu thanh toán thành công:
        - Chuyển trang hiện dòng chữ “Bạn đã đặt hàng thành công” kèm 2 nút bấm “Quay về trang chủ” để quay về trang chủ của ứng dụng và nút bấm “Xem đơn hàng” chuyển hướng đến trang hiển thị thông tin của người dùng và thông tin cơ bản sản phẩm đã đặt.
        - Hệ thống xóa các sản phẩm vừa mua khỏi giỏ hàng (nếu có).
        - Hiển thị ngày giao dự kiến, tiến độ hiện tại của mặt hàng, và 1 nút “Hủy đơn hàng”.
        - Hệ thống cập nhật tồn kho (-1) cho sản phẩm đó ( Nếu về 0, hiển thị hết hàng với các người dùng khác, vô hiệu hóa nút “Thêm vào giỏ hàng” và nút “Mua ngay”).

#### Kịch bản nghiệm thu 2: Mặt hàng không còn

- Given: Khách hàng ở trang thông tin chi tiết sản phẩm hoặc trang giỏ hàng và quyết định mua mặt hàng mình cần, số lượng mặt hàng đã hết.
- When: Người dùng ấn nút “Mua hàng”,  “Đặt hàng”.
- Then:
    - Hệ thống hiển thị thông báo “Mặt hàng đã hết.”
    - Hệ thống vô hiệu hóa nút “Mua hàng” và “Đặt hàng” không tạo đơn hàng.

### 3.3.3. Story 3: Hủy đơn hàng

#### User Story: Là một khách hàng, tôi không có nhu cầu với sản phẩm đã đặt nữa và muốn hủy đơn hàng.

#### Kịch bản nghiệm thu:

- Trạng thái đơn hàng:
    - Chờ thanh toán: Vừa bấm đặt hàng, chờ thanh toán.
    - Chờ xác nhận: Khách đã trả tiền, shop nhận thông báo xác nhận.
    - Chuẩn bị đơn hàng: Shop đóng gói đơn hàng.
    - Đang giao hàng: Giao hàng cho bên vận chuyển.
    - Hoàn thành: Khách nhận được hàng.
    - Đã hủy: Đơn bị hủy (Khách hủy, shop hủy).
- Given: Người dùng ở trang chi tiết đơn hàng và trạng thái đơn hàng là “Chờ thanh toán” và “Chờ xác nhận”.
- When: Khách hàng ấn “Hủy đơn”.
- Then:
    - Hệ thống tiếp nhận yêu cầu hiển thị một danh mục lý do hủy đơn ( “Không còn nhu cầu”, “Có giá tốt hơn”, “Thay đổi địa chỉ/phương thức thanh toán”,… “Khác”), và nút “Xác nhận” để hủy đơn hoặc “Hủy” nếu không quay lại.
        - Với các lý do về thông tin giao hàng, hay phương thức thanh toán thì hệ thống sẽ mở lại màn hình xác nhận đơn hàng để người dùng chỉnh sửa và ấn nút “Lưu”.
    - Sau khi xác nhận hủy hệ thống sẽ trả lại chuyển đổi trạng thái đơn hàng thành “Đã hủy”. Cập nhật lại tồn kho (+1) cho sản phẩm của shop đó.

# 4. Yêu cầu phi chức năng (Non-Functional Requirements - NFR)

## 4.1. Về hiệu năng & Trải nghiệm (Performance)

- Tốc độ tải màn hình và phản hồi tìm kiếm dưới 1-2 giây.
- Cải thiện và tối ưu tính năng vuốt cuộn màn hình đạt chuẩn 60 FPS mượt mà.

## 4.2. Về bảo mật dữ liệu (Security)

- Thông tin dữ liệu của khách hàng sẽ được bảo mật nghiêm ngặt, mọi thông tin quan trọng đều được mã hóa và lưu trong Secure Storage/ Keystore/ Keychain trên máy.
- Khi người dùng đăng xuất mọi dữ liệu cá nhân và token trên máy đều được xóa sạch.

## 4.3. Về khả năng dùng ngoại tuyến (Offline-first & Caching)

- Mọi trang hiển thị sản phẩm đều sẽ bị làm mờ và hiển thị dòng chữ “Mất kết nối mạng. Hãy kiểm tra lại kết nối.” kèm nút “Thử lại”. Mọi tính năng như cuộn trang, tìm kiếm, Đăng xuất khi thao tác sẽ hiển thị nút xoay tròn sau đó hiển thị dòng chữ “Mất kết nối mạng” kèm nút “Thử lại”.
- Dữ liệu giỏ hàng đã được đặt vẫn được lưu lại, khi kết nối lại thành công thì sẽ hiển thị đầy đủ.
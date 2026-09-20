# Thiết kế Lab A3 - XML Layout và tài nguyên

Ngày: 2026-09-20

## Mục tiêu

Xây dựng ứng dụng Android native Java + XML cho Lab A3, package `vn.edu.vhu.ltdd.a3layout`, min SDK 24. Ứng dụng phải thể hiện đầy đủ màn hình đăng nhập ở hướng dọc và ngang, có bản ConstraintLayout để so sánh, dùng tài nguyên tách riêng và hiển thị đúng thông tin sinh viên:

- Họ tên: Võ Văn Quốc Bảo
- MSSV: 231A290036
- Email: 231a290036@vhu.edu.vn
- Không hiển thị lớp theo yêu cầu của sinh viên

Repo đích là `VOVANQUOCBAO/homework` theo chỉ định, dù tài liệu gợi ý tên repo `A3_<MSSV>`.

## Kiến trúc project

Project là một Android application module duy nhất:

- `MainActivity`: nạp layout dọc hoặc layout-land theo cấu hình, xử lý nút đăng nhập và mở bản ConstraintLayout.
- `ConstraintDemoActivity`: hiển thị cùng phần form bằng ConstraintLayout phẳng.
- `res/layout/activity_main.xml`: giao diện dọc dùng ScrollView, LinearLayout và FrameLayout.
- `res/layout-land/activity_main.xml`: giao diện ngang hai cột, giữ nguyên ID mà Java sử dụng.
- `res/layout/activity_constraint_demo.xml`: bản bố cục phẳng, mỗi view có ràng buộc ngang và dọc.
- `res/layout/view_profile_card.xml`: thẻ hồ sơ dùng lại qua `<include>`.
- `res/values`: chuỗi, màu, kích thước và style dùng chung.
- `res/values-night/colors.xml`: bảng màu chế độ tối.

## Giao diện chính

Bản dọc đặt toàn bộ nội dung trong ScrollView. Phần đầu là FrameLayout cao 196dp với nền gradient cao 150dp và avatar 92dp chồng lên mép dưới. Phần nội dung gồm tiêu đề, hai TextInputLayout, hàng ghi nhớ/quên mật khẩu, nút đăng nhập, dải phân cách, đăng nhập trường, đăng ký, thẻ hồ sơ và nút mở bản ConstraintLayout.

Bản ngang chia màn hình theo tỉ lệ 2:3. Cột trái chứa tiêu đề thương hiệu và thẻ hồ sơ; cột phải chứa form. Các ID `main`, `edtStudentId`, `edtPassword`, `cbRemember`, `btnForgot`, `btnLogin`, `btnSchoolLogin`, `btnRegister` và `btnConstraintDemo` giữ nguyên để MainActivity hoạt động ở cả hai hướng.

## Bản ConstraintLayout

Bản so sánh dùng Guideline để thống nhất lề, `0dp` cho kích thước match-constraint và một horizontal chain cho hai nút. Form gồm tiêu đề, MSSV, mật khẩu, ghi nhớ, quên mật khẩu, đăng nhập, đăng ký và nhãn mô tả. Không dùng `match_parent` cho view con trực tiếp của ConstraintLayout.

## Tài nguyên và khả dụng

Mọi chuỗi hiển thị nằm trong `strings.xml`; màu nằm trong `colors.xml`; khoảng cách dùng chung nằm trong `dimens.xml`. Kích thước hình học dùng dp và cỡ chữ dùng sp. Các drawable header, avatar và ô thống kê là shape XML. Style tiêu đề dùng lại được khai báo trong `themes.xml`.

Hai bài nâng cao được chọn:

1. NC1: bảng màu `values-night/colors.xml`.
2. NC3: style chữ tái sử dụng cho tiêu đề và phụ đề.

## Luồng xử lý

Khi tạo MainActivity, Android tự chọn `layout/activity_main.xml` hoặc `layout-land/activity_main.xml`. Activity ghi loại layout đã nạp vào Logcat. Nhấn ĐĂNG NHẬP hiện Snackbar; nội dung Snackbar bổ sung trạng thái ghi nhớ nếu CheckBox được chọn. Nhấn nút xem ConstraintLayout mở `ConstraintDemoActivity`. Các nút còn lại là thành phần minh hoạ giao diện và không tạo luồng nghiệp vụ ngoài phạm vi Lab A3.

## Xử lý lỗi và tính ổn định

Ứng dụng không phụ thuộc mạng hay dữ liệu ngoài. Layout dọc và các cột ngang có vùng cuộn để tránh nội dung bị che trên màn hình nhỏ hoặc khi bàn phím mở. MainActivity chỉ truy cập những ID hiện diện trong cả hai biến thể layout. Màu nền/chữ có cặp tương phản cho cả giao diện sáng và tối.

## Kiểm thử và xác minh

- Kiểm thử cấu trúc sẽ parse XML để xác nhận tài nguyên hợp lệ, không có text hiển thị viết cứng, đúng ID giữa portrait/landscape, có FrameLayout, Space, layout_weight, include và đủ các layout bắt buộc.
- Kiểm thử Java sẽ kiểm tra hai Activity được khai báo trong manifest và các hành vi Snackbar/navigation có trong mã nguồn.
- Chạy toàn bộ test cấu trúc.
- Chạy Gradle build và unit test nếu Android SDK khả dụng trong môi trường.
- Kiểm tra Git diff/trạng thái, số commit và nội dung repo trước khi báo hoàn tất.

## Hồ sơ nộp bài

Repo sẽ có README hướng dẫn mở/chạy project, mô tả hai bài nâng cao và checklist demo. Báo cáo Markdown sẽ có bảng so sánh hai layout và đáp án năm câu ôn tập. Ảnh wireframe vẽ tay, ảnh chạy thật dọc/ngang và video demo không được giả lập bằng nội dung bịa; sinh viên sẽ bổ sung bằng thiết bị hoặc emulator trước khi nộp.

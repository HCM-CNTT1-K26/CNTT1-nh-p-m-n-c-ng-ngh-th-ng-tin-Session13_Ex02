# CNTT1-nh-p-m-n-c-ng-ngh-th-ng-tin-Session13_Ex02
### PHẦN 1. NỀN TẢNG TƯ DUY VÀ QUẢN TRỊ HẠ TẦNG

#### 1. Vận dụng mô hình IPO và DIKW trong vận hành

* **Luồng xử lý thông tin (IPO):**
* Hệ thống hạ tầng mạng luôn sản sinh ra các tệp nhật ký khổng lồ. Thay vì đọc thủ công, ta dùng tệp `log-he-thong.log` làm đầu vào (**Input**).
* Thông qua script tự động hóa thực thi lệnh lọc qua ống dẫn (**Process**), hệ thống tự bóc tách các dòng trạng thái bất thường.
* Đầu ra (**Output**) là tệp báo cáo sự cố được lưu trữ gọn gàng phục vụ công tác rà soát.


* **Chuyển hóa dữ liệu (DIKW):**
* Từ hàng gigabyte log thô chưa có cấu trúc (**Data**), ta lọc ra các mã lỗi cụ thể như `ERROR 404` hay `TIMEOUT` (**Information**).
* Nhìn nhận xu hướng lặp lại của lỗi theo từng trạm cảm biến giúp kỹ sư đúc rút ra nguyên nhân gốc rễ (**Knowledge**).
* Cuối cùng, cấp quản lý sẽ đưa ra quyết định chiến lược về việc nâng cấp phần cứng hoặc mở rộng băng thông mạng (**Wisdom**).



#### 2. Tiêu chuẩn kiến trúc thư mục 4 trụ cột

Một dự án hạ tầng chuẩn mực tuyệt đối không trộn lẫn mã nguồn, dữ liệu vận hành và tài liệu hướng dẫn vào chung một nơi. Cấu trúc chia thành 4 nhánh độc lập:

* `docs/`: Lưu trữ hồ sơ thiết kế, tài liệu kỹ thuật SRS và hướng dẫn vận hành.
* `scripts/`: Chứa toàn bộ các tập lệnh tự động hóa (.bat, .sh).
* `data/`: Nơi đặt cơ sở dữ liệu thiết bị và tệp log thô đầu vào.
* `backup/`: Lưu trữ lịch sử báo cáo phục vụ kiểm toán và truy vết lỗi.
* *Quy tắc định danh:* Sử dụng chuẩn `kebab-case` (ví dụ: `log-he-thong.log`), viết thường toàn bộ, không dùng tiếng Việt có dấu và không chứa khoảng trắng để tránh lỗi đường dẫn khi triển khai trên các hệ điều hành khác nhau.

---

### PHẦN 2. TRIỂN KHAI CÁC MODULE KỸ THUẬT THỰC TẾ

#### Module 1: Khởi tạo nhanh hạ tầng dự án trên Terminal

Thay vì tạo từng thư mục thủ công bằng tay, ta gom nhóm câu lệnh lồng nhau để thiết lập toàn bộ cấu trúc trong vòng một nốt nhạc:

```cmd
mkdir greentech-system\docs greentech-system\scripts greentech-system\data greentech-system\backup && cd greentech-system && type nul > .env

```

* **Kiểm tra trực quan:** Sử dụng lệnh `tree /F` để rà soát lại cây thư mục.
* **Lưu ý bảo mật:** Tệp `.env` đặt ở thư mục gốc dùng để chứa các chuỗi kết nối nhạy cảm, phải được khai báo trong tệp `.gitignore` để tránh bị đẩy lên kho lưu trữ công cộng.

#### Module 2: Xây dựng Robot tự động lọc Log (`scripts/auto_report.bat`)

Đối với các tệp log dung lượng lớn (có thể lên tới hàng trăm MB), việc mở trực tiếp bằng trình soạn thảo văn bản thông thường sẽ làm tràn RAM máy chủ. Giải pháp tối ưu là sử dụng lệnh dòng lệnh kết hợp ống dẫn (Pipe):

```bat
@echo off
if not exist "..\backup" mkdir "..\backup"
echo ===== BAO CAO SU CO HE THONG ===== >> ..\backup\bao-cao-su-co.txt
echo Thoi gian chay: %date% %time% >> ..\backup\bao-cao-su-co.txt
type ..\data\log-he-thong.log | findstr /I "CRITICAL ERROR TIMEOUT" >> ..\backup\bao-cao-su-co.txt
echo [SUCCESS] Da hoan tat trich xuat log.

```

* **Kỹ thuật cốt lõi:** Luôn sử dụng toán tử nối tệp `>>` thay vì dấu ghi đè `>`. Nếu lỡ dùng `>`, toàn bộ lịch sử báo cáo cũ được ghi nhận trước đó sẽ bị xóa sạch hoàn toàn.

#### Module 3: Chuẩn hóa tài liệu vận hành kỹ thuật

* **Đối với Markdown (`docs/README.md`):** Sử dụng các khối mã nguồn (`code block`) để hướng dẫn chạy script trực quan, kết hợp danh sách kiểm tra (`task list`) để nhân viên mới dễ dàng thao tác từng bước mà không cần hỏi lại người phụ trách cũ.
* **Đối với tài liệu Word (`srs-he-thong.docx`):** Bắt buộc phải áp dụng hệ thống Heading Styles chuẩn (Heading 1, 2, 3) để khi chèn Mục lục tự động (Auto TOC), phần mềm sẽ tự động cập nhật số trang chính xác khi tài liệu có sự thay đổi về nội dung.

#### Module 4: Đối soát tài nguyên hạ tầng trên Excel

Để quản lý kho thiết bị và trạng thái trạm không bị sai sót do nhập liệu thủ công, ta áp dụng tư duy cơ sở dữ liệu quan hệ thu nhỏ:

1. **Làm sạch dữ liệu nguồn:** Các chuỗi mã linh kiện thường dễ dính khoảng trắng thừa ở đầu hoặc cuối. Phải chuẩn hóa trước bằng hàm:
```excel
=TRIM(B2)

```


2. **Thiết lập công thức tra cứu VLOOKUP:**
```excel
=VLOOKUP(TRIM(B2), 'danh-muc-linh-kien'!$A$2:$D$100, 2, FALSE)

```


* *Mẹo chuyên môn:* Vùng tham chiếu bắt buộc phải dùng dấu cố định tuyệt đối  để khi kéo công thức xuống các dòng bên dưới, vùng tìm kiếm không bị dịch chuyển lệch tọa độ. Tuyệt đối không để trống dòng hoặc dùng tính năng `Merge Cell` trong bảng dữ liệu thô để tránh làm hỏng bộ lọc `AutoFilter`.

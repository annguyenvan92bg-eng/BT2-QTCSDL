# BT2-QTCSDL
## Phần mở đầu
Thông tin cá nhân
- Tên : Nguyễn Văn An
- Lớp : K59KMT.K01
- MSSV : K235480106002
- Chủ đề : Quản lý thư viện
## Phần 1 : Thiết kế và Khởi tạo Cấu trúc Dữ liệu
- Tên db : QuanlyThuVien_K235480106002
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/057b63f8-d3f1-420b-af71-63b8a77c8299" />
*Ảnh này cho thấy tạo thành cống db với tên QuanlyThuVien_K235480106002

### Mô tả logic
Bài toán quản lý thư viện với 3 bảng chính:
- **Sach**: Lưu thông tin đầu sách (mã sách, tên sách, tác giả, số lượng tồn, đơn giá...)
- **DocGia**: Lưu thông tin bạn đọc (mã độc giả, họ tên, ngày sinh, địa chỉ...)
- **PhieuMuon**: Ghi nhận quá trình mượn/trả sách, kết nối sách và độc giả

Cấu trúc chi tiết từng bảng

#### Bảng 1: Sach (Sách)

| Tên cột | Kiểu dữ liệu | Ràng buộc | Giải thích |
|---------|--------------|-----------|-------------|
| MaSach | VARCHAR(10) | PRIMARY KEY | Mã sách, tự nhập (VD: S001, S002) |
| TenSach | NVARCHAR(200) | NOT NULL | Tên sách, hỗ trợ tiếng Việt |
| TacGia | NVARCHAR(100) | NOT NULL | Tên tác giả |
| NhaXuatBan | NVARCHAR(100) | NOT NULL | Nhà xuất bản |
| NamXuatBan | INT | CHECK (1900-2026) | Năm xuất bản hợp lý |
| SoLuongTon | INT | DEFAULT 0, CHECK >=0 | Số lượng còn trong kho |
| DonGia | MONEY | CHECK >=0 | Giá tiền của sách |
| TheLoai | NVARCHAR(50) | NULL | Thể loại sách (có thể bỏ trống) |

#### Bảng 2: DocGia (Độc giả)

| Tên cột | Kiểu dữ liệu | Ràng buộc | Giải thích |
|---------|--------------|-----------|-------------|
| MaDocGia | VARCHAR(10) | PRIMARY KEY | Mã độc giả, tự nhập (VD: DG001) |
| HoTen | NVARCHAR(100) | NOT NULL | Họ tên đầy đủ |
| NgaySinh | DATE | CHECK <= GETDATE() | Ngày sinh, không được trong tương lai |
| GioiTinh | NVARCHAR(5) | CHECK (Nam hoặc Nữ) | Giới tính |
| DiaChi | NVARCHAR(200) | NULL | Địa chỉ (có thể bỏ trống) |
| SoDienThoai | VARCHAR(15) | NULL | Số điện thoại liên lạc |
| NgayDangKy | DATE | DEFAULT GETDATE() | Ngày đăng ký làm thẻ thư viện |

#### Bảng 3: PhieuMuon (Phiếu mượn)

| Tên cột | Kiểu dữ liệu | Ràng buộc | Giải thích |
|---------|--------------|-----------|-------------|
| MaPhieuMuon | VARCHAR(10) | PRIMARY KEY | Mã phiếu mượn, tự nhập (VD: PM001) |
| MaDocGia | VARCHAR(10) | FOREIGN KEY → DocGia | Mã độc giả mượn sách |
| MaSach | VARCHAR(10) | FOREIGN KEY → Sach | Mã sách được mượn |
| NgayMuon | DATE | DEFAULT GETDATE() | Ngày mượn sách |
| NgayTra | DATE | NULL | Ngày trả (NULL = chưa trả) |
| SoLuongMuon | INT | CHECK >0 | Số lượng sách mượn (1 lần) |
| TinhTrang | NVARCHAR(20) | DEFAULT 'Đang mượn', CHECK (3 giá trị) | Trạng thái: Đang mượn/Đã trả/Quá hạn |
| TienPhat | MONEY | DEFAULT 0, CHECK >=0 | Tiền phạt nếu trả muộn |

####  Các kiểu dữ liệu được sử dụng

| Kiểu dữ liệu | Mục đích | Ví dụ sử dụng trong bài |
|--------------|----------|------------------------|
| VARCHAR(10) | Mã số ngắn (cố định độ dài) | MaSach, MaDocGia, MaPhieuMuon |
| NVARCHAR(200) | Chuỗi Unicode có độ dài thay đổi | Tên sách, tên tác giả, họ tên |
| INT | Số nguyên | Năm xuất bản, số lượng tồn |
| DATE | Chỉ lưu ngày (không lưu giờ) | Ngày sinh, ngày mượn, ngày trả |
| MONEY | Dữ liệu tiền tệ | Đơn giá sách, tiền phạt |
| VARCHAR(15) | Số điện thoại (có thể có dấu cách) | SoDienThoai |

#### Tổng hợp các ràng buộc

**PRIMARY KEY (3 khóa):**
- `Sach.MaSach` - Mỗi sách có một mã duy nhất
- `DocGia.MaDocGia` - Mỗi độc giả có một mã duy nhất
- `PhieuMuon.MaPhieuMuon` - Mỗi phiếu mượn có một mã duy nhất

**FOREIGN KEY (2 khóa):**
- `PhieuMuon.MaDocGia` → `DocGia.MaDocGia` (Không thể tạo phiếu mượn cho độc giả không tồn tại)
- `PhieuMuon.MaSach` → `Sach.MaSach` (Không thể mượn sách chưa có trong thư viện)

**CHECK (6 ràng buộc):**
- `NamXuatBan BETWEEN 1900 AND 2026` - Năm xuất bản hợp lý
- `SoLuongTon >= 0` - Số lượng tồn không âm
- `DonGia >= 0` - Đơn giá không âm
- `NgaySinh <= GETDATE()` - Ngày sinh không trong tương lai
- `GioiTinh IN (N'Nam', N'Nữ')` - Giới tính chỉ là Nam hoặc Nữ
- `SoLuongMuon > 0` - Phải mượn ít nhất 1 cuốn
- `TinhTrang IN (N'Đang mượn', N'Đã trả', N'Quá hạn')` - Chỉ 3 trạng thái hợp lệ
- `TienPhat >= 0` - Tiền phạt không âm

**DEFAULT (4 ràng buộc):**
- `SoLuongTon DEFAULT 0` - Nếu không nhập, mặc định tồn 0
- `NgayDangKy DEFAULT GETDATE()` - Mặc định ngày đăng ký là hôm nay
- `NgayMuon DEFAULT GETDATE()` - Mặc định ngày mượn là hôm nay
- `TinhTrang DEFAULT N'Đang mượn'` - Mặc định phiếu đang mượn
- `TienPhat DEFAULT 0` - Mặc định không bị phạt

### Code SQL
CREATE TABLE [Sach] (
    [MaSach] VARCHAR(10) PRIMARY KEY,          
    [TenSach] NVARCHAR(200) NOT NULL,           
    [TacGia] NVARCHAR(100) NOT NULL,            
    [NhaXuatBan] NVARCHAR(100) NOT NULL,        
    [NamXuatBan] INT CHECK ([NamXuatBan] BETWEEN 1900 AND 2026),  
    [SoLuongTon] INT DEFAULT 0 CHECK ([SoLuongTon] >= 0),         
    [DonGia] MONEY CHECK ([DonGia] >= 0),       
    [TheLoai] NVARCHAR(50) NULL                 
);
GO
CREATE TABLE [DocGia] (
    [MaDocGia] VARCHAR(10) PRIMARY KEY,         
    [HoTen] NVARCHAR(100) NOT NULL,            
    [NgaySinh] DATE CHECK ([NgaySinh] <= GETDATE()),  
    [GioiTinh] NVARCHAR(5) CHECK ([GioiTinh] IN (N'Nam', N'Nữ')),  
    [DiaChi] NVARCHAR(200) NULL,                
    [SoDienThoai] VARCHAR(15) NULL,             
    [NgayDangKy] DATE DEFAULT GETDATE()         
);
GO
CREATE TABLE [PhieuMuon] (
    [MaPhieuMuon] VARCHAR(10) PRIMARY KEY,     
    [MaDocGia] VARCHAR(10) NOT NULL,          
    [MaSach] VARCHAR(10) NOT NULL,              
    [NgayMuon] DATE DEFAULT GETDATE(),          
    [NgayTra] DATE NULL,                        
    [SoLuongMuon] INT CHECK ([SoLuongMuon] > 0),
    [TinhTrang] NVARCHAR(20) DEFAULT N'Đang mượn' CHECK ([TinhTrang] IN (N'Đang mượn', N'Đã trả', N'Quá hạn')),
    [TienPhat] MONEY DEFAULT 0 CHECK ([TienPhat] >= 0),  
    
    FOREIGN KEY ([MaDocGia]) REFERENCES [DocGia]([MaDocGia]),
    FOREIGN KEY ([MaSach]) REFERENCES [Sach]([MaSach])
);

<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/76e01b0a-d3d3-42aa-8405-560d0d3bc413" />
*Ảnh này cho thấy tôi đã tạo thành công 3 bảng với các kiểu dữ liệu đúng yêu cầu

Thực thi các câu lệnh CREATE TABLE để tạo đồng thời 3 bảng: [Sach], [DocGia], [PhieuMuon] trong database QuanLyThuVien_K235480106002, bao gồm các ràng buộc PRIMARY KEY, FOREIGN KEY, CHECK, DEFAULT.

  Giải quyết yêu cầu: xây dựng cấu trúc lưu trữ cho bài toán Quản lý Thư viện với 3 bảng có quan hệ với nhau.
  - Bảng [Sach] lưu thông tin đầu sách (mã sách là khóa chính, số lượng tồn không âm, đơn giá kiểu tiền tệ)
  - Bảng [DocGia] lưu thông tin bạn đọc (mã độc giả là khóa chính, ngày sinh không được là tương lai)
  - Bảng [PhieuMuon] làm bảng trung gian kết nối sách và độc giả thông qua 2 khóa ngoại, đồng thời quản lý tình trạng mượn/trả
## Phần 2 : Xây dựng Function 
**Các loại hàm build-in mà em biết:**

| Hàm | Loại | Công dụng | Ví dụ |
|-----|------|-----------|-------|
| `UPPER()` / `LOWER()` | Chuỗi | Chuyển đổi chữ hoa/thường | `UPPER(N'nguyễn')` → `NGUYỄN` |
| `GETDATE()` | Ngày giờ | Lấy thời gian hiện tại của hệ thống | `GETDATE()` → `2026-04-27 12:38:xx` |
| `DATEDIFF()` | Ngày giờ | Tính khoảng cách giữa 2 mốc thời gian | `DATEDIFF(DAY, '2026-04-01', GETDATE())` |
| `COALESCE()` | Xử lý NULL | Trả về giá trị đầu tiên không NULL | `COALESCE(NULL, 'A', 'B')` → `A` |
| `ISNULL()` | Xử lý NULL | Thay thế NULL bằng giá trị khác | `ISNULL(NULL, 0)` → `0` |

<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/d7ab4362-0c27-431a-b9db-fa2a5e309fa4" />
*Ảnh này là kết quả chạy các câu lệnh SELECT sử dụng system function giúp minh họa các hàm có sẵn trong SQL Server mà e biết

#### Hàm do người dùng tự viết thường mang mục đích 
- Đóng gói logic xử lý phức tạp, có thể tái sử dụng nhiều lần
- Giảm thiểu việc viết lặp lại cùng một đoạn code
- Tăng tính bảo trì (chỉ cần sửa ở 1 nơi khi thay đổi logic)
- Cho phép sử dụng trong SELECT, WHERE, ORDER BY (khác với SP)

#### Có những loại function 
- Scalar Function: Trả về 1 giá trị đơn (int, string, date...)
- Inline Table-Valued Function (ITVF): Trả về bảng, chỉ có 1 câu SELECT
 -Multi-statement Table-Valued Function (MSTVF): Trả về bảng, có thể có nhiều câu lệnh

#### Mỗi loại dùng khi nào?
- Scalar Function: Khi cần tính toán, chuyển đổi, hoặc kiểm tra trả về 1 giá trị
- ITVF: Khi cần lọc dữ liệu đơn giản (hiệu suất cao nhất)
- MSTVF: Khi có logic phức tạp, cần tạo bảng trung gian, vòng lặp

#### Tại sao có nhiều system function rồi vẫn cần tự viết?
- System function chỉ giải quyết các nhu cầu chung, không thể đáp ứng mọi logic nghiệp vụ đặc thù
- Doanh nghiệp mỗi nơi một quy tắc tính toán khác nhau (cách tính phạt, cách tính tuổi...)
- Tái sử dụng logic nghiệp vụ nhiều lần trong các module khác nhau
- Dễ dàng nâng cấp và bảo trì khi quy tắc kinh doanh thay đổi
## Thêm dữ liệu mẫu để kết quả fn được rõ ràng hơn
- Code SQL

INSERT INTO [Sach] ([MaSach], [TenSach], [TacGia], [NhaXuatBan], [NamXuatBan], [SoLuongTon], [DonGia], [TheLoai])
VALUES 
    ('S001', N'Đắc Nhân Tâm', N'Dale Carnegie', N'NXB Tổng hợp', 2020, 10, 120000, N'Kỹ năng'),
    ('S002', N'Nhà Giả Kim', N'Paulo Coelho', N'NXB Văn học', 2018, 5, 85000, N'Tiểu thuyết'),
    ('S003', N'Clean Code', N'Robert Martin', N'NXB Khoa học', 2021, 3, 250000, N'Tin học'),
    ('S004', N'SQL Cơ bản', N'John Doe', N'NXB Giáo dục', 2022, 7, 180000, N'Tin học'),
    ('S005', N'Tư Duy Nhanh', N'Daniel Kahneman', N'NXB Thế giới', 2019, 4, 150000, N'Tâm lý');
  

    INSERT INTO [DocGia] ([MaDocGia], [HoTen], [NgaySinh], [GioiTinh], [DiaChi], [SoDienThoai])
VALUES 
    ('DG001', N'Nguyễn Văn A', '2000-05-15', N'Nam', N'Hà Nội', '0901234567'),
    ('DG002', N'Trần Thị B', '2001-08-20', N'Nữ', N'Hải Phòng', '0901234568'),
    ('DG003', N'Lê Văn C', '1999-12-10', N'Nam', N'Đà Nẵng', '0901234569');


    INSERT INTO [PhieuMuon] ([MaPhieuMuon], [MaDocGia], [MaSach], [NgayMuon], [SoLuongMuon], [TinhTrang])
VALUES 
    ('PM001', 'DG001', 'S001', '2026-04-20', 1, N'Đang mượn'),
    ('PM002', 'DG002', 'S002', '2026-04-25', 2, N'Đang mượn'),
    ('PM003', 'DG001', 'S003', '2026-04-22', 1, N'Đã trả');
  
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/197f0077-b474-4749-9deb-869173a785c4" />
  Ảnh này là kết quả thêm thành công dữ liệu mẫu
  
### 1. Scalar Function
- Mục đích: Tính tổng số sách đang được mượn
- Code SQL

CREATE OR ALTER FUNCTION [dbo].[fn_TongSachDangMuon] ()
RETURNS INT
AS
BEGIN
    DECLARE @Tong INT;
    
    SELECT @Tong = COUNT(*)
    FROM [PhieuMuon]
    WHERE [TinhTrang] = N'Đang mượn';
    
    RETURN @Tong;
END;
GO
SELECT [dbo].[fn_TongSachDangMuon]() AS [TongSachDangMuon];

<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/5040c84b-6223-462b-8118-114c2716d868" />
 *Ảnh này tạo hàm fn_TongSachDangMuon và thực thi để đếm tổng số phiếu mượn đang ở trạng thái "Đang mượn".
 Cho biết có bao nhiêu cuốn sách đang được độc giả mượn mà chưa trả
 
 Kết quả hiển thị một con số
 
### 2. Table-Valued Function
- Mục đích: Lấy danh sách sách còn tồn kho
- Code SQL
CREATE OR ALTER FUNCTION [dbo].[fn_SachConTon] ()
RETURNS TABLE
AS
RETURN
(
    SELECT [MaSach], [TenSach], [TacGia], [SoLuongTon], [DonGia]
    FROM [Sach]
    WHERE [SoLuongTon] > 0
);
GO

SELECT * FROM [dbo].[fn_SachConTon]();

  <img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/9f28a68e-10c3-4972-bf6f-321af5495095" />
* Ảnh này tạo hàm fn_SachConTon và thực thi để liệt kê tất cả sách còn số lượng tồn > 0.

 Lọc ra những đầu sách thư viện còn có thể cho mượn, loại bỏ sách đã hết.
Hiển thị danh sách sách với cột SoLuongTon > 0. Những sách nào tồn kho = 0 sẽ không xuất hiện trong kết quả.

### 3. Multi-statement Table-Valued Function
- Mục đích: Thống kê độc giả có số lần mượn sách
- Code SQl

CREATE OR ALTER FUNCTION [dbo].[fn_ThongKeSachTheoTheLoai] ()
RETURNS @BangThongKe TABLE
(
    [TheLoai] NVARCHAR(50),
    [SoLuongSach] INT,
    [TongSoLuongTon] INT
)
AS
BEGIN
    INSERT INTO @BangThongKe
    SELECT 
        [TheLoai],
        COUNT(*) AS [SoLuongSach],
        SUM([SoLuongTon]) AS [TongSoLuongTon]
    FROM [Sach]
    WHERE [TheLoai] IS NOT NULL
    GROUP BY [TheLoai];
    
    INSERT INTO @BangThongKe
    VALUES (N'TỔNG CỘNG', 
            (SELECT COUNT(*) FROM [Sach]),
            (SELECT SUM([SoLuongTon]) FROM [Sach]));
    
    RETURN;
END;
GO

SELECT * FROM [dbo].[fn_ThongKeSachTheoTheLoai]();

<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/5e8c3b63-e18b-4a0e-ad87-56a529a5dfb1" />
*Ảnh này tạo hàm fn_ThongKeSachTheoTheLoai và thực thi để tổng hợp số lượng sách theo từng thể loại.

 Giúp quản lý thư viện nắm được cơ cấu đầu sách: thể loại nào nhiều sách nhất, thể loại nào có tổng số lượng tồn nhiều nhất.
 Hiển thị bảng có 3 cột: Thể loại, Số lượng đầu sách, Tổng số lượng tồn. Dòng cuối là TỔNG CỘNG - đây là điểm khác biệt của MSTVF so với ITVF vì đã xử lý thêm logic bên trong.

## Phần 3 :  Xây dựng Store Procedure
#### Một vài system sp trong SQL Server
- sp_help :	     Hiển thị thông tin chi tiết của bảng (cột, ràng buộc, khóa)
- sp_columns :	 Chỉ hiển thị thông tin các cột của bảng	
- sp_rename	 :    Đổi tên bảng, cột hoặc ràng buộc	
- sp_who	 :   Xem ai đang kết nối đến SQL Server
- sp_helpdb	 :    Xem thông tin database (kích thước, ngày tạo)	
- sp_tables	 :    Liệt kê tất cả bảng trong database

### 1. Store Procedure đơn giản để thực hiện lệnh INSERT hoặc UPDATE dữ liệu, có kiểm tra điều kiện logic

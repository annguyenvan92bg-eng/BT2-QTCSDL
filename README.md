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
```sql

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

```

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
```sql

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
```
  
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/197f0077-b474-4749-9deb-869173a785c4" />
  Ảnh này là kết quả thêm thành công dữ liệu mẫu
  
### 1. Scalar Function
- Mục đích: Tính tổng số sách đang được mượn
```sql

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
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/5040c84b-6223-462b-8118-114c2716d868" />
 *Ảnh này tạo hàm fn_TongSachDangMuon và thực thi để đếm tổng số phiếu mượn đang ở trạng thái "Đang mượn".
 Cho biết có bao nhiêu cuốn sách đang được độc giả mượn mà chưa trả
 
 Kết quả hiển thị một con số
 
### 2. Table-Valued Function
- Mục đích: Lấy danh sách sách còn tồn kho
```sql
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
```
  <img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/9f28a68e-10c3-4972-bf6f-321af5495095" />
* Ảnh này tạo hàm fn_SachConTon và thực thi để liệt kê tất cả sách còn số lượng tồn > 0.

 Lọc ra những đầu sách thư viện còn có thể cho mượn, loại bỏ sách đã hết.
Hiển thị danh sách sách với cột SoLuongTon > 0. Những sách nào tồn kho = 0 sẽ không xuất hiện trong kết quả.

### 3. Multi-statement Table-Valued Function
- Mục đích: Thống kê độc giả có số lần mượn sách
```sql

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
```
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
- Mục đích : THÊM ĐỘC GIẢ MỚI
```sql
CREATE OR ALTER PROCEDURE [dbo].[sp_ThemDocGia]
    @MaDocGia VARCHAR(10),
    @HoTen NVARCHAR(100),
    @NgaySinh DATE,
    @GioiTinh NVARCHAR(5),
    @DiaChi NVARCHAR(200),
    @SoDienThoai VARCHAR(15)
AS
BEGIN
    IF EXISTS (SELECT 1 FROM [DocGia] WHERE [MaDocGia] = @MaDocGia)
    BEGIN
        PRINT N'LỖI: Mã độc giả ' + @MaDocGia + N' đã tồn tại!';
        RETURN;
    END
   
    IF @GioiTinh NOT IN (N'Nam', N'Nữ')
    BEGIN
        PRINT N'LỖI: Giới tính phải là Nam hoặc Nữ!';
        RETURN;
    END
    IF @NgaySinh > GETDATE()
    BEGIN
        PRINT N'LỖI: Ngày sinh không được lớn hơn ngày hiện tại!';
        RETURN;
    END
    INSERT INTO [DocGia] ([MaDocGia], [HoTen], [NgaySinh], [GioiTinh], [DiaChi], [SoDienThoai])
    VALUES (@MaDocGia, @HoTen, @NgaySinh, @GioiTinh, @DiaChi, @SoDienThoai);
    
    PRINT N'Thành công: Đã thêm độc giả ' + @HoTen;
END;
GO
EXEC [dbo].[sp_ThemDocGia] 'DG004', N'Phạm Thị D', '2002-03-10', N'Nữ', N'Huế', '0912345678';
SELECT * FROM [DocGia];
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/613b4e14-fe70-4131-993e-04191d4e1936" />
 *Ảnh này tạo và thực thi sp_ThemDocGia để thêm độc giả mới.
 
 Tự động kiểm tra mã trùng, giới tính hợp lệ (Nam/Nữ), ngày sinh không trong tương lai trước khi thêm.
 Khi thêm độc giả DG004 thành công, in ra thông báo. Nếu thêm mã đã tồn tại hoặc giới tính sai sẽ báo lỗi ngay lập tức.
 ### 2. Store Procedure có sử dụng tham số OUTPUT để trả về một giá trị tính toán
 - Mục đích : TÍNH TỔNG SỐ SÁCH ĐANG MƯỢN
```sql
CREATE OR ALTER PROCEDURE [dbo].[sp_TongSachDangMuon]
    @MaDocGia VARCHAR(10),
    @TongSo INT OUTPUT
AS
BEGIN
    SELECT @TongSo = COUNT(*)
    FROM [PhieuMuon]
    WHERE [MaDocGia] = @MaDocGia AND [TinhTrang] = N'Đang mượn';
    
    PRINT N'Độc giả ' + @MaDocGia + N' đang mượn ' + CAST(@TongSo AS VARCHAR) + N' cuốn sách';
END;
GO

DECLARE @KQ INT;
EXEC [dbo].[sp_TongSachDangMuon] 'DG001', @KQ OUTPUT;
SELECT @KQ AS [SoSachDangMuon];
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/81e937e4-d401-4cd4-81b5-927bf5fabc02" />
*Ảnh này Tạo sp_TongSachDangMuon và thực thi với tham số OUTPUT.

 Đếm số lượng sách đang mượn của một độc giả, trả về qua tham số OUTPUT để sử dụng cho các xử lý khác.
 Độc giả DG001 đang mượn 1 cuốn sách. Giá trị OUTPUT được lưu vào biến @KQ và hiển thị ra màn hình.
 ### 3. Store Procedure trả về một tập kết quả (Result set) từ lệnh SELECT sau khi đã join nhiều bảng
 - Mục đích : XEM DANH SÁCH PHIẾU MƯỢN
```sql

CREATE OR ALTER PROCEDURE [dbo].[sp_DanhSachPhieuMuon]
AS
BEGIN
    SELECT 
        pm.[MaPhieuMuon],
        dg.[HoTen] AS [TenDocGia],
        s.[TenSach] AS [TenSach],
        pm.[NgayMuon],
        pm.[TinhTrang],
        pm.[TienPhat]
    FROM [PhieuMuon] pm
    JOIN [DocGia] dg ON pm.[MaDocGia] = dg.[MaDocGia]
    JOIN [Sach] s ON pm.[MaSach] = s.[MaSach]
    ORDER BY pm.[NgayMuon] DESC;
END;
GO

EXEC [dbo].[sp_DanhSachPhieuMuon];
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/da78a6f5-a96c-4eed-9514-97ca0e704f22" />
 *Ảnh này Tạo sp_DanhSachPhieuMuon và thực thi để xem danh sách phiếu mượn.
 
 JOIN 3 bảng Sach, DocGia, PhieuMuon để hiển thị đầy đủ thông tin: tên độc giả, tên sách, ngày mượn, trạng thái.
 Trả về bảng kết quả với 6 cột, hiển thị tất cả phiếu mượn đã có trong cơ sở dữ liệu, sắp xếp theo ngày mượn mới nhất.
 ## Phần 4: Trigger và Xử lý logic nghiệp vụ 
 ### 1. Trigger để tự động làm gì đó tại 1 bảng B khi mà dữ liệu thay đổi dữ liệu ở bảng A
-- Bảng A: PhieuMuon (phiếu mượn)

-- Bảng B: Sach (sách)

-- Logic: Khi thêm phiếu mượn (A), tự động giảm số lượng tồn kho của sách (B)
```sql
CREATE OR ALTER TRIGGER [dbo].[trg_PhieuMuon_Insert_GiamTonKho]
ON [PhieuMuon]
AFTER INSERT
AS
BEGIN
    UPDATE [Sach]
    SET [SoLuongTon] = [SoLuongTon] - i.[SoLuongMuon]
    FROM [Sach] s
    INNER JOIN inserted i ON s.[MaSach] = i.[MaSach];
    
    PRINT N'Đã giảm tồn kho sách do có phiếu mượn mới';
END;
```
- Sau đó kiểm tra kết quả
```sql
-- Xem tồn kho trước
SELECT [MaSach], [TenSach], [SoLuongTon] FROM [Sach] WHERE [MaSach] = 'S001';
Thêm phiếu mượn (mượn 2 cuốn)
INSERT INTO [PhieuMuon] ([MaPhieuMuon], [MaDocGia], [MaSach], [NgayMuon], [SoLuongMuon], [TinhTrang])
VALUES ('PM007', 'DG001', 'S001', GETDATE(), 2, N'Đang mượn');
-- Xem tồn kho sau 
SELECT [MaSach], [TenSach], [SoLuongTon] FROM [Sach] WHERE [MaSach] = 'S001';
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/f6566e15-816d-4bfb-9ef6-ec009a1fde25" />
*Ảnh này Tạo trigger trg_PhieuMuon_Insert_UpdateSach và kiểm tra khi thêm phiếu mượn mới.

Tự động giảm số lượng tồn kho sách khi có phiếu mượn mới được tạo.
Sau khi INSERT phiếu mượn PM005 (mượn 2 cuốn S001), tồn kho sách S001 giảm từ 10 xuống 8. Trigger hoạt động đúng.
### 2. Trigger cho Bảng A : Khi insert thì cập nhật dữ liệu vào bảng B; sau đó viết trigger cho bảng B để khi B được cập nhật thì cập nhật sang bảng A 
*Vẫn tiếp tục dựa theo trigger của ý 1 ta làm tiếp
- Bước 1: Tạo Trigger cho bảng A (PhieuMuon) → cập nhật bảng B (Sach)
-- TRIGGER CHO BẢNG A (PhieuMuon)

-- KHI INSERT PHIẾU MƯỢN → CẬP NHẬT TỒN KHO SÁCH
```sql
CREATE OR ALTER TRIGGER [dbo].[trg_A_PhieuMuon_Insert_UpdateSach]
ON [PhieuMuon]
AFTER INSERT
AS
BEGIN
    UPDATE [Sach]
    SET [SoLuongTon] = [SoLuongTon] - i.[SoLuongMuon]
    FROM [Sach] s
    INNER JOIN inserted i ON s.[MaSach] = i.[MaSach];
    
    PRINT N'[Trigger A] Đã cập nhật tồn kho sách (giảm do mượn)';
END;
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/88437b08-6d07-4ab2-a8a6-e4a97588da53" />
 *Ảnh này  Tạo trigger trg_A_PhieuMuon_Insert_UpdateSach trên bảng PhieuMuon (bảng A).

  Khi có phiếu mượn mới được thêm vào bảng A (PhieuMuon), trigger này tự động cập nhật bảng B (Sach) bằng cách giảm số lượng tồn kho của sách tương ứng.
  Trigger được tạo thành công. Logic trong trigger: Lấy dữ liệu từ bảng inserted (chứa các dòng vừa thêm), JOIN với bảng Sach, sau đó UPDATE cột SoLuongTon = SoLuongTon - số lượng mượn. Trigger này chỉ hoạt động khi thực hiện lệnh INSERT vào bảng PhieuMuon.
- Bước 2: Tạo Trigger cho bảng B (Sach) → cập nhật lại bảng A (PhieuMuon)
-- TRIGGER CHO BẢNG B (Sach)
-- KHI TỒN KHO SÁCH GIẢM NHIỀU → TỰ ĐỘNG GIA HẠN PHIẾU MƯỢN
-- Logic: Nếu sách bị giảm tồn kho > 5 cuốn trong 1 lần, các phiếu đang mượn sách đó được gia hạn thêm 3 ngày
```sql
CREATE OR ALTER TRIGGER [dbo].[trg_B_Sach_Update_UpdatePhieuMuon]
ON [Sach]
AFTER UPDATE
AS
BEGIN
    UPDATE [PhieuMuon]
    SET [NgayTra] = DATEADD(DAY, 3, [NgayTra])
    FROM [PhieuMuon] pm
    INNER JOIN inserted i ON pm.[MaSach] = i.[MaSach]
    INNER JOIN deleted d ON i.[MaSach] = d.[MaSach]
    WHERE d.[SoLuongTon] - i.[SoLuongTon] >= 5
        AND pm.[TinhTrang] = N'Đang mượn';
    
    PRINT N'[Trigger B] Đã gia hạn phiếu mượn do sách giảm tồn kho mạnh';
END;
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/a97d47f9-ba81-4b67-82be-961256436e56" />
*Ảnh này Tạo trigger trg_B_Sach_Update_UpdatePhieuMuon trên bảng Sach (bảng B). 

  Khi số lượng tồn kho sách giảm, trigger này kiểm tra nếu mức giảm từ 5 cuốn trở lên trong một lần UPDATE, thì tự động gia hạn thêm 3 ngày cho các phiếu mượn đang mượn sách đó.
  Trigger được tạo thành công. Logic trong trigger: JOIN giữa bảng inserted (giá trị mới), deleted (giá trị cũ) và bảng PhieuMuon. Nếu chênh lệch tồn kho (cũ - mới) >= 5 và phiếu đang trong trạng thái "Đang mượn", thì cập nhật NgayTra tăng thêm 3 ngày. Trigger này chỉ hoạt động khi thực hiện lệnh UPDATE trên bảng Sach.
  
- Bước 3: Thử nghiệm vòng tròn
-- THỬ NGHIỆM: UPDATE BẢNG A SẼ TẠO VÒNG TRÒN VỚI BẢNG B

-- 1. INSERT vào bảng A (PhieuMuon) → kích hoạt Trigger A
-- 2. Trigger A UPDATE bảng B (Sach) → giảm tồn kho 6 cuốn
-- 3. Trigger B thấy giảm tồn kho >=5 → UPDATE bảng A (PhieuMuon) gia hạn
-- 4. UPDATE bảng A lại kích hoạt Trigger A? (tùy loại trigger)
```sql
INSERT INTO [PhieuMuon] ([MaPhieuMuon], [MaDocGia], [MaSach], [NgayMuon], [SoLuongMuon], [TinhTrang])
VALUES ('PM009', 'DG001', 'S001', GETDATE(), 6, N'Đang mượn');
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/01b0acd2-5ec8-4af4-b500-83bde011aaa7" />
*Ảnh này Thực hiện câu lệnh INSERT INTO vào bảng PhieuMuon để thêm phiếu mượn mới (mượn 6 cuốn sách S001).

  Minh họa hậu quả của việc tạo trigger vòng tròn giữa 2 bảng có quan hệ với nhau. Khi INSERT vào bảng A, trigger A cập nhật bảng B; trigger B thấy bảng B thay đổi lại cập nhật bảng A; tạo thành vòng lặp vô hạn.
 Hệ thống báo lỗi
####  Giải thích thông báo lỗi:

**Tại sao xảy ra lỗi?**

| Bước | Hành động | Trigger nào chạy |
|------|-----------|------------------|
| 1 | INSERT vào bảng A (PhieuMuon) | → Kích hoạt trg_A |
| 2 | trg_A UPDATE bảng B (Sach) | → Kích hoạt trg_B |
| 3 | trg_B UPDATE bảng A (PhieuMuon) | → Lại kích hoạt trg_A |
| 4 | Quay lại bước 2 | → Vòng lặp vô hạn |

**Vòng lặp:** A → B → A → B → A → B ... không bao giờ kết thúc.

**Cách SQL Server xử lý:**
- SQL Server giới hạn độ sâu lồng nhau (nested level) tối đa là 32
- Khi đạt đến giới hạn này, hệ thống tự động dừng và báo lỗi
- Mục đích: Tránh treo hệ thống hoặc quá tải bộ nhớ
#### Nhận xét 

| Tình huống | Kết quả | Đánh giá |
|------------|---------|----------|
| Chỉ có 1 trigger (A → B) | ✅ Chạy thành công | **AN TOÀN** |
| Có 2 trigger vòng tròn (A → B và B → A) | ❌ Lỗi nested level exceeded | **NGUY HIỂM** |

**Kết luận:**
1. Trigger rất hữu ích để tự động hóa nghiệp vụ (cập nhật tồn kho khi có phiếu mượn)
2. **Tuyệt đối không nên** tạo trigger vòng tròn giữa 2 bảng có quan hệ với nhau
3. Vòng tròn trigger tạo ra vòng lặp vô hạn, khiến SQL Server báo lỗi và dừng thao tác
4. **Giải pháp:** Chỉ nên có trigger một chiều, phần xử lý ngược lại nên đặt trong Store Procedure để chủ động kiểm soát

## Phần 5 : Cursor và Duyệt dữ liệu
### 1. CURSOR để duyệt qua danh sách của 1 câu lệnh SQL dạng SELECT, duyệt qua từng bản ghi, xử lý riêng từng bản ghi 
BÀI TOÁN ĐẶT RA

Logic  
Thư viện muốn tặng quà Tết cho các độc giả thân thiết. Quy tắc:

Tặng 100.000đ nếu độc giả mượn >= 5 lần

Tặng 50.000đ nếu độc giả mượn từ 3-4 lần

Tặng 20.000đ nếu độc giả mượn từ 1-2 lần

Không tặng nếu chưa mượn lần nào

Đồng thời, ghi log lại việc tặng quà vào bảng riêng.
- Bước 1: Tạo bảng Log để lưu kết quả tặng quà
```sql
CREATE TABLE [LogTangQua] (
    [Id] INT IDENTITY(1,1) PRIMARY KEY,
    [MaDocGia] VARCHAR(10),
    [HoTen] NVARCHAR(100),
    [SoLanMuon] INT,
    [QuaTang] NVARCHAR(100),
    [NgayTang] DATETIME DEFAULT GETDATE()
);
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/a8c7ff77-a698-4fdf-b7e5-709cdeae03a4" />
*Ảnh này thực thi câu lệnh CREATE TABLE để tạo bảng [LogTangQua] dùng để lưu kết quả tặng quà cho độc giả.

Tạo một bảng log riêng để ghi nhận việc xử lý tặng quà . Bảng này gồm các cột: Id (tự động tăng), MaDocGia, HoTen, SoLanMuon, QuaTang, NgayTang. Điều này giúp dễ dàng so sánh kết quả giữa 2 cách sau khi chạy.
 Bảng [LogTangQua] được tạo thành công, hiển thị trong Object Explorer dưới mục Tables. Cấu trúc bảng bao gồm 6 cột với đầy đủ kiểu dữ liệu: INT IDENTITY cho Id, VARCHAR(10) cho MaDocGia, NVARCHAR(100) cho HoTen, INT cho SoLanMuon, NVARCHAR(100) cho QuaTang, và DATETIME mặc định GETDATE() cho NgayTang.

- Bước 2: Giải quyết bằng CURSOR

```sql
-- Duyệt từng độc giả, tính số lần mượn, xử lý riêng từng người
DECLARE @MaDocGia VARCHAR(10);
DECLARE @HoTen NVARCHAR(100);
DECLARE @SoLanMuon INT;
DECLARE @QuaTang NVARCHAR(100);

-- Xóa dữ liệu cũ trong bảng log
DELETE FROM [LogTangQua];

-- Khai báo CURSOR
DECLARE cursor_tangqua CURSOR FOR
    SELECT 
        d.[MaDocGia],
        d.[HoTen],
        COUNT(p.[MaPhieuMuon]) AS [SoLanMuon]
    FROM [DocGia] d
    LEFT JOIN [PhieuMuon] p ON d.[MaDocGia] = p.[MaDocGia]
    GROUP BY d.[MaDocGia], d.[HoTen];

-- Mở CURSOR
OPEN cursor_tangqua;

-- Lấy dòng đầu tiên
FETCH NEXT FROM cursor_tangqua INTO @MaDocGia, @HoTen, @SoLanMuon;

-- Bắt đầu vòng lặp duyệt từng bản ghi
WHILE @@FETCH_STATUS = 0
BEGIN
    -- Xử lý riêng từng bản ghi: Tính quà tặng
    SET @QuaTang = CASE
        WHEN @SoLanMuon >= 5 THEN N'100.000đ'
        WHEN @SoLanMuon >= 3 THEN N'50.000đ'
        WHEN @SoLanMuon >= 1 THEN N'20.000đ'
        ELSE N'Không có quà'
    END;
    
    -- Chèn vào bảng log
    INSERT INTO [LogTangQua] ([MaDocGia], [HoTen], [SoLanMuon], [QuaTang])
    VALUES (@MaDocGia, @HoTen, @SoLanMuon, @QuaTang);
    
    -- In ra thông báo theo từng độc giả
    PRINT N'Đã xử lý xong: ' + @HoTen + N' - Số lần mượn: ' + CAST(@SoLanMuon AS VARCHAR) + N' - Quà: ' + @QuaTang;
    
    -- Lấy dòng tiếp theo
    FETCH NEXT FROM cursor_tangqua INTO @MaDocGia, @HoTen, @SoLanMuon;
END;

-- Đóng và giải phóng CURSOR
CLOSE cursor_tangqua;
DEALLOCATE cursor_tangqua;
-- Xem kết quả
SELECT * FROM [LogTangQua];
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/6e4a4b1e-1962-43a1-82d1-ab3408d952fe" />
*Ảnh này chạy script CURSOR để duyệt từng độc giả, tính số lần mượn và xác định quà tặng.

 Duyệt qua từng bản ghi trong danh sách độc giả, xử lý riêng từng người để tính quà dựa trên số lần mượn.
 Mỗi độc giả được xử lý riêng, thông báo in ra lần lượt từng người, kết quả được lưu vào bảng LogTangQua. 
 Ví dụ: DG001 mượn 2 lần được tặng 20.000đ, DG002 mượn 1 lần được 20.000đ, DG003 chưa mượn không có quà.
- Bước 3: Giải quyết KHÔNG dùng CURSOR (dùng INSERT SELECT)

```sql
-- Xử lý đồng loạt tất cả độc giả cùng lúc
-- Xóa dữ liệu cũ
DELETE FROM [LogTangQua];

-- Chèn dữ liệu bằng INSERT SELECT (xử lý hàng loạt)
INSERT INTO [LogTangQua] ([MaDocGia], [HoTen], [SoLanMuon], [QuaTang])
SELECT 
    d.[MaDocGia],
    d.[HoTen],
    COUNT(p.[MaPhieuMuon]) AS [SoLanMuon],
    CASE 
        WHEN COUNT(p.[MaPhieuMuon]) >= 5 THEN N'100.000đ'
        WHEN COUNT(p.[MaPhieuMuon]) >= 3 THEN N'50.000đ'
        WHEN COUNT(p.[MaPhieuMuon]) >= 1 THEN N'20.000đ'
        ELSE N'Không có quà'
    END AS [QuaTang]
FROM [DocGia] d
LEFT JOIN [PhieuMuon] p ON d.[MaDocGia] = p.[MaDocGia]
GROUP BY d.[MaDocGia], d.[HoTen];

-- Xem kết quả
SELECT * FROM [LogTangQua];
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/06916bba-8db3-47eb-9af4-b0dc01a0f646" />
 *Ảnh này thực thi câu lệnh INSERT SELECT kết hợp với CASE WHEN để xử lý tặng quà cho tất cả độc giả cùng lúc, không dùng CURSOR.
 
  Xử lý bài toán tặng quà Tết cho độc giả dựa trên số lần mượn sách, nhưng thay vì dùng CURSOR duyệt từng dòng, phương pháp này xử lý đồng loạt chỉ với 1 câu lệnh duy nhất. Logic vẫn giống CURSOR: mượn >=5 tặng 100.000đ, từ 3-4 tặng 50.000đ, từ 1-2 tặng 20.000đ, còn lại không có quà.
 Kết quả lưu vào bảng `LogTangQua`
 
- Bước 4: So sánh thời gian xử lý

```sql
-- CÁCH 1: DÙNG CURSOR
PRINT 'CÁCH 1: DÙNG CURSOR';
DECLARE @StartTime1 DATETIME = GETDATE();
PRINT 'Thời gian bắt đầu: ' + CAST(@StartTime1 AS VARCHAR);

-- Chạy CURSOR
DECLARE @MaDG VARCHAR(10), @Ten NVARCHAR(100), @SoLan INT, @Qua NVARCHAR(100);
DELETE FROM [LogTangQua];
DECLARE cur CURSOR FOR SELECT d.[MaDocGia], d.[HoTen], COUNT(p.[MaPhieuMuon]) FROM [DocGia] d LEFT JOIN [PhieuMuon] p ON d.[MaDocGia] = p.[MaDocGia] GROUP BY d.[MaDocGia], d.[HoTen];
OPEN cur;
FETCH NEXT FROM cur INTO @MaDG, @Ten, @SoLan;
WHILE @@FETCH_STATUS = 0
BEGIN
    SET @Qua = CASE WHEN @SoLan >= 5 THEN N'100.000đ' WHEN @SoLan >= 3 THEN N'50.000đ' WHEN @SoLan >= 1 THEN N'20.000đ' ELSE N'Không' END;
    INSERT INTO [LogTangQua] ([MaDocGia], [HoTen], [SoLanMuon], [QuaTang]) VALUES (@MaDG, @Ten, @SoLan, @Qua);
    FETCH NEXT FROM cur INTO @MaDG, @Ten, @SoLan;
END;
CLOSE cur;
DEALLOCATE cur;

DECLARE @EndTime1 DATETIME = GETDATE();
PRINT 'Thời gian kết thúc: ' + CAST(@EndTime1 AS VARCHAR);
PRINT 'Thời gian CURSOR: ' + CAST(DATEDIFF(MILLISECOND, @StartTime1, @EndTime1) AS VARCHAR) + ' ms';
PRINT '';

-- CÁCH 2: KHÔNG DÙNG CURSOR
PRINT 'CÁCH 2: KHÔNG DÙNG CURSOR';
DECLARE @StartTime2 DATETIME = GETDATE();
PRINT 'Thời gian bắt đầu: ' + CAST(@StartTime2 AS VARCHAR);

DELETE FROM [LogTangQua];
INSERT INTO [LogTangQua] ([MaDocGia], [HoTen], [SoLanMuon], [QuaTang])
SELECT d.[MaDocGia], d.[HoTen], COUNT(p.[MaPhieuMuon]),
    CASE WHEN COUNT(p.[MaPhieuMuon]) >= 5 THEN N'100.000đ' WHEN COUNT(p.[MaPhieuMuon]) >= 3 THEN N'50.000đ' WHEN COUNT(p.[MaPhieuMuon]) >= 1 THEN N'20.000đ' ELSE N'Không' END
FROM [DocGia] d LEFT JOIN [PhieuMuon] p ON d.[MaDocGia] = p.[MaDocGia]
GROUP BY d.[MaDocGia], d.[HoTen];

DECLARE @EndTime2 DATETIME = GETDATE();
PRINT 'Thời gian kết thúc: ' + CAST(@EndTime2 AS VARCHAR);
PRINT 'Thời gian KHÔNG CURSOR: ' + CAST(DATEDIFF(MILLISECOND, @StartTime2, @EndTime2) AS VARCHAR) + ' ms';
PRINT '';

-- KẾT LUẬN
PRINT '=== KẾT LUẬN ===';
IF DATEDIFF(MILLISECOND, @StartTime1, @EndTime1) > DATEDIFF(MILLISECOND, @StartTime2, @EndTime2)
    PRINT 'Không dùng CURSOR nhanh hơn CURSOR';
ELSE
    PRINT 'CURSOR nhanh hơn hoặc bằng không dùng CURSOR';

```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/8e6e1108-35a8-4ec9-a47d-6ec098f51b5d" />
*Ảnh này so sánh thời gian thực thi giữa cách dùng CURSOR và cách dùng INSERT SELECT (không CURSOR).

 Đo và in ra thời gian chạy (ms) của cả 2 cách để tìm ra cách nào nhanh hơn.
Kết quả cho thấy
  - Cách dùng CURSOR: thường chậm hơn vì phải duyệt từng dòng, mỗi dòng thực hiện INSERT riêng lẻ.
  - Cách dùng INSERT SELECT: nhanh hơn vì xử lý đồng loạt tất cả dữ liệu trong 1 câu lệnh.
  - **Kết luận:** Không dùng CURSOR nhanh hơn (với dữ liệu càng lớn, chênh lệch càng rõ).

### 2. Bài toán khác, mà chỉ CURSOR mới giải quyết được, còn SQL rất khó giải quyết được
BÀI TOÁN : Gửi thông báo riêng cho từng độc giả với nội dung cá nhân hóa --> Mỗi độc giả nhận 1 thông báo khác nhau dựa trên lịch sử mượn sách của họ

```sql
DECLARE @Ma VARCHAR(10), @Ten NVARCHAR(100), @SoLanMuon INT, @ThongBao NVARCHAR(500);

DECLARE cur_thongbao CURSOR FOR
    SELECT d.[MaDocGia], d.[HoTen], COUNT(p.[MaPhieuMuon])
    FROM [DocGia] d
    LEFT JOIN [PhieuMuon] p ON d.[MaDocGia] = p.[MaDocGia]
    GROUP BY d.[MaDocGia], d.[HoTen];

OPEN cur_thongbao;
FETCH NEXT FROM cur_thongbao INTO @Ma, @Ten, @SoLanMuon;

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @ThongBao = N'Gửi anh/chị ' + @Ten + ', ';
    
    IF @SoLanMuon = 0
        SET @ThongBao = @ThongBao + N'quý thư viện rất nhớ bạn. Hãy ghé thăm thư viện nhé!';
    ELSE IF @SoLanMuon <= 2
        SET @ThongBao = @ThongBao + N'cảm ơn bạn đã mượn ' + CAST(@SoLanMuon AS VARCHAR) + N' lần. Chúc bạn vui!';
    ELSE IF @SoLanMuon <= 4
        SET @ThongBao = @ThongBao + N'chúc mừng bạn đã mượn ' + CAST(@SoLanMuon AS VARCHAR) + N' lần. Bạn nhận được voucher 10%!';
    ELSE
        SET @ThongBao = @ThongBao + N'cảm ơn bạn đã đồng hành cùng thư viện với ' + CAST(@SoLanMuon AS VARCHAR) + N' lần mượn. Bạn là độc giả thân thiết!';
    
    PRINT @ThongBao;
    
    FETCH NEXT FROM cur_thongbao INTO @Ma, @Ten, @SoLanMuon;
END;

CLOSE cur_thongbao;
DEALLOCATE cur_thongbao;
```
<img width="2560" height="1600" alt="image" src="https://github.com/user-attachments/assets/dd282be6-b499-45c2-ab04-9429ac45e163" />
*Ảnh này thực hiện bài toán gửi thông báo cá nhân hóa cho từng độc giả bằng CURSOR.

 Mỗi độc giả nhận một thông báo khác nhau với nội dung được ghép động dựa trên số lần mượn và tên của họ.
- Kết quả cho thấy 
  - CURSOR dễ dàng xử lý bài toán này vì có thể xây dựng nội dung thông báo riêng cho từng dòng.
  - SQL thuần túy rất khó vì:
    1. Phải tạo ra chuỗi thông báo khác nhau cho từng độc giả
    2. Không có câu lệnh nào cho phép "xử lý riêng" từng dòng một cách linh hoạt
    3. Phải dùng các hàm phức tạp như STRING_AGG, CASE lồng nhau rất khó đọc và bảo trì

# BT2-QTCSDL
## Phần mở đầu
Thông tin cá nhân
- Tên : Nguyễn Văn An
- Lớp : K59KMT.K01
- MSSV : K235480106002
- Chủ đề : Quản lý thư viện
## Phần 1 : Thiết kế và Khởi tạo Cấu trúc Dữ liệu
- Tên db : QuanlyThuVien_K235480106002
<img width="2560" height="1600" alt="Ảnh chụp màn hình 2026-04-27 001624" src="https://github.com/user-attachments/assets/27b63e9d-d901-450f-8c0b-a0e84fbe9c3e" />
*Ảnh này cho thấy tạo thành cống db với tên QuanlyThuVien_K235480106002

### Mô tả logic
Bài toán quản lý thư viện với 3 bảng chính:
- **Sach**: Lưu thông tin đầu sách (mã sách, tên sách, tác giả, số lượng tồn, đơn giá...)
- **DocGia**: Lưu thông tin bạn đọc (mã độc giả, họ tên, ngày sinh, địa chỉ...)
- **PhieuMuon**: Ghi nhận quá trình mượn/trả sách, kết nối sách và độc giả

### Các kiểu dữ liệu sử dụng
| Kiểu dữ liệu | Ví dụ sử dụng | Lý do chọn |
|-------------|--------------|-------------|
| INT | MaSach, MaDocGia | Tự động tăng, làm khóa chính |
| NVARCHAR | TenSach, TacGia | Hỗ trợ tiếng Việt có dấu |
| MONEY | DonGia, TienPhat | Phù hợp cho dữ liệu tiền tệ |
| DATE | NgaySinh, NgayMuon | Chỉ lưu ngày, không cần giờ |
| CHAR(3) | GioiTinh | Cố định 3 ký tự (Nam/Nữ) |

### Ràng buộc 
1. **PRIMARY KEY**: MaSach, MaDocGia, MaPhieuMuon
2. **FOREIGN KEY**: 
   - PhieuMuon.MaSach → Sach.MaSach
   - PhieuMuon.MaDocGia → DocGia.MaDocGia
3. **CHECK**:
   - NamXuatBan BETWEEN 1900 AND 2026
   - SoLuongTon >= 0
   - GioiTinh IN ('Nam', 'Nữ')
   - TinhTrang IN (N'Đang mượn', N'Đã trả', N'Quá hạn')
4. **DEFAULT**:
   - SoLuongTon DEFAULT 0
   - NgayDangKy DEFAULT GETDATE()

### Code SQL
USE [QuanLyThuVien_K235480106002];
GO

CREATE TABLE [Sach] (
    [MaSach] INT IDENTITY(1,1) PRIMARY KEY,           
    [TenSach] NVARCHAR(200) NOT NULL,                 
    [TacGia] NVARCHAR(100) NOT NULL,                  
    [NhaXuatBan] NVARCHAR(100) NOT NULL,              
    [NamXuatBan] INT CHECK ([NamXuatBan] >= 1900 AND [NamXuatBan] <= YEAR(GETDATE())), 
    [SoLuongTon] INT DEFAULT 0 CHECK ([SoLuongTon] >= 0), 
    [DonGia] MONEY CHECK ([DonGia] >= 0),            
    [TheLoai] NVARCHAR(50) NULL                      
);
GO

CREATE TABLE [DocGia] (
    [MaDocGia] INT IDENTITY(1,1) PRIMARY KEY,        
    [HoTen] NVARCHAR(100) NOT NULL,                  
    [NgaySinh] DATE CHECK ([NgaySinh] <= GETDATE()),  
    [GioiTinh] CHAR(3) CHECK ([GioiTinh] IN (N'Nam', N'Nữ')), 
    [DiaChi] NVARCHAR(200) NULL,                      
    [SoDienThoai] VARCHAR(15) NULL,                   
    [NgayDangKy] DATE DEFAULT GETDATE()               
);
GO

CREATE TABLE [PhieuMuon] (
    [MaPhieuMuon] INT IDENTITY(1,1) PRIMARY KEY,      
    [MaDocGia] INT NOT NULL,                         
    [MaSach] INT NOT NULL,                            
    [NgayMuon] DATE DEFAULT GETDATE(),                
    [NgayTra] DATE NULL,                              
    [SoLuongMuon] INT CHECK ([SoLuongMuon] > 0),      
    [TinhTrang] NVARCHAR(50) DEFAULT N'Đang mượn' CHECK ([TinhTrang] IN (N'Đang mượn', N'Đã trả', N'Quá hạn')), 
    [TienPhat] MONEY DEFAULT 0,                      
    
    FOREIGN KEY ([MaDocGia]) REFERENCES [DocGia]([MaDocGia]),
    FOREIGN KEY ([MaSach]) REFERENCES [Sach]([MaSach])
);

<img width="2560" height="1600" alt="Ảnh chụp màn hình 2026-04-27 003900" src="https://github.com/user-attachments/assets/3188f829-5eed-4be7-a389-fc2babb7cd5a" />
*Ảnh này cho thấy tôi đã tạo thành công 3 bảng với các kiểu dữ liệu đúng yêu cầu

Thực thi các câu lệnh CREATE TABLE để tạo đồng thời 3 bảng: [Sach], [DocGia], [PhieuMuon] trong database QuanLyThuVien_K235480106002, bao gồm các ràng buộc PRIMARY KEY, FOREIGN KEY, CHECK, DEFAULT.

  Giải quyết yêu cầu: xây dựng cấu trúc lưu trữ cho bài toán Quản lý Thư viện với 3 bảng có quan hệ với nhau.
  - Bảng [Sach] lưu thông tin đầu sách (mã sách là khóa chính, số lượng tồn không âm, đơn giá kiểu tiền tệ)
  - Bảng [DocGia] lưu thông tin bạn đọc (mã độc giả là khóa chính, ngày sinh không được là tương lai)
  - Bảng [PhieuMuon] làm bảng trung gian kết nối sách và độc giả thông qua 2 khóa ngoại, đồng thời quản lý tình trạng mượn/trả

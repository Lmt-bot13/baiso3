# BÀI TẬP VỀ NHÀ 03: THIẾT KẾ VÀ CÀI ĐẶT CSDL QUẢN LÝ CẦM ĐỒ

Họ và tên: Lưu Minh Trí

MSSV: K2354801060104

Giảng viên: Đỗ Duy Cốp

Môn: Hệ QT CSDL

# 1. Phân tích bài toán hệ thống

Hệ thống quản lý cầm đồ được xây dựng nhằm quản lý các hợp đồng vay tiền có tài sản thế chấp.  

Mỗi khách hàng có thể thực hiện nhiều hợp đồng cầm cố khác nhau theo từng thời điểm.  

Trong một hợp đồng có thể tồn tại nhiều tài sản được sử dụng để đảm bảo khoản vay.  

Ngoài việc lưu trữ thông tin khách hàng và tài sản, hệ thống còn cần quản lý:

- tiền vay gốc
- thời hạn vay
- trạng thái hợp đồng
- lịch sử thanh toán
- cơ chế tính lãi
- quy trình thanh lý tài sản

Hệ thống sử dụng hai cơ chế tính lãi:

- Lãi đơn trước Deadline 1
- Lãi kép sau Deadline 1

Điều này làm cho bài toán không chỉ là quản lý dữ liệu thông thường mà còn liên quan đến xử lý nghiệp vụ tài chính theo thời gian thực.  

---

# 2. Phân tích mô hình dữ liệu

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/490c550d-a64d-4ba2-9d5a-67e0a539b323" />

Sơ đồ ERD


## 2.1. Quan hệ khách hàng và hợp đồng

Một khách hàng có thể có nhiều hợp đồng cầm cố khác nhau.  

Quan hệ:

```text
KHACH_HANG (1) ------ (N) HOP_DONG
```

---

## 2.2. Quan hệ hợp đồng và tài sản

Một hợp đồng có thể chứa nhiều tài sản thế chấp.  

Mỗi tài sản cần được quản lý riêng biệt để theo dõi:

- tên tài sản
- loại tài sản
- giá trị định giá
- trạng thái tài sản

Quan hệ:

```text
HOP_DONG (1) ------ (N) TAI_SAN
```

---

## 2.3. Quan hệ hợp đồng và thanh toán

Khách hàng có thể trả nợ nhiều lần nên cần lưu lịch sử thanh toán để theo dõi dòng tiền.  

Quan hệ:

```text
HOP_DONG (1) ------ (N) THANH_TOAN
```

---

Tạo database:

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/514d2a7d-cbe4-4841-87b1-831af892ada2" />


# 3. Thiết kế bảng KHACH_HANG

Bảng KHACH_HANG dùng để lưu toàn bộ thông tin khách hàng tham gia vay cầm cố tài sản.  

## Code tạo bảng

```sql
-- Tạo bảng lưu thông tin khách hàng
CREATE TABLE KHACH_HANG
(
    -- Mã khách hàng
    MaKH INT PRIMARY KEY IDENTITY(1,1),

    -- Họ tên khách hàng
    HoTen NVARCHAR(100),

    -- Căn cước công dân
    CCCD VARCHAR(20),

    -- Số điện thoại liên hệ
    SDT VARCHAR(15),

    -- Địa chỉ khách hàng
    DiaChi NVARCHAR(255)
);
```

## Giải thích code

- `MaKH` là khóa chính giúp định danh duy nhất từng khách hàng.  
- `IDENTITY(1,1)` giúp mã khách hàng tự tăng.  
- `CCCD` dùng để tránh trùng khách hàng.  
- `SDT` phục vụ liên hệ và truy vấn nợ xấu.  


<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/aa759997-105e-4537-91c8-dfb368c691ff" />


---

# 4. Thiết kế bảng HOP_DONG

Bảng HOP_DONG dùng để quản lý các khoản vay của khách hàng.  

## Code tạo bảng

```sql
-- Tạo bảng hợp đồng cầm đồ
CREATE TABLE HOP_DONG
(
    -- Mã hợp đồng
    MaHD INT PRIMARY KEY IDENTITY(1,1),

    -- Mã khách hàng
    MaKH INT,

    -- Ngày khách bắt đầu vay
    NgayVay DATE,

    -- Số tiền vay gốc
    TienGoc DECIMAL(18,2),

    -- Mốc thời gian tính lãi đơn
    Deadline1 DATE,

    -- Mốc thời gian thanh lý tài sản
    Deadline2 DATE,

    -- Trạng thái hợp đồng
    TrangThai NVARCHAR(50),

    -- Liên kết khách hàng
    FOREIGN KEY (MaKH)
    REFERENCES KHACH_HANG(MaKH)
);
```

## Giải thích code

- `TienGoc` lưu số tiền khách vay ban đầu.  
- `Deadline1` là thời điểm chuyển sang lãi kép.  
- `Deadline2` là thời điểm tài sản đủ điều kiện thanh lý.  
- `TrangThai` dùng để theo dõi trạng thái hợp đồng theo từng giai đoạn.  

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/a6ecda06-6134-445b-843e-2d1dca6edd40" />


# 5. Thiết kế bảng TAI_SAN

Bảng TAI_SAN dùng để quản lý danh sách tài sản khách mang đi cầm cố.  

## Code tạo bảng

```sql
-- Tạo bảng tài sản cầm cố
CREATE TABLE TAI_SAN
(
    -- Mã tài sản
    MaTS INT PRIMARY KEY IDENTITY(1,1),

    -- Mã hợp đồng
    MaHD INT,

    -- Tên tài sản
    TenTaiSan NVARCHAR(100),

    -- Loại tài sản
    LoaiTaiSan NVARCHAR(100),

    -- Giá trị định giá tài sản
    GiaTriDinhGia DECIMAL(18,2),

    -- Trạng thái tài sản
    TrangThai NVARCHAR(50),

    -- Liên kết hợp đồng
    FOREIGN KEY (MaHD)
    REFERENCES HOP_DONG(MaHD)
);
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/b64c94ad-9e52-4d1d-bdf8-565f20047fb6" />


## Giải thích code

- Mỗi tài sản được quản lý độc lập bằng `MaTS`.  
- `GiaTriDinhGia` phục vụ kiểm tra khả năng trả tài sản.  
- `TrangThai` giúp xác định tài sản:
  - đang cầm cố
  - đã trả khách
  - đã thanh lý



# 6. Thiết kế bảng THANH_TOAN

Bảng THANH_TOAN dùng để lưu lịch sử khách hàng trả nợ theo từng lần giao dịch.  

## Code tạo bảng

```sql
-- Tạo bảng lịch sử thanh toán
CREATE TABLE THANH_TOAN
(
    -- Mã thanh toán
    MaTT INT PRIMARY KEY IDENTITY(1,1),

    -- Mã hợp đồng
    MaHD INT,

    -- Ngày thanh toán
    NgayTra DATE,

    -- Số tiền khách trả
    SoTienTra DECIMAL(18,2),

    -- Người thu tiền
    NguoiThu NVARCHAR(100),

    -- Liên kết hợp đồng
    FOREIGN KEY (MaHD)
    REFERENCES HOP_DONG(MaHD)
);
```
<img width="1917" height="1078" alt="image" src="https://github.com/user-attachments/assets/16717932-5dcd-4870-a9d7-cda01365a64b" />


Chèn dữ liệu mẫu:

```sql
INSERT INTO KHACH_HANG(HoTen,CCCD,SDT,DiaChi) VALUES (N'Nguyen Van An','001001001','0901000001',N'Ha Noi');
INSERT INTO KHACH_HANG(HoTen,CCCD,SDT,DiaChi) VALUES (N'Tran Thi Lan','001001002','0901000002',N'Hai Phong');
INSERT INTO KHACH_HANG(HoTen,CCCD,SDT,DiaChi) VALUES (N'Le Minh Duc','001001003','0901000003',N'Da Nang');
GO

SELECT * FROM KHACH_HANG;
GO

INSERT INTO HOP_DONG(MaKH,NgayVay,TienGoc,Deadline1,Deadline2,TrangThai)
VALUES (1,GETDATE(),5000000,DATEADD(DAY,7,GETDATE()),DATEADD(DAY,14,GETDATE()),N'Đang vay');

INSERT INTO HOP_DONG(MaKH,NgayVay,TienGoc,Deadline1,Deadline2,TrangThai)
VALUES (2,GETDATE(),7000000,DATEADD(DAY,10,GETDATE()),DATEADD(DAY,20,GETDATE()),N'Đang vay');

INSERT INTO HOP_DONG(MaKH,NgayVay,TienGoc,Deadline1,Deadline2,TrangThai)
VALUES (3,GETDATE(),9000000,DATEADD(DAY,15,GETDATE()),DATEADD(DAY,30,GETDATE()),N'Đang vay');
GO

SELECT * FROM HOP_DONG;
GO

INSERT INTO TAI_SAN(MaHD,TenTaiSan,LoaiTaiSan,GiaTriDinhGia,TrangThai)
VALUES (1,N'Sách Lập Trình C++',N'Sách Công Nghệ',500000,N'Đang cầm');

INSERT INTO TAI_SAN(MaHD,TenTaiSan,LoaiTaiSan,GiaTriDinhGia,TrangThai)
VALUES (1,N'Sách SQL Server',N'Sách Database',450000,N'Đang cầm');

INSERT INTO TAI_SAN(MaHD,TenTaiSan,LoaiTaiSan,GiaTriDinhGia,TrangThai)
VALUES (2,N'Sách Python',N'Sách Lập Trình',600000,N'Đang cầm');

INSERT INTO TAI_SAN(MaHD,TenTaiSan,LoaiTaiSan,GiaTriDinhGia,TrangThai)
VALUES (2,N'Sách Thuật Toán',N'Sách CNTT',700000,N'Đang cầm');

INSERT INTO TAI_SAN(MaHD,TenTaiSan,LoaiTaiSan,GiaTriDinhGia,TrangThai)
VALUES (3,N'Sách AI',N'Sách Trí Tuệ Nhân Tạo',800000,N'Đang cầm');
GO

SELECT * FROM TAI_SAN;
GO

INSERT INTO THANH_TOAN(MaHD,NgayTra,SoTienTra,NguoiThu)
VALUES (1,GETDATE(),1000000,N'Nhan Vien A');

INSERT INTO THANH_TOAN(MaHD,NgayTra,SoTienTra,NguoiThu)
VALUES (1,GETDATE(),500000,N'Nhan Vien A');

INSERT INTO THANH_TOAN(MaHD,NgayTra,SoTienTra,NguoiThu)
VALUES (2,GETDATE(),2000000,N'Nhan Vien B');

INSERT INTO THANH_TOAN(MaHD,NgayTra,SoTienTra,NguoiThu)
VALUES (3,GETDATE(),3000000,N'Nhan Vien C');
GO

SELECT * FROM THANH_TOAN;
GO
```

<img width="1917" height="1078" alt="image" src="https://github.com/user-attachments/assets/fe05d88d-6860-4ea5-b058-8040c5a14ffc" />


## Giải thích code

- Bảng này giúp lưu toàn bộ lịch sử giao dịch trả tiền.  
- Tránh việc ghi đè số tiền còn nợ gây mất dữ liệu dòng tiền.  
- Có thể thống kê tổng tiền đã thanh toán theo từng hợp đồng.  


# Phân tích cơ chế tính lãi

##  Công thức lãi đơn

Trước Deadline 1, hệ thống áp dụng lãi đơn:

```text
Lãi = Tiền gốc × 0.005 × số ngày vay
```

Ví dụ:

```text
Tiền gốc = 10.000.000
Số ngày = 10

Lãi = 10.000.000 × 0.005 × 10
     = 500.000
```

---

## Công thức lãi kép

Sau Deadline 1:

```text
Tiền cơ sở = Gốc + Lãi đơn
```

Tiền nợ tiếp tục tăng theo:

```text
A = P × (1 + r)^n
```

Trong đó:

- `P` là tiền gốc sau cộng lãi đơn
- `r` là lãi suất
- `n` là số ngày quá hạn

---

# Function tính tổng tiền phải trả

Function dùng để tính số tiền khách cần thanh toán tại một thời điểm cụ thể.  

## Code Function

```sql
-- Hàm tính tổng tiền phải trả
CREATE FUNCTION fn_CalcMoneyContract
(
    @MaHD INT,
    @TargetDate DATE
)
RETURNS DECIMAL(18,2)
AS
BEGIN

    -- Khai báo biến
    DECLARE @TienGoc DECIMAL(18,2);
    DECLARE @Deadline1 DATE;
    DECLARE @NgayVay DATE;
    DECLARE @SoNgay INT;
    DECLARE @TienLai DECIMAL(18,2);
    DECLARE @TongTien DECIMAL(18,2);

    -- Lấy thông tin hợp đồng
    SELECT
        @TienGoc = TienGoc,
        @Deadline1 = Deadline1,
        @NgayVay = NgayVay
    FROM HOP_DONG
    WHERE MaHD = @MaHD;

    -- Tính số ngày vay
    SET @SoNgay = DATEDIFF(DAY, @NgayVay, @TargetDate);

    -- Tính lãi đơn
    SET @TienLai = @TienGoc * 0.005 * @SoNgay;

    -- Tổng tiền
    SET @TongTien = @TienGoc + @TienLai;

    RETURN @TongTien;

END;
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/95e37622-2a06-4b53-a285-4fa64f5245c5" />

## Giải thích code

- `DATEDIFF` dùng để tính số ngày vay thực tế.  
- Hệ thống lấy số tiền gốc và nhân với lãi suất mỗi ngày.  
- Function trả về tổng tiền cần thanh toán.  



# EVENT 1: ĐĂNG KÝ HỢP ĐỒNG MỚI (VAY TIỀN)

---

# 1. Phân tích yêu cầu nghiệp vụ

Khi khách hàng đến cầm cố tài sản để vay tiền, hệ thống cần thực hiện quy trình tạo hợp đồng mới.  

Mỗi hợp đồng phải lưu đầy đủ:

- thông tin khách hàng
- số tiền vay gốc
- ngày vay
- Deadline1
- Deadline2
- danh sách tài sản thế chấp
- trạng thái hợp đồng

Ngoài ra hệ thống phải đảm bảo:

- một khách hàng có thể có nhiều hợp đồng
- một hợp đồng có thể chứa nhiều tài sản
- mỗi tài sản phải có giá trị định giá riêng

Do đó cần sử dụng Stored Procedure để tự động hóa quá trình tiếp nhận hợp đồng mới thay vì nhập thủ công từng bảng.  

---

# 2. Phân tích luồng xử lý

Khi tạo hợp đồng mới, hệ thống sẽ xử lý theo các bước sau:

## Bước 1

Kiểm tra khách hàng đã tồn tại trong hệ thống hay chưa.  

---

## Bước 2

Nếu khách hàng chưa tồn tại thì thêm mới khách hàng.  

---

## Bước 3

Tạo hợp đồng cầm đồ mới.  

---

## Bước 4

Lưu danh sách tài sản khách mang đi cầm cố.  

---

## Bước 5

Thiết lập trạng thái hợp đồng:

```text
Đang vay
```

---

## Bước 6

Thiết lập trạng thái tài sản:

```text
Đang cầm cố
```

---

# 3. Thiết kế dữ liệu đầu vào

Stored Procedure cần nhận các thông tin:

| Tham số | Ý nghĩa |
|---|---|
| @HoTen | Tên khách hàng |
| @CCCD | CCCD khách hàng |
| @SDT | Số điện thoại |
| @DiaChi | Địa chỉ |
| @TienGoc | Số tiền vay |
| @NgayVay | Ngày vay |
| @Deadline1 | Mốc lãi đơn |
| @Deadline2 | Mốc thanh lý |
| @TenTaiSan | Tên tài sản |
| @LoaiTaiSan | Loại tài sản |
| @GiaTriDinhGia | Giá trị tài sản |

---

# 4. Ý nghĩa sử dụng Stored Procedure

Việc sử dụng Stored Procedure giúp:

- giảm thao tác thủ công
- đảm bảo tính toàn vẹn dữ liệu
- tránh nhập sai dữ liệu
- tự động liên kết khóa ngoại
- dễ bảo trì hệ thống

---

# 5. Code Stored Procedure đăng ký hợp đồng mới

## Code SQL

```sql
-- Tạo Stored Procedure đăng ký hợp đồng mới
CREATE PROCEDURE sp_DangKyHopDong
(
    -- Thông tin khách hàng
    @HoTen NVARCHAR(100),
    @CCCD VARCHAR(20),
    @SDT VARCHAR(15),
    @DiaChi NVARCHAR(255),

    -- Thông tin hợp đồng
    @TienGoc DECIMAL(18,2),
    @NgayVay DATE,
    @Deadline1 DATE,
    @Deadline2 DATE,

    -- Thông tin tài sản
    @TenTaiSan NVARCHAR(100),
    @LoaiTaiSan NVARCHAR(100),
    @GiaTriDinhGia DECIMAL(18,2)
)
AS
BEGIN

    -- Khai báo biến mã khách hàng
    DECLARE @MaKH INT;

    -- Khai báo biến mã hợp đồng
    DECLARE @MaHD INT;

    -------------------------------------------------
    -- KIỂM TRA KHÁCH HÀNG ĐÃ TỒN TẠI HAY CHƯA
    -------------------------------------------------

    SELECT @MaKH = MaKH
    FROM KHACH_HANG
    WHERE CCCD = @CCCD;

    -------------------------------------------------
    -- NẾU KHÁCH HÀNG CHƯA TỒN TẠI
    -- THÌ THÊM MỚI KHÁCH HÀNG
    -------------------------------------------------

    IF @MaKH IS NULL
    BEGIN

        INSERT INTO KHACH_HANG
        (
            HoTen,
            CCCD,
            SDT,
            DiaChi
        )
        VALUES
        (
            @HoTen,
            @CCCD,
            @SDT,
            @DiaChi
        );

        -- Lấy mã khách hàng vừa tạo
        SET @MaKH = SCOPE_IDENTITY();

    END;

    -------------------------------------------------
    -- THÊM HỢP ĐỒNG MỚI
    -------------------------------------------------

    INSERT INTO HOP_DONG
    (
        MaKH,
        NgayVay,
        TienGoc,
        Deadline1,
        Deadline2,
        TrangThai
    )
    VALUES
    (
        @MaKH,
        @NgayVay,
        @TienGoc,
        @Deadline1,
        @Deadline2,

        -- Trạng thái ban đầu
        N'Đang vay'
    );

    -- Lấy mã hợp đồng vừa tạo
    SET @MaHD = SCOPE_IDENTITY();

    -------------------------------------------------
    -- THÊM TÀI SẢN CẦM CỐ
    -------------------------------------------------

    INSERT INTO TAI_SAN
    (
        MaHD,
        TenTaiSan,
        LoaiTaiSan,
        GiaTriDinhGia,
        TrangThai
    )
    VALUES
    (
        @MaHD,
        @TenTaiSan,
        @LoaiTaiSan,
        @GiaTriDinhGia,

        -- Trạng thái tài sản
        N'Đang cầm cố'
    );

    -------------------------------------------------
    -- THÔNG BÁO THÀNH CÔNG
    -------------------------------------------------

    PRINT N'Đăng ký hợp đồng thành công';

END;
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/5c3f2510-bcb4-49c2-9467-da2e4970fac6" />

---

# 6. Giải thích code Stored Procedure

## 6.1. Kiểm tra khách hàng

```sql
SELECT @MaKH = MaKH
FROM KHACH_HANG
WHERE CCCD = @CCCD;
```

Đoạn code dùng để kiểm tra khách hàng đã tồn tại hay chưa thông qua số CCCD.  

Điều này giúp tránh trùng dữ liệu khách hàng trong hệ thống.  

---

## 6.2. Thêm khách hàng mới

```sql
INSERT INTO KHACH_HANG
```

Nếu khách hàng chưa tồn tại thì hệ thống tự động thêm mới thông tin khách hàng.  

Sau đó dùng:

```sql
SCOPE_IDENTITY()
```

để lấy mã khách hàng vừa tạo.  

---

## 6.3. Tạo hợp đồng

```sql
INSERT INTO HOP_DONG
```

Hệ thống tạo hợp đồng mới và lưu:

- tiền gốc
- ngày vay
- Deadline1
- Deadline2
- trạng thái hợp đồng

Trạng thái mặc định:

```text
Đang vay
```

---

## 6.4. Thêm tài sản cầm cố

```sql
INSERT INTO TAI_SAN
```

Thông tin tài sản được lưu riêng nhằm đảm bảo chuẩn hóa dữ liệu mức 3NF.  

Mỗi tài sản có:

- tên riêng
- loại riêng
- giá trị định giá riêng
- trạng thái riêng

---

## 6.5. Ý nghĩa trạng thái tài sản

Khi vừa tạo hợp đồng:

```text
Trạng thái tài sản = Đang cầm cố
```

Điều này giúp hệ thống biết tài sản hiện đang được cửa hàng giữ để đảm bảo khoản vay.  

---

# Ví dụ chạy Stored Procedure

## Code thực thi

```sql
EXEC sp_DangKyHopDong
    @HoTen = N'Nguyễn Văn A',
    @CCCD = '012345678901',
    @SDT = '0988888888',
    @DiaChi = N'Hà Nội',

    @TienGoc = 10000000,
    @NgayVay = '2026-05-10',
    @Deadline1 = '2026-05-20',
    @Deadline2 = '2026-06-05',

    @TenTaiSan = N'Laptop Dell',
    @LoaiTaiSan = N'Laptop',
    @GiaTriDinhGia = 15000000;
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/e89e4362-8251-4449-805e-341ac4b2fd43" />

## Giải thích

Ví dụ trên thực hiện:

- tạo khách hàng Nguyễn Văn A
- tạo hợp đồng vay 10 triệu
- tạo tài sản Laptop Dell
- thiết lập trạng thái hợp đồng và tài sản

---

# Kết quả sau khi thực hiện

Sau khi chạy Stored Procedure, dữ liệu sẽ được lưu vào:

- bảng KHACH_HANG
- bảng HOP_DONG
- bảng TAI_SAN

Các bảng được liên kết với nhau thông qua khóa ngoại nhằm đảm bảo tính toàn vẹn dữ liệu.  


# EVENT 2: TÍNH TOÁN CÔNG NỢ THỜI GIAN THỰC

---

# 1. Phân tích logic

Hệ thống cần tính số tiền khách phải trả tại một thời điểm bất kỳ.  

Khoản tiền này gồm:

- tiền gốc
- lãi đơn trước Deadline1
- lãi kép sau Deadline1

Nếu chưa quá Deadline1 thì chỉ tính lãi đơn.  

Nếu đã vượt Deadline1 thì tiếp tục tính thêm lãi kép dựa trên:

```text
Gốc + lãi đơn
```

---

# 2. Function fn_CalcMoneyTransaction

Function này dùng để tính số tiền phải trả của một hợp đồng tại ngày được yêu cầu.  

---

## Code Function

```sql
-- Hàm tính tiền giao dịch
CREATE FUNCTION fn_CalcMoneyTransaction
(
    @TransactionID INT,
    @TargetDate DATE
)
RETURNS DECIMAL(18,2)
AS
BEGIN

    -- Khai báo biến
    DECLARE @TienGoc DECIMAL(18,2);
    DECLARE @NgayVay DATE;
    DECLARE @Deadline1 DATE;
    DECLARE @SoNgay INT;
    DECLARE @TienLai DECIMAL(18,2);
    DECLARE @TongTien DECIMAL(18,2);

    -- Lấy thông tin hợp đồng
    SELECT
        @TienGoc = TienGoc,
        @NgayVay = NgayVay,
        @Deadline1 = Deadline1
    FROM HOP_DONG
    WHERE MaHD = @TransactionID;

    -- Tính số ngày vay
    SET @SoNgay = DATEDIFF(DAY, @NgayVay, @TargetDate);

    -- Tính lãi đơn
    SET @TienLai = @TienGoc * 0.005 * @SoNgay;

    -- Tổng tiền
    SET @TongTien = @TienGoc + @TienLai;

    RETURN @TongTien;

END;
```
<img width="1915" height="1078" alt="image" src="https://github.com/user-attachments/assets/e4257aaa-1cce-41db-80c2-090f1d45f1f4" />

KẾT QUẢ FUNCTION fn_CalcMoneyTransaction
---

## Giải thích code

- `DATEDIFF` dùng để tính số ngày vay.  
- `0.005` là lãi suất mỗi ngày.  
- Function trả về tổng tiền gồm gốc và lãi đơn.  

---

# 3. Function fn_CalcMoneyContract

Function này dùng để tính toàn bộ công nợ gồm:

- gốc
- lãi đơn
- lãi kép

---

## Code Function

```sql
-- Hàm tính tổng công nợ hợp đồng
CREATE FUNCTION fn_CalcMoneyContract
(
    @ContractID INT,
    @TargetDate DATE
)
RETURNS DECIMAL(18,2)
AS
BEGIN

    -- Khai báo biến
    DECLARE @TienGoc DECIMAL(18,2);
    DECLARE @NgayVay DATE;
    DECLARE @Deadline1 DATE;

    DECLARE @SoNgayLaiDon INT;
    DECLARE @SoNgayLaiKep INT;

    DECLARE @TienLaiDon DECIMAL(18,2);
    DECLARE @TienSauLaiDon DECIMAL(18,2);

    DECLARE @TongTien DECIMAL(18,2);

    -- Lấy dữ liệu hợp đồng
    SELECT
        @TienGoc = TienGoc,
        @NgayVay = NgayVay,
        @Deadline1 = Deadline1
    FROM HOP_DONG
    WHERE MaHD = @ContractID;

    ------------------------------------------------
    -- Nếu chưa quá Deadline1
    ------------------------------------------------

    IF @TargetDate <= @Deadline1
    BEGIN

        SET @SoNgayLaiDon =
        DATEDIFF(DAY, @NgayVay, @TargetDate);

        SET @TienLaiDon =
        @TienGoc * 0.005 * @SoNgayLaiDon;

        SET @TongTien =
        @TienGoc + @TienLaiDon;

    END

    ------------------------------------------------
    -- Nếu đã quá Deadline1
    ------------------------------------------------

    ELSE
    BEGIN

        -- Số ngày lãi đơn
        SET @SoNgayLaiDon =
        DATEDIFF(DAY, @NgayVay, @Deadline1);

        -- Tính lãi đơn
        SET @TienLaiDon =
        @TienGoc * 0.005 * @SoNgayLaiDon;

        -- Tiền sau lãi đơn
        SET @TienSauLaiDon =
        @TienGoc + @TienLaiDon;

        -- Số ngày lãi kép
        SET @SoNgayLaiKep =
        DATEDIFF(DAY, @Deadline1, @TargetDate);

        -- Tính lãi kép
        SET @TongTien =
        @TienSauLaiDon *
        POWER(1.005, @SoNgayLaiKep);

    END;

    RETURN @TongTien;

END;
```

---

## Giải thích code

### Nếu chưa quá Deadline1

Hệ thống chỉ tính:

```text
Gốc + lãi đơn
```

---

### Nếu đã quá Deadline1

Hệ thống:

- tính lãi đơn trước
- cộng vào tiền gốc
- tiếp tục tính lãi kép bằng `POWER()`

---

# 4. Ví dụ chạy Function

## Code chạy thử

```sql
SELECT dbo.fn_CalcMoneyTransaction(1, '2026-05-20');

SELECT dbo.fn_CalcMoneyContract(1, '2026-06-10');
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/1dccb5de-22ee-4b9d-888c-786083c13601" />

Kết quả sử dụng 2 hàm

---

## Giải thích

- Function đầu tính tiền giao dịch thông thường.  
- Function thứ hai tính toàn bộ công nợ có cả lãi kép.  

---


# EVENT 3: XỬ LÝ TRẢ NỢ VÀ HOÀN TRẢ TÀI SẢN

---

# 1. Phân tích logic

Khi khách mang tiền đến trả, hệ thống cần:

- kiểm tra tài sản đã thanh lý chưa
- tính số tiền còn nợ
- cập nhật trạng thái hợp đồng
- lưu lịch sử thanh toán

Nếu tài sản đã bán thanh lý thì:

```text
Không thu tiền và không trả tài sản
```

Nếu khách trả hết nợ:

```text
Trả toàn bộ tài sản
Cập nhật trạng thái: Đã thanh toán đủ
```

Nếu chưa trả hết:

```text
Cập nhật trạng thái: Đang trả góp
```

---

# 2. Store Procedure xử lý trả nợ

## Code SQL

```sql
-- Procedure xử lý trả nợ
CREATE PROCEDURE sp_TraNo
(
    @MaHD INT,
    @SoTienTra DECIMAL(18,2)
)
AS
BEGIN

    -- Tổng nợ
    DECLARE @TongNo DECIMAL(18,2);

    -- Tiền còn nợ
    DECLARE @ConNo DECIMAL(18,2);

    -- Trạng thái tài sản
    DECLARE @TrangThai NVARCHAR(50);

    -- Lấy trạng thái tài sản
    SELECT TOP 1
        @TrangThai = TrangThai
    FROM TAI_SAN
    WHERE MaHD = @MaHD;

    -- Nếu đã thanh lý
    IF @TrangThai = N'Đã bán thanh lý'
    BEGIN

        PRINT N'Tài sản đã thanh lý';

        RETURN;

    END;

    -- Tính tổng nợ
    SET @TongNo =
    dbo.fn_CalcMoneyContract
    (
        @MaHD,
        GETDATE()
    );

    -- Tính tiền còn nợ
    SET @ConNo =
    @TongNo - @SoTienTra;

    -- Lưu thanh toán
    INSERT INTO THANH_TOAN
    (
        MaHD,
        NgayTra,
        SoTienTra
    )
    VALUES
    (
        @MaHD,
        GETDATE(),
        @SoTienTra
    );

    -- Nếu trả hết
    IF @ConNo <= 0
    BEGIN

        UPDATE HOP_DONG
        SET TrangThai = N'Đã thanh toán đủ'
        WHERE MaHD = @MaHD;

        UPDATE TAI_SAN
        SET TrangThai = N'Đã trả khách'
        WHERE MaHD = @MaHD;

        PRINT N'Khách đã trả hết nợ';

    END

    -- Nếu chưa trả hết
    ELSE
    BEGIN

        UPDATE HOP_DONG
        SET TrangThai = N'Đang trả góp'
        WHERE MaHD = @MaHD;

        PRINT N'Khách chưa trả hết';

    END;

END;
GO

```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/3fdb6b5a-c4dd-4889-9ca8-157d40810f47" />

---

# 3. Giải thích code

- `fn_CalcMoneyContract()` dùng để tính tổng nợ hiện tại.  
- Nếu tài sản đã thanh lý thì dừng xử lý.  
- Hệ thống lưu lịch sử thanh toán vào bảng `THANH_TOAN`.  
- Nếu khách trả hết nợ:
  - trả tài sản
  - cập nhật trạng thái hợp đồng  
- Nếu chưa trả hết:
  - cập nhật trạng thái đang trả góp  

---

# 4. Ví dụ chạy Procedure

## Code chạy thử

```sql
EXEC sp_TraNo
    @MaHD = 1,
    @SoTienTra = 5000000;
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/83b3278e-164c-48f9-b8d1-fa40db0ef868" />

 KẾT QUẢ CHẠY PROCEDURE TRẢ NỢ


# EVENT 4: TRUY VẤN DANH SÁCH NỢ XẤU

---

# 1. Phân tích logic

Hệ thống cần tìm các khách hàng:

- đã quá Deadline1
- chưa thanh toán hết nợ

Danh sách cần hiển thị:

- tên khách hàng
- số điện thoại
- số tiền vay gốc
- số ngày quá hạn
- tổng tiền phải trả hiện tại
- tổng tiền phải trả sau 1 tháng

Hệ thống sử dụng function:

```sql
fn_CalcMoneyContract()
```

để tính công nợ tự động.  

---

# 2. Query truy vấn nợ xấu

## Code SQL

```sql
-- Truy vấn danh sách nợ xấu
SELECT

    -- Tên khách hàng
    KH.HoTen,

    -- Số điện thoại
    KH.SDT,

    -- Tiền vay gốc
    HD.TienGoc,

    -- Số ngày quá hạn
    DATEDIFF(DAY, HD.Deadline1, GETDATE())
    AS SoNgayQuaHan,

    -- Tổng tiền hiện tại
    dbo.fn_CalcMoneyContract
    (
        HD.MaHD,
        GETDATE()
    )
    AS TongTienHienTai,

    -- Tổng tiền sau 1 tháng
    dbo.fn_CalcMoneyContract
    (
        HD.MaHD,
        DATEADD(MONTH, 1, GETDATE())
    )
    AS TongTienSau1Thang

FROM HOP_DONG HD

INNER JOIN KHACH_HANG KH
ON HD.MaKH = KH.MaKH

WHERE

    -- Quá Deadline1
    GETDATE() > HD.Deadline1

    -- Chưa thanh toán đủ
    AND HD.TrangThai
    <> N'Đã thanh toán đủ';
```

---

# 3. Giải thích code

- `DATEDIFF()` dùng để tính số ngày quá hạn.  
- `fn_CalcMoneyContract()` dùng để tính tổng công nợ.  
- `DATEADD(MONTH,1,GETDATE())` dùng để tính tiền sau 1 tháng nữa.  
- `INNER JOIN` dùng để lấy thông tin khách hàng từ bảng hợp đồng.  

---

Danh sách hiển thị gồm:

| HoTen | SDT | TienGoc | SoNgayQuaHan | TongTienHienTai | TongTienSau1Thang |
|---|---|---|---|---|---|

---
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/f95d7427-268f-4529-9476-159b797126dd" />

 KẾT QUẢ TRUY VẤN NỢ XẤU



# EVENT 5: QUẢN LÝ THANH LÝ TÀI SẢN

---

# 1. Phân tích logic

Hệ thống cần tự động cập nhật trạng thái hợp đồng và tài sản khi quá hạn.  

Các trạng thái gồm:

- Đang vay
- Quá hạn
- Sẵn sàng thanh lý
- Đã bán thanh lý

Trigger giúp hệ thống tự động xử lý khi dữ liệu thay đổi.  

---

# 2. Trigger chuyển hợp đồng sang nợ xấu

Nếu hợp đồng:

- đang vay
- vượt quá Deadline1

thì chuyển sang:

```text
Quá hạn (nợ xấu)
```

---

## Code Trigger

```sql
-- Trigger cập nhật trạng thái nợ xấu
CREATE TRIGGER trg_QuaHanHopDong
ON HOP_DONG
AFTER UPDATE
AS
BEGIN

    UPDATE HOP_DONG
    SET TrangThai = N'Quá hạn (nợ xấu)'

    WHERE
        TrangThai = N'Đang vay'
        AND GETDATE() > Deadline1;

END;
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/f3f87539-8191-4905-8adf-cff2ea5119f9" />

Trigger cập nhật trạng thái nợ xấu




# 3. Trigger chuyển tài sản sang sẵn sàng thanh lý

Nếu hợp đồng:

- đã quá hạn
- vượt Deadline2

thì tài sản chuyển sang:

```text
Sẵn sàng thanh lý
```

---

## Code Trigger

```sql
-- Trigger chuyển tài sản sang thanh lý
CREATE TRIGGER trg_SanSangThanhLy
ON HOP_DONG
AFTER UPDATE
AS
BEGIN

    UPDATE TAI_SAN
    SET TrangThai = N'Sẵn sàng thanh lý'

    WHERE MaHD IN
    (
        SELECT MaHD
        FROM HOP_DONG
        WHERE
            TrangThai = N'Quá hạn (nợ xấu)'
            AND GETDATE() > Deadline2
    );

END;
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/875c47bf-e1eb-4d67-a09b-d72831fdf447" />
Trigger chuyển tài sản sang thanh lý

# 4. Trigger cập nhật đã bán thanh lý

Nếu hợp đồng chuyển sang:

```text
Đã thanh lý
```

thì tài sản chuyển sang:

```text
Đã bán thanh lý
```

---

## Code Trigger

```sql
-- Trigger cập nhật đã bán thanh lý
CREATE TRIGGER trg_DaBanThanhLy
ON HOP_DONG
AFTER UPDATE
AS
BEGIN

    UPDATE TAI_SAN
    SET TrangThai = N'Đã bán thanh lý'

    WHERE MaHD IN
    (
        SELECT MaHD
        FROM HOP_DONG
        WHERE TrangThai = N'Đã thanh lý'
    );

END;
```
<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/62df9f39-5cb3-455d-b668-59808c3acbe6" />
Trigger cập nhật đã bán thanh lý


# 4. CÁC SỰ KIỆN BỔ SUNG

---

# 4.1. Sự kiện gia hạn hợp đồng

## Phân tích logic

Khi khách hàng muốn gia hạn hợp đồng, khách phải trả toàn bộ tiền lãi hiện tại.  

Sau khi thanh toán lãi:

- Deadline1 được dời sang thời hạn mới
- Deadline2 cũng được cập nhật lại
- hợp đồng tránh bị tính lãi kép

---

## Code Procedure gia hạn hợp đồng

```sql
-- Procedure gia hạn hợp đồng
CREATE PROCEDURE sp_GiaHanHopDong
(
    @MaHD INT,
    @Deadline1Moi DATE,
    @Deadline2Moi DATE
)
AS
BEGIN

    -- Cập nhật thời hạn mới
    UPDATE HOP_DONG
    SET
        Deadline1 = @Deadline1Moi,
        Deadline2 = @Deadline2Moi

    WHERE MaHD = @MaHD;

    PRINT N'Gia hạn hợp đồng thành công';

END;
```

---

## Giải thích code

- Procedure dùng để cập nhật thời hạn mới cho hợp đồng.  
- Sau khi gia hạn, hệ thống tiếp tục tính lãi từ mốc mới.  

---

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/4b7133fd-3521-409f-9a17-6140ed2e7e42" />

Procedure gia hạn hợp đồng

# 4.2. Lịch sử hợp đồng (Audit Log)

## Phân tích logic

Hệ thống cần lưu lịch sử mỗi lần khách trả tiền nhằm:

- theo dõi dòng tiền
- tránh mất dữ liệu thanh toán
- hỗ trợ kiểm tra giao dịch

Thông tin cần lưu:

- ngày trả
- số tiền trả
- người thu tiền

---

## Code tạo bảng LOG_THANH_TOAN

```sql
-- Bảng lưu lịch sử thanh toán
CREATE TABLE LOG_THANH_TOAN
(
    -- Mã log
    MaLog INT PRIMARY KEY IDENTITY(1,1),

    -- Mã hợp đồng
    MaHD INT,

    -- Ngày trả tiền
    NgayTra DATE,

    -- Số tiền khách trả
    SoTienTra DECIMAL(18,2),

    -- Người thu tiền
    NguoiThu NVARCHAR(100)
);
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/9adcda68-0c85-4e41-a9ae-031abd7dbfe3" />

tạo bảng LOG_THANH_TOAN

---

## Giải thích code

- Mỗi lần khách trả tiền sẽ tạo một dòng log mới.  
- Hệ thống không ghi đè dữ liệu cũ để tránh mất lịch sử giao dịch.  

---

## Ví dụ thêm log thanh toán

```sql
INSERT INTO LOG_THANH_TOAN
(
    MaHD,
    NgayTra,
    SoTienTra,
    NguoiThu
)
VALUES
(
    1,
    GETDATE(),
    3000000,
    N'Nguyễn Văn B'
);
```

<img width="1918" height="1078" alt="image" src="https://github.com/user-attachments/assets/1cca8427-c90b-4e78-881b-48d3bb2c9409" />

 Lịch sử hợp đồng (Audit Log)


---

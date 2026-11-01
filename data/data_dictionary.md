# Data Dictionary - Global Electronic Retailer

Tài liệu giải nghĩa cấu trúc dữ liệu, quan hệ bảng và các trường phái sinh phục vụ phân tích hoạt động kinh doanh chuỗi bán lẻ điện tử toàn cầu (2016 - 2021).

---

## 1. Mô hình dữ liệu (Data Architecture)

Dữ liệu được tổ chức theo mô hình hình sao (Star Schema) chuẩn Business Intelligence:
* **Bảng Fact:** `Sales` (Giao dịch bán hàng)
* **Bảng Dimension:** `Customers`, `Products`, `Stores`, `Exchange_Rates`

```text
       +--------------------+          +--------------------+
       |     Customers      |          |       Stores       |
       |--------------------|          |--------------------|
       | CustomerKey (PK)   |          | StoreKey (PK)      |
       +---------+----------+          +---------+----------+
                 | 1                             | 1
                 |                               |
                 | N                             | N
            +----+-------------------------------+----+
            |                    Sales                |
            |-----------------------------------------|
            | Order Number, Line Item (PK)            |
            | CustomerKey (FK)                        |
            | StoreKey (FK)                           |
            | ProductKey (FK)                         |
            | Order Date, Delivery Date               |
            | Quantity, Currency Code                 |
            +----+-------------------------------+----+
                 | N                             | N
                 |                               |
                 | 1                             | 1
       +---------+----------+          +---------+----------+
       |      Products      |          |   Exchange_Rates   |
       |--------------------|          |--------------------|
       | ProductKey (PK)    |          | Date, Currency (PK)|
       +--------------------+          +--------------------+
```

---

## 2. Chi tiết các bảng dữ liệu gốc (Raw Tables)

### Bảng 1: `Sales` (Fact Table - 62.884 dòng)
Chứa thông tin chi tiết từng dòng đơn hàng phát sinh từ ngày 01/01/2016 đến ngày 20/02/2021.

| Tên trường | Kiểu dữ liệu | Khóa | Diễn giải kinh doanh |
|---|---|:---:|---|
| `Order Number` | Integer | PK | Mã định danh đơn hàng |
| `Line Item` | Integer | PK | Thứ tự sản phẩm trong đơn hàng |
| `Order Date` | Date | | Ngày khách hàng đặt hàng |
| `Delivery Date` | Date | | Ngày đơn hàng được giao thành công (thiếu 79% do mua trực tiếp tại showroom) |
| `CustomerKey` | Integer | FK | Mã định danh khách hàng (liên kết sang `Customers`) |
| `StoreKey` | Integer | FK | Mã cửa hàng (0: Online; 1-66: Showroom vật lý) |
| `ProductKey` | Integer | FK | Mã sản phẩm (liên kết sang `Products`) |
| `Quantity` | Integer | | Số lượng sản phẩm mua |
| `Currency Code` | Varchar(3) | | Loại tiền tệ thanh toán (USD, CAD, AUD, EUR, GBP) |

### Bảng 2: `Customers` (Dimension Table - 15.266 dòng)
Thông tin hồ sơ khách hàng cá nhân.

| Tên trường | Kiểu dữ liệu | Khóa | Diễn giải kinh doanh |
|---|---|:---:|---|
| `CustomerKey` | Integer | PK | Mã định danh duy nhất của khách hàng |
| `Gender` | Varchar(10) | | Giới tính (Male / Female) |
| `Name` | Varchar(100)| | Họ và tên khách hàng |
| `City` | Varchar(100)| | Thành phố cư trú |
| `State Code` | Varchar(20) | | Mã bang/tỉnh (thiếu 10 dòng) |
| `State` | Varchar(100)| | Tên đầy đủ của bang/tỉnh |
| `Zip Code` | Varchar(20) | | Mã bưu chính |
| `Country` | Varchar(50) | | Quốc gia cư trú |
| `Continent` | Varchar(50) | | Châu lục (North America, Europe, Australia) |
| `Birthday` | Date | | Ngày sinh của khách hàng |

### Bảng 3: `Products` (Dimension Table - 2.517 dòng)
Danh mục sản phẩm điện tử, giá vốn và giá niêm yết bán lẻ.

| Tên trường | Kiểu dữ liệu | Khóa | Diễn giải kinh doanh |
|---|---|:---:|---|
| `ProductKey` | Integer | PK | Mã định danh sản phẩm |
| `Product Name` | Varchar(255)| | Tên thương mại đầy đủ của sản phẩm |
| `Brand` | Varchar(50) | | Nhãn hiệu sản xuất (Contoso, Fabrikam, Litware, Adventure Works, ...) |
| `Color` | Varchar(30) | | Màu sắc sản phẩm |
| `Unit Cost USD` | Float | | Giá vốn đơn vị tính bằng USD (sau khi bóc tách ký tự `$`) |
| `Unit Price USD` | Float | | Giá bán lẻ niêm yết tính bằng USD (sau khi bóc tách ký tự `$`) |
| `SubcategoryKey` | Integer | | Mã ngành hàng phụ (32 tiểu ngạch) |
| `Subcategory` | Varchar(100)| | Tên ngành hàng phụ (Desktops, Televisions, Movie DVD, ...) |
| `CategoryKey` | Integer | | Mã ngành hàng chính (8 ngành hàng) |
| `Category` | Varchar(100)| | Tên ngành hàng chính (Computers, Home Appliances, Cameras, ...) |

### Bảng 4: `Stores` (Dimension Table - 67 dòng)
Danh sách mạng lưới 66 showroom bán lẻ và 1 kênh phân phối trực tuyến.

| Tên trường | Kiểu dữ liệu | Khóa | Diễn giải kinh doanh |
|---|---|:---:|---|
| `StoreKey` | Integer | PK | Mã định danh cửa hàng (0: Online; 1-66: Cửa hàng vật lý) |
| `Country` | Varchar(50) | | Quốc gia đặt cửa hàng |
| `State` | Varchar(100)| | Bang/Tỉnh đặt cửa hàng |
| `Square Meters`| Integer | | Diện tích mặt bằng cửa hàng (m2) |
| `Open Date` | Date | | Ngày showroom chính thức khai trương |

### Bảng 5: `Exchange_Rates` (Dimension Table - 11.215 dòng)
Bảng tỷ giá hối đoái hàng ngày quy đổi các ngoại tệ sang USD.

| Tên trường | Kiểu dữ liệu | Khóa | Diễn giải kinh doanh |
|---|---|:---:|---|
| `Date` | Date | PK | Ngày ghi nhận tỷ giá |
| `Currency` | Varchar(3) | PK | Mã tiền tệ (AUD, CAD, EUR, GBP) |
| `Exchange` | Float | | Tỷ giá quy đổi về đồng USD |

---

## 3. Các chỉ tiêu phái sinh & Kỹ thuật Feature Engineering

| Tên chỉ tiêu | Công thức tính toán / Định nghĩa | Ý nghĩa phân tích |
|---|---|---|
| `Channel` | `IF StoreKey == 0 THEN 'Online' ELSE 'In-Store'` | Phân loại kênh bán hàng |
| `Revenue` | `Quantity * Unit Price USD` | Tổng doanh thu đơn hàng (USD) |
| `Cost` | `Quantity * Unit Cost USD` | Tổng giá vốn hàng bán (COGS, USD) |
| `Profit` | `Revenue - Cost` | Lợi nhuận gộp (USD) |
| `Margin (%)` | `(Profit / Revenue) * 100` | Biên lợi nhuận gộp |
| `Delivery Days` | `Delivery Date - Order Date` | Thời gian giao hàng thực tế (chỉ tính cho đơn có giao nhận) |
| `Contract age` | `(Order Date Max - First Purchase Date) / 365.25` (chặn dưới 1.0) | Tuổi hợp đồng của khách hàng (chuẩn hóa tránh bias khách mới) |
| `Recency (R)` | `Order Date Max - Customer Last Purchase Date` | Số ngày kể từ lần giao dịch gần nhất |
| `Frequency (F)` | `Unique Order Days / Contract age` | Tần suất số ngày mua sắm trung bình mỗi năm |
| `Monetary (M)` | `Total Revenue / Contract age` | Giá trị chi tiêu trung bình mỗi năm |
| `R, F, M Score` | Xếp hạng từ 1 đến 4 dựa trên các mốc phân vị 25%, 50%, 75% | Điểm chuẩn hóa hành vi khách hàng |
| `RFM_Code` | Nối chuỗi `str(R) + str(F) + str(M)` (ví dụ: '444', '111') | Mã phân khúc vi mô (64 nhóm kết hợp) |
| `BCG_Matrix` | Phân loại 4 nhóm: Star (VIP), Question Mark (Tiềm năng), Cash Cow (Trung thành), Dog (Vãng lai) | Ma trận định vị phân khúc khách hàng chiến lược |


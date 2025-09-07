# importPubHTML[huong-dan-importPubHtml.md](https://github.com/user-attachments/files/22196501/huong-dan-importPubHtml.md)
# 📖 Hướng dẫn sử dụng script `importPubHtml`

## 1. Mục đích

Hàm `importPubHtml` dùng để **lấy dữ liệu từ Google Sheets đã publish (ở
dạng CSV)** và chép dữ liệu vào file Google Sheets nội bộ của bạn.\
- Có thể lấy từ nhiều file (nhiều link CSV).\
- Có thể đổ dữ liệu vào nhiều sheet khác nhau.\
- Có thể loại bỏ những cột không cần thiết trước khi ghi.

------------------------------------------------------------------------

## 2. Cấu trúc code

``` js
function importPubHtml() {
  // 1. Link dữ liệu nguồn (file Google Sheet publish ra)
  var dataNgay = "LINK_1";  
  var dataLuong = "LINK_2";

  // 2. Lấy file Google Sheet hiện tại
  var ss = SpreadsheetApp.getActiveSpreadsheet();

  // 3. Lấy sheet cần ghi dữ liệu
  var sheet1 = ss.getSheets()[0]; // sheet đầu tiên
  var sheet2 = ss.getSheets()[1]; // sheet thứ hai

  // 4. Gọi API để lấy dữ liệu CSV
  var response1 = UrlFetchApp.fetch(dataNgay);
  var response2 = UrlFetchApp.fetch(dataLuong);

  // 5. Lấy nội dung text CSV
  var text1 = response1.getContentText();
  var text2 = response2.getContentText();

  // 6. Chuyển CSV thành mảng 2 chiều
  const data1 = Utilities.parseCsv(text1);
  var data2 = Utilities.parseCsv(text2);

  // 7. Loại bỏ một số cột trong data2 (theo chỉ số cột)
  const loaibo = [11, 12, 18]; 
  data2 = data2.map(row => row.filter((_, i) => !loaibo.includes(i)));

  // 8. Xóa dữ liệu cũ
  sheet1.clearContents();
  sheet2.clearContents();

  // 9. Ghi dữ liệu mới vào sheet
  sheet1.getRange(1, 1, data1.length, data1[0].length).setValues(data1);
  console.log("Đã xong data Ngày");

  sheet2.getRange(1, 1, data2.length, data2[0].length).setValues(data2);
  console.log("Đã xong data Lương");
}
```

------------------------------------------------------------------------

## 3. Giải thích từng phần

### 🔗 Link dữ liệu nguồn

``` js
var dataNgay = "https://docs.google.com/spreadsheets/d/e/.../pub?gid=2061515320&single=true&output=csv";
```

-   Đây là link xuất dữ liệu CSV từ một **Google Sheet được publish ra
    web**.\
-   `gid=2061515320`: chính là **ID của sheet (tab)** trong file đó.

👉 Cách lấy `gid`:\
1. Mở Google Sheet.\
2. Chọn tab bạn muốn.\
3. Trên URL sẽ thấy `gid=XXXX`. Copy con số này.

Ví dụ:

    https://docs.google.com/spreadsheets/d/FILE_ID/edit#gid=2061515320

→ `gid = 2061515320`

------------------------------------------------------------------------

### 📄 Chỉ định Sheet trong file đích

``` js
var sheet1 = ss.getSheets()[0]; // sheet đầu tiên
var sheet2 = ss.getSheets()[1]; // sheet thứ hai
```

-   `ss.getSheets()[0]`: sheet đầu tiên trong file.\
-   `ss.getSheets()[1]`: sheet thứ hai.

👉 Nếu muốn lấy theo tên sheet:

``` js
var sheet1 = ss.getSheetByName("Ngày công");
var sheet2 = ss.getSheetByName("Lương");
```

------------------------------------------------------------------------

### 🧹 Xóa dữ liệu cũ trước khi ghi

``` js
sheet1.clearContents();
sheet2.clearContents();
```

Đảm bảo dữ liệu mới **ghi đè hoàn toàn** và không bị dính dữ liệu thừa
từ lần trước.

------------------------------------------------------------------------

### ✂️ Loại bỏ cột

``` js
const loaibo = [11, 12, 18];
data2 = data2.map(row => row.filter((_, i) => !loaibo.includes(i)));
```

-   `loaibo = [11, 12, 18]`: nghĩa là bỏ **cột thứ 12, 13 và 19** (vì
    index tính từ 0).\
-   Mỗi hàng (`row`) sẽ được lọc bỏ những cột nằm trong danh sách này.

👉 Nếu muốn loại bỏ theo **tên cột (header)** thì cần viết thêm một đoạn
code xử lý dòng đầu tiên.

------------------------------------------------------------------------

## 4. Cách thay đổi / bổ sung

### 🔹 Thay file nguồn

-   Chỉ cần đổi link `dataNgay`, `dataLuong` sang link CSV mới.\
-   Đảm bảo bạn đã **Publish to web** file đó trong Google Sheets.

### 🔹 Bổ sung thêm sheet

Ví dụ muốn thêm `dataThuong` → sheet3:

``` js
var dataThuong = "LINK_3";
var response3 = UrlFetchApp.fetch(dataThuong);
var text3 = response3.getContentText();
var data3 = Utilities.parseCsv(text3);

var sheet3 = ss.getSheets()[2]; // hoặc getSheetByName("Thưởng")
sheet3.clearContents();
sheet3.getRange(1, 1, data3.length, data3[0].length).setValues(data3);
```

### 🔹 Đổi danh sách cột cần loại bỏ

``` js
const loaibo = [0, 2, 5]; // bỏ cột 1, 3, 6
```

------------------------------------------------------------------------

## 5. Lịch chạy tự động

-   Vào **Apps Script \> Triggers \> Add Trigger**.\
-   Chọn hàm `importPubHtml`.\
-   Chọn lịch: ví dụ mỗi 6 tiếng, mỗi ngày, ...

------------------------------------------------------------------------

👉 Với tài liệu này, nhân sự của bạn chỉ cần:\
1. Lấy link CSV của file nguồn (với gid đúng).\
2. Cập nhật link trong code.\
3. Chỉnh cột cần bỏ.\
4. Chỉnh sheet đích.

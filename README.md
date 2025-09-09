[huong-dan-importPubHtml-v2.md](https://github.com/user-attachments/files/22229131/huong-dan-importPubHtml-v2.md)
# 📘 Hướng dẫn sử dụng hàm `importPubHtml` (phiên bản tối ưu theo index)

Hàm này giúp tự động **lấy dữ liệu từ Google Sheet đã publish (CSV)** và
**đồng bộ về file Google Sheet của bạn**.

------------------------------------------------------------------------

## 🚀 Cách hoạt động

-   Sử dụng **link gốc (BASE_URL + gid + SUFFIX)** để tạo URL CSV.
-   Duyệt qua danh sách `jobs`, mỗi phần tử trong mảng tương ứng với một
    sheet đích.
-   Sheet được xác định **theo index của job** (job đầu tiên = sheet 0,
    job thứ hai = sheet 1, ...).
-   Tự động đổi tên sheet theo config trong `jobs`.
-   Nếu sheet chưa tồn tại → tạo mới.
-   Xóa dữ liệu cũ và ghi dữ liệu mới từ file nguồn.

------------------------------------------------------------------------

## 📝 Cấu trúc config `jobs`

``` js
const jobs = [
  [<gid>, <sheetName>, <removeCols>]
];
```

### Tham số:

1.  **gid**: ID của sheet con trong Google Sheet nguồn

    -   Lấy bằng cách xem link public CSV. Ví dụ:

            https://docs.google.com/spreadsheets/.../pub?gid=2061515320&single=true&output=csv

        → `gid = 2061515320`

2.  **sheetName**: Tên sheet trong file Google Sheet đích (sẽ được đổi
    tên cho khớp).

3.  **removeCols**: Mảng các số cột cần loại bỏ (theo index bắt đầu từ
    0).

    -   Ví dụ `[11, 12, 18]` nghĩa là loại bỏ 3 cột 12, 13, 19.

------------------------------------------------------------------------

## 📂 Ví dụ đầy đủ

``` js
function importPubHtml() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const BASE_URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vRcAGHkcxNak7SrXxwkkft0g3R2r4cIqrKrv5HcT-wqPmDDnFZGjYURQyiEanrBBTX2JzgcmiMyL7RU/pub?gid=";
  const SUFFIX   = "&single=true&output=csv";

  const jobs = [
    [2061515320, "Data_Ngay", []],
    [1583919753, "Data_Luong", [11, 12, 18]]
  ];

  jobs.forEach(([gid, sheetName, removeCols], idx) => {
    let url  = BASE_URL + gid + SUFFIX;
    let data = Utilities.parseCsv(UrlFetchApp.fetch(url).getContentText());
    if (removeCols.length) data = data.map(r => r.filter((_, i) => !removeCols.includes(i)));

    let sheet = ss.getSheets()[idx] || ss.insertSheet();
    sheet.setName(sheetName);
    sheet.clearContents();
    if (data.length) sheet.getRange(1, 1, data.length, data[0].length).setValues(data);

    console.log("Đã xong:", sheetName);
  });
}
```

------------------------------------------------------------------------

## 🔧 Tùy chỉnh

-   **Thêm sheet mới**: chỉ cần thêm 1 dòng trong `jobs`.\
-   **Đổi tên sheet**: thay đổi `sheetName` trong config.\
-   **Loại bỏ cột**: thêm index cột cần loại bỏ vào `removeCols`.\
-   **Thứ tự sheet trong file đích**: phụ thuộc vào vị trí trong `jobs`.

------------------------------------------------------------------------

## ⏰ Chạy tự động

-   Mở menu **Extensions \> Triggers** trong Google Apps Script.\
-   Đặt trigger cho hàm `importPubHtml` chạy **5 tiếng một lần** (hoặc
    chu kỳ khác).

------------------------------------------------------------------------

✅ Với cách này, bạn có thể quản lý nhiều file Google Sheet publish mà
không lo trùng dữ liệu, sheet luôn được đồng bộ theo config.

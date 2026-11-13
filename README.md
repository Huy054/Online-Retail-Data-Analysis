# Phân tích dữ liệu bán hàng trực tuyến bằng Python

## 1. Giới thiệu đề tài

Dự án sử dụng Python để đọc, kiểm tra, làm sạch, phân tích và trực quan hóa bộ dữ liệu **Online Retail II** từ UCI Machine Learning Repository. Notebook được xây dựng phục vụ bài giữa kỳ A5 và có thể chạy bằng **Google Colab hoặc Visual Studio Code (VS Code)** với Jupyter Notebook.

**Tệp notebook đề xuất sử dụng:** `A5_PhanTichDuLieuBanHang_Portable.ipynb`.

## 2. Nguồn dữ liệu

- Nguồn: https://archive.ics.uci.edu/dataset/502/online+retail+ii
- Tệp Excel đầu vào: `online_retail_II.xlsx` (cần tải từ UCI; giải nén nếu được cung cấp ở dạng `.zip`).
- Quy mô dữ liệu gốc: 1.067.371 dòng, 8 cột, từ hai sheet của tệp Excel.
- Các cột: `Invoice`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `Price`, `Customer ID`, `Country`.
- Thời gian giao dịch: 12/2009–12/2011 (không đủ toàn bộ tháng 12/2011).

> **Lưu ý:** Notebook phân tích cả hai sheet, cần đủ bộ dữ liệu Excel gốc thay vì chỉ một sheet đã trích xuất.

## 3. Yêu cầu môi trường

- Python 3.
- Các thư viện: `pandas`, `numpy`, `matplotlib`, `seaborn`, `openpyxl`.
- Nếu chạy trên VS Code, cần cài tiện ích **Python** và **Jupyter** của Microsoft và chọn Python kernel thích hợp.

Cài thư viện (trong Terminal của VS Code hoặc môi trường Python tương ứng):

```bash
python -m pip install pandas numpy matplotlib seaborn openpyxl ipykernel
```

Trong Google Colab, phần lớn thư viện thường đã có sẵn. Nếu thiếu, có thể chạy trong một ô Code:

```python
%pip install pandas numpy matplotlib seaborn openpyxl
```

## 4. Tổ chức tệp

Đặt notebook và tệp Excel trong **cùng thư mục làm việc**:

```text
A5_PhanTichDuLieuBanHang/
├── A5_PhanTichDuLieuBanHang_Portable.ipynb
├── online_retail_II.xlsx
└── README.md
```

Notebook sử dụng **đường dẫn tương đối** như `online_retail_II.xlsx`, không phụ thuộc đường dẫn riêng của Colab là `/content/`. Khi chạy, chương trình đọc và lưu tệp theo **thư mục làm việc hiện tại** của Python.

Nếu không tìm thấy tệp đầu vào, hãy kiểm tra tên tệp và thư mục làm việc bằng:

```python
from pathlib import Path
print('Thư mục làm việc:', Path.cwd())
print('Có tệp dữ liệu:', Path('online_retail_II.xlsx').exists())
```

### 4.1. Cách chạy trên Google Colab

1. Truy cập https://colab.research.google.com/ và mở notebook `A5_PhanTichDuLieuBanHang_Portable.ipynb`.
2. Trong thanh **Tệp / Files**, tải lên file `online_retail_II.xlsx`. Theo mặc định, Colab làm việc ở thư mục `/content`.
3. Chọn **Runtime / Môi trường thực thi → Run all / Chạy tất cả**.
4. Kiểm tra các bảng phân tích, biểu đồ và tải file CSV/PNG đầu ra từ thanh **Tệp / Files**.

**Lưu ý:** Tệp tải lên Colab có thể mất khi phiên làm việc kết thúc; nếu chạy lại ở một phiên mới thì cần tải file Excel lên lại.

### 4.2. Cách chạy trên Visual Studio Code (VS Code)

1. Cài Python 3 và VS Code; cài hai extension **Python** và **Jupyter**.
2. Tạo/mở một thư mục dự án chứa notebook, `README.md` và `online_retail_II.xlsx` như sơ đồ mục 4.
3. Mở Terminal trong VS Code và cài các thư viện ở mục 3 vào đúng môi trường Python sẽ sử dụng.
4. Mở file `.ipynb`, chọn **Select Kernel** và chọn môi trường Python đã cài thư viện.
5. Chạy các ô từ trên xuống bằng **Run All** (hoặc chạy từng ô theo thứ tự).
6. Nếu báo `FileNotFoundError`, kiểm tra `Path.cwd()` bằng đoạn mã tại mục 4; đảm bảo Excel nằm trong chính thư mục đó hoặc điều chỉnh đường dẫn trong ô đọc dữ liệu.

**Không nên dùng “Run Python File” để chạy trực tiếp `.ipynb` như file `.py`**; hãy mở notebook bằng giao diện Jupyter của VS Code.

## 5. Quy trình xử lý và phân tích

- **Mục 6–7:** Đọc cả hai sheet Excel; xem kích thước, tên cột, kiểu dữ liệu và thống kê mô tả.
- **Mục 8:** Kiểm tra dữ liệu thiếu, trùng lặp, hóa đơn hủy, số lượng và giá bán bất thường; làm sạch và lưu CSV.
- **Mục 9.1–9.5:** Phân tích tổng quan doanh thu, theo tháng, mã hàng, khách hàng và quốc gia.
- **Mục 9.6–9.7:** Thống kê doanh thu từng dòng bằng NumPy và bảng tổng hợp Pivot Table.
- **Mục 10:** Xây dựng 5 biểu đồ thuộc 4 dạng: đường, cột, tròn và histogram.
- **Mục 11:** Rút ra nhận xét có ý nghĩa, nêu hạn chế và kết luận.

**Quy tắc làm sạch:** Loại dòng trùng hoàn toàn, hóa đơn có mã bắt đầu bằng `C`, dòng có `Quantity <= 0` hay `Price <= 0`, dòng thiếu hoặc rỗng `Description`. Dòng thiếu `Customer ID` vẫn được giữ cho doanh thu tổng thể và chỉ bị bỏ khi phân tích khách hàng. Cột doanh thu `Revenue` được tính bằng `Quantity × Price`.

Doanh thu tính được chỉ phản ánh dữ liệu sau các quy tắc làm sạch này, **không phải doanh thu kế toán ròng**.

## 6. Đầu ra sau khi chạy

Các tệp được lưu trong thư mục làm việc hiện tại (Colab thường là `/content/`; VS Code thường là thư mục dự án khi mở notebook tại đó):

```text
online_retail_clean.csv
bieu_do_1_doanh_thu_thang.png
bieu_do_2_san_pham.png
bieu_do_3_khach_hang.png
bieu_do_4_quoc_gia.png
bieu_do_5_so_luong.png
```

Theo kết quả đã lưu trong notebook: dữ liệu sau làm sạch có 1.007.913 dòng. Kết quả chính thức cần dựa trên lần chạy lại thực tế với đúng file Excel nguồn.

**Lưu ý:** CSV được xuất tại mục 8 trước khi tạo thêm cột `Month` ở mục 9; do đó, muốn phân tích theo tháng từ CSV thì cần tạo cột này từ `InvoiceDate`.

## 7. Lưu ý và hạn chế

- Một phần dữ liệu thiếu mã khách hàng.
- Dòng trùng hoàn toàn không phải lúc nào cũng chắc chắn là giao dịch sai.
- Các dòng hủy/trả hàng, giá trị âm, điều chỉnh đã được xử lý theo mục đích phân tích doanh thu bán hàng, không tính doanh thu ròng.
- Một số mã như `M` (Manual) và `DOT` (DOTCOM POSTAGE) cần phân biệt với sản phẩm hàng hóa thông thường.
- Dữ liệu ghi nhận giai đoạn 2009–2011, không phản ánh xu hướng bán lẻ hiện tại.

## 8. Tài liệu tham khảo

UCI Machine Learning Repository. *Online Retail II*. https://archive.ics.uci.edu/dataset/502/online+retail+ii

# VietTraffic AI: Hệ Thống Giám Sát Giao Thông Thông Minh & Nhận Diện Biển Số

Hệ thống tích hợp trí tuệ nhân tạo (Computer Vision) ứng dụng trong giám sát giao thông đô thị tại Việt Nam. Dự án bao gồm các module chính: Phát hiện phương tiện đi sai làn đường, nhận diện biển số xe sử dụng mô hình kết hợp (Ensemble OCR), và giao diện giám sát (Dashboard Web).

---

## 📌 Các Tính Năng Chính

### 1. Phát hiện phương tiện đi sai làn đường (`find_wrong_lane`)
* Sử dụng **YOLO Tracking** để theo dõi vết di chuyển (trajectory) của các phương tiện qua nút giao.
* Tự động xác định làn xuất phát và làn đích bằng các vùng đa giác (polygon) cấu hình sẵn.
* Phát hiện các hành vi vi phạm phổ biến (Theo Nghị định 168/2024/NĐ-CP):
  - **CASE_3**: Đi thẳng trên làn bắt buộc rẽ trái.
  - **CASE_4**: Rẽ trái trên làn bắt buộc đi thẳng.
  - **CASE_6**: Đổi sang làn/vùng đi thẳng rồi mới rẽ trái (đè vạch liền).
* Hỗ trợ công cụ hiệu chuẩn trực quan **GUI (`calibrate_config.py`)** để vẽ và tùy chỉnh các làn đường trực tiếp trên hình ảnh hoặc video.

### 2. Nhận diện biển số xe Việt Nam (`train_ocr` & `find_license_plate`)
* Pipeline nhận diện biển số xe tự động từ luồng video hoặc ảnh crop.
* Sử dụng giải thuật **OCR Ensemble**:
  - Mô hình chính: **PyTorch CTC OCR**.
  - Mô hình bổ trợ: **PaddleOCR** để hiệu chỉnh các lỗi ký tự ở phần đuôi biển số (đặc biệt là định dạng dạng `123.45`).
* Áp dụng quy tắc kinh nghiệm của hệ thống biển số Việt Nam:
  - Phân loại màu sắc biển số (Trắng, Vàng, Xanh, Đỏ) dựa trên không gian màu HSV.
  - Tự động sửa lỗi nhầm lẫn phổ biến như `A4`/`T4`/`L4` thành `AA`/`TA`/`LA` dựa theo màu sắc và loại xe.

### 3. Giao diện Giám sát & Dashboard (`demo`)
* Cung cấp Web Dashboard hiển thị danh sách các phương tiện vi phạm kèm hình ảnh chụp bằng chứng (crop xe, crop biển số, thông tin luật vi phạm, mức phạt tương ứng).
* Tích hợp API cung cấp dữ liệu vi phạm thời gian thực.

---

## 📂 Cấu Trúc Thư Mục Dự Án

```text
system_camera_ai/
│
├── find_wrong_lane/          # Module phát hiện xe đi sai làn đường
│   ├── calibration_frames/   # Ảnh mẫu dùng để hiệu chuẩn làn đường
│   ├── calibrate_config.py   # Giao diện GUI để vẽ cấu hình làn đường
│   ├── lane_config.json      # Tệp cấu hình tọa độ các làn đường
│   ├── main.py               # Script chạy phát hiện sai làn
│   └── readme.md             # Hướng dẫn chi tiết cho module sai làn
│
├── find_license_plate/       # Module phát hiện biển số xe từ video/ảnh
│   ├── main.py               # Script chạy phát hiện biển số
│   └── models/               # Chứa các trọng số YOLO phát hiện biển số
│
├── train_ocr/                # Module nhận diện ký tự biển số xe (OCR)
│   ├── src/                  # Mã nguồn mô hình PyTorch CTC và pipeline gộp
│   ├── models/               # Lưu trữ các mô hình OCR (PaddleOCR & PyTorch CTC)
│   ├── main.py               # Script chạy nhận diện hàng loạt
│   └── README.md             # Hướng dẫn chi tiết module OCR
│
├── demo/                     # Web Dashboard và API giám sát
│   ├── apps/api/             # Backend API cung cấp dữ liệu vi phạm
│   └── web/dashboard/        # Frontend Dashboard giao diện người dùng
│
├── docs/                     # Tài liệu thiết kế và đặc tả dự án
├── data/                     # Thư mục chứa dữ liệu video/ảnh đầu vào mẫu
├── result/                   # Kết quả đầu ra tổng hợp
└── README.md                 # Tài liệu hướng dẫn chung dự án (Tệp tin này)
```

---

## 🛠️ Hướng Dẫn Cài Đặt

### 1. Yêu cầu hệ thống
* Hệ điều hành: Windows / Linux / macOS
* Phiên bản Python: **Python 3.8 - 3.10** (Khuyến nghị 3.9)
* Khuyên dùng card đồ họa hỗ trợ CUDA để hệ thống xử lý thời gian thực mượt mà hơn.

### 2. Cài đặt thư viện
Tạo môi trường ảo và cài đặt các thư viện cần thiết:
```bash
# Tạo môi trường ảo (tùy chọn)
python -m venv venv
source venv/bin/activate  # Trên Linux/macOS
# Hoặc chạy trên Windows:
.\venv\Scripts\activate

# Cài đặt các thư viện chính
pip install -r requirements.txt
```
*(Nếu chưa có file `requirements.txt`, bạn có thể cài đặt thủ công các thư viện chính bao gồm: `ultralytics`, `opencv-python`, `pandas`, `numpy`, `paddlepaddle`, `paddleocr`, `torch`, `torchvision`, `PyQt5`/`PySide6` tùy module)*

---

## 🚀 Hướng Dẫn Chạy Dự Án

### 1. Chạy phát hiện xe đi sai làn đường
Di chuyển vào thư mục `find_wrong_lane` và chạy script:
```bash
cd find_wrong_lane

# Chạy nhanh chỉ xuất kết quả (không hiển thị màn hình video)
python main.py --no-video

# Chạy hiển thị luồng video debug trực quan
python main.py
```
Kết quả vi phạm (bao gồm ảnh xe vi phạm, file CSV, file SQLite DB) sẽ được xuất ra thư mục `find_wrong_lane/outputs/`.

*Để mở công cụ vẽ lại làn đường:*
```bash
python calibrate_config.py
```

### 2. Chạy nhận diện biển số OCR
Di chuyển vào thư mục `train_ocr`:
1. Đặt các ảnh biển số cần nhận diện vào thư mục `train_ocr/input/`.
2. Chạy lệnh:
   ```bash
   cd train_ocr
   python main.py
   ```
3. Xem kết quả tại thư mục `train_ocr/output/results.csv`.

---

## 📜 Tài Liệu & Báo Cáo
Báo cáo khoa học chi tiết về thiết kế hệ thống, thuật toán phát hiện sai làn và thử nghiệm mô hình OCR Ensemble có sẵn tại:
* [Báo cáo chi tiết (PDF)](bao_cao_viettraffic_ocr.pdf)
* [Slide thuyết trình dự án (PDF)](slide_viettraffic_ocr.pdf)

---

## 👥 Tác Giả & Bản Quyền
Dự án được phát triển bởi **Đỗ Công Trí** nhằm mục đích nghiên cứu và ứng dụng công nghệ AI vào bài toán tối ưu hóa an toàn giao thông tại Việt Nam. Mọi đóng góp và phản hồi vui lòng tạo Issue hoặc Pull Request trên Repository này.


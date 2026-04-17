# Hệ Thống Phân Tích Mã Độc - Malware Analysis System

## Mục Lục
1. [Giới Thiệu](#giới-thiệu)
2. [Kết Quả Đã Hoàn Thành](#kết-quả-đã-hoàn-thành)
3. [Kiến Trúc Hệ Thống](#kiến-trúc-hệ-thống)
4. [Cài Đặt & Chạy Ứng Dụng](#cài-đặt--chạy-ứng-dụng)
5. [Hạn Chế](#hạn-chế)
6. [Các Tính Năng Chính](#các-tính-năng-chính)
7. [Cấu Trúc Thư Mục](#cấu-trúc-thư-mục)

---

## Giới Thiệu

Hệ thống phân tích mã độc (Malware Analysis System) là ứng dụng web xây dựng trên Flask. Cho phép upload file quét, phân tích tĩnh với YARA rules, trích xuất chuỗi, phân loại mã độc, và xuất báo cáo JSON. Dự án là công cụ giáo dục cho phân tích malware cơ bản.

---

## Kết Quả Đã Hoàn Thành

### 1. Giao Diện Web Người Dùng
- Upload file drag-and-drop
- Hiển thị lịch sử báo cáo quét
- Dashboard hiển thị trạng thái các file

### 2. Backend Phân Tích
- Memory Scanner: Quét file từ bộ nhớ mà không cần lưu vào ổ cứng
- PE File Analysis: Phân tích file PE (Portable Executable)
  - Entropy analysis
  - Import/Export table analysis
  - IMPHash detection
- String Extraction: Trích xuất chuỗi từ file nhị phân
  - Hỗ trợ FLOSS (Free Library for Obfuscated String Solving)
  - Fallback method nếu FLOSS không có sẵn
- YARA Rules Engine: Biên dịch rules từ thư mục, fallback string matching, hỗ trợ Keylogger/Ransomware/Trojan rules

### 3. Phân Loại & Đánh Giá
- Rule-based classifier đánh giá mức độ nguy hiểm
- Phát hiện njRAT, bladabindi, hostr, ransomware, keylogger
- Tính toán confidence score (0-100)
- Nhãn phân loại: benign, suspicious, malicious, keylogger, ransomware

### 4. Báo Cáo
- Báo cáo JSON chi tiết với metadata, kết quả quét, phân loại
- Lưu trữ báo cáo trong thư mục results/
- Hiển thị báo cáo trong dashboard

### 5. Cơ Sở Dữ Liệu Rules
- Capa rules library (1000+ rules)
- Custom YARA rules (Keylogger, Ransomware, Trojan)
- Known IMPHash database cho malware families

---

## Kiến Trúc Hệ Thống

### Sơ Đồ Luồng Xử Lý

```
┌─────────────────┐
│  Web Interface  │ (Flask Web App)
│  index.html     │
└────────┬────────┘
         │ Upload file
         ▼
┌─────────────────────────┐
│  Memory Scanner         │ (app/memory_scanner.py)
│  - Load file to memory  │
│  - Calculate hash       │
│  - Analyze entropy      │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  PE File Analyzer       │ (pefile library)
│  - Parse PE headers     │
│  - Extract imports      │
│  - Detect IMPHash       │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  String Extractor       │ (FLOSS / Fallback)
│  - Extract strings      │
│  - Filter & rank        │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  YARA Scanner           │ (app/yara_wrapper.py)
│  - Compile rules        │
│  - Scan file against    │
│    rules database       │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  Classifier             │ (app/classifier.py)
│  - Calculate score      │
│  - Assign label         │
│  - Generate reasons     │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│  Report Generator       │ (web/app.py)
│  - Format results       │
│  - Save JSON            │
│  - Return to web UI     │
└─────────────────────────┘
```

### Các Module Chính

| Module | Chức Năng |
|--------|----------|
| `app/memory_scanner.py` | Quét file từ memory, phân tích PE, tính entropy |
| `app/classifier.py` | Phân loại mã độc dựa trên điểm số |
| `app/rule_manager.py` | Load & compile YARA rules |
| `app/yara_wrapper.py` | Wrapper YARA với fallback mode |
| `web/app.py` | Flask web application, routing, upload handling |
| `web/templates/` | HTML templates (index.html, report.html) |

---

## Cài Đặt & Chạy Ứng Dụng

### 1. Tải Dự Án
```bash
git clone https://github.com/haind16/BTL_PTMD/MalwareScan
```

### 2. Cài Đặt Dependencies
```bash
pip install -r requirements.txt
```

Dependencies: flask>=2.0.0, pytest>=6.0.0, pefile>=2022.1.1, werkzeug>=2.0.0

### 3. Chạy Ứng Dụng
```bash
python run_web.py
```
Ứng dụng sẽ khởi động tại http://localhost:5000

### 4. Dừng
Nhấn CTRL+C

---

## Hạn Chế

- Chỉ phân tích tĩnh, không có dynamic analysis hay sandboxing
- Phát hiện phụ thuộc vào chất lượng YARA rules
- Chỉ hỗ trợ PE files tốt, không hỗ trợ ELF, Mach-O hay file archive
- Không có authentication, rate limiting hay encryption
- YARA-Python là tùy chọn, fallback mode yếu hơn
- String extraction chậm với file lớn
- Single-threaded development server, không phù hợp production
- Capa rules chiếm dung lượng lớn (~1GB)

---

## Các Tính Năng Chính

### Feature 1: File Upload & Scanning
User uploads file -> Memory scan -> Entropy check -> PE analysis -> String extraction -> YARA matching -> Classifier -> Report

### Feature 2: Entropy Analysis
- Phát hiện encryption/packing
- Threshold: 7.5 -> high entropy

### Feature 3: PE Import Detection
- Phát hiện network imports (ws2_32, wininet, winhttp)
- Phát hiện API gọi nguy hiểm (CreateRemoteThread, VirtualAlloc)
- Tìm IMPHash matches trong database

### Feature 4: String Matching
- Tìm suspicious strings (cmd.exe, powershell, regsvr32)
- Tìm malware signatures (njRAT, bladabindi, hostr)
- Tìm ransomware patterns (encrypt, ransom, HOW_TO_DECRYPT)
- Tìm keylogger patterns (GetAsyncKeyState, SetWindowsHookEx)

### Feature 5: Classification
Score: High entropy +30, Network imports +10, IMPHash match +35, Suspicious string +5-20, Ransom +30, Keylogger +25

Label: 0-25 benign, 25-50 suspicious, 50-75 malicious, 75-100 severe

### Feature 6: Report Export
JSON format với metadata, analysis results, classification, timestamp

---

## Cấu Trúc Thư Mục

```
BTL_PTMD/
├── README.md                    # README gốc
├── README_COMPLETE.md           # README chi tiết
│
├── MalwareScan/                 # Thư mục chính
│   ├── requirements.txt          # Dependencies
│   ├── run_web.py               # Script khởi động
│   ├── app/                     # Core modules
│   │   ├── classifier.py        # Rule-based classifier
│   │   ├── memory_scanner.py    # File analysis từ memory
│   │   ├── rule_manager.py      # YARA rule manager
│   │   └── yara_wrapper.py      # YARA wrapper với fallback
│   ├── web/                     # Flask web app
│   │   ├── app.py               # Flask application
│   │   ├── templates/           # HTML templates
│   │   └── uploads/             # Uploaded files
│   ├── rules/                   # YARA rules
│   │   ├── Keylogger.yar
│   │   ├── ransomware.yar
│   │   ├── trojan.yar
│   │   └── capa-rules/          # CAPA behavior rules
│   └── results/                 # Generated reports
├── Report/                      # Project documentation
└── .git/                        # Git repository
```

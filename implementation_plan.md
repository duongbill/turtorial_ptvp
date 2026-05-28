# 📋 Implementation Plan: Trang Web Hướng Dẫn Cài Đặt Dự Án BOCS

> [!IMPORTANT]
> Đây là kế hoạch triển khai (Plan) — chưa phải code. Hãy review và phản hồi trước khi bắt tay vào code.

---

## 🔍 1. Phân Tích Gap — Những gì yêu cầu ban đầu CÒN THIẾU

Sau khi phân tích toàn bộ codebase dự án BOCS (Bio-Optimal Control System), tôi phát hiện **7 điểm quan trọng** cần bổ sung vào nội dung trang hướng dẫn:

### 1.1. Thiếu bước cài đặt Git
- Dự án cần `git clone` để tải source code, nhưng hướng dẫn chưa đề cập việc cài đặt **Git** — một phần mềm mà nhiều học sinh chưa có.
- **Bổ sung**: Thêm mục hướng dẫn cài Git (link download, hướng dẫn cài trên Windows).

### 1.2. Thiếu hướng dẫn cấu hình file `.env`
- Dự án có 2 file `.env.example` cần thiết:
  - `backend/.env.example` → chứa cấu hình `SIMULATION_ENABLED=true`, `MQTT_ENABLED=false`
  - `frontend/.env.example` → chứa `NEXT_PUBLIC_API_BASE_URL`, `NEXT_PUBLIC_WS_URL`
- Khi chạy bằng Docker, các biến này được truyền qua `docker-compose.yml` nên **không cần** cấu hình thủ công (cần giải thích rõ cho học sinh).
- **Bổ sung**: Thêm Note giải thích rằng Docker tự xử lý `.env`, không cần tạo tay.

### 1.3. Thiếu thông tin kiến trúc hệ thống (3 services)
- `docker-compose.yml` khởi chạy **3 services**: `mqtt` (Mosquitto), `backend` (FastAPI), `frontend` (Next.js).
- Hướng dẫn ban đầu chỉ đề cập frontend tại `localhost:3000`, nhưng thiếu giải thích:
  - `localhost:8000` → Backend API + Swagger Docs
  - `localhost:8000/docs` → Giao diện test API
  - `localhost:1883` → MQTT Broker (không cần truy cập trình duyệt)
- **Bổ sung**: Thêm bảng "Địa chỉ truy cập sau khi khởi chạy" liệt kê cả 3 endpoints.

### 1.4. Thiếu lệnh dừng và quản lý Docker
- Học sinh cần biết cách **dừng hệ thống** khi không dùng, tránh Docker chiếm tài nguyên.
- **Bổ sung**: Thêm mục nhỏ "Các lệnh Docker hữu ích":
  - `docker-compose down` → dừng tất cả
  - `docker-compose logs -f` → xem log
  - `docker-compose ps` → xem trạng thái
  - Hoặc giới thiệu file `docker.ps1` có sẵn trong dự án.

### 1.5. Thiếu hướng dẫn PowerShell Execution Policy (Windows)
- File [docker.ps1](file:///d:/workspace/job2025/co_giang/ptvp_2026/docker.ps1) yêu cầu chạy lệnh:
  ```powershell
  Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
  ```
- Nhiều máy Windows mặc định **chặn chạy script PowerShell**. Cần cảnh báo học sinh.
- **Bổ sung**: Thêm Warning box giải thích lệnh Execution Policy.

### 1.6. Thiếu bước kiểm tra Docker Desktop đã Start chưa
- Docker Desktop trên Windows **không tự chạy** sau khi khởi động máy (trừ khi đã bật tự động).
- **Bổ sung**: Thêm bước "Mở Docker Desktop và đợi biểu tượng cá voi ở khay hệ thống chuyển sang màu xanh/trắng".

### 1.7. Thiếu lưu ý về thời gian build lần đầu
- Lần đầu chạy `docker-compose up --build`, Docker cần tải ~2-3GB images (Node, Python, Mosquitto).
- **Bổ sung**: Thêm Note về dung lượng tải và thời gian dự kiến (5-15 phút tùy mạng).

---

## 🏗️ 2. Cấu Trúc Nội Dung Chính Thức (Đã Bổ Sung)

Trang web sẽ gồm **7 Section** (tăng từ 5 lên 7 so với yêu cầu ban đầu):

### Section 1: Tổng Quan & Chuẩn Bị
| Nội dung | Chi tiết |
|----------|----------|
| Giới thiệu dự án | Hệ thống IoT BOCS, container hóa bằng Docker, 3 services: MQTT + Backend + Frontend |
| Checklist cấu hình | Table có icon checkbox |
| OS | Windows 10/11 (Home 21H2+, Pro, Enterprise) hoặc macOS |
| RAM | Tối thiểu 8GB, khuyến khích 16GB |
| Ổ cứng | Trống tối thiểu 10GB SSD |
| Quyền | Administrator |
| **🆕 Mạng Internet** | **Ổn định, cần thiết cho lần build đầu tiên (~2-3GB download)** |

### Section 2: Cài Đặt Git *(🆕 BỔ SUNG)*
| Nội dung | Chi tiết |
|----------|----------|
| Tải Git | Link `https://git-scm.com/downloads` |
| Cài đặt | Chạy installer, giữ mọi tùy chọn mặc định |
| Kiểm tra | Code block: `git --version` |
| ⚠️ Warning | "Nếu sử dụng file ZIP thay vì git clone, có thể bỏ qua bước này" |

### Section 3: Cài Đặt Visual Studio Code
| Nội dung | Chi tiết |
|----------|----------|
| Bước 3.1 | Tải tại `https://code.visualstudio.com/` |
| Bước 3.2 | Cài đặt + ⚠️ Warning: Tích chọn "Open with Code" & "Add to PATH" |
| Bước 3.3 | Cài Extensions: `Docker` (Microsoft), `Prettier - Code formatter` |
| **🆕 Bước 3.4** | **Extension bổ sung nên cài: `ESLint`, `Python` (Microsoft) để debug thuận tiện** |

### Section 4: Cài Đặt Docker & Kích Hoạt Ảo Hóa
| Nội dung | Chi tiết |
|----------|----------|
| Bước 4.1 | Kiểm tra Virtualization trong Task Manager |
| Bước 4.2 | Kích hoạt WSL 2: `wsl --install` + Restart |
| Bước 4.3 | Tải Docker Desktop, tích "Use WSL 2", Restart |
| Bước 4.4 | Kiểm tra: `docker --version` + `docker compose version` |
| **🆕 Bước 4.5** | **Mở Docker Desktop, đợi icon cá voi chuyển trạng thái "Running" trước khi tiếp tục** |

### Section 5: Khởi Chạy Dự Án Thực Tế
| Nội dung | Chi tiết |
|----------|----------|
| Bước 5.1 | Tải source: `git clone <link_repo>` hoặc giải nén ZIP |
| Bước 5.2 | Mở thư mục gốc bằng VS Code (nhìn thấy `docker-compose.yml`) |
| Bước 5.3 | Build: `docker-compose up -d --build` + giải thích flag |
| Bước 5.4 | ✅ Success: Truy cập `http://localhost:3000` |
| **🆕 Bước 5.5** | **Bảng "Địa chỉ truy cập":  `localhost:3000` (Dashboard), `localhost:8000/docs` (API Swagger)** |
| **🆕 Bước 5.6** | **Lệnh quản lý Docker cơ bản: dừng, xem log, restart** |

### Section 6: Xử Lý Lỗi Thường Gặp (Troubleshooting)
| # | Lỗi | Cách sửa |
|---|------|----------|
| 1 | `docker: command not found` | Bật Docker Desktop hoặc restart VS Code |
| 2 | `Ports are not available` | Tắt XAMPP/Skype hoặc đổi port trong `docker-compose.yml` |
| 3 | Yêu cầu cập nhật WSL kernel | Chạy `wsl --update` |
| **🆕 4** | **Docker Desktop "starting" mãi không xong** | **Restart máy, kiểm tra Virtualization đã Enabled** |
| **🆕 5** | **Build quá lâu hoặc lỗi network** | **Kiểm tra mạng, thử `docker-compose down` rồi `up` lại** |
| **🆕 6** | **PowerShell chặn chạy script (.ps1)** | **Chạy `Set-ExecutionPolicy RemoteSigned -Scope Process`** |
| **🆕 7** | **Lỗi "no space left on device"** | **Chạy `docker system prune -a` để dọn rác Docker** |

### Section 7: Tham Khảo Nhanh *(🆕 BỔ SUNG)*
| Nội dung | Chi tiết |
|----------|----------|
| Bảng tổng hợp lệnh | Tất cả commands quan trọng ở một chỗ (quick reference) |
| Link hữu ích | Docker docs, VS Code docs, GitHub repo |
| File `docker.ps1` | Giới thiệu script quản lý Docker có sẵn trong dự án |

---

## 🎨 3. Quy Hoạch UI/UX

### 3.1. Layout

```
┌──────────────────────────────────────────────────────────┐
│                     VIEWPORT                             │
│  ┌───────────┐  ┌──────────────────────────────────────┐ │
│  │           │  │                                      │ │
│  │  SIDEBAR  │  │           MAIN CONTENT               │ │
│  │  280px    │  │           max-width: 900px            │ │
│  │  fixed    │  │           padding: 40px               │ │
│  │           │  │                                      │ │
│  │  ┌──────┐ │  │  ┌──────────────────────────────┐    │ │
│  │  │ Logo │ │  │  │  Section Header              │    │ │
│  │  └──────┘ │  │  │  H2 + Description             │    │ │
│  │           │  │  └──────────────────────────────┘    │ │
│  │  Nav      │  │                                      │ │
│  │  ├─ §1   │  │  ┌──────────────────────────────┐    │ │
│  │  ├─ §2   │  │  │  Content Block               │    │ │
│  │  ├─ §3   │  │  │  - Text paragraphs           │    │ │
│  │  ├─ §4   │  │  │  - Checklist tables          │    │ │
│  │  ├─ §5   │  │  │  - Code blocks + Copy btn    │    │ │
│  │  ├─ §6   │  │  │  - Alert boxes               │    │ │
│  │  └─ §7   │  │  │  - Accordion items           │    │ │
│  │           │  │  └──────────────────────────────┘    │ │
│  │           │  │                                      │ │
│  └───────────┘  └──────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
```

### 3.2. Design Tokens (CSS Variables)

```css
:root {
  /* === Colors === */
  --bg-page:        #f4f6f8;
  --bg-sidebar:     #1a1a2e;
  --bg-code:        #2d3748;
  --text-primary:   #2d3748;
  --text-sidebar:   #a0aec0;
  --text-white:     #ffffff;
  --accent:         #007acc;       /* VS Code blue */
  --accent-hover:   #00adb5;       /* Tech teal */

  /* Alert colors */
  --note-bg:        #e3f2fd;
  --note-border:    #1976d2;
  --note-text:      #0d47a1;
  --warning-bg:     #fff3e0;
  --warning-border: #f57c00;
  --warning-text:   #e65100;
  --success-bg:     #e8f5e9;
  --success-border: #43a047;
  --success-text:   #1b5e20;

  /* === Spacing === */
  --sidebar-width:  280px;
  --content-max-w:  900px;
  --radius:         6px;

  /* === Typography === */
  --font-main:      'Inter', 'Segoe UI', sans-serif;
  --font-code:      'Consolas', 'Courier New', monospace;
}
```

### 3.3. Các Component CSS Chính

| Component | Selector | Đặc điểm |
|-----------|----------|-----------|
| Sidebar | `.sidebar` | `position: fixed`, dark bg, scroll nội bộ |
| Nav Item | `.nav-item` | Hover → accent color, active → border-left |
| Section | `.section` | `margin-bottom: 3rem`, divider line |
| Code Block | `.code-block` | `position: relative`, dark bg, monospace |
| Copy Button | `.copy-btn` | `position: absolute; top-right`, hover effect |
| Alert Note | `.alert.note` | Blue scheme, `border-left: 5px solid` |
| Alert Warning | `.alert.warning` | Orange scheme |
| Alert Success | `.alert.success` | Green scheme |
| Checklist Table | `.checklist-table` | Striped rows, checkbox icons |
| Troubleshoot Item | `.troubleshoot-item` | Accordion hoặc card dạng click-to-expand |
| Step Number | `.step-number` | Viên tròn số thứ tự, accent color |

---

## 📂 4. Quy Hoạch File

```
docs/setup-guide/
├── index.html          # Toàn bộ structure + content (Semantic HTML5)
├── style.css           # Toàn bộ styling (Pure CSS3, có comment phân khu)
└── (script inline)     # JS ngắn gọn ở cuối <body> trong index.html
```

### 4.1. Cấu trúc HTML (Skeleton)

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hướng Dẫn Cài Đặt Dự Án BOCS</title>
    <meta name="description" content="...">
    <link rel="stylesheet" href="style.css">
    <link href="Google Fonts - Inter" rel="stylesheet">
</head>
<body>
    <!-- Sidebar Navigation -->
    <nav class="sidebar" id="sidebar">
        <div class="sidebar-header">Logo + Title</div>
        <ul class="nav-list">
            <li><a href="#section-1">§1 Tổng quan</a></li>
            <li><a href="#section-2">§2 Cài Git</a></li>
            <li><a href="#section-3">§3 Cài VS Code</a></li>
            <li><a href="#section-4">§4 Cài Docker</a></li>
            <li><a href="#section-5">§5 Khởi chạy</a></li>
            <li><a href="#section-6">§6 Xử lý lỗi</a></li>
            <li><a href="#section-7">§7 Tham khảo</a></li>
        </ul>
    </nav>

    <!-- Main Content -->
    <main class="main-content">
        <section id="section-1">...</section>
        <section id="section-2">...</section>
        <section id="section-3">...</section>
        <section id="section-4">...</section>
        <section id="section-5">...</section>
        <section id="section-6">...</section>
        <section id="section-7">...</section>
    </main>

    <!-- JavaScript -->
    <script>
        // 1. Smooth scroll
        // 2. Copy to clipboard
        // 3. Active nav highlight on scroll
        // 4. Accordion toggle (Troubleshooting)
    </script>
</body>
</html>
```

### 4.2. Cấu trúc CSS (Sections)

```css
/* =========================================
   style.css - Setup Guide BOCS
   ========================================= */

/* === 1. Reset & Base === */
/* === 2. CSS Variables (Design Tokens) === */
/* === 3. Typography === */
/* === 4. Layout (Sidebar + Main) === */
/* === 5. Navigation === */
/* === 6. Section Styling === */
/* === 7. Code Blocks === */
/* === 8. Copy Button === */
/* === 9. Alert Boxes === */
/* === 10. Checklist Table === */
/* === 11. Step Numbers === */
/* === 12. Troubleshooting Accordion === */
/* === 13. Quick Reference Table === */
/* === 14. Responsive (Mobile) === */
/* === 15. Animations & Transitions === */
```

### 4.3. JavaScript Functions

| Function | Mô tả |
|----------|--------|
| `initSmoothScroll()` | Smooth scroll khi click nav links, offset cho fixed sidebar |
| `initCopyButtons()` | Click copy → sao chép code vào clipboard → hiển thị "Copied!" 2 giây |
| `initScrollSpy()` | Highlight nav item tương ứng khi cuộn đến section đó |
| `initAccordion()` | Toggle mở/đóng các mục Troubleshooting |

---

## ✅ 5. Checklist Nghiệm Thu

- [ ] **Layout**: Sidebar cố định, main content cuộn mượt
- [ ] **Responsive**: Sidebar ẩn trên mobile (≤768px), hiện hamburger menu
- [ ] **Smooth Scroll**: Click sidebar → cuộn mượt đến section
- [ ] **Scroll Spy**: Nav item active đúng khi cuộn
- [ ] **Copy Button**: Click → copy text → hiện "Copied!" → trả lại "Copy" sau 2s
- [ ] **Alert Boxes**: 3 loại Note/Warning/Success hiển thị đúng màu
- [ ] **Code Blocks**: Nền tối, font monospace, có syntax highlight cơ bản
- [ ] **Checklist Table**: Có icon checkbox, striped rows
- [ ] **Accordion**: Click toggle mở/đóng trong Troubleshooting
- [ ] **Nội dung đầy đủ**: Tất cả 7 sections có đủ text, lệnh, và alert boxes
- [ ] **Semantic HTML**: Sử dụng đúng `<nav>`, `<main>`, `<section>`, `<article>`, `<code>`, `<pre>`
- [ ] **Font**: Google Fonts Inter được load
- [ ] **Accessibility**: Các link có `aria-label`, contrast đạt chuẩn WCAG AA

---

## ⏱️ 6. Kế Hoạch Triển Khai (Timeline)

| Phase | Nội dung | Ước lượng |
|-------|----------|-----------|
| **Phase 1** | Tạo `style.css`: Design tokens, layout grid, sidebar, typography | ~15 min |
| **Phase 2** | Tạo `index.html`: HTML skeleton + toàn bộ nội dung 7 sections | ~25 min |
| **Phase 3** | CSS components: Code blocks, alerts, checklist, accordion | ~15 min |
| **Phase 4** | JavaScript: Smooth scroll, copy, scroll spy, accordion | ~10 min |
| **Phase 5** | Polish: Transitions, hover effects, responsive mobile | ~10 min |
| **Phase 6** | Review & test trên browser | ~5 min |

**Tổng ước lượng: ~80 phút**

---

## 📌 7. Lưu Ý Quan Trọng

> [!WARNING]
> Nội dung hướng dẫn được thiết kế **đặc thù cho dự án BOCS** — không phải hướng dẫn chung. Các thông tin như port `3000`, `8000`, `1883`, file `docker-compose.yml`, file `docker.ps1` đều lấy trực tiếp từ codebase thực tế.

> [!NOTE]
> Trang web sẽ được đặt tại `docs/setup-guide/` trong thư mục dự án, sẵn sàng để commit lên GitHub cùng với source code.

---

## 🤔 Câu Hỏi Cần Phản Hồi Trước Khi Code

1. **Link GitHub repo**: Bạn muốn chèn URL repo thật vào lệnh `git clone` hay để placeholder `<link_repo>`?
2. **Phần cài Git**: Có cần giữ mục này không? Hay học sinh đã biết dùng Git rồi và chỉ cần file ZIP?
3. **Section 7 (Tham khảo nhanh)**: Có muốn thêm mục này hay giữ 5 sections gốc là đủ?
4. **Responsive mobile**: Có cần thiết kế cho mobile (hamburger menu) hay chỉ cần desktop?
5. **Logo/Banner**: Có muốn tôi generate hình ảnh minh họa (logo BOCS, screenshot minh họa) hay chỉ thuần text?
6. **Ngôn ngữ**: Toàn bộ viết bằng tiếng Việt, đúng không?

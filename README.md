
# 📄 TÀI LIỆU CI/CD – Build Kernel LG V50 Android 12 với KernelSU Next

## 🧰 1. Mô tả tổng quan
Dự án sử dụng Jenkins để tự động hóa quy trình build kernel cho thiết bị **LG V50 (Android 12)**, tích hợp với **KernelSU Next**. Quy trình CI/CD bao gồm các bước:
- Lấy mã nguồn từ GitHub (qua webhook)
- Thiết lập và build kernel sử dụng toolchain Clang & GCC
- Tích hợp KernelSU Next
- Đóng gói kernel với AnyKernel3
- Tự động upload lên SourceForge nếu build thành công

---

## 🏗️ 2. Cấu trúc Pipeline Jenkins

```groovy
pipeline {
  agent any
  ...
  stages {
    stage('Checkout')     // Clone source từ GitHub
    stage('Build Kernel') // Thiết lập & build kernel
    stage('Upload')       // Upload bản build lên SourceForge (nếu có)
  }
  post {
    success { ... } // Log khi build thành công
    failure { ... } // Log khi build thất bại
  }
}
```

---

## ⚙️ 3. Biến môi trường

| Biến        | Ý nghĩa                                               |
|-------------|--------------------------------------------------------|
| `CLANG_PATH` | Đường dẫn tới toolchain Clang dùng để build           |
| `CROSS_PATH` | Toolchain GCC cross-compile cho ARM                   |
| `SF_USER`    | Tên người dùng SourceForge để upload bản build        |

---

## 🪛 4. Chi tiết các stage

### ✅ `Checkout`
- **Mục đích**: Lấy mã nguồn mới nhất từ repository GitHub thông qua webhook
- **Công cụ**: `checkout scm`

---

### ⚙️ `Build Kernel`
- **Thiết lập biến môi trường** cho kiến trúc ARM64 và Clang toolchain
- **Tích hợp KernelSU Next**:
  ```bash
  curl -LSs "https://raw.githubusercontent.com/KernelSU-Next/KernelSU-Next/next/kernel/setup.sh" | bash -
  ```
- **Chạy script `ubuntu.sh`** để:
  - Clean môi trường
  - Thiết lập `defconfig` (vendor/dragon_flash_defconfig)
  - Biên dịch kernel với Clang + GCC toolchain
  - Đóng gói `Image.gz-dtb` vào AnyKernel3
  - Tạo file flashable zip: `release/Dragon-AK3.zip`

---

### ☁️ `Upload to SourceForge`
- **Kiểm tra** nếu file `release/Dragon-AK3.zip` tồn tại
- **Đổi tên file** theo timestamp: `HHMM-DDMMYYYY-Dragon-AK3.zip`
- **Upload qua SCP** tới SourceForge thư mục `lg-v50-oss/DragonKernel/`

---

## 💥 5. Xử lý sau khi build

| Trạng thái | Hành động                                     |
|------------|-----------------------------------------------|
| Thành công | Log: `✅ Build thành công.`                    |
| Thất bại   | Log: `❌ Build thất bại. Vui lòng kiểm tra lại log.` |

---

## 📜 6. Chi tiết file build kernel `ubuntu.sh`

| Bước                            | Mô tả                                                     |
|---------------------------------|------------------------------------------------------------|
| `make clean && make mrproper`  | Xóa sạch môi trường build trước                           |
| `make vendor/dragon_flash_defconfig` | Tải cấu hình defconfig của kernel                 |
| `make -j$(nproc)`              | Tiến hành build với số lượng luồng tối đa                 |
| `git clone AnyKernel3`         | Clone khung đóng gói zip                                  |
| `cp Image.gz-dtb`              | Copy kernel vào thư mục zip                               |
| `zip`                          | Đóng gói zip có thể flash qua TWRP                        |

---

## 📦 7. Kết quả đầu ra
- **File Flashable Zip**: `release/HHMM-DDMMYYYY-Dragon-AK3.zip`
- **Vị trí**: Upload lên SourceForge tại:
  ```
  https://sourceforge.net/projects/lg-v50-oss/files/DragonKernel/
  ```

---

## 📈 8. Lợi ích CI/CD
- ✅ Tự động hóa toàn bộ quy trình build kernel
- ✅ Tích hợp nhanh KernelSU NExT phiên bản mới nhất
- ✅ Giảm rủi ro lỗi do thao tác tay
- ✅ Tăng tốc độ phân phối kernel mới đến cộng đồng

---

## 🧠 9. Hướng phát triển
- Tích hợp thông báo tự động qua Telegram/Discord sau khi build
- Xác minh SHA256 file sau khi build & upload

# Hướng dẫn tạo bộ cài đặt (Installer) trên Windows

Tài liệu này hướng dẫn cách tạo bộ cài đặt (installer) cho ứng dụng Warp trên hệ điều hành Windows từ mã nguồn (codebase).

---

## 1. Yêu cầu chuẩn bị (Prerequisites)

Để tạo bộ cài đặt, máy của bạn cần được cài đặt sẵn các công cụ sau:

1. **Rust & Cargo Toolchain**: Được cài đặt và cấu hình đầy đủ để build mã nguồn Rust của Warp.
2. **Inno Setup Compiler**: Công cụ đóng gói ứng dụng Windows.
   - Tải và cài đặt tại: [Inno Setup Download](https://jrsoftware.org/isdl.php).
   - Sau khi cài đặt, trình biên dịch dòng lệnh (`ISCC.exe`) thường nằm tại `C:\Program Files (x86)\Inno Setup 6\ISCC.exe`. Hãy thêm đường dẫn này vào biến môi trường `PATH` hệ thống hoặc gọi trực tiếp từ CLI.

---

## 2. Cách 1: Sử dụng Script đóng gói tự động (Khuyên dùng)

Thư mục dự án đã cung cấp sẵn script PowerShell để tự động hóa toàn bộ quy trình từ biên dịch mã nguồn đến đóng gói installer: [bundle.ps1](file:///d:/Projects/Canhan/warp/script/windows/bundle.ps1).

Script này sẽ tự động:
1. Biên dịch mã nguồn Rust (`cargo build`) với các tính năng (features) và cấu hình tối ưu theo kênh release chỉ định.
2. Chuẩn bị các tài nguyên đóng gói (fonts, dlls, các skills đi kèm, settings schema...) thông qua script [prepare_bundled_resources.ps1](file:///d:/Projects/Canhan/warp/script/windows/prepare_bundled_resources.ps1).
3. Sử dụng công cụ Inno Setup biên dịch file [windows-installer.iss](file:///d:/Projects/Canhan/warp/script/windows/windows-installer.iss) để đóng gói thành file cài đặt `.exe` duy nhất.

### Lệnh thực thi từ thư mục gốc dự án:

Mở PowerShell tại thư mục gốc của dự án và chạy một trong các lệnh sau tùy thuộc vào Release Channel mong muốn:

* **Tạo bộ cài đặt bản Dev (WarpDev - Mặc định):**
  ```powershell
  .\script\windows\bundle.ps1 -CHANNEL dev
  ```
* **Tạo bộ cài đặt bản Stable (Warp chính thức):**
  ```powershell
  .\script\windows\bundle.ps1 -CHANNEL stable
  ```
* **Tạo bộ cài đặt bản Open Source (Warp Oss):**
  ```powershell
  .\script\windows\bundle.ps1 -CHANNEL oss
  ```

### Các tùy chọn tham số (Parameters) hữu ích:
- `-DEBUG_BUILD`: Sử dụng profile debug để biên dịch ứng dụng.
- `-SKIP_BUILD_BINARY`: Chỉ chạy đóng gói Installer từ file thực thi đã được build sẵn trước đó (bỏ qua Cargo build).
- `-SKIP_BUILD_INSTALLER`: Chỉ biên dịch Cargo build mà không tạo file cài đặt `.exe`.
- `-ARCH`: Chỉ định kiến trúc CPU target: `x64` (mặc định) hoặc `arm64`.

---

## 3. Cách 2: Đóng gói thủ công (Manual)

Nếu cần kiểm soát chi tiết quy trình hoặc tùy chỉnh sâu cấu hình, bạn có thể thực hiện thủ công các bước sau:

### Bước 2.1: Biên dịch Cargo Binary
Biên dịch ứng dụng Warp bằng Cargo:
```powershell
cargo build -p warp --profile <profile> --bin <binary_name> --target <target_triple>
```

### Bước 2.2: Chuẩn bị tài nguyên đóng gói
Chạy script chuẩn bị tài nguyên để sao chép fonts, dlls, licenses và generate settings schema vào thư mục tài nguyên đích:
```powershell
.\script\windows\prepare_bundled_resources.ps1 -DestinationDir "target\<target_triple>\<profile>\resources" -Channel <channel> -CargoProfile <profile>
```

### Bước 2.3: Chạy biên dịch Inno Setup (`iscc`)
Chạy trình biên dịch Inno Setup dòng lệnh (`iscc`) bằng cách truyền vào các tham số preprocessor để ghi đè cấu hình mặc định:
```powershell
iscc .\script\windows\windows-installer.iss /DReleaseChannel=<channel> /DMyAppExeName=<binary_name>.exe /DTargetProfileDir=target\<target_triple>\<profile> /DMyAppName=<app_name> /DMyAppVersion=<version> /DArch=<arch> /DOutputName=<output_installer_name>
```

Sau khi hoàn tất biên dịch, file cài đặt sẽ được sinh ra tại thư mục: `.\script\windows\Output\`.

---

## 4. Tham khảo thêm

- Chi tiết các tùy biến tiền xử lý (preprocessor definitions), cấu hình môi trường và hướng dẫn tạo icon ứng dụng có thể được xem thêm tại file hướng dẫn gốc của dự án: [README.md](file:///d:/Projects/Canhan/warp/script/windows/README.md).

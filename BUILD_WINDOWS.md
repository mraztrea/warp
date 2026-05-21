# Build Warp Cho Windows

Tai lieu nay ghi lai cach build Warp tren Windows theo dung pipeline cua repo nay.

## Ket qua da build tren may nay

- Binary: `target/x86_64-pc-windows-msvc/rlto/warp-oss.exe`
- Installer: `script/windows/Output/WarpOssSetup.exe`

## Cach nhanh nhat de build ra file `.exe`

Mo PowerShell tai thu muc goc repo va chay:

```powershell
winget install Google.Protobuf
$env:PROTOC = "$env:LOCALAPPDATA\Microsoft\WinGet\Packages\Google.Protobuf_Microsoft.Winget.Source_8wekyb3d8bbwe\bin\protoc.exe"
.\script\windows\bundle.ps1 -CHANNEL oss -SKIP_BUILD_INSTALLER
```

Lenh tren tao ra file:

```text
target\x86_64-pc-windows-msvc\rlto\warp-oss.exe
```

Ghi chu:

- Lan build dau tien rat lau. Tren may nay mat khoang 25 phut.
- Neu ban vua cai `Google.Protobuf` ma khong mo duoc `protoc`, hay dong va mo lai PowerShell, hoac giu nguyen dong `$env:PROTOC = ...` nhu o tren.

## Cach build ra installer `.exe`

Neu muon co file cai dat Windows, can them `cargo-about` va Inno Setup:

```powershell
cargo install --locked cargo-about@0.8.4
winget install --id JRSoftware.InnoSetup --accept-source-agreements --accept-package-agreements
$env:PROTOC = "$env:LOCALAPPDATA\Microsoft\WinGet\Packages\Google.Protobuf_Microsoft.Winget.Source_8wekyb3d8bbwe\bin\protoc.exe"
$env:PATH = "$env:LOCALAPPDATA\Programs\Inno Setup 6;$env:PATH"
.\script\windows\bundle.ps1 -CHANNEL oss -RELEASE_TAG 0.1.0
```

Lenh tren tao ra file:

```text
script\windows\Output\WarpOssSetup.exe
```

## Neu da build xong binary va chi muon dong goi installer

```powershell
$env:PATH = "$env:LOCALAPPDATA\Programs\Inno Setup 6;$env:PATH"
.\script\windows\bundle.ps1 -CHANNEL oss -SKIP_BUILD_BINARY -RELEASE_TAG 0.1.0
```

Lenh nay nhanh hon vi khong compile lai Rust.

## Cach chay ban build

Chay truc tiep binary:

```powershell
.\target\x86_64-pc-windows-msvc\rlto\warp-oss.exe
```

Hoac chay installer:

```powershell
.\script\windows\Output\WarpOssSetup.exe
```

## Loi thuong gap

### 1. `no such command: about`

Thieu `cargo-about`.

```powershell
cargo install --locked cargo-about@0.8.4
```

### 2. `protoc` khong tim thay

Cai Protobuf va dat bien moi truong `PROTOC` nhu o tren.

### 3. `ISCC` khong tim thay

Thieu Inno Setup hoac terminal chua nhan `PATH` moi.

```powershell
winget install --id JRSoftware.InnoSetup --accept-source-agreements --accept-package-agreements
$env:PATH = "$env:LOCALAPPDATA\Programs\Inno Setup 6;$env:PATH"
```

## Lenh dang dung trong repo nay

Repo nay co script Windows rieng tai `script/windows/bundle.ps1`.
Tren Windows, nen uu tien script nay thay vi `cargo bundle --bin warp`.

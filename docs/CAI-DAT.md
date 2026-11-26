# Hướng dẫn cài đặt chi tiết

Công cụ chỉ chạy với khoá API của [RevidAPI](https://revidapi.com). Mọi lệnh (kể cả `phan-tich` không tốn credit) đều kiểm khoá trước khi làm việc.

## Bước 0. Lấy khoá API

1. Đăng ký hoặc đăng nhập tại https://revidapi.com.
2. Vào **Cài đặt** (https://revidapi.com/dashboard/settings), bấm sao chép khoá `sk_...`.
3. Nạp credit nếu cần. Xem số dư bất cứ lúc nào bằng `reup so-du`.

Không dán khoá vào lệnh, vào file trong repo, hay gửi cho người khác.

## macOS

```bash
# 1. Công cụ cần có
xcode-select --install            # git (bỏ qua nếu đã có)
brew install python@3.12 ffmpeg   # cần Homebrew: https://brew.sh

# 2. Tải mã và cài
git clone https://github.com/revidapicom/revidapi-reup-video
cd revidapi-reup-video
python3 -m venv .venv && source .venv/bin/activate
pip install -e .

# 3. Lưu khoá (chỉ bạn đọc được)
mkdir -p ~/.revidapi && printf '%s' 'sk_...' > ~/.revidapi/khoa && chmod 600 ~/.revidapi/khoa

# 4. Kiểm
reup so-du
```

Lỗi `CERTIFICATE_VERIFY_FAILED`: công cụ đã dùng chứng chỉ `certifi`, chỉ cần `pip install -U certifi`.

## Windows 10/11

Mở **PowerShell** (không cần quyền Admin):

```powershell
# 1. Công cụ cần có
winget install -e --id Python.Python.3.12
winget install -e --id Gyan.FFmpeg
winget install -e --id Git.Git
# Đóng PowerShell, mở lại để nhận PATH mới

# 2. Tải mã và cài
git clone https://github.com/revidapicom/revidapi-reup-video
cd revidapi-reup-video
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1      # nếu bị chặn: Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
pip install -e .

# 3. Lưu khoá
New-Item -ItemType Directory -Force "$HOME\.revidapi" | Out-Null
Set-Content -NoNewline -Path "$HOME\.revidapi\khoa" -Value 'sk_...'

# 4. Kiểm
reup so-du
```

Đường dẫn có dấu cách thì đặt trong ngoặc kép: `reup chay "C:\Users\Toi\Video mau\a.mp4"`.

## VPS / máy chủ Linux (Ubuntu 22.04+)

```bash
sudo apt update && sudo apt install -y python3-venv python3-pip ffmpeg git
git clone https://github.com/revidapicom/revidapi-reup-video
cd revidapi-reup-video
python3 -m venv .venv && . .venv/bin/activate
pip install -e .
mkdir -p ~/.revidapi && printf '%s' 'sk_...' > ~/.revidapi/khoa && chmod 600 ~/.revidapi/khoa
reup so-du
```

- Whisper `small` cần khoảng 2GB RAM. VPS 1GB thì dùng `--whisper base`.
- Chạy lâu, không sợ rớt SSH: `tmux new -s reup` rồi chạy lệnh trong đó.
- Lấy file về máy: `scp vps:~/revidapi-reup-video/reup-ra/du-an/*.mp4 .`

## Lỗi thường gặp

| Thông báo | Cách xử lý |
|---|---|
| `Chưa có khoá` | Làm lại bước lưu khoá, hoặc `export REVIDAPI_KEY=sk_...` cho phiên hiện tại. |
| `Khoá RevidAPI không hợp lệ hoặc đã bị thu hồi` | Sao chép lại khoá ở trang Cài đặt. Khoá cũ không còn dùng được sau khi tạo khoá mới. |
| `Khoá có ký tự lạ` | Khoá bị dán thừa dấu cách hoặc xuống dòng. Ghi lại file bằng `printf '%s'`. |
| Báo hết credit / không đủ số dư | Nạp thêm ở revidapi.com rồi chạy lại. |
| `ffmpeg: command not found` | Chưa cài ffmpeg, hoặc chưa mở lại cửa sổ dòng lệnh sau khi cài. |

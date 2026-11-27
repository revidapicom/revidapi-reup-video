---
name: reup-video
description: Dựng lại video theo video mẫu bằng repo revidapi-reup-video và MCP RevidAPI — tải mẫu, phân tích frame/lời/nhịp, viết kịch bản mới, tạo ảnh/video AI (grok, kling), lồng giọng, dựng video dọc. Dùng khi người dùng gửi link TikTok/Douyin/FB/YouTube hoặc file mp4 và nói "làm lại video này", "reup", "phân tích rồi dựng lại".
---

# Reup video theo mẫu

Chép **công thức** của video mẫu (hook, nhịp, cấu trúc, thủ pháp hình), không chép hình, nhạc, câu chữ của người khác.

Khoá ở `~/.revidapi/khoa` (hoặc biến `REVIDAPI_KEY`). **Không bao giờ in khoá ra**, không ghi khoá vào file trong repo.

## 0. Kiểm máy (0 credit)

```bash
reup so-du                      # khoá đúng + số dư
claude mcp list | grep revidapi # MCP đã nối chưa (xem README mục "Dùng với Claude Code + MCP RevidAPI")
```

Chưa có `reup` thì cài theo README. Chưa có MCP thì vẫn làm được toàn bộ bằng `reup`; MCP chỉ để Claude tự gọi tạo ảnh/video từng cảnh.

## 1. Lấy video mẫu

- Link: `reup chay <link> ...` tự tải. Muốn tự lấy link gốc thì dùng tool MCP `tai_video` (`nen_tang` + `url`).
- File mp4 của khách: dùng thẳng đường dẫn.

## 2. Phân tích (0 credit)

```bash
reup phan-tich <link-hoặc-file> --ten mau1
```

Ra `reup-ra/mau1/phan-tich/`: `tong-quan.jpg` (lưới 12 frame), `frames/`, `canh.json`, `nhip.json`, `loi.txt`.

**Read `tong-quan.jpg` và 3–5 frame trong `frames/`** để nhìn bố cục thật, rồi viết cho người dùng một bảng ngắn:

| Mục | Ghi gì |
|---|---|
| Hook 3 giây đầu | câu mở (lấy ở `nhip.json` → `hook_3s`) + hình mở |
| Nhịp | `so_canh`, `canh_tb_giay`, `tu_moi_giay` |
| Cấu trúc | hook → vấn đề → nội dung chính → chốt |
| Cảnh có người | `canh_co_nguoi` — những cảnh này sẽ vẽ lại bằng AI |
| Thủ pháp hình | phụ đề, chữ nhấn, chuyển cảnh, góc máy |

## 3. Kịch bản — DỪNG Ở CỔNG DUYỆT

```bash
reup chay <link-hoặc-file> --ten mau1 --yeu-cau "<chủ đề mới của khách>" --giay 30
```

Lệnh dừng trước khi tiêu credit, in ước tính và để lại `reup-ra/mau1/kich-ban.json`. Đọc file đó, sửa cho đúng ý khách (chỉ nói điều đúng về sản phẩm của khách, số liệu phải kiểm), báo **ước tính credit + số dư** và **chờ khách đồng ý**. Không tự thêm `--dong-y`.

## 4. Hình cho từng cảnh

Hai cách, chọn một:

**A. Để `reup` tự làm** (chỉ chạy sau khi khách đã đồng ý; `--dong-y` là bắt buộc):

```bash
reup dung reup-ra/mau1 --hinh goc --dong-y                                          # cảnh có người vẽ lại bằng ảnh AI, cảnh khác giữ frame
reup dung reup-ra/mau1 --hinh goc --video-ai --model-video kling-2.5-turbo --dong-y  # cảnh có người thành video AI
```

**B. Claude tự gọi MCP RevidAPI từng cảnh** (khi cần điều khiển prompt từng cảnh):

1. Với mỗi cảnh có người: nhìn frame tương ứng, viết prompt mô tả **bố cục, góc máy, ánh sáng, hành động** (không mô tả khuôn mặt người thật trong mẫu).
2. `tao_anh` — `model: "gpt-image-2"`, `size: "1024x1536"` (dọc) → lấy link ảnh.
3. `tao_video` — `model: "kling-2.5-turbo"` hoặc `"grok-i2v"`, `img_url: <link ảnh bước 2>`, `aspect_ratio: "9:16"`, `prompt`: chuyển động ngắn (một hành động + một chuyển động máy).
4. `trang_thai_job` với `job_id` nhận được, hỏi lại mỗi 20–30 giây (thường 1–5 phút).
5. Tải video về `reup-ra/mau1/` rồi dựng bằng `reup dung reup-ra/mau1 --dong-y`.

Luôn **truyền `model` rõ ràng**. Giá thật xem `GET https://revidapi.com/v1/video/models` hoặc tool `bang_gia`. Mốc rẻ nhất tại thời điểm viết: `kling-2.5-turbo` 720p 5 giây ≈ 45 credit, `grok-i2v` 480p 6 giây ≈ 53 credit, ảnh `gpt-image-2` ≈ 18 credit. Mỗi cảnh video AI là một lượt tính tiền — cộng đủ vào ước tính trước khi xin duyệt.

## 5. Kiểm trước khi gửi (máy đo, không đoán)

```bash
ffprobe -v error -show_entries format=duration:stream=codec_type,width,height -of default=nw=1 reup-ra/mau1/*.mp4
reup so-du    # so với số dư trước → credit đã tiêu thật
```

Xem ít nhất 3 frame (đầu, giữa, cuối): phụ đề không đè hình, chữ không tràn. Báo khách: đường dẫn file, thời lượng, credit tiêu thật, điều chưa làm.

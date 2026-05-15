---
name: making-image
description: Tạo ảnh blog chuẩn SEO kích cỡ 1238×829 pixels, định dạng WebP từ ảnh gốc trong thư mục photos. Khi user gõ "/making-image" hoặc yêu cầu tạo/xử lý ảnh blog, skill sẽ resize + convert toàn bộ ảnh trong thư mục nguồn và lưu vào thư mục output.
---

# Making Image — Ảnh Blog Chuẩn SEO Whale Island Resort

## Mục đích

Tạo ảnh chuẩn cho bài viết SEO trên blog Whale Island Resort:
- **Kích thước**: 1238 × 829 pixels (tỉ lệ ~3:2, chuẩn blog)
- **Định dạng**: WebP (nhẹ hơn JPEG ~30%, tốt cho Core Web Vitals)
- **Nguồn ảnh**: `C:\Users\ADMIN\Claude\.claude\skills\making-image\photos wir\`
- **Output**: `C:\Users\ADMIN\Claude\.claude\skills\making-image\output\`

## Khi nào dùng skill này

Khi user gõ `/making-image` hoặc nói "tạo ảnh blog", "xử lý ảnh cho bài viết", "convert ảnh WebP", ví dụ:
- `/making-image` — xử lý toàn bộ ảnh trong thư mục photos
- `/making-image whale-bay` — xử lý và đặt prefix tên file là `whale-bay`
- `/making-image --folder "tên thư mục con"` — xử lý thư mục con cụ thể

Trích phần đứng sau `/making-image` làm **prefix tên file** (nếu có). Nếu không có prefix, giữ nguyên tên file gốc.

## Quy trình thực hiện

### Bước 1 — Kiểm tra thư mục và ảnh nguồn

Kiểm tra thư mục nguồn có ảnh không:
```
C:\Users\ADMIN\Claude\.claude\skills\making-image\photos wir\
```

Các định dạng ảnh được hỗ trợ: `.jpg`, `.jpeg`, `.png`, `.webp`, `.bmp`, `.tiff`, `.heic`

Nếu thư mục trống hoặc không có ảnh hợp lệ → báo user biết và hướng dẫn thêm ảnh vào thư mục.

### Bước 2 — Tạo thư mục output

Tạo thư mục output nếu chưa tồn tại:
```
C:\Users\ADMIN\Claude\.claude\skills\making-image\output\
```

### Bước 3 — Xử lý ảnh bằng Python (Pillow)

Chạy script Python để resize + convert. Logic xử lý:

**Chiến lược crop để đạt 1238×829**:
1. Tính tỉ lệ scale để ảnh **cover** toàn bộ khung 1238×829 (không để lộ nền trắng)
2. Crop phần dư ở giữa (center crop) — giữ chủ thể ở trung tâm
3. Export WebP với quality=85 (cân bằng chất lượng và dung lượng)

Script Python cần chạy:

```python
import os
from PIL import Image

SRC = r"C:\Users\ADMIN\Claude\.claude\skills\making-image\photos wir"
DST = r"C:\Users\ADMIN\Claude\.claude\skills\making-image\output"
TARGET_W, TARGET_H = 1238, 829
QUALITY = 85
MIN_SIZE_BYTES = 100 * 1024  # 100 kB tối thiểu

os.makedirs(DST, exist_ok=True)

supported = {'.jpg', '.jpeg', '.png', '.webp', '.bmp', '.tiff'}
files = [f for f in os.listdir(SRC) if os.path.splitext(f)[1].lower() in supported]

if not files:
    print("Không tìm thấy ảnh trong thư mục nguồn.")
else:
    for fname in files:
        src_path = os.path.join(SRC, fname)
        stem = os.path.splitext(fname)[0]
        # Nếu có prefix truyền vào, dùng PREFIX_stem.webp
        # (thay PREFIX bằng giá trị thực khi chạy)
        out_name = stem + ".webp"
        dst_path = os.path.join(DST, out_name)

        with Image.open(src_path) as img:
            img = img.convert("RGB")
            orig_w, orig_h = img.size

            # Center crop để đạt tỉ lệ 1238:829
            scale = max(TARGET_W / orig_w, TARGET_H / orig_h)
            new_w = int(orig_w * scale)
            new_h = int(orig_h * scale)
            img = img.resize((new_w, new_h), Image.LANCZOS)

            left = (new_w - TARGET_W) // 2
            top = (new_h - TARGET_H) // 2
            img = img.crop((left, top, left + TARGET_W, top + TARGET_H))

            # Lưu với quality ban đầu rồi kiểm tra dung lượng
            img.save(dst_path, "WEBP", quality=QUALITY, method=6)

            # Nếu file < 100 kB, tăng dần quality cho đến khi đạt tối thiểu
            q = QUALITY
            while os.path.getsize(dst_path) < MIN_SIZE_BYTES and q < 99:
                q += 5
                img.save(dst_path, "WEBP", quality=q, method=6)

            file_kb = os.path.getsize(dst_path) / 1024
            print(f"✓ {fname} → {out_name} ({orig_w}×{orig_h} → {TARGET_W}×{TARGET_H}) | {file_kb:.0f} kB | quality={q}")

    print(f"\nXong! {len(files)} ảnh đã lưu tại: {DST}")
```

Khi có prefix (ví dụ user gõ `/making-image whale-bay`), thay tên output thành `whale-bay-01.webp`, `whale-bay-02.webp`... (đánh số theo thứ tự file).

### Bước 4 — Báo cáo kết quả

Sau khi xử lý xong, báo user:
- Số ảnh đã xử lý thành công
- Đường dẫn thư mục output
- Danh sách tên file đã tạo (dạng bảng nếu nhiều hơn 3 file)
- Dung lượng trung bình mỗi file (nếu có thể tính được)
- Hỏi: "Bạn muốn thêm prefix tên file cho SEO không? (ví dụ: `whale-island-beach-01.webp`)"

## Ghi chú kỹ thuật

| Thông số | Giá trị | Lý do |
|----------|---------|-------|
| Kích thước | 1238 × 829 px | Chuẩn blog WordPress/Ghost, tỉ lệ 3:2 |
| Định dạng | WebP | Nhẹ hơn JPEG ~30%, được hỗ trợ bởi Google |
| Quality | 85 (tự tăng đến 99 nếu cần) | Bắt đầu ở 85, tăng dần 5 mỗi bước nếu file < 100 kB |
| Dung lượng tối thiểu | 100 kB | Tránh ảnh quá nhỏ/mờ khi Google index |
| Crop strategy | Center crop | Giữ chủ thể ảnh ở trung tâm |
| Resize filter | LANCZOS | Chất lượng cao nhất khi downscale |

## Xử lý lỗi thường gặp

- **Pillow chưa cài**: Chạy `python -m pip install Pillow` trước
- **File HEIC không đọc được**: Cài thêm `python -m pip install pillow-heif` và import `pillow_heif` ở đầu script
- **Ảnh bị kéo méo**: Không dùng `img.resize((TARGET_W, TARGET_H))` thẳng — phải dùng center crop như script trên
- **Tên file có ký tự đặc biệt/tiếng Việt**: Pillow trên Windows xử lý được, nhưng nên rename file output thành ASCII để SEO-friendly

## Sau khi tạo ảnh

Hỏi user:
1. Có muốn rename file theo slug bài viết SEO không? (ví dụ: `whale-island-snorkeling-guide-01.webp`)
2. Có ảnh nào cần crop khác (focus vào góc trái/phải thay vì center) không?
3. Có muốn tạo thêm thumbnail kích cỡ khác (ví dụ: 800×533 cho featured image, 400×267 cho thumbnail) không?

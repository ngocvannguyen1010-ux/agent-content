---
name: research
description: Phân tích search intent chi tiết cho từ khóa SEO phục vụ blog Whale Island Resort. Khi user gõ "/research [chủ đề]" hoặc "/research [từ khóa]", skill sẽ nghiên cứu sâu Who/What/Why/Insight của từ khóa đó và xuất ra file markdown trong thư mục research-output/.
---

# Research Search Intent — Whale Island Resort Blog

## Bối cảnh thương hiệu

**Whale Island Resort (Hòn Ông)** là resort cao cấp nằm trên đảo Hòn Ông, vịnh Vân Phong, tỉnh Khánh Hòa, Việt Nam. Đặc điểm:
- Resort biệt lập trên đảo riêng, không gian hoang sơ, gần gũi thiên nhiên
- Nổi tiếng với các hoạt động: lặn biển, ngắm cá voi, kayak, snorkeling, yoga, retreat
- Phân khúc: trung – cao cấp, hướng đến khách yêu thiên nhiên, gia đình, cặp đôi, dân văn phòng thành thị cần nghỉ dưỡng
- Khách hàng chính: người Việt Nam (Hà Nội, TP.HCM, Đà Nẵng), khách nước ngoài đến Khánh Hòa

Mọi phân tích search intent đều phải đặt trong bối cảnh này — không phân tích chung chung.

## Khi nào dùng skill này

Khi user gõ `/research <chủ đề/từ khóa>`, ví dụ:
- `/research du lịch Hòn Ông`
- `/research resort biệt lập Khánh Hòa`
- `/research lặn ngắm san hô Nha Trang`
- `/research retreat yoga ven biển`

Trích phần đứng sau `/research` làm **từ khóa mục tiêu**. Nếu user không cung cấp từ khóa, hỏi lại trước khi làm.

## Quy trình thực hiện

### Bước 0 — Thu thập dữ liệu SERP thực tế (BẮT BUỘC)

Trước khi phân tích, dùng tool tìm kiếm để thu thập dữ liệu thực tế từ web. Đây là bước **bắt buộc** — không được bỏ qua, vì insight phải bám vào SERP thật chứ không suy luận thuần.

**Ưu tiên dùng Tavily MCP** (nếu có sẵn — tool tên `tavily-search` hoặc `mcp__tavily__tavily-search`) vì trả về nội dung trang đầy đủ chứ không chỉ snippet. Fallback sang `WebSearch` nếu Tavily không khả dụng.

Đối với Tavily, các tham số khuyến nghị:
- `search_depth`: `"advanced"` để có kết quả chất lượng cao
- `max_results`: 5–8
- `include_raw_content`: `true` để lấy nội dung trang thật
- `country`: `"vietnam"` (nếu API hỗ trợ) để ưu tiên kết quả tiếng Việt

Ngoài tìm kiếm, nếu cần đọc sâu một trang cụ thể (ví dụ bài review dài trên webtretho), dùng tool `tavily-extract` thay vì `WebFetch` để có nội dung đầy đủ.

Chạy **4 truy vấn song song** (gọi search 4 lần trong cùng một message):

1. **`<từ khóa>`** — lấy top 10 kết quả đang rank để hiểu:
   - Đối thủ là ai (OTA như Booking/Agoda/Traveloka, blog du lịch lớn, công ty tour, hay resort khác?)
   - Loại nội dung đang win (listicle, review, guide, landing page)
   - Gap nội dung mà blog Whale Island Resort có thể chen vào

2. **`<từ khóa> review` hoặc `<từ khóa> kinh nghiệm`** — tìm bài chia sẻ thật của khách đã đi, đặc biệt trên webtretho, otofun, hội du lịch Facebook (nếu Google trả về). Phục vụ phần Insight/Pain point.

3. **`<từ khóa> reddit` hoặc `<từ khóa> forum`** — lấy thảo luận thật để hiểu pain point và câu hỏi user hay đặt. Reddit thường có insight chân thật hơn blog SEO.

4. **`<từ khóa cốt lõi không có địa danh> nên đi đâu`** hoặc một biến thể câu hỏi phổ biến — để bắt được People Also Ask gián tiếp.

Nếu cả Tavily và `WebSearch` đều không có sẵn (lỗi/không có quyền), thông báo cho user và tiếp tục với cảnh báo "phân tích dưới đây dựa trên suy luận, không có dữ liệu SERP thực tế".

**Ghi chú vào output**: tại Mục 1 của file kết quả, thêm tiểu mục **"Top 5 đối thủ đang rank"** liệt kê domain + tiêu đề, và tiểu mục **"Câu hỏi user thường đặt (từ SERP/forum)"** liệt kê 5–10 câu thật sự lấy được.

### Bước 1 — Phân tích từ khóa

Sau khi có dữ liệu SERP, làm rõ:
1. **Loại intent SEO**: Informational / Navigational / Commercial / Transactional — kiểm chứng bằng SERP (nếu top 10 toàn landing page bán hàng thì intent là Commercial/Transactional).
2. **Mức độ thương mại** (1–10): càng cao càng gần quyết định đặt phòng
3. **Giai đoạn funnel**: Awareness / Consideration / Decision
4. **Biến thể từ khóa & long-tail liên quan**: 5–10 biến thể, **ưu tiên lấy từ "Related searches" trong kết quả WebSearch** trước khi tự nghĩ ra.

### Bước 2 — Xác định nhiều nhóm Who

Một từ khóa thường có **nhiều nhóm người tìm kiếm khác nhau**. Liệt kê tối thiểu 2, tối đa 5 nhóm Who. Với mỗi nhóm, mô tả:
- **Tên persona** ngắn gọn (vd: "Cặp đôi văn phòng 28–35 tuổi Hà Nội")
- **Nhân khẩu học**: độ tuổi, giới tính, nghề nghiệp, thu nhập (VNĐ/tháng), trình độ học vấn, khu vực sinh sống, tình trạng hôn nhân/gia đình
- **Hành vi số**: dùng kênh nào (Google, TikTok, Instagram, Facebook group, Booking, Agoda), thiết bị nào
- **Họ đã biết gì**: kiến thức nền về chủ đề trước khi search (vd: đã từng đi Nha Trang, biết Hòn Ông nhưng chưa biết giá; hoặc hoàn toàn chưa biết)
- **Họ chưa biết gì**: lỗ hổng kiến thức mà bài viết cần lấp

### Bước 3 — What (thông tin họ muốn tìm)

Sắp xếp theo **thứ tự ưu tiên giảm dần**. Với mỗi điểm:
- Mô tả cụ thể thông tin (không nói chung chung)
- Lý do tại sao điểm đó quan trọng với persona đó
- Định dạng kỳ vọng (bảng giá, ảnh, video, checklist, review thật…)

Ví dụ tốt: "Giá phòng cụ thể theo mùa cao điểm/thấp điểm — vì họ cần tính ngân sách chuyến đi 4N3Đ cho 2 người trong khoảng 8–12 triệu."

### Bước 4 — Why (lý do tìm kiếm)

Chia thành 2 nhóm:
- **Khách quan** (hoàn cảnh bên ngoài): kỳ nghỉ lễ sắp tới, sếp duyệt OOO, deadline dự án vừa xong, vé máy bay Vietjet đang sale, có 4 ngày phép còn lại trong năm…
- **Chủ quan** (tâm lý bên trong): muốn thoát khỏi thành phố ồn ào, muốn check-in sống ảo, muốn làm mới mối quan hệ, muốn thưởng cho bản thân sau dự án, FOMO sau khi thấy bạn đi…

### Bước 5 — Insight (động lực sâu & pain point)

Đây là phần **quan trọng nhất** — phải đi xuống tận tâm lý:
- **Động lực sâu** (deep motivation): khao khát thực sự đằng sau hành vi search. Không phải "đi du lịch" — mà có thể là "cần một nơi không có sóng điện thoại để chồng/vợ thực sự nhìn vào mắt nhau"
- **Pain point** (nỗi đau): điều gì đang làm họ khó chịu/lo lắng đến mức phải hành động? (vd: "ngột ngạt với 60h/tuần ở công ty", "con cái dán mắt vào iPad, sợ mất kết nối gia đình", "lần cuối đi biển đã 3 năm trước, cảm thấy đời mình bế tắc")
- **Rào cản tâm lý**: lo ngại gì cản họ ra quyết định? (vd: sợ ra đảo bất tiện, sợ đắt mà không xứng, sợ thời tiết xấu, sợ không phù hợp với người già/trẻ con)
- **Khoảnh khắc "Aha"**: nếu bài viết giải quyết được rào cản nào thì họ sẽ đặt phòng?

### Bước 6 — Xuất file markdown

Tạo file tại: `research-output/<slug-tu-khoa>-<YYYY-MM-DD>.md`

Slug: bỏ dấu, thay khoảng trắng bằng `-`, lowercase. Ví dụ: từ khóa "Du lịch Hòn Ông" → slug `du-lich-hon-ong`.

Nếu thư mục `research-output/` chưa tồn tại, tạo mới.

## Template file output

```markdown
# Search Intent Research: <Từ khóa gốc>

> Phân tích cho blog Whale Island Resort | Ngày: <YYYY-MM-DD>

## 1. Tổng quan từ khóa

- **Từ khóa chính**: <từ khóa>
- **Loại intent**: <Informational/Navigational/Commercial/Transactional>
- **Mức độ thương mại**: <x/10>
- **Giai đoạn funnel**: <Awareness/Consideration/Decision>
- **Biến thể & long-tail liên quan** (từ SERP + suy luận):
  - <biến thể 1>
  - <biến thể 2>
  - ...

### Top 5 đối thủ đang rank (từ WebSearch)
| # | Domain | Tiêu đề | Loại nội dung |
|---|--------|---------|---------------|
| 1 | <domain> | <title> | <listicle/review/landing/guide> |
| 2 | ... | ... | ... |

### Câu hỏi user thường đặt (từ SERP, forum, review)
- <câu hỏi 1 — kèm nguồn nếu có>
- <câu hỏi 2>
- ...

### Gap nội dung — cơ hội cho Whale Island Resort
<1–2 đoạn: top 10 đang thiếu gì, blog Whale Island Resort có thể chen vào ở góc nào>

## 2. Who — Chân dung người tìm kiếm

### Persona 1: <Tên ngắn>
- **Nhân khẩu học**: tuổi, giới tính, nghề nghiệp, thu nhập, học vấn, khu vực, gia đình
- **Hành vi số**: kênh tiếp cận, thiết bị, giờ online
- **Đã biết gì**: <kiến thức nền>
- **Chưa biết gì**: <lỗ hổng cần lấp>

### Persona 2: <Tên ngắn>
... (lặp lại cấu trúc, ít nhất 2 persona, tối đa 5)

## 3. What — Thông tin họ muốn tìm (ưu tiên giảm dần)

1. **<Thông tin quan trọng nhất>** — vì sao quan trọng, định dạng kỳ vọng
2. **<Thông tin quan trọng thứ 2>** — ...
3. ...
(7–12 mục)

## 4. Why — Lý do tìm kiếm

### Khách quan
- <lý do 1>
- <lý do 2>
- ...

### Chủ quan
- <lý do 1>
- <lý do 2>
- ...

## 5. Insight — Động lực sâu & nỗi đau

### Động lực sâu (Deep Motivation)
<2–4 đoạn phân tích, viết như câu chuyện chứ không gạch đầu dòng khô khan>

### Pain point (Nỗi đau)
- <nỗi đau cụ thể 1>
- <nỗi đau cụ thể 2>
- ...

### Rào cản tâm lý cản trở quyết định
- <rào cản 1>
- <rào cản 2>
- ...

### Khoảnh khắc "Aha" — bài viết cần chạm gì để chuyển đổi
<1–2 đoạn>

## 6. Gợi ý content cho blog Whale Island Resort

- **Góc tiếp cận đề xuất**: <góc viết phù hợp nhất>
- **Tiêu đề gợi ý** (3–5 phương án):
  1. <tiêu đề 1>
  2. <tiêu đề 2>
  ...
- **CTA phù hợp**: <CTA gì, dẫn đến trang nào của resort>
- **Yếu tố cần có**: ảnh/video, bảng giá, FAQ, review khách thật, bản đồ di chuyển…
```

## Nguyên tắc viết

1. **Cụ thể hơn chung chung**: thay vì "khách trẻ", viết "nhân viên văn phòng 26–32 tuổi, thu nhập 18–35 triệu/tháng, đang sống ở Hà Nội".
2. **Đặt mọi insight trong bối cảnh Whale Island Resort** — không viết như nghiên cứu thị trường du lịch chung. Ví dụ: nếu từ khóa là "lặn biển", phải nghĩ ngay đến điểm lặn của Hòn Ông và đối tượng đến đảo lặn.
3. **Một từ khóa = nhiều persona** — không gom chung. Cặp đôi đi tuần trăng mật khác hoàn toàn với gia đình có con nhỏ.
4. **Insight phải đi xa hơn "muốn nghỉ dưỡng"** — phải chạm được tâm lý thật sự (cô đơn, kiệt sức, FOMO, sợ tuổi tác, muốn níu giữ hôn nhân…).
5. **Không bịa số liệu**: nếu nói thu nhập, đưa khoảng hợp lý theo mặt bằng Việt Nam 2026; không phát minh thống kê không có nguồn.
6. **Viết tiếng Việt tự nhiên**, không Google-translate-từ-tiếng-Anh.

## Sau khi xuất file

Báo cho user:
- Đường dẫn file đã tạo
- Tóm tắt 3–5 dòng về insight nổi bật nhất tìm được
- Hỏi user có muốn đi sâu hơn vào persona nào, hoặc nghiên cứu thêm từ khóa liên quan không

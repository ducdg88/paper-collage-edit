---
name: paper-collage-edit
description: Edit video người nói khổ ngang 16:9 (YouTube) hoặc dọc 9:16 (TikTok/Reels/Shorts) (bài giảng, giải thích, học thuật, explainer) theo style "cắt dán giấy xé", viền giấy xé trắng, texture bút sáp, chữ viết tay hiện dần, stop-motion 12fps, chuyển cảnh xé giấy, chữ bật ra theo tay người nói, ảnh/video minh họa dán băng dính, cảnh kết tách nền. Pipeline Python thuần (Pillow + numpy + ffmpeg), không cần After Effects. Dùng khi người dùng nói "edit style giấy xé", "edit kiểu cắt dán", "paper collage", "paper cut-out", "video dọc style giấy xé", gửi video người nói kèm kịch bản muốn dựng animation giấy, hoặc yêu cầu sửa/render lại một đoạn đã dựng bằng skill này. Không dùng cho việc cắt ghép thô, lồng phụ đề thường, dựng video không có người nói, hay khi người dùng cần hiệu ứng kiểu khác (motion graphics phẳng, chữ karaoke).
---

# Paper collage edit (style giấy xé)

Pipeline Python thuần (Pillow + numpy + ffmpeg pipe). Mỗi đoạn video là một file `renderN.py` định nghĩa timeline cảnh; thư viện chung là `scripts/render.py`.

## Yêu cầu
- Python 3.10+, `ffmpeg`/`ffprobe` trong PATH.
- `pip install pillow numpy scipy faster-whisper mediapipe rembg onnxruntime`
- Tuỳ chọn: model `selfie_segmenter.tflite` (chỉ cho kỹ thuật làm tối nền), tải từ
  `https://storage.googleapis.com/mediapipe-models/image_segmenter/selfie_segmenter/float16/latest/selfie_segmenter.tflite`.

## Khổ hình: 16:9 và 9:16
Chọn khổ bằng biến môi trường **trước khi** import `render`:
- `PAPER_ASPECT=16:9` (mặc định) → 1920x1080
- `PAPER_ASPECT=9:16` → 1080x1920

`render.py` export `W, H, CX, CY, VERTICAL, SAFE, FIT`:
- `FIT` = filter ffmpeg scale + crop giữa, đưa source bất kỳ về đúng canvas (source 16:9 → output 9:16 cũng được, nhưng kiểm tra người nói có bị cắt không; nếu người lệch khung thì thay `crop` bằng `crop=W:H:x:y` theo vị trí người). Mọi chỗ đọc source phải có `"-vf", R.FIT`.
- `SAFE = (x0, y0, x1, y1)` = vùng an toàn. Ở 9:16 chừa trên 220px, dưới 480px, phải 160px cho UI TikTok/Reels (caption, nút like/share), **chữ và keyword quan trọng phải nằm trong SAFE**.
- Bố cục bằng `CX/CY/SAFE` và đơn vị `U = min(W, H) / 1080`, không hardcode toạ độ 1920x1080 như các file ví dụ cũ (các ví dụ đều làm cho 16:9).

Khác biệt khi dựng 9:16:
- Người nói thường chiếm giữa khung → chữ/keyword đặt **trên đầu** hoặc **khoảng 1/3 dưới** (trên vùng caption), không đặt hai bên.
- Chỉ 1 keyword/1 hình mỗi lúc, xếp theo chiều dọc; cỡ chữ giữ như 16:9 (câu chính ~100 đến 120px, chip ≥ 66px) vì màn điện thoại nhỏ.
- Cảnh full animation: tờ giấy/note chiếm ~bề ngang SAFE, hình minh hoạ xếp trên dưới thay vì trái phải.
- `torn_wipe` và `background` tự theo kích thước canvas.

`scripts/template.py` là file mẫu tối thiểu chạy được cho cả hai khổ (1 cảnh overlay + 1 cảnh full + torn wipe):
```
python template.py input.mp4 output.mp4 6
PAPER_ASPECT=9:16 python template.py input.mp4 output.mp4 6
```

## Setup workdir
Mỗi dự án một thư mục làm việc riêng, chạy script từ đó (đường dẫn tương đối `fonts/`, `models/`, `v2/`…):
```
mkdir work && cd work
SK=~/.claude/skills/paper-collage-edit
cp $SK/scripts/*.py .                 # render.py = thư viện, tên module bắt buộc là "render"
cp $SK/examples/render2.py .          # các ví dụ sau import helper (tag, scrim, arrow, crop_origin) từ render2
cp -r $SK/fonts $SK/models .
```
`render2.py` import `tracks` (jar_tracks) và glob `v2/cut/` lúc import, nếu chỉ dùng helper, tạo `v2/det.json` rỗng (`[]`) hoặc bỏ dòng import tracks.

Bắt đầu đoạn mới bằng cách copy `template.py` thành `renderN.py` rồi thêm cảnh. Mỗi file ví dụ/script có `SRC = "input.mp4"` (hoặc nhận đường dẫn qua argv), đổi thành video nguồn thật.

## Quy trình mỗi đoạn
1. `ffprobe` source, hỏi/chốt khổ output (16:9 hay 9:16); contact sheet `fps=0.5,tile` để thấy bố cục (người đứng đâu, vùng trống).
2. Transcribe word-level: faster-whisper `medium`, `initial_prompt` chứa thuật ngữ chuyên ngành. Timestamp từng từ → mốc pop của chữ.
3. Lên timeline: xen kẽ cảnh người nói (`ov`/`sp`, overlay lên video) và cảnh full animation (`fs`). Chuyển giữa ov↔fs và fs↔fs bằng `torn_wipe` 0.45s.
4. Chụp snapshot nhiều mốc (xem `scripts/snap_example.py`), **đọc ảnh để QC trước khi render full**.
5. Render full (≈ 3 đến 10 phút/đoạn 1 phút), lấy vài frame từ file thành phẩm để kiểm tra, rồi mở folder cho người dùng.

## Thư viện (scripts/render.py)
- `Sprite(size, draw_fn)` → 3 biến thể viền xé (boil); `place(canvas, sprite, cx, cy, st, scale, rot)` có jitter stop-motion.
- `pop(t, t0, t1)` bật nảy vào / thu ra; `write(c, text, x, y, t, t0, dur, size, color, anchor)` = chữ viết tay hiện dần trái→phải.
- `Label(text, size, color)` = nhãn giấy màu; `note_fn` = giấy kẻ dòng/gáy lò xo; `crayon_circle`, `torn_wipe`, `background(color)`.
- Nhân vật giấy `people()`, `brain_fn`, `swatches_fn`, `hair_fn`, `box_fn`, `film`/`clock`…
- Font: Pangolin (đủ dấu tiếng Việt), PatrickHand, Itim. **Không có glyph `→ ✓ ✗ ≠`**, dùng "-", vẽ mũi tên/dấu tick bằng ImageDraw.
- Palette: NAVY, MUST(vàng), PINK, CORAL, TEAL, PURPLE, WOOD, INK(chữ xanh), RED.

## Ví dụ (examples/), tên file = tên module khi import
| File | Nội dung / kỹ thuật chính | Helper định nghĩa |
|---|---|---|
| `render2.py` | tracking vật theo tay, khoanh đỏ + mũi tên, freeze-frame + đếm ngược 5s, câu hỏi trên/dưới có scrim | `tag`, `scrim`, `arrow`, `crop_origin` |
| `render3.py` | ảnh chân dung dán giấy, hình vẽ giấy + keyword | không có |
| `render4.py` | chữ bật theo tay (MediaPipe), ảnh/clip stock dán băng dính | `framed`, `photo`, `Clip` |
| `render5.py` | source đen (chỉ có voice) → toàn bộ collage full màn, ảnh/clip tư liệu | `Clip`, `slam`, `dashed`, `oscar_fn`, `angry_fn` |
| `render6.py` | footage tư liệu full màn rồi chuyển về người nói | `book_fn`, `magnifier_fn` |
| `render7.py` | ảnh thật Wikimedia, đám đông, thư chỉ trích, tivi/báo có ảnh bên trong | `tv_fn`, `tv_screen` |
| `render8.py` | làm tối nền nhấn mạnh (selfie segmenter) | không có |

Ví dụ phụ thuộc asset riêng (thư mục `vN/assets`, `vN/clips`, transcript, tracks) không kèm theo, đọc để học cách dựng, không chạy thẳng.

## Kỹ thuật
- **Tracking vật theo tay** (render2 + `jar_detect.py`/`jar_tracks.py`): tìm blob màu (vd nắp đỏ) từng frame, bám nearest-neighbour từ seed → khoanh đỏ, làm tối nền chừa lỗ sáng (`dim_spot`), nhãn + mũi tên chỉ lên.
- **Chữ bật theo tay** (render4 + `hands_detect.py`): MediaPipe HandLandmarker (`models/hand_landmarker.task`, tasks API, VIDEO mode) → chọn tay rảnh; chip đặt cạnh lòng bàn tay lúc pop, tránh mặt, tự dời khi chồng nhau.
- **Tách nền cảnh kết** (`cutout_birefnet_lite.py`): rembg `birefnet-general-lite` (~8 đến 20s/frame CPU) + `binary_fill_holes`; chạy 12fps (stop-motion) để rẻ. `birefnet-portrait` đẹp hơn nhưng ~60s/frame. Không chạy rembg song song với render (tốn RAM).
  Scale cả frame theo 1 hệ số cố định sau khi crop vùng quanh người, không crop theo bbox từng frame (người sẽ giật vị trí). Viền giấy xé trắng quanh mask.
- **Chèn thời gian**: freeze-frame (câu hook quá ngắn) và màn đếm ngược 5s thay khoảng im lặng; map thời gian out↔src (`o()`, `src_of()`), audio ghép bằng `atrim`+`concat`, tiếng tick tổng hợp bằng numpy.
- **Media minh họa**: ảnh/clip đặt trên giấy trắng viền xé + 2 băng dính (`framed`), nền giấy màu; clip Pexels tải bằng `https://www.pexels.com/download/video/<id>/`, cắt trước về 1280x720 30fps rồi đọc tuần tự.
- **Ảnh người nổi tiếng**: `python commons_search.py "<query>" <prefix> <N>` → tải ảnh Wikimedia Commons (in kèm license) vào `assets/`.
- **Làm tối nền nhấn mạnh** (render8 + `person_mask_segment.py`): selfie segmenter chỉ cho các khung nhấn mạnh → nền tối + khử màu, người giữ nguyên sáng, keyword to hiện trong vùng tối. Hợp cho câu hỏi/thuật ngữ then chốt.
- **Nhãn "đập" vào màn**: scale 3.2→1 trong 0.18s + rung khung 0.4s + tiếng thump (numpy) mix bằng `adelay`+`amix`.

## Nguyên tắc thẩm mỹ (áp dụng mặc định)
- Chữ phải **to**, dễ đọc; nhãn giấy chip ≥ 66px, câu chính 100 đến 124px.
- Câu hỏi gửi khán giả: chữ **trên + dưới** khung hình với **gradient đen** (`scrim`), không để khối note một bên.
- Liệt kê từ ngữ: chữ **bật ra theo tay** người nói, linh hoạt vị trí, không chỉ 2 khối cố định hai bên.
- **Ít chữ, nhiều hình**: một câu giải thích → 1 hình vẽ giấy (ADN, cán cân, biển cảnh báo…) + **1 keyword**; không xếp nhiều chip câu dài.
- **Dùng nhiều ảnh thật**, không chỉ vẽ + chữ: nhắc tên người nổi tiếng → tìm ảnh thật; "tung hô" → ảnh đám đông thật, "chỉ trích" → thẻ bình luận + phong thư bay vào, "truyền thông" → trang báo/tivi có ảnh thật bên trong.
- Chữ **không bao giờ đè mặt** người nói, vùng mặt tính theo zoom camera; không đủ chỗ thì đổi bên / thu nhỏ.
- Người nói giữ ở đầu video; đoạn dài chèn cảnh full animation cho đỡ nhàm; zoom punch / slow push tạo nhịp.
- Cân bằng hai bên khung hình (vd hình bên phải, câu tiếp theo pop bên trái).
- Cảnh thuật ngữ/kết: tách nền người, viền giấy trắng, **thu nhỏ + hạ thấp**, chữ lớn phía trên đầu.
- Khoảng lặng chờ khán giả trả lời → màn đếm ngược 5s full màn hình có tick-tock.
- Nhãn phủ định mạnh ("KHÔNG PHẢI ĐÂU!") đập thẳng vào màn hình, không che mặt người.
- File tải về có thể lẫn `.exe` giả video → **không chạy**, cảnh báo người dùng.
- Tải YouTube hay bị chặn → nhờ người dùng tự tải file.

# paper-collage-edit

**Tiếng Việt** | [English](#english)

Skill cho Claude Code: dựng video người nói khổ **16:9 (YouTube)** và **9:16 (TikTok, Reels, Shorts)** theo phong cách **cắt dán giấy xé**. Viền giấy trắng, nét bút sáp, chữ viết tay hiện dần, stop-motion 12fps, chuyển cảnh xé giấy, ảnh dán băng dính, tách nền người nói. Tất cả bằng Python và ffmpeg, không cần After Effects.

Skill gốc của [Annie-246](https://github.com/Annie-246/paper-collage-edit) (giấy phép MIT). Bản này do [ducpt.com](https://ducpt.com/?utm_source=github&utm_medium=readme&utm_campaign=paper-collage-edit) đóng gói lại: README song ngữ, kiểm tự động (CI), SKILL.md rà theo chuẩn skill của DUCPT.

## Demo

![demo](docs/demo-skool.gif)

Video demo đầy đủ (79 giây, không tiếng): [docs/demo-skool.mp4](docs/demo-skool.mp4). Video do tác giả gốc dựng hoàn toàn bằng skill này.

Kết quả `scripts/template.py` ở hai khổ (trên 16:9, dưới 9:16):

![16:9](docs/demo-16x9.jpg)
![9:16](docs/demo-9x16.jpg)

## Cài đặt

1. Chép thư mục `paper-collage-edit` vào thư mục skill của Claude Code:
   - Windows: `C:\Users\<tên bạn>\.claude\skills\`
   - macOS, Linux: `~/.claude/skills/`

   ```bash
   git clone https://github.com/ducdg88/paper-collage-edit ~/.claude/skills/paper-collage-edit
   ```
2. Cài ffmpeg (có `ffmpeg` và `ffprobe` trong PATH) và thư viện Python 3.10 trở lên:

   ```bash
   pip install -r requirements.txt
   ```
3. Mở Claude Code, gửi video kèm kịch bản và nói: *"edit video này style giấy xé"*. Làm TikTok hay Reels thì thêm *"khổ dọc 9:16"*.

## Chạy thử nhanh

```bash
cd scripts
python template.py input.mp4 out_16x9.mp4 6
PAPER_ASPECT=9:16 python template.py input.mp4 out_9x16.mp4 6
```

Windows PowerShell: `$env:PAPER_ASPECT="9:16"; python template.py input.mp4 out_9x16.mp4 6`

## Trong repo có gì

- `SKILL.md`: hướng dẫn cho Claude (quy trình, kỹ thuật, nguyên tắc thẩm mỹ).
- `scripts/render.py`: thư viện hiệu ứng giấy cho cả 16:9 và 9:16. `scripts/template.py`: file mẫu tối thiểu chạy được. Các script phụ: bắt tay, tách nền, tìm ảnh Wikimedia, chụp khung hình để kiểm.
- `examples/`: 7 file dựng mẫu (16:9) từ dự án thật, đọc để học cách dựng.
- `fonts/`: Pangolin, Patrick Hand, Itim (SIL Open Font License).
- `models/hand_landmarker.task`: MediaPipe (Apache 2.0).

---

## English

A Claude Code skill that edits talking-head videos in **16:9 (YouTube)** and **9:16 (TikTok, Reels, Shorts)** in a **torn-paper collage** style: white torn-paper borders, crayon texture, handwriting that writes itself, 12fps stop-motion, paper-tear transitions, taped photos and speaker cutouts. Pure Python and ffmpeg, no After Effects.

Original skill by [Annie-246](https://github.com/Annie-246/paper-collage-edit) (MIT License). This copy is repackaged by [ducpt.com](https://ducpt.com/?utm_source=github&utm_medium=readme&utm_campaign=paper-collage-edit): bilingual README, automated checks (CI), and a SKILL.md reviewed against the DUCPT skill standard.

### Install

1. Copy the `paper-collage-edit` folder into your Claude Code skills folder (`~/.claude/skills/`, or `C:\Users\<you>\.claude\skills\` on Windows):

   ```bash
   git clone https://github.com/ducdg88/paper-collage-edit ~/.claude/skills/paper-collage-edit
   ```
2. Install ffmpeg (`ffmpeg` and `ffprobe` on PATH) and the Python 3.10+ libraries:

   ```bash
   pip install -r requirements.txt
   ```
3. In Claude Code, send a video with its script and say: *"edit this video in torn-paper style"*. Add *"vertical 9:16"* for TikTok or Reels.

### Quick test

```bash
cd scripts
python template.py input.mp4 out_16x9.mp4 6
PAPER_ASPECT=9:16 python template.py input.mp4 out_9x16.mp4 6
```

### What is inside

- `SKILL.md`: instructions for Claude (workflow, techniques, visual rules). Written in Vietnamese.
- `scripts/render.py`: the paper effect library for both aspect ratios. `scripts/template.py`: a minimal runnable template. Helper scripts: hand detection, background cutout, Wikimedia image search, QC snapshots.
- `examples/`: 7 real project render files (16:9) to learn from.
- `fonts/`: Pangolin, Patrick Hand, Itim (SIL Open Font License).
- `models/hand_landmarker.task`: MediaPipe (Apache 2.0).

---

**Made by DUCPT.** Học cách vận hành Công ty 1 người với AI / Learn to run a one-person company with AI: [ducpt.com](https://ducpt.com/?utm_source=github&utm_medium=readme&utm_campaign=paper-collage-edit)

Giấy phép / License: mã nguồn [MIT](LICENSE) (bản quyền thuộc tác giả gốc Annie-246 / copyright of the original author Annie-246). Fonts: SIL OFL. MediaPipe model: Apache 2.0.

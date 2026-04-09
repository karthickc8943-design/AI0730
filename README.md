# AI0730 - JARVIS Modern Local AI Assistant

JARVIS is a local-first AI assistant project with:

- a ChatGPT-style web UI (`modern_chat.py`)
- tool-routing chat + automations
- voice wake/sleep conversation loop
- local image generation via ComfyUI
- file summarization (including OCR fallback for scanned PDFs)
- YouTube transcript summarization with fallback logic
- web URL fetch + summarize

This repository is designed for day-to-day personal assistant workflows on Linux.

---

## Features

### 1) Modern chat interface
- Single chat surface in `modern_chat.py`
- Compact input row with:
  - message box
  - attach icon (`📎`)
  - send button
- In-chat previews for generated images and converted files
- Auto task detection badge (`Detected: ...`)

### 2) AI routing and system actions
- Fast pattern routing + AI fallback (`command_router.py`, `core.py`)
- System automation commands (`system_automation.py`), including:
  - opening desktop apps
  - media controls
  - calendar actions
  - screenshot/lock/time/date/jokes

### 3) Voice assistant behavior
- Voice controls integrated in UI
- Wake phrases and sleep behavior implemented in `modern_chat.py`
- Supports:
  - wake by phrase
  - follow-up conversation while awake
  - auto sleep after silence timeout
  - explicit sleep phrases
  - explicit shutdown phrase

### 4) Local image generation with ComfyUI
- Image generation uses local ComfyUI server (no Pollinations dependency)
- Flow:
  - health check (`/system_stats`)
  - queue workflow (`/prompt`)
  - wait for result (`/history/{prompt_id}`)
  - fetch image (`/view`)

### 5) File intelligence
- Summarize `.txt`, `.docx`, `.csv/.xlsx`, code files, etc.
- PDF extraction with fallback logic:
  - direct text extraction
  - OCR fallback for scanned/image-based PDFs (if tools available)

### 6) YouTube and web summarization
- YouTube URL -> transcript -> summary
- Multi-path transcript fallback (API + `yt-dlp` subtitle extraction)
- URL fetch + HTML text extraction + summary

---

## Project Structure

```text
ai1/
├── __init__.py
├── chat_memory.py
├── command_router.py
├── core.py
├── faster_whisper_stt.py
├── file_converter.py
├── file_mgmt.py
├── image.py
├── jarvis.py
├── jarvis_calendar.py
├── system_automation.py
├── tts_jarvis.py
├── upload.py
├── video_analysis.py
├── web_search.py
└── README.md
```

UI/entry scripts in home directory:

- `/home/karthick/modern_chat.py` (primary UI)
- `/home/karthick/ai_chat.py` (full legacy multi-tab UI and shared functions)

---

## Environment and Requirements

Recommended platform:
- Linux (tested on Ubuntu-based setup)
- NVIDIA GPU (optional but recommended)
- Conda Python environment

Core Python packages used in this stack:
- `gradio`
- `requests`
- `pillow`
- `pytesseract`
- `PyPDF2`
- `python-docx`
- `pandas`
- `faster-whisper`
- `youtube-transcript-api`
- `yt-dlp`
- `torch`, `torchvision`, `torchaudio`, `torchsde`

System tools that improve reliability:
- `ffmpeg`
- `tesseract-ocr`
- `poppler-utils` (`pdftoppm`)

---

## Setup

## 1) Clone

```bash
git clone https://github.com/karthickc8943-design/ai0730.git
cd ai0730
```

## 2) Install dependencies (example)

Use your conda/python environment:

```bash
/home/karthick/anaconda3/bin/python -m pip install gradio requests pillow pytesseract PyPDF2 python-docx pandas faster-whisper youtube-transcript-api yt-dlp
```

If PyPI DNS is blocked in your network, use mirror:

```bash
/home/karthick/anaconda3/bin/python -m pip install <package> -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
```

## 3) ComfyUI

Install and run ComfyUI separately:

```bash
cd ~/ComfyUI
/home/karthick/anaconda3/bin/python main.py --listen 127.0.0.1 --port 8188
```

Optional custom endpoint:

```bash
export COMFYUI_URL="http://127.0.0.1:8188"
```

---

## Running the App

Primary UI:

```bash
/home/karthick/anaconda3/bin/python /home/karthick/modern_chat.py
```

Open in browser:
- `http://localhost:7866`

---

## How Auto Mode Works

The app auto-detects intent from user text + attachment type.

Examples:
- `generate an image of ...` -> image generation
- upload image + `analyze image text` -> OCR image analysis
- upload PDF + `summarize short` -> file summary
- `summarize this youtube video <url>` -> YouTube summary
- `<https://example.com/article>` -> web fetch summary

Task badge displays what mode was selected.

---

## Voice Behavior

Voice loop in `modern_chat.py`:

- starts in sleep mode
- wakes on wake phrase(s)
- ignores non-wake speech while sleeping
- supports follow-up while awake
- goes to sleep on sleep phrases
- auto-sleeps after silence timeout
- supports explicit shutdown phrase

Logs are shown in `Chat Log` tab.

---

## Local Image Generation (ComfyUI details)

Implemented in:
- `/home/karthick/modern_chat.py`
- `/home/karthick/ai_chat.py`
- `system_automation.py` (`image_gen` action)

Current default checkpoint name in workflow:
- `v1-5-pruned-emaonly.safetensors`

If your ComfyUI model name differs, update this checkpoint string in the workflow definitions.

---

## Troubleshooting

### ComfyUI connection refused
Error:
- `Failed to establish a new connection: 127.0.0.1:8188`

Fix:
- Start ComfyUI server
- verify:
  ```bash
  curl http://127.0.0.1:8188/system_stats
  ```

### Missing Python dependencies for ComfyUI
Common missing modules:
- `torchvision`
- `torchaudio`
- `torchsde`

Install with your active Python:
```bash
/home/karthick/anaconda3/bin/python -m pip install torchvision torchaudio torchsde
```

### Transformers/Hugging Face version mismatch
Pin compatible versions:

```bash
/home/karthick/anaconda3/bin/python -m pip install --force-reinstall --no-cache-dir "transformers==4.57.3" "huggingface-hub==0.36.2"
```

### YouTube transcript blocked by IP
If transcript API fails, app attempts `yt-dlp` subtitle fallback.
If both fail, video likely has unavailable captions in your region/session.

---

## Security and Privacy

- Designed for local usage
- ComfyUI image generation is local when server runs locally
- Web/YouTube features access external services by design
- Avoid committing secrets, tokens, or private local files

---

## Development Notes

- Keep code modular inside `ai1/`
- `modern_chat.py` is the primary UX layer
- `ai_chat.py` still contains useful shared functions
- Use conservative filtering in STT to avoid dropping valid user commands

---

## License

This project is licensed under the MIT License.
See the `LICENSE` file for details.

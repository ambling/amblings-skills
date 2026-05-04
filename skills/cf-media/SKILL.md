---
name: cf-media
description: Manage media processing service and analyze one or multiple files for a specific purpose
---

# cf-media

Manage the media processing service and use it to analyze one or multiple media files (images, videos, audio) for a specific purpose or question.

**User's instruction:** {{args}}

## Your Task

1. **Check/Start service** - Ensure the service is running
2. **Parse the request** - Identify the files and the purpose/question
3. **Determine analysis type** - Based on file types and purpose
4. **Call media processing service** - Use appropriate endpoints
5. **Aggregate results** - Combine analysis from multiple files if applicable
6. **Synthesize answer** - Provide a coherent response addressing the user's purpose

## Service Overview

The media processing service is running at `http://localhost:8654` with these endpoints:

| Endpoint | Input | Description |
|----------|-------|-------------|
| `POST /v1/describe-image` | Image | Generate detailed description |
| `POST /v1/visual-qa` | Image + Question | Answer questions about image |
| `POST /v1/ocr` | Image | Extract text from image |
| `POST /v1/extract-document` | Image | Extract financial data |
| `POST /v1/describe-video` | Video | Analyze video content |
| `POST /v1/analyze-audio` | Audio | Analyze audio content |
| `POST /v1/transcribe` | Audio | Transcribe speech to text |

## Approach

### 0. Service Management

**Check if service is running:**
```bash
curl -s http://localhost:8654/health
```

**Start service manually (if not using launchd/systemd):**
```bash
cd /Users/ambling/Projects/if-accountant/src/services/media-processing-service
uvicorn media_processing_service.app:app --host 0.0.0.0 --port 8654
```

**Service management (macOS launchd):**
```bash
# Check status
launchctl list | grep media-processing

# Restart service
launchctl bootout gui/$(id -u)/com.informationfactory.media-processing
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.informationfactory.media-processing.plist

# View logs
tail -f /Users/ambling/.local/share/media-processing-service/service.log
```

**Service management (Linux systemd):**
```bash
# Check status
systemctl --user status com.informationfactory.media-processing

# Restart service
systemctl --user restart com.informationfactory.media-processing

# View logs
journalctl --user -u com.informationfactory.media-processing -f
```

### 1. Parse User Request

Extract:
- **Files**: One or more file paths
- **Purpose**: What the user wants to know or accomplish

### 2. Determine Analysis Strategy

Based on file types and purpose:

**Single File:**
- Image + General question → `visual-qa`
- Image + "Describe" → `describe-image`
- Image + "Extract text" → `ocr`
- Video → `describe-video`
- Audio → `analyze-audio` or `transcribe`

**Multiple Files:**
- Compare/contrast → Analyze each, then compare
- Find specific content → Query each file
- Summarize collection → Analyze each, then aggregate

### 3. Execute Analysis

**Single file analysis:**
```bash
# Image description
curl -X POST http://localhost:8654/v1/describe-image \
  -F "file=@/path/to/image.jpg"

# Visual QA
curl -X POST http://localhost:8654/v1/visual-qa \
  -F "file=@/path/to/image.jpg" \
  -F "question=What color is the car?"

# Video analysis
curl -X POST http://localhost:8654/v1/describe-video \
  -F "file=@/path/to/video.mp4"

# Audio analysis
curl -X POST http://localhost:8654/v1/analyze-audio \
  -F "file=@/path/to/audio.mp3"
```

### 4. Aggregate Results

For multiple files:
- Store each result
- Look for patterns, similarities, differences
- Synthesize a coherent answer

### 5. Present Findings

- Address the original purpose/question
- Reference specific files when relevant
- Highlight key findings

## Input Format

Natural language with file paths:

**Single file:**
- "Describe this image: /path/to/image.jpg"
- "What's in this video: /path/to/video.mp4"
- "Extract text from: /path/to/document.jpg"

**Multiple files:**
- "Compare these images: /path/to/img1.jpg /path/to/img2.jpg"
- "Find all images with red cars: /path/to/*.jpg"
- "Describe all videos in: /path/to/folder/"

**With specific question:**
- "Are there people in these images? /path/to/img1.jpg /path/to/img2.jpg"
- "What's the common theme: /path/to/img1.jpg /path/to/img2.jpg /path/to/img3.jpg"

## Output Format

Structured response:

**Single file:**
```
File: /path/to/file.jpg
Type: Image
Analysis: {result from service}
```

**Multiple files:**
```
File 1: /path/to/file1.jpg
Analysis: {result}

File 2: /path/to/file2.jpg
Analysis: {result}

---
Synthesis: {combined insights addressing the purpose}
```

## Examples

### Example 1: Single Image Description

**Input:**
```
Describe this image: ~/Pictures/vacation.jpg
```

**Execution:**
```bash
curl -s -X POST http://localhost:8654/v1/describe-image \
  -F "file=@/Users/ambling/Pictures/vacation.jpg"
```

**Output:**
```
File: ~/Pictures/vacation.jpg
Type: Image
Analysis: A beach scene with clear blue water, white sand, and palm trees...```

---

### Example 2: Visual QA

**Input:**
```
What color is the car in ~/Documents/parking-lot.jpg?
```

**Execution:**
```bash
curl -s -X POST http://localhost:8654/v1/visual-qa \
  -F "file=@/Users/ambling/Documents/parking-lot.jpg" \
  -F "question=What color is the car?"
```

**Output:**
```
File: ~/Documents/parking-lot.jpg
Type: Image
Question: What color is the car?
Answer: The car in the image is red...```

---

### Example 3: Multiple Image Comparison

**Input:**
```
Compare these product photos: ~/Products/product1.jpg ~/Products/product2.jpg
```

**Execution:**
```bash
# Analyze each image
for img in ~/Products/product1.jpg ~/Products/product2.jpg; do
  echo "=== $img ==="
  curl -s -X POST http://localhost:8654/v1/describe-image -F "file=@$img"
done
```

**Output:**
```
File 1: ~/Products/product1.jpg
Analysis: A red ceramic vase with blue floral patterns...

File 2: ~/Products/product2.jpg
Analysis: A blue ceramic vase with red floral patterns...

---
Synthesis: Both images show ceramic vases with complementary color schemes.
Product 1 has a red base with blue flowers, while Product 2 has a blue base 
with red flowers. They appear to be from the same collection with inverted 
color palettes.
```

---

### Example 4: Video Analysis

**Input:**
```
What happens in this video: ~/Videos/tutorial.mp4
```

**Execution:**
```bash
curl -s -X POST http://localhost:8654/v1/describe-video \
  -F "file=@/Users/ambling/Videos/tutorial.mp4"
```

**Output:**
```
File: ~/Videos/tutorial.mp4
Type: Video
Analysis: This video shows a step-by-step tutorial for assembling furniture...```

---

### Example 5: OCR from Multiple Images

**Input:**
```
Extract all text from these receipts: ~/Receipts/receipt1.jpg ~/Receipts/receipt2.jpg
```

**Execution:**
```bash
for receipt in ~/Receipts/receipt1.jpg ~/Receipts/receipt2.jpg; do
  echo "=== $receipt ==="
  curl -s -X POST http://localhost:8654/v1/ocr -F "file=@$receipt"
done
```

**Output:**
```
File 1: ~/Receipts/receipt1.jpg
Text: STORE NAME, 123 Main St, Total: $45.99...

File 2: ~/Receipts/receipt2.jpg
Text: STORE NAME, 456 Oak Ave, Total: $23.50...

---
Synthesis: Two receipts from the same store at different locations.
Total combined: $69.49
```

---

### Example 6: Find Specific Content Across Images

**Input:**
```
Find any images with text containing "ERROR": ~/Logs/*.jpg
```

**Execution:**
```bash
for img in ~/Logs/*.jpg; do
  result=$(curl -s -X POST http://localhost:8654/v1/ocr -F "file=@$img")
  if echo "$result" | grep -q "ERROR"; then
    echo "FOUND in $img:"
    echo "$result"
  fi
done
```

## Error Handling

**Service Errors:**
- **Service unavailable**: `curl http://localhost:8654/health` - if fails, start/restart service
- **503 Service Unavailable**: LLM backend is down
- **Connection refused**: Service not running, start it

**Request Errors:**
- **400 Bad Request**: Invalid input or processing error
- **422 Validation Error**: Missing required fields
- **Invalid file type**: Verify file extension matches expected format

**Processing Notes:**
- **Large files**: Video/audio may take longer to process
- **Model limitations**: Audio analysis requires audio-capable model (Ultravox, Voxtral, etc.)

---

## Service Configuration

**Environment variables** (in `~/.config/media-processing-service/environment`):
- `LLM_BASE_URL`: LLM API endpoint (default: `http://z790home:8004/v1`)
- `LLM_API_KEY`: API key for LLM (default: `sk-empty` for local)
- `LLM_MODEL`: Model name (default: `Qwen3.5-27B.Q4_K_M.gguf`)
- `LLM_TIMEOUT`: Request timeout in seconds (default: `120`)
- `MEDIA_PROCESSING_PORT`: Service port (default: `8654`)
- `MEDIA_PROCESSING_HOST`: Service host (default: `0.0.0.0`)

**To change configuration:**
```bash
nano ~/.config/media-processing-service/environment
# Then restart the service
launchctl bootout gui/$(id -u)/com.informationfactory.media-processing
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.informationfactory.media-processing.plist
```

---

## Quick Reference

**Common operations:**

| Operation | Command |
|-----------|--------|
| Health check | `curl http://localhost:8654/health` |
| Service info | `curl http://localhost:8654/` |
| Describe image | `curl -X POST http://localhost:8654/v1/describe-image -F "file=@path.jpg"` |
| Visual QA | `curl -X POST http://localhost:8654/v1/visual-qa -F "file=@path.jpg" -F "question=..."` |
| OCR | `curl -X POST http://localhost:8654/v1/ocr -F "file=@path.jpg"` |
| Video analysis | `curl -X POST http://localhost:8654/v1/describe-video -F "file=@path.mp4"` |
| Audio analysis | `curl -X POST http://localhost:8654/v1/analyze-audio -F "file=@path.mp3"` |

## Verification

Before analyzing files:
```bash
# Check service health
curl http://localhost:8654/health

# List available endpoints
curl http://localhost:8654/
```

## Notes

- **Image formats**: jpg, jpeg, png, gif, bmp
- **Video formats**: mp4, avi, mov, mkv, webm
- **Audio formats**: mp3, wav, ogg, m4a, flac
- **Max sizes**: Images 10MB, Video 100MB, Audio 50MB
- **Video analysis**: Extracts frames internally (up to 32 frames, max 8 used)
- **Audio analysis**: Requires audio-capable model (not available with default qwen3.5-27b)

---

## Service Installation

**First-time setup:**
```bash
cd /Users/ambling/Projects/if-accountant/src/services/media-processing-service
./install-user.sh
```

**Update service:**
```bash
./install-user.sh --update
```

**Service location:**
- Installation: `~/.local/share/media-processing-service/`
- Config: `~/.config/media-processing-service/environment`
- Logs: `~/.local/share/media-processing-service/service.log`

**Convenience scripts:**
```bash
~/.local/share/media-processing-service/start.sh
~/.local/share/media-processing-service/stop.sh
~/.local/share/media-processing-service/restart.sh
~/.local/share/media-processing-service/health-check.sh
```

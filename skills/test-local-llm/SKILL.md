---
name: test-local-llm
description: Test local LLM endpoints for capability and performance
---

# test-local-llm

Test local LLM endpoints directly (native llama.cpp/vLLM connections) for capability and performance, then generate a report.

**User's instruction:** test-local-llm to test both native endpoint and litellm endpoint as above and generate a report in @docs/operations/local-llm-tests/<date>-<model>-<setting>.md in terms of both capability (text, thinking, image, video, audio) and performance (token/s)

## Overview

This skill tests local LLM endpoints comprehensively, covering:
- **Text capabilities**: Simple queries, reasoning, code generation
- **Thinking mode**: Internal reasoning behavior and token consumption
- **Vision**: Image understanding (colors, objects, scenes)
- **Video**: Frame extraction and analysis
- **Performance**: Tokens per second for prompt processing and generation

## Your Task

1. Test the native endpoint directly (e.g., http://z790home:8003, http://z790home:8004)
2. Test multiple capabilities: text, thinking, vision, video
3. Measure performance metrics: prompt tokens/s, generation tokens/s
4. Generate a dated report in `docs/operations/local-llm-tests/`

## Approach

### 1. Environment Setup
- Check if `.env` file exists and source it for API keys (if needed)
- Verify endpoints are accessible:
  - llama.cpp 9B: `http://z790home:8003/health`
  - llama.cpp 27B: `http://z790home:8004/health`
  - vLLM 9B: `http://z790home:8002/health`
  - sglang 35B: `http://z790home:8000/health`
- Get model info from endpoints (`/props`, `/v1/models`)

### 2. Run Test Suite

Create and execute a comprehensive test script covering:

**Text Tests:**
- Simple factual query (e.g., "What is 2+2?")
- Creative writing (e.g., haiku)
- Code generation (simple function)

**Thinking Mode Tests:**
- Query with and without thinking enabled
- Measure reasoning token consumption
- Compare response quality

**Vision Tests:**
- Small base64 image (1x1 pixel color test)
- Real image from `~/iCloud/ClawFarm/Tests/` if available
- Resize images to 512x512 for faster processing using ffmpeg

**Video Tests:**
- Extract frame from video in `~/iCloud/ClawFarm/Tests/`
- Test frame analysis at multiple timestamps (start, middle, end)

**Performance Metrics:**
- Prompt processing speed (tokens/s)
- Generation speed (tokens/s)
- Total tokens used
- Thinking token ratio

### 3. Generate Report

Create a report file at:
```
docs/operations/local-llm-tests/YYYY-MM-DD-<model>-<setting>.md
```

Where:
- `<model>`: Model identifier (e.g., `qwen3.5-9b-gguf`, `qwen3.5-27b-gguf`)
- `<setting>`: Configuration identifier (e.g., `default`, `128k-context`, `benchmark`)

### 4. Report Template

```markdown
# Local LLM Test Report: {Model Name}

**Date**: {YYYY-MM-DD HH:MM}
**Model**: {model_id}
**Backend**: {llama.cpp | vLLM | sglang}
**Context**: {context_window} tokens

## Configuration

- **Native Endpoint**: {url}
- **Quantization**: {Q4_0 | Q4_K_M | FP8 | etc.}
- **Vision Support**: {Yes | No} (mmproj: {loaded | not loaded})
- **Slots**: {number of concurrent requests}

## Test Results

### Text Capabilities

| Test | Result | Speed | Notes |
|------|--------|-------|-------|
| Simple Query | {result} | {tokens/s} | {notes} |
| Creative Writing | {result} | {tokens/s} | {notes} |
| Code Generation | {result} | {tokens/s} | {notes} |

### Thinking Mode

- **Thinking Enabled**: {Yes | No}
- **Reasoning Tokens**: {count}
- **Final Answer Tokens**: {count}
- **Total Tokens**: {count}

### Vision Capabilities

| Test | Result | Confidence | Notes |
|------|--------|------------|-------|
| Color Recognition | {color} | {high/medium/low} | {notes} |
| Object Detection | {objects} | {high/medium/low} | {notes} |
| Scene Description | {description} | {high/medium/low} | {notes} |
| Video Frame (T={time}s) | {description} | {high/medium/low} | {notes} |

### Performance Metrics

| Metric | Value |
|--------|-------|
| Prompt Speed | {tokens/s} |
| Generation Speed | {tokens/s} |
| Total Tokens | {count} |
| Thinking Token Ratio | {percentage} |

## Capability Matrix

| Capability | Working | Notes |
|------------|---------|-------|
| Text Generation | ✓ / ✗ | |
| Thinking Mode | ✓ / ✗ | |
| Vision (Image) | ✓ / ✗ | |
| Vision (Video Frame) | ✓ / ✗ | |
| Audio | ✓ / ✗ | Not tested |

## Issues & Notes

- {any issues encountered}
- {configuration notes}
- {performance bottlenecks}
- {recommendations}

## Test Environment

- **Backend**: {llama.cpp | vLLM | sglang}
- **Model Path**: Server on {host:port}
- **Test Files**: {files used}

## Conclusion

{summary of findings and recommendations}

---
*Report generated: {date}*
*Model: {model_id}*
*Backend: {backend}*
```

## Input Format

Optional arguments:
- `--model <model_id>`: Specify model to test (default: auto-detect)
- `--endpoint <url>`: Custom endpoint URL
- `--quick`: Run quick tests only (skip video)
- `--output <path>`: Custom report path

## Output Format

Markdown report saved to `docs/operations/local-llm-tests/YYYY-MM-DD-<model>-<setting>.md`

## Examples

### Basic Usage
```
/test-local-llm
```
Tests all configured local models and generates dated reports.

### Test Specific Model
```
/test-local-llm --model qwen3.5-27b-gguf
```

### Quick Test (Skip Video)
```
/test-local-llm --quick
```

## Testing Checklist

Before running tests, ensure:
- [ ] Endpoints are accessible (`curl` test)
- [ ] Test media files exist in `~/iCloud/ClawFarm/Tests/`
- [ ] ffmpeg is available for image/video processing
- [ ] Sufficient disk space for reports
- [ ] Report directory exists: `docs/operations/local-llm-tests/`

## Available Endpoints

| Port | Backend | Model | Context |
|------|----------|-------|---------|
| 8000 | sglang | Qwen3.5-35B-A3B | 24K |
| 8002 | vLLM | Qwen3.5-9B | 32K |
| 8003 | llama.cpp | Qwen3.5-9B-GGUF | 32K |
| 8004 | llama.cpp | Qwen3.5-27B-GGUF | 128K |

## Related Skills

- `openclaw` - OpenClaw specific queries
- `feature` - Implement new features
- `test` - Run test suites

## References

- `@workspace/openclaw.json` - OpenClaw model configuration
- `@docs/operations/local-llm-tests/` - Previous test reports

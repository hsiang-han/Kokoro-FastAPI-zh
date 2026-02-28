# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在处理本仓库代码时提供指引。
This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目简介 / What This Project Is

[Kokoro-82M](https://huggingface.co/hexgrad/Kokoro-82M) TTS 模型的 FastAPI 封装。在 8880 端口暴露兼容 OpenAI 的 `/v1/audio/speech` 接口，以及开发/调试接口。支持 NVIDIA GPU（CUDA）、Apple Silicon（MPS）、CPU 和 ROCm。
FastAPI wrapper for the [Kokoro-82M](https://huggingface.co/hexgrad/Kokoro-82M) TTS model. Exposes an OpenAI-compatible `/v1/audio/speech` endpoint on port 8880, plus dev/debug endpoints. Supports NVIDIA GPU (CUDA), Apple Silicon (MPS), CPU, and ROCm.

## 常用命令 / Commands

### 环境安装 / Setup
```bash
uv venv && source .venv/bin/activate
uv pip install -e ".[test,cpu]"   # CPU 开发环境 / CPU dev setup
uv pip install -e ".[test,gpu]"   # GPU 开发环境 / GPU dev setup
```

### 下载模型（首次运行前必须执行）/ Download Model (required before first run)
```bash
python docker/scripts/download_model.py --output api/src/models/v1_0
```

### 本地运行 / Run Locally
```bash
./start-cpu.sh    # CPU（设置环境变量并启动 uvicorn）/ CPU (sets env vars and starts uvicorn)
./start-gpu.sh    # NVIDIA GPU
```

### Docker 运行 / Run via Docker
```bash
docker compose -f docker/cpu/docker-compose.yml up --build
docker compose -f docker/gpu/docker-compose.yml up --build
```

### 测试 / Tests
```bash
uv run pytest                                      # 全部测试 / All tests
uv run pytest api/tests/ --asyncio-mode=auto --cov=api --cov-report=term-missing
uv run pytest api/tests/test_openai_endpoints.py  # 单个测试文件 / Single test file
uv run pytest api/tests/test_tts_service.py -k "test_name"  # 单个测试 / Single test
```

### 代码格式化与检查 / Lint & Format
```bash
ruff format .
ruff check . --fix
```

## 架构 / Architecture

### 请求流程 / Request Flow
```
HTTP Request → FastAPI Router → TTSService → ModelManager → KokoroV1 (inference)
                                          ↘ VoiceManager (voice tensor loading)
                          ↓
                   AudioService (format conversion) → StreamingAudioWriter → Response
```

### 核心模块 / Key Modules

**`api/src/main.py`** — FastAPI 应用初始化、生命周期管理（启动时模型预热）、路由挂载。
FastAPI app initialization, lifespan (model warmup on startup), router mounting.

**`api/src/core/config.py`** — 通过 `Settings(BaseSettings)` 管理所有配置，从环境变量或 `.env` 读取。关键配置项：`USE_GPU`、`MODEL_DIR`、`VOICES_DIR`、`TARGET_MIN_TOKENS`、`TARGET_MAX_TOKENS`、`ABSOLUTE_MAX_TOKENS`。全局 `settings` 单例。
All settings via `Settings(BaseSettings)`. Reads from environment or `.env`. Key settings: `USE_GPU`, `MODEL_DIR`, `VOICES_DIR`, `TARGET_MIN_TOKENS`, `TARGET_MAX_TOKENS`, `ABSOLUTE_MAX_TOKENS`. Global `settings` singleton.

**`api/src/core/model_config.py`** — 模型专属配置（模型文件路径 `v1_0/kokoro-v1_0.pth`、语音缓存大小）。全局 `model_config` 单例。
Model-specific config (model filename path `v1_0/kokoro-v1_0.pth`, voice cache size). Global `model_config` singleton.

**`api/src/inference/kokoro_v1.py`** — `KokoroV1(BaseModelBackend)`：封装 `kokoro` 包中的 `KModel` 与 `KPipeline`，每个 `lang_code` 保存一个 `KPipeline`，处理设备选择（cuda/mps/cpu）。
`KokoroV1(BaseModelBackend)`: wraps `KModel` + `KPipeline` from the `kokoro` package. Stores one `KPipeline` per `lang_code`. Handles device selection (cuda/mps/cpu).

**`api/src/inference/model_manager.py`** — `ModelManager` 单例。通过 `initialize_with_warmup()` 在启动时加载模型，`generate()` 委托给 `KokoroV1`。
`ModelManager` singleton. Loads model on startup via `initialize_with_warmup()`. `generate()` delegates to `KokoroV1`.

**`api/src/inference/voice_manager.py`** — `VoiceManager` 单例。从 `api/src/voices/v1_0/` 加载 `.pt` 语音张量，支持语音混合（加权张量组合）。
`VoiceManager` singleton. Loads `.pt` voice tensors from `api/src/voices/v1_0/`. Supports voice mixing (weighted tensor combinations).

**`api/src/services/tts_service.py`** — `TTSService`：统筹文本分块 → 音频生成 → 格式转换流程。并发通过 `asyncio.Semaphore(4)` 限制，`generate_audio_stream()` 以流式方式产出 `AudioChunk` 对象。
`TTSService`: orchestrates text chunking → audio generation → format conversion. Concurrency limited with `asyncio.Semaphore(4)`. `generate_audio_stream()` yields `AudioChunk` objects for streaming.

**`api/src/services/text_processing/`** — 文本处理流水线：`normalizer.py`（预处理）、`phonemizer.py`（封装 misaki）、`text_processor.py`（`process_text_chunk`、`smart_split` 按句子边界分割长文本）。
Text pipeline: `normalizer.py` (pre-processing), `phonemizer.py` (wraps misaki), `text_processor.py` (`process_text_chunk`, `smart_split` for chunking long text at sentence boundaries).

**`api/src/services/audio.py`** — `AudioService.convert_audio()` 处理格式转换（mp3/wav/opus/flac/pcm）。`AudioNormalizer` 剪除流式分块间的静音。
`AudioService.convert_audio()` handles format conversion (mp3/wav/opus/flac/pcm). `AudioNormalizer` trims silence between streaming chunks.

**`api/src/services/streaming_audio_writer.py`** — `StreamingAudioWriter`：带状态的流式音频编码器，包含正确的容器头信息。
`StreamingAudioWriter`: stateful encoder for streaming audio output with proper container headers.

**`api/src/routers/openai_compatible.py`** — `/v1/audio/speech`、`/v1/audio/voices`、`/v1/audio/voices/combine`、`/v1/models`。同时处理流式与非流式响应，可通过临时文件提供可选下载链接。
`/v1/audio/speech`, `/v1/audio/voices`, `/v1/audio/voices/combine`, `/v1/models`. Handles both streaming and non-streaming responses, optional download links via temp files.

**`api/src/routers/development.py`** — `/dev/phonemize`、`/dev/generate_from_phonemes`、`/dev/captioned_speech`（词级时间戳）。
`/dev/phonemize`, `/dev/generate_from_phonemes`, `/dev/captioned_speech` (word-level timestamps).

**`api/src/routers/debug.py`** — `/debug/threads`、`/debug/storage`、`/debug/system`、`/debug/session_pools`。
`/debug/threads`, `/debug/storage`, `/debug/system`, `/debug/session_pools`.

**`api/src/structures/schemas.py`** — Pydantic 模型：`OpenAISpeechRequest`、`CaptionedSpeechRequest`、`NormalizationOptions`、`WordTimestamp`。
Pydantic models: `OpenAISpeechRequest`, `CaptionedSpeechRequest`, `NormalizationOptions`, `WordTimestamp`.

**`api/src/core/openai_mappings.json`** — 将 OpenAI 语音/模型名称映射到 Kokoro 内部名称。
Maps OpenAI voice/model names to internal Kokoro names.

### 单例模式 / Singletons Pattern

`ModelManager` 与 `VoiceManager` 均采用单例模式（`_instance` 类变量），配合异步 `get_manager()` 工厂函数。`TTSService` 也作为全局单例在 OpenAI 路由中使用，通过锁保护的 `get_tts_service()` 获取。
Both `ModelManager` and `VoiceManager` use the singleton pattern (`_instance` class variable) with async `get_manager()` factory functions. `TTSService` is also used as a global singleton in the OpenAI router via a lock-guarded `get_tts_service()`.

### 文本分块 / Text Chunking

长文本按句子边界拆分为多个分块，可通过环境变量配置：`TARGET_MIN_TOKENS`（175）、`TARGET_MAX_TOKENS`（250）、`ABSOLUTE_MAX_TOKENS`（450）。每个分块独立处理，音频以流式方式传输/拼接。
Long text is split into chunks at sentence boundaries. Configurable via env vars: `TARGET_MIN_TOKENS` (175), `TARGET_MAX_TOKENS` (250), `ABSOLUTE_MAX_TOKENS` (450). Each chunk is processed independently and audio is streamed/stitched.

### 语音混合 / Voice Mixing

语音以 `voice1+voice2` 或 `voice1(2)+voice2(1)`（加权）格式指定，语音张量在推理前进行数值混合。语音名称的首字母决定默认 `lang_code`（例如 `af_` → 美式英语，`jf_` → 日语）。
Voices are specified as `voice1+voice2` or `voice1(2)+voice2(1)` (weighted). The voice tensors are blended numerically before inference. The first letter of the voice name determines the default `lang_code` (e.g., `af_` → American English, `jf_` → Japanese).

### 模型文件位置 / Model Files Location

- 模型 / Model: `api/src/models/v1_0/kokoro-v1_0.pth` + `config.json`
- 语音 / Voices: `api/src/voices/v1_0/*.pt`

### 测试结构 / Test Structure

`api/tests/` 中的测试使用 pytest-asyncio，`asyncio_mode = auto`。`conftest.py` 提供已模拟的 `ModelManager`、`VoiceManager` 和 `TTSService` fixtures。从磁盘加载真实语音张量（`af_bella.pt`）和预生成的音频 numpy 数组，以确保测试的一致性。
Tests in `api/tests/` use pytest-asyncio with `asyncio_mode = auto`. `conftest.py` provides mocked `ModelManager`, `VoiceManager`, and `TTSService` fixtures. A real voice tensor (`af_bella.pt`) and pre-generated audio numpy array are loaded from disk for consistent testing.

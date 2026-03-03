[![CI](https://github.com/hsiang-han/Kokoro-FastAPI-zh/actions/workflows/ci.yml/badge.svg?branch=master)](https://github.com/hsiang-han/Kokoro-FastAPI-zh/actions/workflows/ci.yml)

## 本版本变更说明 / Fork Changes

> 基于 [remsky/Kokoro-FastAPI](https://github.com/remsky/Kokoro-FastAPI) 的分支版本，以下为关键差异。
> Forked from [remsky/Kokoro-FastAPI](https://github.com/remsky/Kokoro-FastAPI), with key upstream differences listed below.

### PyTorch & CUDA 升级（支持 RTX 50 系列 GPU）/ PyTorch & CUDA Upgrade (RTX 50-series GPU Support)

PyTorch 升级至 2.10.0，CUDA 路线切换到 cu128（12.8），用于适配 NVIDIA Blackwell（RTX 5060–5090，sm_120）。

Upgraded to PyTorch 2.10.0 with CUDA cu128 (12.8) for NVIDIA Blackwell support.

参考 / References: [Issue #365](https://github.com/remsky/Kokoro-FastAPI/issues/365) · [PR #390](https://github.com/remsky/Kokoro-FastAPI/pull/390)

### Kokoro-82M-v1.1-zh 迁移（中文优化 + 宿主机可扩展模型/语音目录） / Kokoro-82M-v1.1-zh Migration (Chinese optimization + host-extensible model/voice directories)

已迁移到 `hexgrad/Kokoro-82M-v1.1-zh`，并按社区方案完成基线适配（Issue #214 / PR #237）。

- 默认配置：模型 `v1_1_zh/kokoro-v1_1-zh.pth`，语音 `zf_094`，`REPO_ID=hexgrad/Kokoro-82M-v1.1-zh`
- 中英混读：`lang_code='z'` 时启用 `en_callable`，英文片段由英文 pipeline 处理
- 下载与依赖：`docker/scripts/download_model.py` 支持下载 v1.1-zh 模型与 `voices/*.pt`，新增依赖 `huggingface-hub`
- Docker 映射已按 v1.1-zh 模型与语音目录调整（具体挂载路径请以各平台 compose 为准）

Migrated to `hexgrad/Kokoro-82M-v1.1-zh` with baseline adaptation based on the community approach (Issue #214 / PR #237).

- Default config: model `v1_1_zh/kokoro-v1_1-zh.pth`, voice `zf_094`, `REPO_ID=hexgrad/Kokoro-82M-v1.1-zh`
- Mixed Chinese/English reading: when `lang_code='z'`, `en_callable` is enabled and English segments are handled by the English pipeline
- Download and dependencies: `docker/scripts/download_model.py` supports downloading the v1.1-zh model and `voices/*.pt`, with added `huggingface-hub` dependency
- Docker mappings were updated for v1.1-zh model/voice directories (final mount paths should follow each platform compose file)

参考 / References: [Issue #214](https://github.com/remsky/Kokoro-FastAPI/issues/214) · [PR #237](https://github.com/remsky/Kokoro-FastAPI/pull/237)

---

## 快速导航 / Quick Navigation

- [一条命令启动（预构建镜像） / One-command Start](#一条命令启动预构建镜像--one-command-start-prebuilt-image)
- [测试范围说明 / Testing Scope](#测试范围说明--testing-scope)
- [源码部署 / Source Deployment](#源码部署--source-deployment)
- [免源码部署入口 / No-source Deployment Entry](#免源码部署入口--no-source-deployment-entry)
- [镜像标签策略 / Image Tagging Strategy](#镜像标签策略--image-tagging-strategy)
- [最小环境变量 / Minimal Environment Variables](#最小环境变量--minimal-environment-variables)
- [切换到 v1.0（英文模型） / Switch to v1.0](#切换到-v10英文模型--switch-to-v10)
- [Get Started](#get-started)

## 测试范围说明 / Testing Scope

- 已完成实测：NVIDIA RTX 50 系列（Blackwell）+ CUDA 路线，以及 CPU 路线。
- 当前未实测：ROCm（受限于测试硬件条件），相关支持为配置级/依赖级适配，欢迎社区反馈。

- Verified in real environment: NVIDIA RTX 50 series (Blackwell) + CUDA stack, plus CPU path.
- Not yet tested in real hardware: ROCm (hardware-limited on our side). Support is currently at configuration/dependency compatibility level; community feedback is welcome.

## 一条命令启动（预构建镜像） / One-command Start (Prebuilt Image)

不编译源码、最简单直接：

```bash
docker run --rm -p 8880:8880 ghcr.io/hsiang-han/kokoro-fastapi-zh-cpu:latest
```

- 适用：CPU、Apple Silicon（M1/M2/M3）或先快速验证是否可用

启动后访问：

- API 服务地址：`http://localhost:8880`
- API 文档地址：`http://localhost:8880/docs`
- Web 界面地址：`http://localhost:8880/web`

NVIDIA GPU（同样一条命令）：

```bash
docker run --rm --gpus all -p 8880:8880 ghcr.io/hsiang-han/kokoro-fastapi-zh-gpu:latest
```

English:

The simplest way without building from source:

```bash
docker run --rm -p 8880:8880 ghcr.io/hsiang-han/kokoro-fastapi-zh-cpu:latest
```

- Use this for CPU, Apple Silicon (M1/M2/M3), or a quick smoke test

After startup, visit:

- API endpoint: `http://localhost:8880`
- API docs: `http://localhost:8880/docs`
- Web UI: `http://localhost:8880/web`

NVIDIA GPU (also one command):

```bash
docker run --rm --gpus all -p 8880:8880 ghcr.io/hsiang-han/kokoro-fastapi-zh-gpu:latest
```

## 源码编译部署 / Source Deployment

### Docker Compose

按下面步骤执行，第一次部署也可以成功。

1. 准备环境
   - 安装并启动 [Docker Desktop](https://www.docker.com/products/docker-desktop/)
   - 打开终端，确认 Docker 可用：
    ```bash
    docker --version
    docker compose version
    ```

2. 拉取代码（使用本仓库）

    ```bash
    git clone https://github.com/hsiang-han/Kokoro-FastAPI-zh.git
    cd Kokoro-FastAPI-zh
    ```

3. 选择运行模式（只选一个）
   - NVIDIA 显卡：
     ```bash
     cd docker/gpu
     ```
   - CPU（无 NVIDIA 显卡 / Apple Silicon / 仅想先跑通）：
     ```bash
     cd docker/cpu
     ```

4. 启动服务
   ```bash
   docker compose up --build
   ```
   - 首次启动会自动下载模型与语音，可能需要几分钟，请等待日志出现 `Uvicorn running on`。

5. 验证是否启动成功
   - 浏览器打开：`http://localhost:8880/docs`（能打开即成功）
   - Linux/macOS 终端验证：
     ```bash
     curl http://localhost:8880/v1/models
     ```
   - Windows PowerShell 验证：
     ```powershell
     Invoke-RestMethod http://localhost:8880/v1/models
     ```
   - API 服务地址：`http://localhost:8880`
   - API 文档地址：`http://localhost:8880/docs`
   - Web 界面地址：`http://localhost:8880/web`

6. 常用后续操作
   - 停止服务：在当前终端按 `Ctrl + C`
   - 后台运行：`docker compose up -d`
   - 查看日志：`docker compose logs -f`
   - 更新后重建：`docker compose up --build -d`

> 提示：如果你是 M1/M2/M3 Mac，请使用 `docker/cpu`。当前 GPU compose 依赖 CUDA，不支持 Apple Silicon。

English:

Follow the steps below. First-time deployment should work directly.

1. Prepare environment
   - Install and start [Docker Desktop](https://www.docker.com/products/docker-desktop/)
   - Open a terminal and verify Docker is available:
     ```bash
     docker --version
     docker compose version
     ```

2. Clone this repository

    ```bash
    git clone https://github.com/hsiang-han/Kokoro-FastAPI-zh.git
    cd Kokoro-FastAPI-zh
    ```

3. Choose one runtime mode
   - NVIDIA GPU:
     ```bash
     cd docker/gpu
     ```
   - CPU (no NVIDIA GPU / Apple Silicon / quick smoke test):
     ```bash
     cd docker/cpu
     ```

4. Start service
   ```bash
   docker compose up --build
   ```
   - First startup will auto-download model and voices. It may take a few minutes; wait until logs show `Uvicorn running on`.

5. Verify startup success
   - Open in browser: `http://localhost:8880/docs` (if reachable, deployment is successful)
   - Linux/macOS terminal check:
     ```bash
     curl http://localhost:8880/v1/models
     ```
   - Windows PowerShell check:
     ```powershell
     Invoke-RestMethod http://localhost:8880/v1/models
     ```
   - API endpoint: `http://localhost:8880`
   - API docs: `http://localhost:8880/docs`
   - Web UI: `http://localhost:8880/web`

6. Common follow-up operations
   - Stop service: press `Ctrl + C` in current terminal
   - Run in background: `docker compose up -d`
   - View logs: `docker compose logs -f`
   - Rebuild after update: `docker compose up --build -d`

> Tip: If you use M1/M2/M3 Mac, use `docker/cpu`. The current GPU compose depends on CUDA and does not support Apple Silicon.

## 免源码部署入口 / No-source Deployment Entry

- 部署文档：`docs/deployment/unraid-and-prebuilt-images.md`
- Unraid / Compose Stack（GPU）：`docker/unraid/stack.gpu.image.yml`
- Unraid / Compose Stack（CPU）：`docker/unraid/stack.cpu.image.yml`
- Unraid CA 模板（统一 CPU/GPU）：`unraid/templates/kokoro-fastapi-zh.xml`

English:

- Deployment guide: `docs/deployment/unraid-and-prebuilt-images.md`
- Unraid / Compose Stack (GPU): `docker/unraid/stack.gpu.image.yml`
- Unraid / Compose Stack (CPU): `docker/unraid/stack.cpu.image.yml`
- Unraid CA template (unified CPU/GPU): `unraid/templates/kokoro-fastapi-zh.xml`

### Unraid 可视化 Docker Compose 部署（不需要编译源码）

下面是只用 Unraid 网页界面完成部署的方法。

English:

Below is a deployment method using only the Unraid web UI.

#### 1) 前置条件

- 你已经安装并启用 Unraid 的 Docker 功能
- 你已经安装 Docker Compose Manager（或可创建 Stack 的 Compose 插件）
- 如果用 GPU 版，主机需已配置 NVIDIA 驱动/runtime
- 已创建挂载目录并配置好权限（参见 [挂载目录权限问题](#6-挂载目录权限问题permission-denied)）

English:

- Unraid Docker service is installed and enabled
- Docker Compose Manager (or another Stack-capable compose plugin) is installed
- For GPU mode, host NVIDIA driver/runtime must be configured
- Mount directories are created with proper permissions (see [permission section](#6-挂载目录权限问题permission-denied))

#### 2) 在 Unraid WebUI 创建 Stack

1. 进入 `Docker` 页面，打开 `Compose` / `Stacks` 管理页面
2. 点击 `Add New Stack`
3. Stack 名称建议填：`kokoro-fastapi-zh-gpu`（或 `kokoro-fastapi-zh-cpu`）
4. 将下面对应 YAML 直接粘贴到编辑框

English:

1. Go to `Docker` page and open `Compose` / `Stacks`
2. Click `Add New Stack`
3. Suggested stack name: `kokoro-fastapi-zh-gpu` (or `kokoro-fastapi-zh-cpu`)
4. Paste one of the YAML files below into the editor

#### 3) 粘贴 YAML（GPU 二选一）

English: Paste YAML (choose one)

**GPU（NVIDIA）**

```yaml
name: kokoro-fastapi-zh-gpu

services:
  kokoro-fastapi:
    image: ghcr.io/hsiang-han/kokoro-fastapi-zh-gpu:latest
    container_name: kokoro-fastapi-zh-gpu
    labels:
      net.unraid.docker.icon: https://raw.githubusercontent.com/hsiang-han/Kokoro-FastAPI-zh/master/assets/unraid-icon.png
    volumes:
      - /mnt/user/appdata/kokoro-fastapi-zh/models:/app/api/src/models
      - /mnt/user/appdata/kokoro-fastapi-zh/voices:/app/api/src/voices
    ports:
      - "8880:8880"
    environment:
      - PYTHONPATH=/app:/app/api
      - USE_GPU=true
      - PYTHONUNBUFFERED=1
      - REPO_ID=hexgrad/Kokoro-82M-v1.1-zh
      - DEFAULT_VOICE=zf_094
      - KOKORO_V1_FILE=v1_1_zh/kokoro-v1_1-zh.pth
      - VOICES_DIR=/app/api/src/voices/v1_1_zh
      - DEFAULT_VOICE_CODE=z
      - API_LOG_LEVEL=DEBUG
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

**CPU**

```yaml
name: kokoro-fastapi-zh-cpu

services:
  kokoro-fastapi:
    image: ghcr.io/hsiang-han/kokoro-fastapi-zh-cpu:latest
    container_name: kokoro-fastapi-zh-cpu
    labels:
      net.unraid.docker.icon: https://raw.githubusercontent.com/hsiang-han/Kokoro-FastAPI-zh/master/assets/unraid-icon.png
    volumes:
      - /mnt/user/appdata/kokoro-fastapi-zh/models:/app/api/src/models
      - /mnt/user/appdata/kokoro-fastapi-zh/voices:/app/api/src/voices
    ports:
      - "8880:8880"
    environment:
      - PYTHONPATH=/app:/app/api
      - USE_GPU=false
      - PYTHONUNBUFFERED=1
      - REPO_ID=hexgrad/Kokoro-82M-v1.1-zh
      - DEFAULT_VOICE=zf_094
      - KOKORO_V1_FILE=v1_1_zh/kokoro-v1_1-zh.pth
      - VOICES_DIR=/app/api/src/voices/v1_1_zh
      - DEFAULT_VOICE_CODE=z
      - API_LOG_LEVEL=DEBUG
```

#### 4) 部署与验证

1. 点击 `Deploy`
2. 首次启动会自动下载模型/语音到：
    - `/mnt/user/appdata/kokoro-fastapi-zh/models/v1_1_zh/`
    - `/mnt/user/appdata/kokoro-fastapi-zh/voices/v1_1_zh/`
3. 在容器日志看到 `Uvicorn running on` 后，浏览器打开：`http://<你的UnraidIP>:8880/docs`
4. 能打开接口文档页面即部署成功

- API 服务地址：http://localhost:8880
- API 文档地址：http://localhost:8880/docs
- Web 界面地址：http://localhost:8880/web

English:

1. Click `Deploy`
2. First startup will auto-download model/voices to:
  - `/mnt/user/appdata/kokoro-fastapi-zh/models/v1_1_zh/`
  - `/mnt/user/appdata/kokoro-fastapi-zh/voices/v1_1_zh/`
3. Once logs show `Uvicorn running on`, open `http://<your Unraid IP>:8880/docs`
4. If docs page opens, deployment is successful

- API endpoint: http://localhost:8880
- API docs: http://localhost:8880/docs
- Web UI: http://localhost:8880/web

#### 5) 常见问题

- 端口冲突：把 YAML 里的 `8880:8880` 改成 `18880:8880`（外部端口可改）
- 拉取镜像失败：确认 Unraid 主机能访问 GHCR，必要时先手动 pull
- GPU 不生效：确认已安装 NVIDIA 驱动插件并启用 Docker 的 NVIDIA runtime
- 想固定版本：将 `:latest` 改为固定 tag（如 `:v0.2.4-zh`）
- 挂载目录权限错误（`Permission denied`）：见下方"权限问题"说明

English:

- Port conflict: change `8880:8880` to `18880:8880` (external port is customizable)
- Image pull failure: confirm Unraid host can access GHCR; manually pull if needed
- GPU not working: confirm NVIDIA driver plugin is installed and Docker NVIDIA runtime is enabled
- Pin version: replace `:latest` with a fixed tag (e.g. `:v0.2.4-zh`)
- Mount permission error (`Permission denied`): see the permission section below

#### 6) 挂载目录权限问题（Permission Denied）

容器内进程以非 root 用户 `appuser`（UID 1000）运行。如果宿主机挂载目录的所有者不是 UID 1000，容器将无法写入，出现类似错误：

```
PermissionError: [Errno 13] Permission denied: 'api/src/models/v1_1_zh'
```

**修复方法（在 Unraid 终端执行，仅需首次部署前操作一次）：**

```bash
# 1. 预创建挂载目录 / Create mount directories
mkdir -p /mnt/user/appdata/kokoro-fastapi-zh/models/v1_1_zh
mkdir -p /mnt/user/appdata/kokoro-fastapi-zh/voices/v1_1_zh

# 2. 将目录所有者设为 UID 1000（容器内 appuser） / Set owner to UID 1000 (container appuser)
chown -R 1000:1000 /mnt/user/appdata/kokoro-fastapi-zh

# 3. 设置读写权限 / Set read-write permissions
chmod -R 775 /mnt/user/appdata/kokoro-fastapi-zh
```

> **提示**：Unraid 默认以 root 登录终端，以上命令可直接执行，仅首次部署前需要操作一次。

English:

The container process runs as non-root user `appuser` (UID 1000). If mounted host directories are not owned by UID 1000, writes will fail with errors like:

**Fix (run once in Unraid terminal before first deployment):**

```bash
# 1. Create mount directories
mkdir -p /mnt/user/appdata/kokoro-fastapi-zh/models/v1_1_zh
mkdir -p /mnt/user/appdata/kokoro-fastapi-zh/voices/v1_1_zh

# 2. Set owner to UID 1000 (container appuser)
chown -R 1000:1000 /mnt/user/appdata/kokoro-fastapi-zh

# 3. Set read-write permissions
chmod -R 775 /mnt/user/appdata/kokoro-fastapi-zh
```

> **Tip**: Unraid terminal is root by default, so these commands can be executed directly and only once before first deployment.

## 镜像标签策略 / Image Tagging Strategy

- 推荐生产环境固定版本标签（例如：`v0.2.4-zh`），避免 `latest` 漂移。
- 建议约定：
- GPU：`ghcr.io/hsiang-han/kokoro-fastapi-zh-gpu:v0.2.4-zh`
- CPU：`ghcr.io/hsiang-han/kokoro-fastapi-zh-cpu:v0.2.4-zh`
- `latest` 可用于快速试用，不建议用于可复现部署。

English:

- In production, pin fixed tags (for example `v0.2.4-zh`) to avoid `latest` drift.
- Suggested convention:
- GPU: `ghcr.io/hsiang-han/kokoro-fastapi-zh-gpu:v0.2.4-zh`
- CPU: `ghcr.io/hsiang-han/kokoro-fastapi-zh-cpu:v0.2.4-zh`
- `latest` is fine for quick trials, but not recommended for reproducible deployment.

## 最小环境变量 / Minimal Environment Variables

| 变量 / Variable | 示例值 / Example | 说明 / Description |
|---|---|---|
| `REPO_ID` | `hexgrad/Kokoro-82M-v1.1-zh` | 模型仓库 / Model repository |
| `DEFAULT_VOICE` | `zf_094` | 默认语音 / Default voice |
| `KOKORO_V1_FILE` | `v1_1_zh/kokoro-v1_1-zh.pth` | 模型权重相对路径 / Relative model weight path |
| `VOICES_DIR` | `/app/api/src/voices/v1_1_zh` | 容器内语音目录 / Voice directory in container |
| `DEFAULT_VOICE_CODE` | `z`（可选） | 强制语言代码（中英混读建议） / Force language code (recommended for mixed Chinese-English reading) |
| `API_LOG_LEVEL` | `DEBUG` | 日志等级 / Log level |

### 手动准备模型/语音文件时的目录约定（重要） / Directory conventions when manually preparing model/voice files (important)

当你关闭自动下载（例如 `DOWNLOAD_MODEL=false`）并手动放置文件时，请按下面结构准备，避免路径不匹配：

- 容器内模型根目录：`/app/api/src/models`
- 容器内语音根目录：`/app/api/src/voices`
- 默认模型文件应位于：`/app/api/src/models/v1_1_zh/kokoro-v1_1-zh.pth`
- 默认语音目录应位于：`/app/api/src/voices/v1_1_zh/*.pt`

若使用当前 compose 映射：

- `./models:/app/api/src/models`
- `./voices:/app/api/src/voices`

则宿主机目录应为：

- `./models/v1_1_zh/kokoro-v1_1-zh.pth`
- `./voices/v1_1_zh/*.pt`

English:

If you disable auto-download (for example `DOWNLOAD_MODEL=false`) and provide files manually, use the structure below to avoid path mismatch:

- Model root in container: `/app/api/src/models`
- Voice root in container: `/app/api/src/voices`
- Default model file should be at: `/app/api/src/models/v1_1_zh/kokoro-v1_1-zh.pth`
- Default voice directory should be at: `/app/api/src/voices/v1_1_zh/*.pt`

If using current compose mappings:

- `./models:/app/api/src/models`
- `./voices:/app/api/src/voices`

Then host paths should be:

- `./models/v1_1_zh/kokoro-v1_1-zh.pth`
- `./voices/v1_1_zh/*.pt`

## 切换到 v1.0（英文模型） / Switch to v1.0

本分支默认是 v1.1-zh。如果你要切换到 v1.0（英文模型），请使用：

- GPU v1.0 配置文件：`docker/gpu/stack.v1_0.yml`

### 1) 准备目录

在 `docker/gpu/` 下准备：

- `models/v1_0/`
- `voices/v1_0/`

### 2) 下载模型与语音（来源）

- 模型仓库（v1.0）：https://huggingface.co/hexgrad/Kokoro-82M
- 语音目录： https://huggingface.co/hexgrad/Kokoro-82M/tree/main/voices

推荐使用 Hugging Face CLI 下载（在 `docker/gpu` 目录执行）：

```bash
huggingface-cli download hexgrad/Kokoro-82M kokoro-v1_0.pth config.json --local-dir ./models/v1_0
huggingface-cli download hexgrad/Kokoro-82M --include "voices/*.pt" --local-dir ./_tmp_kokoro_v1_0
cp ./_tmp_kokoro_v1_0/voices/*.pt ./voices/v1_0/
```

> Windows PowerShell 可将最后一行替换为：
>
> `Copy-Item .\_tmp_kokoro_v1_0\voices\*.pt .\voices\v1_0\`

### 3) 启动 v1.0 配置

在 `docker/gpu/` 目录执行：

```bash
docker compose -f stack.v1_0.yml up --build
```

该配置已预设：

- `REPO_ID=hexgrad/Kokoro-82M`
- `KOKORO_V1_FILE=v1_0/kokoro-v1_0.pth`
- `VOICES_DIR=/app/api/src/voices/v1_0`

English:

This fork defaults to v1.1-zh. If you want v1.0 (English model), use:

- GPU v1.0 compose file: `docker/gpu/stack.v1_0.yml`

1) Prepare directories under `docker/gpu/`:

- `models/v1_0/`
- `voices/v1_0/`

```bash
mkdir -p ./models/v1_0
mkdir -p ./voices/v1_0
```

2) Download model and voices:

- Model repository (v1.0): https://huggingface.co/hexgrad/Kokoro-82M
- Voice directory: https://huggingface.co/hexgrad/Kokoro-82M/tree/main/voices

Recommended: use Hugging Face CLI (run in `docker/gpu`):

```bash
huggingface-cli download hexgrad/Kokoro-82M kokoro-v1_0.pth config.json --local-dir ./models/v1_0
huggingface-cli download hexgrad/Kokoro-82M --include "voices/*.pt" --local-dir ./_tmp_kokoro_v1_0
cp ./_tmp_kokoro_v1_0/voices/*.pt ./voices/v1_0/
```

> In Windows PowerShell, replace the last line with:
>
> `Copy-Item .\_tmp_kokoro_v1_0\voices\*.pt .\voices\v1_0\`

3) Start v1.0 stack:

```bash
docker compose -f stack.v1_0.yml up --build
```

This stack is preconfigured with:

- `REPO_ID=hexgrad/Kokoro-82M`
- `KOKORO_V1_FILE=v1_0/kokoro-v1_0.pth`
- `VOICES_DIR=/app/api/src/voices/v1_0`

<p align="center">
  <img src="githubbanner.png" alt="Kokoro TTS Banner">
</p>

# <sub><sub>_`FastKoko`_ </sub></sub>
[![Tests](https://img.shields.io/badge/tests-69-darkgreen)]()
[![Coverage](https://img.shields.io/badge/coverage-54%25-tan)]()
[![Try on Spaces](https://img.shields.io/badge/%F0%9F%A4%97%20Try%20on-Spaces-blue)](https://huggingface.co/spaces/Remsky/Kokoro-TTS-Zero)

[![Kokoro](https://img.shields.io/badge/kokoro-0.9.2-BB5420)](https://github.com/hexgrad/kokoro)
[![Misaki](https://img.shields.io/badge/misaki-0.9.3-B8860B)](https://github.com/hexgrad/misaki)

[![Tested at Model Commit](https://img.shields.io/badge/last--tested--model--commit-1.0::9901c2b-blue)](https://huggingface.co/hexgrad/Kokoro-82M/commit/9901c2b79161b6e898b7ea857ae5298f47b8b0d6)

Dockerized FastAPI wrapper for [Kokoro-82M](https://huggingface.co/hexgrad/Kokoro-82M) text-to-speech model
- Multi-language support (English, Japanese, Chinese, _Vietnamese soon_)
- OpenAI-compatible Speech endpoint, NVIDIA GPU accelerated or CPU inference with PyTorch 
- ONNX support coming soon, see v0.1.5 and earlier for legacy ONNX support in the interim
- Debug endpoints for monitoring system stats, integrated web UI on localhost:8880/web
- Phoneme-based audio generation, phoneme generation
- Per-word timestamped caption generation
- Voice mixing with weighted combinations

### Integration Guides
 [![Helm Chart](https://img.shields.io/badge/Helm%20Chart-black?style=flat&logo=helm&logoColor=white)](https://github.com/remsky/Kokoro-FastAPI/wiki/Setup-Kubernetes) [![DigitalOcean](https://img.shields.io/badge/DigitalOcean-black?style=flat&logo=digitalocean&logoColor=white)](https://github.com/remsky/Kokoro-FastAPI/wiki/Integrations-DigitalOcean) [![SillyTavern](https://img.shields.io/badge/SillyTavern-black?style=flat&color=red)](https://github.com/remsky/Kokoro-FastAPI/wiki/Integrations-SillyTavern)
[![OpenWebUI](https://img.shields.io/badge/OpenWebUI-black?style=flat&color=white)](https://github.com/remsky/Kokoro-FastAPI/wiki/Integrations-OpenWebUi)

- Docker 免源码部署 + Unraid 上架说明: `docs/deployment/unraid-and-prebuilt-images.md`
- Unraid Stack 示例: `docker/unraid/stack.gpu.image.yml` / `docker/unraid/stack.cpu.image.yml`
- Unraid CA 模板: `unraid/templates/kokoro-fastapi-zh.xml`

## Get Started

<details>
<summary>Quickest Start (docker run)</summary>


Pre built images are available to run, with arm/multi-arch support, and baked in models
Refer to the core/config.py file for a full list of variables which can be managed via the environment

```bash
# the `latest` tag can be used, though it may have some unexpected bonus features which impact stability.
 Named versions should be pinned for your regular usage.
 Feedback/testing is always welcome

docker run -p 8880:8880 ghcr.io/hsiang-han/kokoro-fastapi-zh-cpu:latest # CPU, or:
docker run --gpus all -p 8880:8880 ghcr.io/hsiang-han/kokoro-fastapi-zh-gpu:latest  # NVIDIA GPU
```


</details>

<details>

<summary>Quick Start (docker compose) </summary>

For detailed source-based deployment steps, see the Chinese section above:
`源码部署 / Source Deployment`.

Quick command reference:

```bash
git clone https://github.com/hsiang-han/Kokoro-FastAPI-zh.git
cd Kokoro-FastAPI-zh
cd docker/gpu   # or: cd docker/cpu
docker compose up --build
```

Apple Silicon (M1/M2/M3) users should use `docker/cpu`.

Original detailed steps (kept for compatibility):

1. Install prerequisites, and start the service using Docker Compose (Full setup including UI):
   - Install [Docker](https://www.docker.com/products/docker-desktop/)
   - Clone the repository:
    ```bash
    git clone https://github.com/remsky/Kokoro-FastAPI.git
    cd Kokoro-FastAPI

    cd docker/gpu  # For GPU support
    # or cd docker/cpu  # For CPU support
    docker compose up --build

    # *Note for Apple Silicon (M1/M2) users:
    # The current GPU build relies on CUDA, which is not supported on Apple Silicon.
    # If you are on an M1/M2/M3 Mac, please use the `docker/cpu` setup.
    # MPS (Apple's GPU acceleration) support is planned but not yet available.

    # Models will auto-download, but if needed you can manually download:
    python docker/scripts/download_model.py --output api/src/models/v1_1_zh --voices-output api/src/voices/v1_1_zh

    # Or run directly via UV:
    ./start-gpu.sh  # For GPU support
    ./start-cpu.sh  # For CPU support
    ```
</details>
<details>
<summary>Direct Run (via uv) </summary>

1. Install prerequisites ():
   - Install [astral-uv](https://docs.astral.sh/uv/)
   - Install [espeak-ng](https://github.com/espeak-ng/espeak-ng) in your system if you want it available as a fallback for unknown words/sounds. The upstream libraries may attempt to handle this, but results have varied.
   - Clone the repository:
        ```bash
        git clone https://github.com/remsky/Kokoro-FastAPI.git
        cd Kokoro-FastAPI
        ```
        
        Run the [model download script](https://github.com/remsky/Kokoro-FastAPI/blob/master/docker/scripts/download_model.py) if you haven't already
     
        Start directly via UV (with hot-reload)
        
        Linux and macOS
        ```bash
        ./start-cpu.sh OR
        ./start-gpu.sh 
        ```

        Windows
        ```powershell
        .\start-cpu.ps1 OR
        .\start-gpu.ps1 
        ```

</details>

<details open>
<summary> Up and Running? </summary>


Run locally as an OpenAI-Compatible Speech Endpoint
    
```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8880/v1", api_key="not-needed"
)

with client.audio.speech.with_streaming_response.create(
    model="kokoro",
    voice="af_sky+af_bella", #single or multiple voicepack combo
    input="Hello world!"
  ) as response:
      response.stream_to_file("output.mp3")
```
  
- The API will be available at http://localhost:8880
- API Documentation: http://localhost:8880/docs

- Web Interface: http://localhost:8880/web

<div align="center" style="display: flex; justify-content: center; gap: 10px;">
  <img src="assets/docs-screenshot.png" width="42%" alt="API Documentation" style="border: 2px solid #333; padding: 10px;">
  <img src="assets/webui-screenshot.png" width="42%" alt="Web UI Screenshot" style="border: 2px solid #333; padding: 10px;">
</div>

</details>

## Features 

<details>
<summary>OpenAI-Compatible Speech Endpoint</summary>

```python
# Using OpenAI's Python library
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8880/v1", api_key="not-needed")
response = client.audio.speech.create(
    model="kokoro",  
    voice="af_bella+af_sky", # see /api/src/core/openai_mappings.json to customize
    input="Hello world!",
    response_format="mp3"
)

response.stream_to_file("output.mp3")
```
Or Via Requests:
```python
import requests


response = requests.get("http://localhost:8880/v1/audio/voices")
voices = response.json()["voices"]

# Generate audio
response = requests.post(
    "http://localhost:8880/v1/audio/speech",
    json={
        "model": "kokoro",  
        "input": "Hello world!",
        "voice": "af_bella",
        "response_format": "mp3",  # Supported: mp3, wav, opus, flac
        "speed": 1.0
    }
)

# Save audio
with open("output.mp3", "wb") as f:
    f.write(response.content)
```

Quick tests (run from another terminal):
```bash
python examples/assorted_checks/test_openai/test_openai_tts.py # Test OpenAI Compatibility
python examples/assorted_checks/test_voices/test_all_voices.py # Test all available voices
```
</details>

<details>
<summary>Voice Combination</summary>

- Weighted voice combinations using ratios (e.g., "af_bella(2)+af_heart(1)" for 67%/33% mix)
- Ratios are automatically normalized to sum to 100%
- Available through any endpoint by adding weights in parentheses
- Saves generated voicepacks for future use

Combine voices and generate audio:
```python
import requests
response = requests.get("http://localhost:8880/v1/audio/voices")
voices = response.json()["voices"]

# Example 1: Simple voice combination (50%/50% mix)
response = requests.post(
    "http://localhost:8880/v1/audio/speech",
    json={
        "input": "Hello world!",
        "voice": "af_bella+af_sky",  # Equal weights
        "response_format": "mp3"
    }
)

# Example 2: Weighted voice combination (67%/33% mix)
response = requests.post(
    "http://localhost:8880/v1/audio/speech",
    json={
        "input": "Hello world!",
        "voice": "af_bella(2)+af_sky(1)",  # 2:1 ratio = 67%/33%
        "response_format": "mp3"
    }
)

# Example 3: Download combined voice as .pt file
response = requests.post(
    "http://localhost:8880/v1/audio/voices/combine",
    json="af_bella(2)+af_sky(1)"  # 2:1 ratio = 67%/33%
)

# Save the .pt file
with open("combined_voice.pt", "wb") as f:
    f.write(response.content)

# Use the downloaded voice file
response = requests.post(
    "http://localhost:8880/v1/audio/speech",
    json={
        "input": "Hello world!",
        "voice": "combined_voice",  # Use the saved voice file
        "response_format": "mp3"
    }
)

```
<p align="center">
  <img src="assets/voice_analysis.png" width="80%" alt="Voice Analysis Comparison" style="border: 2px solid #333; padding: 10px;">
</p>
</details>

<details>
<summary>Multiple Output Audio Formats</summary>

- mp3
- wav
- opus 
- flac
- m4a
- pcm

<p align="center">
<img src="assets/format_comparison.png" width="80%" alt="Audio Format Comparison" style="border: 2px solid #333; padding: 10px;">
</p>

</details>

<details>
<summary>Streaming Support</summary>

```python
# OpenAI-compatible streaming
from openai import OpenAI
client = OpenAI(
    base_url="http://localhost:8880/v1", api_key="not-needed")

# Stream to file
with client.audio.speech.with_streaming_response.create(
    model="kokoro",
    voice="af_bella",
    input="Hello world!"
) as response:
    response.stream_to_file("output.mp3")

# Stream to speakers (requires PyAudio)
import pyaudio
player = pyaudio.PyAudio().open(
    format=pyaudio.paInt16, 
    channels=1, 
    rate=24000, 
    output=True
)

with client.audio.speech.with_streaming_response.create(
    model="kokoro",
    voice="af_bella",
    response_format="pcm",
    input="Hello world!"
) as response:
    for chunk in response.iter_bytes(chunk_size=1024):
        player.write(chunk)
```

Or via requests:
```python
import requests

response = requests.post(
    "http://localhost:8880/v1/audio/speech",
    json={
        "input": "Hello world!",
        "voice": "af_bella",
        "response_format": "pcm"
    },
    stream=True
)

for chunk in response.iter_content(chunk_size=1024):
    if chunk:
        # Process streaming chunks
        pass
```

<p align="center">
  <img src="assets/gpu_first_token_timeline_openai.png" width="45%" alt="GPU First Token Timeline" style="border: 2px solid #333; padding: 10px; margin-right: 1%;">
  <img src="assets/cpu_first_token_timeline_stream_openai.png" width="45%" alt="CPU First Token Timeline" style="border: 2px solid #333; padding: 10px;">
</p>

Key Streaming Metrics:
- First token latency @ chunksize
    - ~300ms  (GPU) @ 400 
    - ~3500ms (CPU) @ 200 (older i7)
    - ~<1s    (CPU) @ 200 (M3 Pro)
- Adjustable chunking settings for real-time playback 

*Note: Artifacts in intonation can increase with smaller chunks*
</details>

## Processing Details
<details>
<summary>Performance Benchmarks</summary>

Benchmarking was performed on generation via the local API using text lengths up to feature-length books (~1.5 hours output), measuring processing time and realtime factor. Tests were run on: 
- Windows 11 Home w/ WSL2 
- NVIDIA 4060Ti 16gb GPU @ CUDA 12.1
- 11th Gen i7-11700 @ 2.5GHz
- 64gb RAM
- WAV native output
- H.G. Wells - The Time Machine (full text)

<p align="center">
  <img src="assets/gpu_processing_time.png" width="45%" alt="Processing Time" style="border: 2px solid #333; padding: 10px; margin-right: 1%;">
  <img src="assets/gpu_realtime_factor.png" width="45%" alt="Realtime Factor" style="border: 2px solid #333; padding: 10px;">
</p>

Key Performance Metrics:
- Realtime Speed: Ranges between 35x-100x (generation time to output audio length)
- Average Processing Rate: 137.67 tokens/second (cl100k_base)
</details>
<details>
<summary>GPU Vs. CPU</summary>

```bash
# GPU: Requires NVIDIA GPU with CUDA 12.8 support (~35x-100x realtime speed)
cd docker/gpu
docker compose up --build

# CPU: PyTorch CPU inference
cd docker/cpu
docker compose up --build

```
*Note: Overall speed may have reduced somewhat with the structural changes to accommodate streaming. Looking into it* 
</details>

<details>
<summary>Natural Boundary Detection</summary>

- Automatically splits and stitches at sentence boundaries 
- Helps to reduce artifacts and allow long form processing as the base model is only currently configured for approximately 30s output

The model is capable of processing up to a 510 phonemized token chunk at a time, however, this can often lead to 'rushed' speech or other artifacts. An additional layer of chunking is applied in the server, that creates flexible chunks with a `TARGET_MIN_TOKENS` , `TARGET_MAX_TOKENS`, and `ABSOLUTE_MAX_TOKENS` which are configurable via environment variables, and set to 175, 250, 450 by default

</details>

<details>
<summary>Timestamped Captions & Phonemes</summary>

Generate audio with word-level timestamps without streaming:
```python
import requests
import base64
import json

response = requests.post(
    "http://localhost:8880/dev/captioned_speech",
    json={
        "model": "kokoro",
        "input": "Hello world!",
        "voice": "af_bella",
        "speed": 1.0,
        "response_format": "mp3",
        "stream": False,
    },
    stream=False
)

with open("output.mp3","wb") as f:

    audio_json=json.loads(response.content)
    
    # Decode base 64 stream to bytes
    chunk_audio=base64.b64decode(audio_json["audio"].encode("utf-8"))
    
    # Process streaming chunks
    f.write(chunk_audio)
    
    # Print word level timestamps
    print(audio_json["timestamps"])
```

Generate audio with word-level timestamps with streaming:
```python
import requests
import base64
import json

response = requests.post(
    "http://localhost:8880/dev/captioned_speech",
    json={
        "model": "kokoro",
        "input": "Hello world!",
        "voice": "af_bella",
        "speed": 1.0,
        "response_format": "mp3",
        "stream": True,
    },
    stream=True
)

f=open("output.mp3","wb")
for chunk in response.iter_lines(decode_unicode=True):
    if chunk:
        chunk_json=json.loads(chunk)
        
        # Decode base 64 stream to bytes
        chunk_audio=base64.b64decode(chunk_json["audio"].encode("utf-8"))
        
        # Process streaming chunks
        f.write(chunk_audio)
        
        # Print word level timestamps
        print(chunk_json["timestamps"])
```
</details>

<details>
<summary>Phoneme & Token Routes</summary>

Convert text to phonemes and/or generate audio directly from phonemes:
```python
import requests

def get_phonemes(text: str, language: str = "a"):
    """Get phonemes and tokens for input text"""
    response = requests.post(
        "http://localhost:8880/dev/phonemize",
        json={"text": text, "language": language}  # "a" for American English
    )
    response.raise_for_status()
    result = response.json()
    return result["phonemes"], result["tokens"]

def generate_audio_from_phonemes(phonemes: str, voice: str = "af_bella"):
    """Generate audio from phonemes"""
    response = requests.post(
        "http://localhost:8880/dev/generate_from_phonemes",
        json={"phonemes": phonemes, "voice": voice},
        headers={"Accept": "audio/wav"}
    )
    if response.status_code != 200:
        print(f"Error: {response.text}")
        return None
    return response.content

# Example usage
text = "Hello world!"
try:
    # Convert text to phonemes
    phonemes, tokens = get_phonemes(text)
    print(f"Phonemes: {phonemes}")  # e.g. ðɪs ɪz ˈoʊnli ɐ tˈɛst
    print(f"Tokens: {tokens}")      # Token IDs including start/end tokens

    # Generate and save audio
    if audio_bytes := generate_audio_from_phonemes(phonemes):
        with open("speech.wav", "wb") as f:
            f.write(audio_bytes)
        print(f"Generated {len(audio_bytes)} bytes of audio")
except Exception as e:
    print(f"Error: {e}")
```

See `examples/phoneme_examples/generate_phonemes.py` for a sample script.
</details>

<details>
<summary>Debug Endpoints</summary>

Monitor system state and resource usage with these endpoints:

- `/debug/threads` - Get thread information and stack traces
- `/debug/storage` - Monitor temp file and output directory usage
- `/debug/system` - Get system information (CPU, memory, GPU)
- `/debug/session_pools` - View ONNX session and CUDA stream status

Useful for debugging resource exhaustion or performance issues.
</details>

<details>
<summary>Logging</summary>

Global API [loguru logging level](https://loguru.readthedocs.io/en/stable/api/logger.html#levels) can be set using the `API_LOG_LEVEL` environment variable. Defaults to `DEBUG`.

**Docker**

Modify the appropriate compose `yml` or append to command line.
```bash
docker run --env 'API_LOG_LEVEL=WARNING' ...
```

**Direct via UV**

Linux and macOS
```bash
export API_LOG_LEVEL=WARNING
./start-cpu.sh OR
./start-gpu.sh
```

Windows
```powershell
$env:API_LOG_LEVEL = 'WARNING'
.\start-cpu.ps1 OR
.\start-gpu.ps1
```
</details>

## Known Issues & Troubleshooting

<details>
<summary>Missing words & Missing some timestamps</summary>

The api will automaticly do text normalization on input text which may incorrectly remove or change some phrases. This can be disabled by adding `"normalization_options":{"normalize": false}` to your request json:
```python
import requests

response = requests.post(
    "http://localhost:8880/v1/audio/speech",
    json={
        "input": "Hello world!",
        "voice": "af_heart",
        "response_format": "pcm",
        "normalization_options":
        {
            "normalize": False
        }
    },
    stream=True
)

for chunk in response.iter_content(chunk_size=1024):
    if chunk:
        # Process streaming chunks
        pass
```
  
</details>

<details>
<summary>Versioning & Development</summary>

**Branching Strategy:**
*   **`release` branch:** Contains the latest stable build, recommended for production use. Docker images tagged with specific versions (e.g., `v0.3.0`) are built from this branch.
*   **`master` branch:** Used for active development. It may contain experimental features, ongoing changes, or fixes not yet in a stable release. Use this branch if you want the absolute latest code, but be aware it might be less stable. The `latest` Docker tag often points to builds from this branch.

Note: This is a *development* focused project at its core. 

If you run into trouble, you may have to roll back a version on the release tags if something comes up, or build up from source and/or troubleshoot + submit a PR.

Free and open source is a community effort, and there's only really so many hours in a day. If you'd like to support the work, feel free to open a PR, buy me a coffee, or report any bugs/features/etc you find during use.

  <a href="https://www.buymeacoffee.com/remsky" target="_blank">
    <img 
      src="https://cdn.buymeacoffee.com/buttons/v2/default-violet.png" 
      alt="Buy Me A Coffee" 
      style="height: 30px !important;width: 110px !important;"
    >
  </a>

  
</details>

<details>
<summary>Linux GPU Permissions</summary>

Some Linux users may encounter GPU permission issues when running as non-root. 
Can't guarantee anything, but here are some common solutions, consider your security requirements carefully

### Option 1: Container Groups (Likely the best option)
```yaml
services:
  kokoro-tts:
    # ... existing config ...
    group_add:
      - "video"
      - "render"
```

### Option 2: Host System Groups
```yaml
services:
  kokoro-tts:
    # ... existing config ...
    user: "${UID}:${GID}"
    group_add:
      - "video"
```
Note: May require adding host user to groups: `sudo usermod -aG docker,video $USER` and system restart.

### Option 3: Device Permissions (Use with caution)
```yaml
services:
  kokoro-tts:
    # ... existing config ...
    devices:
      - /dev/nvidia0:/dev/nvidia0
      - /dev/nvidiactl:/dev/nvidiactl
      - /dev/nvidia-uvm:/dev/nvidia-uvm
```
⚠️ Warning: Reduces system security. Use only in development environments.

Prerequisites: NVIDIA GPU, drivers, and container toolkit must be properly configured.

Visit [NVIDIA Container Toolkit installation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) for more detailed information

</details>

## Model and License

<details open>
<summary>Model</summary>

This API uses the [Kokoro-82M](https://huggingface.co/hexgrad/Kokoro-82M) model from HuggingFace. 

Visit the model page for more details about training, architecture, and capabilities. I have no affiliation with any of their work, and produced this wrapper for ease of use and personal projects.
</details>
<details>
<summary>License</summary>
This project is licensed under the Apache License 2.0 - see below for details:

- The Kokoro model weights are licensed under Apache 2.0 (see [model page](https://huggingface.co/hexgrad/Kokoro-82M))
- The FastAPI wrapper code in this repository is licensed under Apache 2.0 to match
- The inference code adapted from StyleTTS2 is MIT licensed

The full Apache 2.0 license text can be found at: https://www.apache.org/licenses/LICENSE-2.0
</details>

</details open>

## Contributor Stats
![Alt](https://repobeats.axiom.co/api/embed/f9694366bf96febc749d592316ff0a275fe77219.svg "Repobeats analytics image")
</details>

<a href="https://github.com/remsky/Kokoro-FastAPI/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=remsky/Kokoro-FastAPI" />
</a>

Made with [contrib.rocks](https://contrib.rocks).
# Docker 分享与 Unraid 上架指南

本文对应两个目标：

1. 让其他人不需要下载源码和本地构建，只通过 Docker Compose/Stack 运行。
2. 让 Unraid 用户可以通过 Community Applications（CA）一键安装。

## 1) 免源码部署（预构建镜像）

### 你需要先做一次发布

仓库已新增 workflow：`.github/workflows/publish-images.yml`。

- 在 GitHub 上发布 Release（建议 tag 例如 `v1.1.0`），会自动构建并推送：
  - `ghcr.io/<你的用户名>/<仓库名>-cpu:<tag>`
  - `ghcr.io/<你的用户名>/<仓库名>-gpu:<tag>`
  - 以及对应 `:latest`
- 首次发布后，请在 GitHub Packages 中把镜像可见性改为 Public（否则别人 pull 会 401）。

### 给用户的最简部署方式

直接提供以下 compose 文件即可：

- GPU: `docker/unraid/stack.gpu.image.yml`
- CPU: `docker/unraid/stack.cpu.image.yml`

用户只需：

1. 复制 compose 文件到自己的机器。
2. 修改 `KOKORO_IMAGE` 为你发布的镜像名（如果与你当前默认值不同）。
3. 执行 `docker compose up -d`。

可选 `.env` 示例：

```env
KOKORO_IMAGE=ghcr.io/hsiang-han/kokoro-fastapi-v1.1_zh-gpu:v1.1.0
KOKORO_PORT=8880
KOKORO_MODELS_PATH=/mnt/user/appdata/kokoro-fastapi-v1_1_zh/models
KOKORO_VOICES_PATH=/mnt/user/appdata/kokoro-fastapi-v1_1_zh/voices
KOKORO_OUTPUT_PATH=/mnt/user/appdata/kokoro-fastapi-v1_1_zh/output
DEFAULT_VOICE=zf_094
DEFAULT_VOICE_CODE=z
```

说明：

- 模型文件会落在 `KOKORO_MODELS_PATH/v1_1_zh/`。
- 语音文件会自动下载到 `KOKORO_VOICES_PATH/v1_1_zh/`（子目录会自动创建）。

## 2) Unraid App 市场上架

### A. 准备 Unraid 模板

仓库已新增模板：`unraid/templates/kokoro-fastapi-gpu.xml`。

请确认以下字段指向你的仓库：

- `Repository`
- `Support`
- `Project`
- `TemplateURL`
- `Icon`

### B. 让用户一键安装（两种方式）

#### 方式 1（最快）：用户添加你的模板仓库

1. Unraid 安装 Community Applications 插件。
2. 在 CA 设置里添加你的模板仓库或模板 URL。
3. 用户在 Apps 搜索 `Kokoro-FastAPI-v1.1-zh`，点击 Install。

#### 方式 2（覆盖面更大）：提交到公共模板仓库

把 `unraid/templates/kokoro-fastapi-gpu.xml` 提交到社区维护的模板仓库（或按其提交流程发 PR）。

说明：不同时间点社区的收录仓库和流程可能变化，建议先查看当前 CA 社区文档/论坛置顶规则后再提交。

## 3) 推荐的发布节奏

1. 更新 `VERSION`。
2. 创建 GitHub Release（触发镜像发布）。
3. 验证镜像可拉取：

```bash
docker pull ghcr.io/<你的用户名>/<仓库名>-gpu:<tag>
```

4. 如模板中使用了固定 tag，同步更新 `Repository`。
5. 提交模板更新（你的模板仓库 / 公共模板仓库）。

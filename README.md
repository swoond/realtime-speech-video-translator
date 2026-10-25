# realtime-speech-video-translator
Real-time speech translation subtitles that run entirely on your machine. Transcribe mic audio or video files with Whisper, translate with a bundled offline model, and export bilingual (EN/中文) SRT — no API key, no cloud, no GPU needed. Includes a browser UI, CLI, and optional other models' API support.\

#  Realtime Speech Translator

**对着麦克风说话，或丢进一个视频文件 —— 本地识别语音、翻译成中文，实时滚动双语字幕，结束自动存成 `.srt`。**

![Python](https://img.shields.io/badge/python-3.11%20%7C%203.12-blue)
![Offline](https://img.shields.io/badge/runs-100%25%20offline-success)
![API Key](https://img.shields.io/badge/API%20key-not%20required-brightgreen)
![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20Linux%20%7C%20macOS-lightgrey)

一个跑在自己电脑上的**语音翻译字幕工具**。识别用 [Whisper](https://github.com/openai/whisper)（本地推理），翻译默认用本地 [argos](https://github.com/argosopentech/argos-translate) 语言包 —— **不联网、不花钱、不需要 API Key** 就能产出中英双语字幕。也可以把识别或翻译换成你自己的任意 OpenAI 兼容接口。

## ✨ 特性

- **三种用法**：网页版（推荐）/ 命令行实时字幕 / 音视频文件批量转写
- **开箱即用**：识别模型 + 离线翻译语言包本地跑，默认配置下就能出双语
- **可换更强引擎**：识别、翻译都能切到 OpenAI 兼容接口（DeepSeek / 通义 / 智谱 / OpenAI / 本地 Ollama）
- **视频免装 ffmpeg**：`mp4 / mkv / mov / webm` 由 PyAV 直接解码
- **隐私友好**：音频只在本机 `127.0.0.1` 内流动；不选在线接口就不会离开这台电脑
- **零前端构建**：单端口同时提供页面与 WebSocket，不需要 Node / npm
- **结果可带走**：导出双语 SRT / TXT，或一键复制全文

## 🚀 快速开始

### 环境要求

| 项目 | 要求 |
| --- | --- |
| Python | **3.11 或 3.12**（不支持 3.13，RealtimeSTT 限定 `<3.13`） |
| 内存 | 8 GB 起 |
| 磁盘 | 约 4 GB（依赖 + 模型，torch 占大头） |

### 安装

```bash
git clone https://github.com/<your-name>/realtime-speech-translator.git
cd realtime-speech-translator

python -m venv .venv
# Windows: .venv\Scripts\activate
# Linux/macOS: source .venv/bin/activate

pip install -r requirements.txt RealtimeSTT==1.1.2
python src/download_models.py        # 下载识别模型与翻译语言包（约 630 MB）
```

> **国内网络**：模型下载脚本已内置 HuggingFace 镜像与断点续传；pip 建议加 `-i https://mirrors.aliyun.com/pypi/simple/`。

Windows 用户也可以直接运行一键安装脚本，它会自动找解释器并处理依赖：

```powershell
.\scripts\00_setup_env.ps1
```

### 运行

```bash
python src/web_server.py --open      # 启动网页版，自动打开 http://127.0.0.1:8765
```

Windows 下也可以**直接双击 `启动网页版.bat`**：自动找环境、端口被占用会自动往后顺延。

## 🖥️ 三种用法

| 场景 | 怎么做 |
| --- | --- |
| **实时字幕**（听网课、开会、面对面交流） | 网页版点「开始识别」→ 说话 → 原文实时出现，整句结束后补上中文 |
| **视频 / 音频转字幕** | 网页版切到「🎬 音视频文件」→ 拖入文件 → 得到双语 SRT |
| **命令行 / 批处理** | `python src/transcribe_file.py --input lecture.mp4` |

实时字幕的命令行版本：`python src/realtime_mic.py --language en --target-lang zh`。

## 📊 性能参考

CPU + `small`（int8）实测：

| 指标 | 数值 |
| --- | --- |
| 转写速度 | 约 **11× 实时**（3 分 23 秒音频 ≈ 18 秒） |
| 识别准确率 | 官方测试音频 **WER 0.00%** |
| 内存占用 | 约 350 MB（含识别模型），再加离线翻译约 480 MB |

GPU 基本不需要：CTranslate2 在 Blackwell（RTX 50 系）上 int8 内核不可用，实测 GPU 反而比 CPU 慢。

## 🔌 接入自己的 API（可选）

默认全本地，无需任何配置。想要更好的翻译质量，在网页版「⚙ API 设置」里填：

| 字段 | 说明 |
| --- | --- |
| 接口地址 | 填到 `/v1`，例如 `https://api.deepseek.com/v1` |
| API Key | 默认只存浏览器本地；点「保存到项目配置」才会写进 `data/api.json`（已在 `.gitignore`） |
| 模型名 | 例如 `deepseek-chat` |

识别同样可以换成云端：走 `POST {base_url}/audio/transcriptions`（OpenAI Whisper API 兼容）。填完可用「测试接口」按钮当场验证连通性。

## ⚙️ 配置

所有参数都在 `config.yaml`，常用几项：

| 参数 | 作用 |
| --- | --- |
| `asr.model` | 识别档位：`tiny` / `base` / `small`（默认，均衡）/ `medium` |
| `asr.device` / `compute_type` | 默认 `cpu` + `int8` |
| `translate.backend` | `argos-direct`（默认，本地离线）/ `none` / `openai` |
| `translate.source_lang` / `target_lang` | 默认 `en` → `zh` |
| `data/glossary.csv` | 术语表：提升专业词识别率并统一译法（性价比最高的优化） |

## 📁 项目结构

```
src/        识别 / 翻译 / 字幕 / 网页服务 等 Python 模块
scripts/    PowerShell 一键脚本（安装、下载模型、转写、实时、网页）
web/        网页前端（原生 HTML/CSS/JS，无需构建）
data/       术语表与评测素材
models/     模型权重（下载后生成）
output/     字幕与转写结果
upstream/   上游开源库源码快照（参考与测试素材）
```

## 🧩 技术栈

[RealtimeSTT](https://github.com/KoljaB/RealtimeSTT) · [faster-whisper](https://github.com/SYSTRAN/faster-whisper) (CTranslate2) · [Argos Translate](https://github.com/argosopentech/argos-translate) · [PyAV](https://github.com/PyAV-Org/PyAV) · websockets

## ❓ 常见问题

**浏览器提示没有麦克风权限？**
必须用 `http://127.0.0.1:端口` 或 `http://localhost:端口` 打开，用局域网 IP 会被浏览器的安全上下文策略拒绝。

**模型下载失败？**
脚本已内置 hf-mirror 镜像与断点续传；也可以手动把模型放进 `models/faster-whisper/`。网页上的「模型」下拉只会列出本地已有的档位。

**翻译质量不满意？**
默认用的是约 80 MB 的离线小模型，难免出错（例如专有名词）。两个办法：切到「自定义 API」用大模型，或把错词写进 `data/glossary.csv` 强制纠正。

**能用麦克风以外的声音吗？**
可以。用系统内录（如 Windows 的立体声混音）在系统设置里选为默认输入设备即可。

## 🙏 致谢

本项目基于以下优秀的开源工作，均为 MIT 协议：

- [RealtimeSTT](https://github.com/KoljaB/RealtimeSTT) — 实时语音识别引擎
- [faster-whisper](https://github.com/SYSTRAN/faster-whisper) — Whisper 的 CTranslate2 加速实现
- [Argos Translate](https://github.com/argosopentech/argos-translate) — 离线翻译引擎
- [Whisper](https://github.com/openai/whisper) — 语音识别模型

## 📄 License

MIT，沿用上述上游项目的许可协议。

## 📚 更多文档

- [ENVIRONMENT.md](ENVIRONMENT.md) — 环境清单、版本约束与已知问题
- [ROADMAP.md](ROADMAP.md) — 优化路线与评测方法
- [EXPERIMENTS.md](EXPERIMENTS.md) — 实验记录与实测数据

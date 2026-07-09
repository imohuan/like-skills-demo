---
name: whisper-cpp
description: "使用 whisper.cpp 进行本地离线语音转文字。纯 C/C++ 实现，零 Python 依赖，单个 exe 即可运行。适用于需要将音频文件转录为文本的场景。当用户提到 语音识别、语音转文字、音频转录、whisper、whisper.cpp、stt、speech to text、字幕提取 等关键词时触发此技能。"
agent_created: true
---

# Whisper.cpp 语音转文字

使用 whisper.cpp 在本地离线进行语音转文字，无需 Python、无需网络。

## 前置条件

### 1. 下载 whisper.cpp 可执行文件

从 GitHub Releases 下载预编译的 Windows exe：

```
https://github.com/ggml-org/whisper.cpp/releases/latest
```

推荐选择 `whisper-bin-x64.zip`（标准版，体积最小，无额外依赖）。

如果有 NVIDIA 显卡，可选 `whisper-cublas-x.x.x-bin-x64.zip` 获得 GPU 加速。

下载后解压到任意目录，得到 `whisper-cli.exe`。

### 2. 下载模型文件

模型文件从 HuggingFace 下载，放到与 `whisper-cli.exe` 同目录：

| 模型 | 大小 | 速度 | 精度 | 推荐场景 |
|------|------|------|------|----------|
| ggml-tiny.bin | 75 MB | 最快 | 一般 | 快速测试 |
| ggml-base.bin | 142 MB | 快 | 较好 | 英文为主 |
| ggml-small.bin | 466 MB | 适中 | 好 | **中文推荐** |
| ggml-medium.bin | 1.5 GB | 较慢 | 很好 | 高精度需求 |
| ggml-large-v3.bin | 2.9 GB | 慢 | 最好 | 最高精度 |

下载示例（以 small 模型为例）：

```
curl -L -o ggml-small.bin "https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-small.bin"
```

其他模型替换 URL 中的 `small` 为目标名称即可。

## 使用方法

### 基本转录

```powershell
.\whisper-cli.exe -m <模型文件> -f <音频文件> -l <语言> -otxt
```

### 参数说明

| 参数 | 说明 | 示例 |
|------|------|------|
| `-m` | 模型文件路径 | `-m ggml-small.bin` |
| `-f` | 输入音频文件路径 | `-f "F:\audio\recording.mp3"` |
| `-l` | 语言代码（auto=自动检测） | `-l zh` / `-l auto` |
| `-otxt` | 输出为 txt 文件 | |
| `-osrt` | 输出为 srt 字幕文件 | |
| `-ovtt` | 输出为 vtt 字幕文件 | |
| `-oj` | 输出为 JSON（含时间戳） | |
| `-ocsv` | 输出为 CSV | |
| `-of` | 输出文件路径前缀 | `-of output_dir/result` |
| `-t` | 线程数（默认 4） | `-t 8` |
| `-pp` | 是否打印处理进度 | `-pp` |
| `-np` | 不显示进度条 | `-np` |
| `-tr` | 是否翻译为英文 | `-tr` |

### 常用命令示例

**中文语音转文字：**
```powershell
.\whisper-cli.exe -m ggml-small.bin -f "F:\音频\interview.mp3" -l zh -otxt
```

**输出带时间戳的 JSON：**
```powershell
.\whisper-cli.exe -m ggml-small.bin -f "input.m4a" -l zh -oj -of result
```

**输出 SRT 字幕：**
```powershell
.\whisper-cli.exe -m ggml-small.bin -f "video.mp4" -l zh -osrt
```

**自动检测语言并翻译为英文：**
```powershell
.\whisper-cli.exe -m ggml-medium.bin -f "speech.wav" -l auto -tr -otxt
```

**使用 GPU 加速（需 CUDA 版 whisper-cli + NVIDIA 显卡）：**
```powershell
.\whisper-cli.exe -m ggml-medium.bin -f "audio.wav" -l zh -otxt -ng
```

## 支持的音频格式

wav, mp3, m4a, ogg, flac, opus, aac, wma 等常见格式。ffmpeg 可处理的格式均支持。

## 已知安装路径参考

以下路径仅为参考，实际使用时根据环境调整：

- `whisper-cli.exe` 位置：`C:\UserApp\Bin\Release\whisper-cli.exe`
- 模型文件位置：`C:\UserApp\Bin\Release\ggml-small.bin`
- 模型下载缓存：`.workbuddy/binaries/python/envs/default/Lib/site-packages/` (Python 版)
- ffmpeg：系统已安装

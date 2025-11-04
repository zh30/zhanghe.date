---
title: Kokoro-82M 语音生成模型
date: 2025-01-20
---

大家好！今天要跟大家聊聊一个新晋的 TTS 模型——Kokoro！它只有 8200 万参数，但实力不容小觑，在 TTS 模型领域闯出了不小的名堂！

2024 年 12 月 25 日，Kokoro v0.19 版本的权重以 Apache 2.0 许可证发布，精度达到 fp32。截止到 2025 年 1 月 2 日，已经发布了 10 个不同的声音包，并且 v0.19 版本也推出了 .onnx 格式。在发布前的几周里，Kokoro v0.19 在 TTS 模型竞赛中一举夺魁，在单音竞技场中击败了众多对手，用更少的参数和数据取得了更高的 Elo 积分。

一起来看看它的成绩单：

| 模型         | 参数    | 许可证      | 训练数据   |
| ------------ | ------- | ----------- | ---------- |
| Kokoro v0.19 | 8200 万 | Apache      | <100 小时  |
| XTTS v2      | 4.67 亿 | CPML        | >1 万小时  |
| Edge TTS     |         | 专有        |            |
| MetaVoice    | 12 亿   | Apache      | 10 万小时  |
| Parler Mini  | 8.8 亿  | Apache      | 4.5 万小时 |
| Fish Speech  | ~5 亿   | CC-BY-NC-SA | 100 万小时 |

Kokoro 的出色表现暗示着传统 TTS 模型的规模效应可能比我们之前预想的还要陡峭。想要体验 Kokoro 的魅力？快来访问 hf.co/spaces/hexgrad/Kokoro-TTS

# 如何使用 Kokoro？

## 在 Google Colab 上运行

### 1️⃣ 静默安装依赖项

```
!git lfs install
!git clone https://huggingface.co/hexgrad/Kokoro-82M
%cd Kokoro-82M
!apt-get -qq -y install espeak-ng > /dev/null 2>&1
!pip install -q phonemizer torch transformers scipy munch
```

### 2️⃣ 构建模型并加载默认声音包

```python
from models import build_model
import torch
device = 'cuda' if torch.cuda.is_available() else 'cpu'
MODEL = build_model('kokoro-v0_19.pth', device)
VOICE_NAME = ['af', 'af_bella', 'af_sarah', 'am_adam', 'am_michael', 'bf_emma', 'bf_isabella', 'bm_george', 'bm_lewis', 'af_nicole', 'af_sky'][0]  # 默认语音
VOICEPACK = torch.load(f'voices/{VOICE_NAME}.pt', weights_only=True).to(device)
print(f'加载语音：{VOICE_NAME}')
```

### 3️⃣ 生成音频并获取音素

```python
from kokoro import generate
text = "这真是一个无法回答的问题。就像问一个未出生的孩子，他未来会过得好吗？他们甚至还没出生。"
audio, out_ps = generate(MODEL, text, VOICEPACK, lang=VOICE_NAME[0])
```

### 4️⃣ 播放音频并打印音素

```python
from IPython.display import display, Audio
display(Audio(data=audio, rate=24000, autoplay=True))
print(out_ps)
```

## 在[浏览器](https://huggingface.co/onnx-community/Kokoro-82M-ONNX#javascript)上运行

通过 [Transformers.js](https://huggingface.co/docs/transformers.js/index)可以轻松调用

### 首先，使用 NPM 安装 `kokoro-js`库

```bash
npm i kokoro-js
```

### 然后就可以调用生成语音了

```js
import { KokoroTTS } from "kokoro-js"

const model_id = "onnx-community/Kokoro-82M-ONNX"
const tts = await KokoroTTS.from_pretrained(model_id, {
  dtype: "q8", // Options: "fp32", "fp16", "q8", "q4", "q4f16"
})

const text = "Life is like a box of chocolates. You never know what you're gonna get."
const audio = await tts.generate(text, {
  // Use `tts.list_voices()` to list all available voices
  voice: "af_bella",
})
audio.save("audio.wav")
```

更多细节，请参考项目[仓库](https://huggingface.co/hexgrad/Kokoro-82M)。

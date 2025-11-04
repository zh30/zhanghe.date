---
title: 谷歌 Imagen 3 图像生成模型来啦
date: 2025-02-07
---

哇！谷歌 Imagen 3 图像生成模型来啦，Gemini API 抢先体验！

谷歌最新的图像生成神器 Imagen 3 终于要和大家见面啦！这次他们选择通过 Gemini API 率先开放体验，简直是给开发者们送福利！不过先别激动，初期只有付费用户才能尝鲜，免费用户嘛，再等等，好东西总是值得期待的！

Imagen 3 的厉害之处在于，它能生成各种风格的图像，而且质量超高，几乎看不到瑕疵。不管是逼真到让人分不清真假的写实照片，还是充满艺术气息的印象派风景，甚至各种抽象画作和可爱的动漫角色，它都能轻松驾驭。更棒的是，它对提示语的理解能力大大提升，能更好地将你的想法变成高质量的图像。可以说，Imagen 3 在各种图像生成评测中都表现出了顶尖的水平！而且每张图只要 0.03 美元，还能自己调整图像比例和生成数量，简直不要太划算！

为了防止有人滥用这个工具制造虚假信息，Imagen 3 生成的所有图像都会自动添加一个肉眼不可见的数字水印 SynthID，表明这些图像是由 AI 生成的。

## 快速上手：Gemini API 中的 Imagen 3

想体验一下 Imagen 3 的魔力吗？下面这段 Python 代码可以帮助你通过 Gemini API 生成图像：

```python
from google import genai
from google.genai import types
from PIL import Image
from io import BytesIO

client = genai.Client(api_key='GEMINI_API_KEY')

response = client.models.generate_images(
    model='imagen-3.0-generate-002',
    prompt='a portrait of a sheepadoodle wearing cape', # 给出一个提示语：一只穿着披风的绵羊贵宾犬的画像
    config=types.GenerateImagesConfig(
        number_of_images=1,
    )
)
for generated_image in response.generated_images:
  image = Image.open(BytesIO(generated_image.image.image_bytes))
  image.show()
  Image generated
```

## 更多信息，尽在 Gemini API 开发者文档

如果你想了解更多关于提示语的技巧和图像风格，可以去 Gemini API 的开发者文档里看看。更新后的[技术报告](https://storage.googleapis.com/deepmind-media/imagen/imagen_3_tech_report_update_dec2024_v3.pdf#page=26)的附录 D 还包含了关于评分、方法论和性能改进的详细信息。

谷歌表示，这次率先在 Gemini API 中开放生成式媒体模型只是第一步，未来他们会推出更多功能，让开发者们可以将生成式媒体和语言模型结合起来，创造出更多有趣的应用！是不是很期待呢？

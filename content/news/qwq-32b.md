---
title: 哇塞！这个 QwQ-32B 模型简直开挂了！
date: 2025-03-06
---

最近研究表明，通过强化学习 (RL) 来提升模型性能，比单纯的预训练和后训练效果更好哦！像是 DeepSeek R1 就用冷启动数据和多阶段训练，变得超级聪明，擅长深度思考和复杂推理。

咱们这次的研究也关注这个方向，想看看 RL 到底能把大语言模型变得多厉害。所以就有了 QwQ-32B，一个拥有 320 亿参数的模型，但性能竟然能媲美 DeepSeek-R1（它可是有 6710 亿参数，虽然只激活了 370 亿）。这说明，如果基础模型足够强大，拥有丰富的世界知识，再用 RL 加持，效果杠杠的！更酷的是，我们还把智能体的能力融入到推理模型里，让它在思考的时候可以灵活运用各种工具，并且根据环境反馈调整自己的推理过程。这些进步不仅展示了 RL 的巨大潜力，也为通往通用人工智能 (AGI) 的道路打下了坚实的基础。

QwQ-32B 已经在 [Hugging Face](https://huggingface.co/Qwen/QwQ-32B) 和 [ModelScope](https://modelscope.cn/models/Qwen/QwQ-32B) 上开源啦，用的 Apache 2.0 许可，大家可以通过 [Qwen Chat](https://chat.qwen.ai/?models=Qwen2.5-Plus) 体验。

## 效果怎么样？

为了评估 QwQ-32B 的实力，我们用了一系列测试，包括数学推理、代码能力和通用问题解决能力。下面的图表展示了 QwQ-32B 与其他领先模型的对比，比如 DeepSeek-R1-Distilled-Qwen-32B, DeepSeek-R1-Distilled-Llama-70B, o1-mini，当然也包括原版的 DeepSeek-R1。

![](https://qianwen-res.oss-accelerate-overseas.aliyuncs.com/qwq-32b-final.jpg)

## 强化学习大法

我们从一个冷启动的 checkpoint 开始，采用基于结果奖励的强化学习 (RL) 扩展方法。一开始，我们专门针对数学和编码任务进行 RL 训练。并没有使用传统的奖励模型，而是使用了一个数学问题的准确性验证器来确保最终解决方案的正确性，以及一个代码执行服务器来评估生成的代码是否能成功通过预定义的测试用例。随着训练的进行，这两个领域的性能都在持续提升。在第一阶段之后，我们又添加了一个针对通用能力的 RL 阶段。它使用通用奖励模型和一些基于规则的验证器进行训练。我们发现，用少量步骤进行这个阶段的 RL 训练，可以提高其他通用能力，比如指令遵循、与人类偏好对齐和智能体性能，而不会显著降低数学和编码方面的性能。

## 怎么用 QwQ-32B 呢？

下面是一些简单的例子，展示了如何通过 Hugging Face Transformers 和阿里云 DashScope API 使用 QwQ-32B。

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "Qwen/QwQ-32B"

model = AutoModelForCausalLM.from_pretrained(
model_name,
torch_dtype="auto",
device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained(model_name)

prompt = "How many r's are in the word \"strawberry\""
messages = [
{"role": "user", "content": prompt}
]
text = tokenizer.apply_chat_template(
messages,
tokenize=False,
add_generation_prompt=True
)

model_inputs = tokenizer([text], return_tensors="pt").to(model.device)

generated_ids = model.generate(
**model_inputs,
max_new_tokens=32768
)
generated_ids = [
output_ids[len(input_ids):] for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
]

response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]
print(response)
```

```python
from openai import OpenAI
import os

# 初始化 OpenAI 客户端
client = OpenAI(
# 如果没有配置环境变量，请替换成你的 API Key: api_key="sk-xxx"
# 如何获取 API Key：https://help.aliyun.com/zh/model-studio/developer-reference/get-api-key
api_key=os.getenv("DASHSCOPE_API_KEY"),
base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)

reasoning_content = ""
content = ""

is_answering = False

completion = client.chat.completions.create(
model="qwq-32b",
messages=[
{"role": "user", "content": "Which is larger, 9.9 or 9.11?"}
],
stream=True,
# 取消注释下面这行，可以在最后一个 chunk 中返回 token 使用情况
# stream_options={
#     "include_usage": True
# }
)

print("\n" + "=" * 20 + "推理内容" + "=" * 20 + "\n")

for chunk in completion:
# 如果 chunk.choices 是空的，打印使用情况
if not chunk.choices:
print("\n使用情况:")
print(chunk.usage)
else:
delta = chunk.choices[0].delta
# 打印推理内容
if hasattr(delta, 'reasoning_content') and delta.reasoning_content is not None:
print(delta.reasoning_content, end='', flush=True)
reasoning_content += delta.reasoning_content
else:
if delta.content != "" and is_answering is False:
print("\n" + "=" * 20 + "内容" + "=" * 20 + "\n")
is_answering = True
# 打印内容
print(delta.content, end='', flush=True)
content += delta.content
```

## 未来展望

这次 Qwen 在扩展强化学习 (RL) 以增强推理能力方面迈出了第一步。通过这次尝试，我们不仅看到了扩展 RL 的巨大潜力，也认识到预训练语言模型中蕴藏着无限可能。在我们努力开发下一代 Qwen 的过程中，我们有信心，将更强大的基础模型与由规模化计算资源驱动的 RL 相结合，将推动我们更接近实现通用人工智能 (AGI)。此外，我们还在积极探索将智能体与 RL 相结合，以实现长程推理，目标是利用推理时间扩展来释放更大的智能。

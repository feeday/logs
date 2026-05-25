图像反推提示词

## 正反提示词
```
# 自动安装依赖
try:
    import requests
except ImportError:
    import os
    os.system("pip install requests")
    import requests

import base64

# API 配置  0.02 折合每张  https://poixe.com/pricing?i=token
url = "https://api.poixe.com/v1/chat/completions"

headers = {
    "Content-Type": "application/json",
    "Authorization": "sk-2"
}

# 本地图片路径
image_path = r"C:\Users\X\Downloads\douyin\无人\douyin_20260525_0173.jpg"

# 读取图片
with open(image_path, "rb") as image_file:
    base64_image = base64.b64encode(image_file.read()).decode("utf-8")

# 提示词
prompt = """
你是世界级 AI 绘图提示词反推专家。

请根据图片内容，输出：

========================
【中文反推提示词】
========================

要求：
- 使用中文
- 尽可能详细
- 包含：
  - 人物
  - 外貌
  - 表情
  - 动作
  - 服装
  - 场景
  - 光影
  - 构图
  - 摄影风格
  - 镜头语言
  - 色彩
  - 氛围
  - 画质
  - 细节

========================
【English Prompt】
========================

要求：
- 使用专业英文 AI 绘图 Prompt
- 适用于：
  - Midjourney
  - Flux
  - SDXL
  - Stable Diffusion
- 使用英文逗号分隔
- 尽可能详细
- 包含高质量绘图词汇

========================
【负面提示词 Negative Prompt】
========================

输出英文 negative prompt。

========================
【风格标签】
========================

中英文都输出。

========================
【摄影参数】
========================

例如：
- 85mm
- f1.8
- cinematic lighting
- depth of field
- volumetric light
- HDR
- RAW photo

不要解释。
直接输出结果。
"""

# 请求数据
data = {
    "model": "gpt-4o",
    "messages": [
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": prompt
                },
                {
                    "type": "image_url",
                    "image_url": {
                        "url": f"data:image/jpeg;base64,{base64_image}"
                    }
                }
            ]
        }
    ],
    "temperature": 0.8
}

# 发送请求
response = requests.post(url, headers=headers, json=data)

# 输出结果
if response.status_code == 200:
    result = response.json()
    print(result["choices"][0]["message"]["content"])
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

## 一句话提示词
```
# 自动安装依赖
try:
    import requests
except ImportError:
    import os
    os.system("pip install requests")
    import requests

import base64

# API 配置  0.02 折合每张  https://poixe.com/pricing?i=token
url = "https://api.poixe.com/v1/chat/completions"

headers = {
    "Content-Type": "application/json",
    "Authorization": "sk-"
}


# 图片路径
image_path = r"C:\Users\X\Downloads\douyin\无人\douyin_20260525_0173.jpg"

# 读取图片
with open(image_path, "rb") as image_file:
    base64_image = base64.b64encode(image_file.read()).decode("utf-8")

# 更稳定的 Prompt
prompt = """
Describe this image as a single ultra-detailed AI art prompt in English.

Requirements:
- Output only ONE single line
- No explanation
- No markdown
- Include:
subject, facial features, hairstyle, expression, pose, clothing, environment, composition, cinematic lighting, shadows, atmosphere, textures, materials, color grading, depth of field, camera angle, lens, photography settings, artistic style, realistic details, visual aesthetics, high quality render tags

Style:
cinematic, ultra detailed, realistic, professional photography, HDR, volumetric lighting, shallow depth of field, RAW photo, masterpiece quality

Optimized for:
Midjourney, Flux, SDXL, Stable Diffusion
"""

# 请求数据
data = {
    "model": "gpt-4o",
    "messages": [
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": prompt
                },
                {
                    "type": "image_url",
                    "image_url": {
                        "url": f"data:image/jpeg;base64,{base64_image}"
                    }
                }
            ]
        }
    ],
    "temperature": 0.8,
    "max_tokens": 800
}

# 请求
response = requests.post(url, headers=headers, json=data)

# 输出
if response.status_code == 200:
    result = response.json()
    print(result["choices"][0]["message"]["content"])
else:
    print("Error:", response.status_code)
    print(response.text)
```
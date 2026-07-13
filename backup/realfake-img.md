图像真伪检测

# AI图片真伪检测模型成本对比

| 模型 | 平台 | 单张成本（美元） | 单张成本（人民币≈7.2汇率） | 速度 | 特点 | 推荐用途 |
|---|---|---:|---:|---|---|---|
| Gemini-3.1-flash-lite | Google | $0.000462～$0.000469 | ≈0.0033～0.0034元/张 | 快（约1.7～1.8秒） | 成本极低，响应快，适合大规模视觉分析 | **批量初筛首选** |
| GPT-4o | Azure/OpenAI | $0.00325～$0.00418 | ≈0.023～0.030元/张 | 中等（约6.7～6.9秒） | 综合视觉理解能力强，细节判断稳定 | **重点图片复核** |
| Grok-3 | xAI | $0.0128～$0.0162 | ≈0.092～0.117元/张 | 较慢（约8.6～11秒） | 推理能力强，但成本最高 | **疑难案例分析** |


# 成本规模对比

| 检测数量 | Gemini-3.1-flash-lite | GPT-4o | Grok-3 |
|---|---:|---:|---:|
| 100张 | ≈0.33元 | ≈2.5～3元 | ≈9～12元 |
| 1000张 | ≈3.3元 | ≈25～30元 | ≈90～120元 |
| 1万张 | ≈33元 | ≈250～300元 | ≈900～1200元 |
| 10万张 | ≈330元 | ≈2500～3000元 | ≈9000～12000元 |


# AI图片真伪检测推荐方案

| 阶段 | 模型 | 作用 |
|---|---|---|
| 第一层 | Gemini-3.1-flash-lite | 全量扫描，快速判断真实/AI伪造 |
| 第二层 | GPT-4o | 对低置信度、疑似AI图片进行复核 |
| 第三层 | Grok-3 | 对复杂案例、争议图片进行最终分析 |


# 推荐策略

10000张图片：

- Gemini初筛：
  - 成本约33元
  - 快速得到全部结果

- 筛选其中5%异常图片：
  - 500张 × GPT-4o
  - 成本约12～15元

- 极少量争议图片：
  - Grok-3复核


综合成本：
约50元左右完成1万张高质量AI真伪检测。


# 综合评价

|指标|Gemini-3.1-flash-lite|GPT-4o|Grok-3|
|---|---|---|---|
|成本|★★★★★|★★★|★|
|速度|★★★★★|★★★|★★|
|视觉理解|★★★★|★★★★★|★★★★★|
|批量检测|★★★★★|★★★|★★|
|复杂伪造识别|★★★☆|★★★★★|★★★★★|
|性价比|★★★★★|★★★★|★★|

结论：

> 大规模AI图片鉴伪，Gemini-3.1-flash-lite负责99%的初筛，GPT-4o/Grok-3负责疑难复核，是成本和准确率最优组合。












```
# -*- coding: utf-8 -*-

import sys
import subprocess
import importlib
import os
import base64
import mimetypes
import time
import re
from pathlib import Path


# ==================================================
# 自动安装缺失依赖
# ==================================================

def install_package(package):
    try:
        importlib.import_module(package)
    except ImportError:
        print(f"正在安装依赖: {package}")
        subprocess.check_call([
            sys.executable,
            "-m",
            "pip",
            "install",
            package
        ])


required_packages = [
    "requests",
    "pandas",
    "tqdm",
    "openpyxl"
]

for pkg in required_packages:
    install_package(pkg)


import requests
import pandas as pd
from tqdm import tqdm


# ==================================================
# API 配置
# ==================================================

API_URL = "https://api.poixe.com/v1/chat/completions"

# 不建议把 Key 写死在代码里
# 运行后手动输入更安全
API_KEY = input("请输入你的 Poixe API Key：").strip()

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_KEY}"
}


# ==================================================
# 模型选择
# ==================================================

"""
可选模型：

1. gpt-4o
2. grok-3
3. gemini-3.1-flash-lite
"""

MODEL = "gemini-3.1-flash-lite"


# ==================================================
# 图片目录与输出文件
# ==================================================

image_dir = Path(r"C:\img")
output_excel = Path(r"C:\真伪检测结果.xlsx")

INCLUDE_SUBFOLDERS = True


# ==================================================
# 支持格式
# ==================================================

image_ext = {
    ".jpg",
    ".jpeg",
    ".png",
    ".webp",
    ".bmp"
}


# ==================================================
# AI 检测提示词
# ==================================================

prompt = """
你是一名专业的AI生成内容鉴定专家。

请分析上传图片是真实摄影还是AI生成图片。

请从以下方面检查：

1. 人体结构
- 手指数量
- 手部结构
- 四肢比例
- 身体姿态

2. 面部细节
- 五官比例
- 眼睛
- 牙齿
- 耳朵
- 皮肤纹理

3. 光影关系
- 光源方向
- 阴影一致性
- 反射是否符合物理规律

4. 图像物理规律
- 透视
- 景深
- 运动模糊
- 遮挡关系

5. 背景异常
- 文字错误
- 重复纹理
- 边缘融合

6. AI生成特征
- AI水印
- 生成痕迹
- 换脸痕迹

判断：

A 真人实拍

B AI生成、AI修改、换脸、深度伪造

必须选择概率最高结果。

严格输出格式：

最终判断：真实 / AI伪造

置信度：0-100%

最关键三个判断依据：

1.

2.

3.

不要输出其他内容。
"""


# ==================================================
# 图片转 Base64
# ==================================================

def encode_image(path):
    with open(path, "rb") as f:
        return base64.b64encode(f.read()).decode("utf-8")


def get_mime_type(path):
    mime, _ = mimetypes.guess_type(str(path))

    if mime:
        return mime

    suffix = path.suffix.lower()

    if suffix in [".jpg", ".jpeg"]:
        return "image/jpeg"
    elif suffix == ".png":
        return "image/png"
    elif suffix == ".webp":
        return "image/webp"
    elif suffix == ".bmp":
        return "image/bmp"

    return "image/jpeg"


# ==================================================
# 解析模型输出
# ==================================================

def parse_result(text):
    judgment = ""
    confidence = ""
    reason1 = ""
    reason2 = ""
    reason3 = ""

    m = re.search(r"最终判断[:：]\s*(真实|AI伪造)", text)
    if m:
        judgment = m.group(1)

    m = re.search(r"置信度[:：]\s*([0-9]{1,3}%?)", text)
    if m:
        confidence = m.group(1)
        if not confidence.endswith("%"):
            confidence += "%"

    reasons = re.findall(r"\n\s*[1-3][.、]\s*(.+)", text)

    if len(reasons) > 0:
        reason1 = reasons[0].strip()
    if len(reasons) > 1:
        reason2 = reasons[1].strip()
    if len(reasons) > 2:
        reason3 = reasons[2].strip()

    return judgment, confidence, reason1, reason2, reason3


# ==================================================
# 调用模型
# ==================================================

def analyze_image(path, retry=2):
    img64 = encode_image(path)
    mime_type = get_mime_type(path)

    data = {
        "model": MODEL,
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
                            "url": f"data:{mime_type};base64,{img64}"
                        }
                    }
                ]
            }
        ],
        "temperature": 0.2
    }

    last_error = ""

    for attempt in range(retry + 1):
        try:
            response = requests.post(
                API_URL,
                headers=headers,
                json=data,
                timeout=120
            )

            if response.status_code == 200:
                result = response.json()
                return result["choices"][0]["message"]["content"].strip()

            last_error = f"HTTP {response.status_code}: {response.text}"

        except Exception as e:
            last_error = str(e)

        if attempt < retry:
            time.sleep(2)

    return "ERROR: " + last_error


# ==================================================
# 获取图片列表
# ==================================================

def get_image_files(folder):
    if INCLUDE_SUBFOLDERS:
        files = [
            p for p in folder.rglob("*")
            if p.is_file() and p.suffix.lower() in image_ext
        ]
    else:
        files = [
            p for p in folder.iterdir()
            if p.is_file() and p.suffix.lower() in image_ext
        ]

    return sorted(files)


# ==================================================
# 主程序
# ==================================================

def main():
    if not image_dir.exists():
        print(f"图片目录不存在：{image_dir}")
        return

    image_files = get_image_files(image_dir)

    if not image_files:
        print("没有找到支持格式的图片。")
        return

    print(f"共找到 {len(image_files)} 张图片，开始检测...")
    print(f"使用模型：{MODEL}")

    rows = []

    for img_path in tqdm(image_files, desc="检测进度"):
        result_text = analyze_image(img_path)

        if result_text.startswith("ERROR:"):
            judgment = "检测失败"
            confidence = ""
            reason1 = result_text
            reason2 = ""
            reason3 = ""
        else:
            judgment, confidence, reason1, reason2, reason3 = parse_result(result_text)

        rows.append({
            "文件名": img_path.name,
            "完整路径": str(img_path),
            "最终判断": judgment,
            "置信度": confidence,
            "依据1": reason1,
            "依据2": reason2,
            "依据3": reason3,
            "模型原始输出": result_text
        })

    df = pd.DataFrame(rows)

    output_excel.parent.mkdir(parents=True, exist_ok=True)
    df.to_excel(output_excel, index=False)

    print("\n检测完成！")
    print(f"结果已保存到：{output_excel}")


if __name__ == "__main__":
    main()
```




```
# -*- coding: utf-8 -*-

import os
import base64
import requests
import pandas as pd
from tqdm import tqdm


# ===============================
# API 配置
# ===============================

url = "https://api.poixe.com/v1/chat/completions"

headers = {
    "Content-Type": "application/json",
    "Authorization": "sk-你的KEY"
}


# ===============================
# 图片目录
# ===============================

image_dir = r"c:\img"

output_excel = r"c:\img\真伪检测结果.xlsx"



# ===============================
# 支持图片格式
# ===============================

image_ext = [
    ".jpg",
    ".jpeg",
    ".png",
    ".webp"
]



# ===============================
# AI检测提示词
# ===============================

prompt = """

你是一名专业的 AI 生成内容鉴定专家。

请对上传的图像进行真实性分析。

结合以下证据判断：

1. 人物肢体细节：

- 手指数量
- 手部结构
- 四肢比例
- 身体姿态
- 是否符合人体运动规律


2. 面部细节：

- 五官比例
- 眼睛
- 牙齿
- 耳朵
- 皮肤纹理
- 表情自然程度


3. 光影与反射：

- 光源方向
- 阴影关系
- 镜面反射
- 环境光一致性


4. 物理规律：

- 透视关系
- 景深
- 运动模糊
- 遮挡关系


5. 背景异常：

- 背景变形
- 文字错误
- 重复纹理
- 边缘融合


6. 图像质量：

- 压缩痕迹
- 局部噪声
- AI生成伪影


7. AI证据：

- 是否存在AI水印
- 是否存在生成标记
- 是否存在换脸痕迹


请判断：

A. 真人实拍或正常摄影图片

B. AI生成、AI换脸、深度伪造、生成式AI修改图片


规则：

- 必须选择 A 或 B。
- 不允许回答无法判断。
- 没有AI水印不能作为真实依据。
- 根据所有证据选择概率最高结果。


严格输出：

最终判断：真实 / AI伪造

置信度：0—100%

最关键的三个判断依据：

1.

2.

3.


不要输出其他内容。

"""



# ===============================
# 图片编码
# ===============================

def encode_image(path):

    with open(path, "rb") as f:

        return base64.b64encode(
            f.read()
        ).decode("utf-8")



# ===============================
# 调用模型
# ===============================

def analyze_image(path):

    img64 = encode_image(path)


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

                            "url":
                            f"data:image/jpeg;base64,{img64}"

                        }

                    }

                ]

            }

        ],

        "temperature":0.2

    }



    response = requests.post(

        url,

        headers=headers,

        json=data,

        timeout=120

    )


    if response.status_code == 200:

        result = response.json()

        return result["choices"][0]["message"]["content"]


    else:

        return "ERROR:" + response.text




# ===============================
# 解析结果
# ===============================

def parse_result(text):


    result = {

        "最终判断":"",
        "置信度":"",
        "依据1":"",
        "依据2":"",
        "依据3":""

    }



    lines = text.split("\n")


    for line in lines:


        line=line.strip()


        if "最终判断" in line:

            result["最终判断"] = (
                line.replace("最终判断：","")
                .replace("最终判断:","")
                .strip()
            )


        elif "置信度" in line:

            result["置信度"] = (
                line.replace("置信度：","")
                .replace("置信度:","")
                .strip()
            )


        elif line.startswith("1."):

            result["依据1"]=line[2:].strip()


        elif line.startswith("2."):

            result["依据2"]=line[2:].strip()


        elif line.startswith("3."):

            result["依据3"]=line[2:].strip()



    return result




# ===============================
# 获取图片列表（包含子文件夹）
# ===============================

files=[]


for root, dirs, filenames in os.walk(image_dir):

    for f in filenames:

        ext=os.path.splitext(f)[1].lower()


        if ext in image_ext:

            full_path=os.path.join(
                root,
                f
            )

            files.append(full_path)



print("====================")
print("发现图片数量:",len(files))
print("====================")




# ===============================
# 批量检测
# ===============================

rows=[]


for path in tqdm(files):


    try:


        ai_result = analyze_image(path)


        parsed = parse_result(
            ai_result
        )


        rows.append({

            "文件名":
            os.path.relpath(
                path,
                image_dir
            ),

            **parsed

        })



    except Exception as e:


        rows.append({

            "文件名":
            os.path.relpath(
                path,
                image_dir
            ),

            "错误":
            str(e)

        })




# ===============================
# 导出Excel
# ===============================


df=pd.DataFrame(rows)


df.to_excel(

    output_excel,

    index=False

)



print("====================")
print("检测完成")
print("输出文件:")
print(output_excel)
print("====================")
```
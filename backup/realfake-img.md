图像真伪检测

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
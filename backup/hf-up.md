- https://colab.new
- https://hf-mirror.com
- https://huggingface.co/datasets/datxy/demo/blob/main/test/eso1242a.psb
- https://hf-mirror.com/datasets/datxy/demo/blob/main/test/eso1242a.psb

```
from huggingface_hub import login, HfApi
import requests
from tqdm import tqdm

login()

repo_id = "datxy/demo"

url_list = [
    {
        "url": "https://cdn2.eso.org/images/original/eso1242a.psb",
        "name": "eso1242a.psb"
    }
]

api = HfApi()
api.create_repo(repo_id=repo_id, repo_type="dataset", exist_ok=True)

for item in url_list:
    url = item["url"]
    name = item["name"]

    print(f"\n⬇️ downloading: {name}")

    r = requests.get(url, stream=True)
    total = int(r.headers.get("content-length", 0))

    with open(name, "wb") as f, tqdm(total=total, unit="B", unit_scale=True) as bar:
        for chunk in r.iter_content(1024 * 1024):
            if chunk:
                f.write(chunk)
                bar.update(len(chunk))

    print(f"☁️ uploading to test/ folder")

    api.upload_file(
        path_or_fileobj=name,
        path_in_repo=f"test/{name}",
        repo_id=repo_id,
        repo_type="dataset"
    )

    print(f"✅ done: test/{name}")
```
![ScreenShot_2026-07-04_133815_204.png](https://i.imgur.com/77UHCUb.png)

```
import os
from huggingface_hub import snapshot_download, hf_hub_download

# =========================
# 🌍 镜像（可选）
# Windows PowerShell 
# $env:HF_ENDPOINT = "https://hf-mirror.com"
# =========================
os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"


# =========================
# 🧠 自动识别 repo 类型
# =========================
def detect_repo_type(repo_id: str):

    r = repo_id.lower()

    if any(x in r for x in ["dataset", "data", "demo"]):
        return "dataset"

    return "model"


# =========================
# 🚀 ULTRA CORRECT ENGINE
# =========================
def download(repo_id, file_list=None, local_dir="./downloads"):

    os.makedirs(local_dir, exist_ok=True)

    repo_type = detect_repo_type(repo_id)

    print("\n==============================")
    print("🚀 ULTRA CORRECT ENGINE START")
    print("==============================")
    print("📦 Repo:", repo_id)
    print("🧠 Type:", repo_type)
    print("📁 Dir :", local_dir)
    print("==============================\n")

    # =========================
    # CASE 1：全量下载（唯一正确方式）
    # =========================
    if not file_list:

        print("📦 FULL SNAPSHOT MODE")

        path = snapshot_download(
            repo_id=repo_id,
            repo_type=repo_type,
            local_dir=local_dir,
            resume_download=True,
            local_dir_use_symlinks=False
        )

        print("\n✅ 完成:", path)
        return


    # =========================
    # CASE 2：单文件（正确方式，不猜路径）
    # =========================
    print("📦 SINGLE FILE MODE")

    for file in file_list:

        print("\n⬇️ 目标文件:", file)

        try:

            # ❗ 不再猜 test/ data/
            # ❗ 只使用真实路径
            path = hf_hub_download(
                repo_id=repo_id,
                filename=file,
                repo_type=repo_type,
                local_dir=local_dir
            )

            print("✅ 成功:", path)

        except Exception as e:
            print("❌ 失败:", file)
            print("原因:", str(e))


# =========================
# 🚀 示例使用
# =========================
if __name__ == "__main__":

    # =========================
    # 🔥 你只需要改这里
    # =========================

    REPO = "datxy/demo"

    # 👉 None = 全量下载
    FILES = None

    # 👉 指定文件（必须写完整路径！！）
    # FILES = ["test/eso1242a.psb"]

    download(
        repo_id=REPO,
        file_list=FILES,
        local_dir="./downloads"
    )
```
![ScreenShot_2026-07-04_150813_644.png](https://i.imgur.com/kT69aFt.png)
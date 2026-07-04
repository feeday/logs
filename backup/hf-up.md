- https://colab.new
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
```
# -*- coding: utf-8 -*-

from pathlib import Path
import shutil
import csv
import re

# ================= 配置区 =================

TXT_FILE = Path(r"d:\line.txt")

SEARCH_DIR = Path(r"d:\1")

TARGET_DIR = Path(r"d:\2")

INCLUDE_SUBFOLDERS = True

OVERWRITE = True

# True：保留源目录结构
# False：所有文件直接放到 TARGET_DIR
KEEP_FOLDER_STRUCTURE = True

# 日志文件
LOG_FILE = TARGET_DIR / "list.csv"

# ================= 操作模式 =================
# 0 = 预览（只打印不操作）
# 1 = 复制
# 2 = 移动
MODE = 1
# ===========================================


# ================= TXT清洗 =================
def clean_text(x: str):
    """去除不可见字符和空白字符"""
    if not x:
        return ""

    x = x.replace("\ufeff", "")
    x = x.replace("\u200b", "")
    x = x.replace("\u200c", "")
    x = x.replace("\u200d", "")

    x = re.sub(r"\s+", "", x)

    return x.lower()


def read_txt_names(txt_file: Path):
    """
    读取TXT中的文件名。

    支持：
    1
    1.mp4
    xxx/1.mp4
    G:\\xxx\\1.mp4
    """
    names = []

    with open(txt_file, "r", encoding="utf-8-sig") as f:
        for line in f:
            line = clean_text(line)

            if not line:
                continue

            # 如果是表格复制出来的内容，取最后一列
            if "\t" in line:
                parts = [
                    clean_text(p)
                    for p in line.split("\t")
                    if p.strip()
                ]

                if parts:
                    line = parts[-1]

            name = Path(line).name

            if name in ["文件名", "完整路径"]:
                continue

            # 只匹配文件名主体，不匹配扩展名
            names.append(Path(name).stem.lower())

    return names


# ================= 重名处理 =================
def get_unique_path(path: Path):
    """目标文件已存在时，生成不重名路径"""
    if not path.exists():
        return path

    parent = path.parent
    stem = path.stem
    suffix = path.suffix

    i = 1

    while True:
        new_path = parent / f"{stem}_{i}{suffix}"

        if not new_path.exists():
            return new_path

        i += 1


# ================= 生成目标路径 =================
def build_target_path(src: Path):
    """
    根据 KEEP_FOLDER_STRUCTURE 生成目标路径。
    """

    if KEEP_FOLDER_STRUCTURE:
        # 获取文件相对于源目录的路径
        relative_path = src.relative_to(SEARCH_DIR)

        # 在目标目录下重建同样的目录结构
        target_file = TARGET_DIR / relative_path

    else:
        # 不保留目录结构，全部放到目标根目录
        target_file = TARGET_DIR / src.name

    return target_file


# ================= 文件操作 =================
def process_file(src: Path, dst: Path):
    # 创建目标文件所在目录
    dst.parent.mkdir(parents=True, exist_ok=True)

    if MODE == 0:
        print(f"[预览] {src} -> {dst}")
        return "预览"

    elif MODE == 1:
        shutil.copy2(str(src), str(dst))
        print(f"[复制] {src} -> {dst}")
        return "复制"

    elif MODE == 2:
        shutil.move(str(src), str(dst))
        print(f"[移动] {src} -> {dst}")
        return "移动"

    else:
        raise ValueError("MODE 必须是 0、1 或 2")


# ================= 主流程 =================
def main():
    mode_map = {
        0: "预览",
        1: "复制",
        2: "移动",
    }

    print(f"当前模式：{MODE}（{mode_map.get(MODE)}）")
    print(f"保留目录结构：{KEEP_FOLDER_STRUCTURE}")
    print(f"包含子目录：{INCLUDE_SUBFOLDERS}")
    print(f"允许覆盖：{OVERWRITE}")
    print("-" * 80)

    if not TXT_FILE.exists():
        print(f"TXT不存在：{TXT_FILE}")
        return

    if not SEARCH_DIR.exists():
        print(f"源目录不存在：{SEARCH_DIR}")
        return

    TARGET_DIR.mkdir(parents=True, exist_ok=True)

    txt_names = read_txt_names(TXT_FILE)
    txt_set = set(txt_names)

    print(f"TXT中读取到：{len(txt_names)} 个文件名")
    print(f"去重后数量：{len(txt_set)}")

    if INCLUDE_SUBFOLDERS:
        all_files = [
            p
            for p in SEARCH_DIR.rglob("*")
            if p.is_file()
        ]
    else:
        all_files = [
            p
            for p in SEARCH_DIR.iterdir()
            if p.is_file()
        ]

    print(f"源目录中找到文件数量：{len(all_files)}")
    print("-" * 80)

    success = 0
    fail = 0

    matched_names = set()
    log_rows = []

    for file in all_files:
        match_key = file.stem.lower()

        if match_key not in txt_set:
            continue

        target_file = build_target_path(file)

        if target_file.exists():
            if OVERWRITE:
                # 直接使用原路径，复制或移动时覆盖
                pass
            else:
                # 不覆盖时自动添加序号
                target_file = get_unique_path(target_file)

        try:
            action = process_file(file, target_file)

            matched_names.add(match_key)

            if MODE == 0:
                log_type = "预览"
            else:
                success += 1
                log_type = "成功"

            log_rows.append([
                log_type,
                match_key,
                str(file),
                str(target_file),
                action,
            ])

        except Exception as e:
            fail += 1

            print(f"[错误] {file} -> {e}")

            log_rows.append([
                "失败",
                match_key,
                str(file),
                str(target_file),
                str(e),
            ])

    not_found = txt_set - matched_names

    for name in sorted(not_found):
        log_rows.append([
            "未找到",
            name,
            "",
            "",
            "源目录中未匹配到",
        ])

    with open(
        LOG_FILE,
        "w",
        newline="",
        encoding="utf-8-sig",
    ) as f:
        writer = csv.writer(f)

        writer.writerow([
            "状态",
            "匹配文件名",
            "原始路径",
            "目标路径",
            "备注",
        ])

        writer.writerows(log_rows)

    print("-" * 80)
    print("全部完成")
    print(f"成功：{success}")
    print(f"失败：{fail}")
    print(f"未找到：{len(not_found)}")
    print(f"日志：{LOG_FILE}")


if __name__ == "__main__":
    main()
```
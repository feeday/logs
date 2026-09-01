```
from pathlib import Path

def delete_db_files_recursively(target_directory):
    # 将字符串路径转换为 Path 对象
    base_path = Path(target_directory)
    
    # rglob('*') 表示递归查找所有文件，过滤出 .db 文件
    # 如果只想删除当前层级不包含子目录，使用 glob('*.db')
    db_files = list(base_path.rglob('*.db'))
    
    if not db_files:
        print("未找到 .db 文件。")
        return

    print(f"找到 {len(db_files)} 个 .db 文件，准备删除...")
    
    for db_file in db_files:
        try:
            db_file.unlink() # 删除文件
            print(f"已删除: {db_file}")
        except PermissionError:
            print(f"无法删除 (被占用或无权限): {db_file}")
        except Exception as e:
            print(f"错误: {e}")

# --- 使用示例 ---
target_folder = r"e:\data" # 请替换为你的路径
delete_db_files_recursively(target_folder)
```
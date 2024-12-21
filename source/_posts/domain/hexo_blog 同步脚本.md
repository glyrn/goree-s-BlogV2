---
title: hexo_blog 同步脚本
date: 2024-12-21 22:53:16
categories: domain
updated: 2024-12-21 23:54:45
cover: /img/default.avif
---
> 使用python且支持幂等

```python
import os
import shutil

def sync_md_files(source_dir, target_dir):
    """
    完全同步源目录和目标目录中的 .md 文件，删除目标目录中不再存在于源目录中的文件，
    并复制源目录中新添加或修改的文件。
    """
    source_dir = os.path.normpath(source_dir)
    target_dir = os.path.normpath(target_dir)

    if not os.path.exists(source_dir):
        print(f"源目录 {source_dir} 不存在。")
        return

    if not os.path.exists(target_dir):
        os.makedirs(target_dir)

    skipped = 0
    copied = 0
    deleted = 0

    # 遍历源目录中的所有 .md 文件
    source_files = {}
    for root, _, files in os.walk(source_dir):
        for file in files:
            if file.endswith('.md'):
                source_file = os.path.join(root, file)
                relative_path = os.path.relpath(root, source_dir)
                target_path = os.path.join(target_dir, relative_path)
                target_file = os.path.join(target_path, file)
                
                source_files[target_file] = source_file

                # 如果目标文件夹不存在，创建它
                os.makedirs(target_path, exist_ok=True)

                # 检查目标文件是否存在，且修改时间一致
                if os.path.exists(target_file):
                    source_mtime = os.path.getmtime(source_file)
                    target_mtime = os.path.getmtime(target_file)
                    if source_mtime <= target_mtime:
                        skipped += 1
                        continue
                
                try:
                    # 复制文件
                    shutil.copy2(source_file, target_file)
                    copied += 1
                except Exception as e:
                    print(f"复制失败: {source_file} -> {target_file}，错误: {e}")

    # 检查目标目录中不再存在于源目录中的文件并删除
    target_files = {}
    for root, _, files in os.walk(target_dir):
        for file in files:
            if file.endswith('.md'):
                target_file = os.path.join(root, file)
                target_files[target_file] = os.path.join(root, file)

    for target_file in target_files:
        if target_file not in source_files:
            try:
                os.remove(target_file)
                deleted += 1
            except Exception as e:
                print(f"删除失败: {target_file}，错误: {e}")

    print(f"同步完成：跳过 {skipped} 个文件，复制 {copied} 个文件，删除 {deleted} 个文件。")

if __name__ == "__main__":
    source_directory = "D:\obsidianKnowledgeV2.0"
    target_directory = "D:\hexo\source\_posts"
    sync_md_files(source_directory, target_directory)


```

目标地址
/d/hexo/source/_posts

源地址
/d/obsidianKnowledgeV2.0
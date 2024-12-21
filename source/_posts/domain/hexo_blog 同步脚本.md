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

def copy_md_files(source_dir, target_dir):
    """
    复制源目录中的所有 .md 文件到目标目录，保持原有目录结构。
    如果文件未更改，则跳过复制。
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

    for root, _, files in os.walk(source_dir):
        for file in files:
            if file.endswith('.md'):
                source_file = os.path.join(root, file)
                relative_path = os.path.relpath(root, source_dir)
                target_path = os.path.join(target_dir, relative_path)
                target_file = os.path.join(target_path, file)
                
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
    
    print(f"已完成：跳过 {skipped} 个文件，复制 {copied} 个文件。")

if __name__ == "__main__":  
    source_directory = "D:\obsidianKnowledgeV2.0"    
    target_directory = "D:\hexo\source\_posts"   
    copy_md_files(source_directory, target_directory)

```

目标地址
/d/hexo/source/_posts

源地址
/d/obsidianKnowledgeV2.0
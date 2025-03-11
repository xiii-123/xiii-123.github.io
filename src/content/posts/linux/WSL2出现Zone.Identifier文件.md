---
title: WSL2出现:Zone.Identifier
published: 2025-03-11
description: 'WSL2出现:Zone.Identifier如何解决？'
image: ''
tags: [WSL2]
category: 'WSL2'
draft: false 
---

# 原因
从 Windows 直接下载文件或移动文件到 WSL 目录时，会出现类似 `:Zone.Identifier` 的文件。

其中包含了一些跟关联文件有关的元数据。

该文件因为微软的 NTFS 功能而出现，虽然没有实际用途。

但因为文件名包含 : 冒号，所以可能会破坏某些 Linux 脚本的运行。

所以需要处理它。

# 解决方法
- 移动文件时，直接将文件从资源管理器拖动到VSCode中，有概率不产生 `:Zone.Identifier` 文件；
- 当产生该文件时，通过这条命令删除该文件：`find . -name "*:Zone.Identifier" -type f -delete`。
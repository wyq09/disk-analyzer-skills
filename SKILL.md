---
name: disk-analyzer
description: 磁盘空间分析工具。当用户请求「分析磁盘空间」「查看磁盘占用」「清理磁盘」「磁盘满了」「空间不足」「哪些文件占用空间大」或类似需求时使用。只进行分析和提供建议，绝不直接删除任何文件。
author: wuyiqun
license: MIT
---

# 磁盘空间分析 Skill

## 核心原则

**重要：此 skill 只进行分析和提供建议，绝对禁止直接删除任何文件或文件夹！**

所有清理操作必须由用户手动确认和执行。

## 使用场景

当用户请求以下任何操作时，使用此 skill：
- 分析磁盘空间
- 查看磁盘占用情况
- 磁盘空间不足
- 磁盘满了怎么办
- 哪些文件占用空间大
- 大文件查找
- 清理磁盘建议
- 系统垃圾文件分析
- 目录大小统计

## 工作流程

### 1. 确定分析目标

首先确认用户想要分析的路径：
- 如果用户指定了路径，分析该路径
- 如果用户未指定，询问用户想分析哪个目录（默认可分析用户主目录）

### 2. 获取磁盘整体情况

使用以下命令获取磁盘概览：

**macOS / Linux:**
```bash
# 查看磁盘整体使用情况
df -h

# 查看指定目录所在分区的使用情况
df -h /path/to/directory
```

### 3. 分析目录大小

**macOS / Linux:**
```bash
# 分析当前目录下各子目录大小（按大小排序，显示前20个）
du -sh */ 2>/dev/null | sort -hr | head -20

# 分析指定目录下各子目录大小
du -sh /path/to/directory/*/ 2>/dev/null | sort -hr | head -20

# 查找大文件（超过100MB）
find /path/to/directory -type f -size +100M 2>/dev/null | head -20

# 查找大文件并显示大小
find /path/to/directory -type f -size +100M -exec ls -lh {} \; 2>/dev/null | awk '{print $5, $9}' | sort -hr | head -20
```

### 4. 常见可清理目录检查

根据操作系统，检查以下常见的可清理位置：

**macOS:**
```bash
# 系统缓存
du -sh ~/Library/Caches 2>/dev/null

# 系统日志
du -sh ~/Library/Logs 2>/dev/null

# 应用程序支持文件（可能有旧应用残留）
ls -la ~/Library/Application\ Support/ 2>/dev/null

# 下载目录
du -sh ~/Downloads 2>/dev/null

# 废纸篓
du -sh ~/.Trash 2>/dev/null

# Xcode 派生数据（开发者）
du -sh ~/Library/Developer/Xcode/DerivedData 2>/dev/null

# npm 缓存
du -sh ~/.npm 2>/dev/null

# yarn 缓存
du -sh ~/Library/Caches/Yarn 2>/dev/null

# pip 缓存
du -sh ~/Library/Caches/pip 2>/dev/null

# Homebrew 缓存
du -sh ~/Library/Caches/Homebrew 2>/dev/null

# Docker 相关
du -sh ~/Library/Containers/com.docker.docker 2>/dev/null
```

**Linux:**
```bash
# APT 缓存 (Debian/Ubuntu)
du -sh /var/cache/apt/archives 2>/dev/null

# yum 缓存 (RHEL/CentOS)
du -sh /var/cache/yum 2>/dev/null

# 系统日志
du -sh /var/log 2>/dev/null

# 临时文件
du -sh /tmp 2>/dev/null

# 用户缓存
du -sh ~/.cache 2>/dev/null

# npm 缓存
du -sh ~/.npm 2>/dev/null

# pip 缓存
du -sh ~/.cache/pip 2>/dev/null
```

### 5. 生成分析报告

根据收集的信息，生成结构化的分析报告：

```markdown
# 磁盘空间分析报告

## 磁盘概览
- 总容量: XXX GB
- 已使用: XXX GB (XX%)
- 可用空间: XXX GB

## 大文件/目录分析
| 路径 | 大小 | 说明 |
|------|------|------|
| ... | ... | ... |

## 可清理项目建议
| 路径 | 大小 | 风险等级 | 清理建议 |
|------|------|----------|----------|
| ... | ... | ... | ... |

## 清理命令建议（请手动执行）
# 低风险清理
[具体的清理命令]

# 中等风险清理（请先确认）
[具体的清理命令]
```

## 可清理项目分类

### 低风险（通常可安全清理）

| 类型 | 路径 | 说明 |
|------|------|------|
| 废纸篓 | ~/.Trash (macOS) | 已删除的文件暂存 |
| 下载目录 | ~/Downloads | 用户下载的文件 |
| 浏览器缓存 | ~/Library/Caches/浏览器名称 | 浏览器临时文件 |
| npm 缓存 | ~/.npm | Node.js 包缓存 |
| pip 缓存 | ~/.cache/pip 或 ~/Library/Caches/pip | Python 包缓存 |
| 系统临时文件 | /tmp | 临时文件（重启可能清空） |

### 中等风险（清理前请确认）

| 类型 | 路径 | 说明 |
|------|------|------|
| Xcode 派生数据 | ~/Library/Developer/Xcode/DerivedData | Xcode 编译缓存 |
| Docker 镜像 | Docker 系统存储 | 未使用的 Docker 镜像 |
| Homebrew 缓存 | ~/Library/Caches/Homebrew | 软件包下载缓存 |
| 应用程序缓存 | ~/Library/Caches | 各应用程序缓存 |

### 高风险（不建议清理或需谨慎）

| 类型 | 路径 | 说明 |
|------|------|------|
| 应用程序支持 | ~/Library/Application Support | 应用配置和数据 |
| 系统库文件 | /Library, /System | 系统核心文件 |
| 用户配置 | ~/.config, ~/.* 配置文件 | 用户应用配置 |

## 清理命令参考

**重要：以下命令仅供参考，请在执行前仔细确认！**

### macOS 清理命令

```bash
# 清空废纸篓（低风险）
rm -rf ~/.Trash/*

# 清理 npm 缓存（低风险）
npm cache clean --force

# 清理 yarn 缓存（低风险）
yarn cache clean

# 清理 pip 缓存（低风险）
pip cache purge

# 清理 Homebrew 缓存（低风险）
brew cleanup --prune=all

# 清理 Xcode 派生数据（中等风险，下次编译会重新生成）
rm -rf ~/Library/Developer/Xcode/DerivedData/*

# 查看 Docker 空间使用情况
docker system df

# 清理 Docker 未使用资源（中等风险）
docker system prune
```

### Linux 清理命令

```bash
# 清理 APT 缓存（Debian/Ubuntu，低风险）
sudo apt-get clean
sudo apt-get autoremove

# 清理 yum 缓存（RHEL/CentOS，低风险）
sudo yum clean all

# 清理 pip 缓存（低风险）
pip cache purge

# 清理 npm 缓存（低风险）
npm cache clean --force
```

## 报告输出格式

每次分析完成后，输出以下格式的报告：

```
📊 磁盘空间分析报告
==================

💻 磁盘概览:
   总容量: 500 GB
   已使用: 350 GB (70%)
   可用空间: 150 GB

📁 最大的目录/文件:
   1. ~/Library (45 GB) - 系统库文件
   2. ~/Projects (30 GB) - 项目文件
   3. ~/Downloads (15 GB) - 下载文件
   ...

🗑️ 可清理项目建议:

   低风险 (建议清理):
   ┌─────────────────────────────────┬────────┬─────────────────────┐
   │ 路径                            │ 大小   │ 说明                │
   ├─────────────────────────────────┼────────┼─────────────────────┤
   │ ~/.Trash                        │ 5 GB   │ 废纸篓              │
   │ ~/.npm                          │ 2 GB   │ npm 缓存            │
   │ ~/Downloads                     │ 15 GB  │ 下载文件（请确认）  │
   └─────────────────────────────────┴────────┴─────────────────────┘

   中等风险 (请先确认):
   ┌─────────────────────────────────┬────────┬─────────────────────┐
   │ 路径                            │ 大小   │ 说明                │
   ├─────────────────────────────────┼────────┼─────────────────────┤
   │ ~/Library/Caches               │ 8 GB   │ 应用缓存            │
   │ ~/Library/Developer/Xcode      │ 10 GB  │ Xcode 数据          │
   └─────────────────────────────────┴────────┴─────────────────────┘

📝 建议的清理命令 (请手动复制执行):

   # 清空废纸篓 (可释放约 5 GB)
   rm -rf ~/.Trash/*

   # 清理 npm 缓存 (可释放约 2 GB)
   npm cache clean --force

⚠️ 注意事项:
   - 以上命令仅供参考，请在执行前确认路径正确
   - 建议先备份重要数据
   - 如有疑问，请逐个确认后再执行

💡 预计可释放空间: 约 22 GB
```

## 注意事项

1. **绝对不自动删除文件**：只提供建议，所有删除操作由用户手动执行
2. **风险评估**：对每个可清理项目标注风险等级
3. **路径验证**：在给出建议前，先验证路径是否存在
4. **空间预估**：尽量提供可释放空间的预估值
5. **备份提醒**：提醒用户在清理前备份重要数据

## 特殊目录说明

### macOS 特有目录

- `~/Library`：包含应用程序支持文件、缓存、偏好设置等
- `~/Library/Caches`：可安全清理，应用会自动重建
- `~/Library/Application Support`：包含应用数据，清理需谨慎
- `~/Library/Developer`：开发工具相关，Xcode 派生数据可清理
- `~/Library/Containers`：沙盒应用数据

### 开发相关目录

- `node_modules`：Node.js 项目依赖，可通过 npm install 重建
- `.venv` / `venv`：Python 虚拟环境，可重建
- `__pycache__`：Python 缓存，可安全删除
- `target`（Maven）/ `build`（Gradle）：Java 构建输出，可重建
- `.gradle`：Gradle 缓存，可清理部分

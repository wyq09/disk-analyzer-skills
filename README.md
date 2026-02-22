# Disk Analyzer Skill / 磁盘空间分析技能

[English](#english) | [中文](#中文)

---

<a name="english"></a>
## English

### Overview

A Claude Code skill for analyzing disk space usage and providing cleanup recommendations. **This skill only analyzes and provides suggestions - it NEVER automatically deletes any files or folders.**

### Features

- Analyze disk space usage overview
- Identify large files and directories
- **Deep cache analysis** - Detailed breakdown of `~/Library/Caches` contents
- **Identify unused/orphaned cache files** from uninstalled applications
- **Cache safety classification** - Know which caches are safe to clean
- Check common cleanup locations (caches, logs, temp files, etc.)
- Risk assessment for each cleanup item (🟢 Low / 🟡 Medium / 🔴 High)
- Generate structured analysis reports
- Provide manual cleanup commands for user execution

### Installation

Copy the `SKILL.md` file to your Claude Code skills directory:

```bash
# Create skills directory if not exists
mkdir -p ~/.claude/skills/disk-analyzer

# Copy the skill file
cp SKILL.md ~/.claude/skills/disk-analyzer/
```

Or clone this repository:

```bash
git clone git@github.com:wyq09/disk-analyzer-skills.git
cd disk-analyzer-skills
cp SKILL.md ~/.claude/skills/disk-analyzer/
```

### Usage

In Claude Code, use any of the following commands:

- `/disk-analyzer` - Start disk analysis
- "分析磁盘空间" (Analyze disk space)
- "查看磁盘占用" (Check disk usage)
- "磁盘满了怎么办" (Disk full, what to do)
- "哪些文件占用空间大" (Which files are taking up space)
- "清理磁盘建议" (Disk cleanup suggestions)

### How It Works

1. **Disk Overview**: Gets overall disk usage with `df -h`
2. **Directory Analysis**: Analyzes subdirectory sizes with `du` commands
3. **Large File Detection**: Finds files larger than 100MB/500MB
4. **Cleanup Check**: Examines common cleanup locations:
   - Trash (~/.Trash)
   - Downloads folder
   - npm/yarn/pip caches
   - Homebrew cache
   - Xcode derived data
   - Application caches
5. **Report Generation**: Produces a structured report with cleanup recommendations

### Risk Levels

| Level | Description | Examples |
|-------|-------------|----------|
| Low | Safe to clean, auto-regenerable | Trash, npm cache, pip cache |
| Medium | Confirm before cleaning | App caches, Docker images, Xcode derived data |
| High | Not recommended | App support files, system libraries, user configs |

### Sample Output

```
📊 Disk Space Analysis Report
============================

💻 Disk Overview:
   Total: 926 GB
   Used: 864 GB (93%)
   Available: 3.9 GB

📁 Largest Directories:
   1. ~/Library (45 GB) - System library files
   2. ~/Projects (30 GB) - Project files
   3. ~/Downloads (15 GB) - Downloaded files

🗑️ Cleanup Recommendations:

   Low Risk (Recommended):
   ┌─────────────────────────┬────────┬─────────────────┐
   │ Path                    │ Size   │ Description     │
   ├─────────────────────────┼────────┼─────────────────┤
   │ ~/.Trash                │ 5 GB   │ Trash           │
   │ ~/.npm                  │ 2 GB   │ npm cache       │
   └─────────────────────────┴────────┴─────────────────┘

📝 Suggested Cleanup Commands (Execute Manually):

   # Empty trash (~5 GB)
   rm -rf ~/.Trash/*

   # Clean npm cache (~2 GB)
   npm cache clean --force

💡 Estimated Reclaimable Space: ~7 GB
```

### Safety Guarantees

- **No Automatic Deletion**: All cleanup commands are provided as reference only
- **Risk Labels**: Every cleanup item is labeled with risk level
- **Manual Execution**: Users must manually copy and execute cleanup commands
- **Path Validation**: Paths are verified before making recommendations

---

<a name="中文"></a>
## 中文

### 概述

一个用于分析磁盘空间使用情况并提供清理建议的 Claude Code 技能。**此技能只进行分析和提供建议，绝不自动删除任何文件或文件夹。**

### 功能特性

- 分析磁盘空间使用概览
- 识别大文件和大目录
- **深度缓存分析** - 详细分解 `~/Library/Caches` 内容
- **识别无用/孤立缓存文件** - 检测已卸载应用的残留缓存
- **缓存安全分类** - 明确哪些缓存可以安全清理
- 检查常见可清理位置（缓存、日志、临时文件等）
- 为每个清理项目评估风险等级（🟢 低 / 🟡 中 / 🔴 高）
- 生成结构化分析报告
- 提供手动清理命令供用户执行

### 安装方法

将 `SKILL.md` 文件复制到你的 Claude Code 技能目录：

```bash
# 如果技能目录不存在则创建
mkdir -p ~/.claude/skills/disk-analyzer

# 复制技能文件
cp SKILL.md ~/.claude/skills/disk-analyzer/
```

或克隆此仓库：

```bash
git clone git@github.com:wyq09/disk-analyzer-skills.git
cd disk-analyzer-skills
cp SKILL.md ~/.claude/skills/disk-analyzer/
```

### 使用方法

在 Claude Code 中，使用以下任一命令：

- `/disk-analyzer` - 启动磁盘分析
- "分析磁盘空间"
- "查看磁盘占用"
- "磁盘满了怎么办"
- "哪些文件占用空间大"
- "清理磁盘建议"
- "大文件查找"
- "系统垃圾文件分析"

### 工作原理

1. **磁盘概览**：使用 `df -h` 获取整体磁盘使用情况
2. **目录分析**：使用 `du` 命令分析子目录大小
3. **大文件检测**：查找大于 100MB/500MB 的文件
4. **清理检查**：检查常见可清理位置：
   - 废纸篓 (~/.Trash)
   - 下载目录
   - npm/yarn/pip 缓存
   - Homebrew 缓存
   - Xcode 派生数据
   - 应用程序缓存
5. **报告生成**：生成包含清理建议的结构化报告

### 风险等级

| 等级 | 描述 | 示例 |
|------|------|------|
| 低风险 | 可安全清理，会自动重建 | 废纸篓、npm 缓存、pip 缓存 |
| 中等风险 | 清理前请确认 | 应用缓存、Docker 镜像、Xcode 派生数据 |
| 高风险 | 不建议清理 | 应用支持文件、系统库文件、用户配置 |

### 示例输出

```
📊 磁盘空间分析报告
==================

💻 磁盘概览:
   总容量: 926 GB
   已使用: 864 GB (93%)
   可用空间: 3.9 GB

📁 最大的目录/文件:
   1. ~/Library (45 GB) - 系统库文件
   2. ~/Projects (30 GB) - 项目文件
   3. ~/Downloads (15 GB) - 下载文件

🗑️ 可清理项目建议:

   低风险 (建议清理):
   ┌─────────────────────────┬────────┬─────────────────┐
   │ 路径                    │ 大小   │ 说明            │
   ├─────────────────────────┼────────┼─────────────────┤
   │ ~/.Trash                │ 5 GB   │ 废纸篓          │
   │ ~/.npm                  │ 2 GB   │ npm 缓存        │
   └─────────────────────────┴────────┴─────────────────┘

📝 建议的清理命令 (请手动复制执行):

   # 清空废纸篓 (可释放约 5 GB)
   rm -rf ~/.Trash/*

   # 清理 npm 缓存 (可释放约 2 GB)
   npm cache clean --force

💡 预计可释放空间: 约 7 GB
```

### 安全保证

- **不自动删除**：所有清理命令仅供参考
- **风险标签**：每个清理项目都标注了风险等级
- **手动执行**：用户必须手动复制并执行清理命令
- **路径验证**：在提供建议前会验证路径是否存在

### 支持的系统

- macOS（完整支持）
- Linux（支持常见清理位置）

### 常见清理命令参考

#### macOS

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

# 清理 Xcode 派生数据（中等风险）
rm -rf ~/Library/Developer/Xcode/DerivedData/*

# 清理 Docker 未使用资源（中等风险）
docker system prune
```

#### Linux

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

## License

MIT License

## Author

wuyiqun

## Contributing

Issues and pull requests are welcome!

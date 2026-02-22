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
- **深度缓存分析**
- **详细分析 Library/Caches**

## 工作流程

### 1. 确定分析目标

首先确认用户想要分析的路径：
- 如果用户指定了路径，分析该路径
- 如果用户未指定，询问用户想分析哪个目录（默认可分析用户主目录）
- 如果用户要求深度分析，执行额外的缓存目录详细分析

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

### 5. 深度缓存分析（重点功能）

当发现 `~/Library/Caches` 占用空间较大时（>5GB），或用户要求深度分析时，执行以下详细分析：

#### 5.1 分析 ~/Library/Caches 子目录

```bash
# 列出 Caches 目录下所有子目录大小，按大小排序
du -sh ~/Library/Caches/*/ 2>/dev/null | sort -hr | head -30

# 查找缓存目录中的大文件
find ~/Library/Caches -type f -size +100M 2>/dev/null -exec ls -lh {} \; | awk '{print $5, $9}' | sort -hr | head -20
```

#### 5.2 常见缓存目录安全等级分类

根据缓存类型，将结果分为以下几类：

**🟢 低风险 - 可安全清理（自动重建）**

| 缓存目录 | 说明 | 清理命令 |
|----------|------|----------|
| `com.apple.Safari/Webkit` | Safari 浏览器缓存 | `rm -rf ~/Library/Caches/com.apple.Safari/WebKit` |
| `com.google.Chrome` | Chrome 浏览器缓存 | 在 Chrome 设置中清理 |
| `com.microsoft.edgemac` | Edge 浏览器缓存 | 在 Edge 设置中清理 |
| `com.mozilla.firefox` | Firefox 浏览器缓存 | 在 Firefox 设置中清理 |
| `com.apple.dt.Xcode` | Xcode 临时缓存 | `rm -rf ~/Library/Caches/com.apple.dt.Xcode` |
| `Homebrew` | Homebrew 下载缓存 | `brew cleanup --prune=all` |
| `pip` | Python pip 缓存 | `pip cache purge` |
| `Yarn` | Yarn 包缓存 | `yarn cache clean` |
| `node-gyp` | Node.js 编译缓存 | `rm -rf ~/Library/Caches/node-gyp` |
| `electron` | Electron 缓存 | `rm -rf ~/Library/Caches/electron` |
| `electron-builder` | Electron 构建缓存 | `rm -rf ~/Library/Caches/electron-builder` |
| `CocoaPods` | iOS 依赖缓存 | `pod cache clean --all` |
| `gradle` | Gradle 缓存 | `rm -rf ~/Library/Caches/gradle` |
| `miguelgrinberg.Flask` | Flask 缓存 | 可安全删除 |
| `com.apple.bird` | iCloud 同步缓存 | 系统自动管理 |

**🟡 中等风险 - 清理前请确认**

| 缓存目录 | 说明 | 注意事项 |
|----------|------|----------|
| `com.apple.Phojos` | 照片缓存 | 清理后可能需要重新加载照片 |
| `com.apple.music` | Apple Music 缓存 | 清理后可能需要重新下载 |
| `com.apple.podcasts` | 播客缓存 | 清理后需要重新下载 |
| `com.apple.spotlight` | Spotlight 索引缓存 | 系统会自动重建 |
| `com.apple.nsurlsessiond` | 系统 URL 会话缓存 | 系统自动管理 |
| `com.apple.geod` | 地图缓存 | 清理后地图加载变慢 |
| `com.apple.HelpData` | 帮助文档缓存 | 可清理，需要时重新下载 |
| `com.adobe.*` | Adobe 系列缓存 | 清理后可能需要重新登录 |
| `com.microsoft.*` | Microsoft 应用缓存 | 清理后可能需要重新登录 |
| `com.jetbrains.*` | JetBrains IDE 缓存 | 清理后 IDE 需要重建索引 |

**🔴 高风险 - 不建议清理**

| 缓存目录 | 说明 | 原因 |
|----------|------|------|
| `com.apple.loginservices` | 登录服务缓存 | 可能影响系统登录 |
| `com.apple.coreservices` | 核心服务缓存 | 系统关键组件 |
| `com.apple.apsd` | Apple 推送服务 | 系统通知服务 |
| `com.apple.security` | 安全相关缓存 | 系统安全组件 |

#### 5.3 识别无用/过期文件

执行以下检查识别可能无用的文件：

```bash
# 查找超过 30 天未访问的缓存文件
find ~/Library/Caches -type f -atime +30 -size +10M 2>/dev/null | head -20

# 查找超过 90 天未修改的缓存文件
find ~/Library/Caches -type f -mtime +90 -size +10M 2>/dev/null | head -20

# 查找已卸载应用的残留缓存（检查是否有对应应用）
ls ~/Library/Caches/ | while read dir; do
  app_name=$(echo "$dir" | sed 's/com\.//;s/\./ /g')
  # 检查应用是否还存在
  if ! mdfind "kMDItemKind == 'Application'" 2>/dev/null | grep -qi "$app_name"; then
    echo "可能已卸载应用的缓存: $dir"
  fi
done
```

#### 5.4 深度分析其他缓存位置

**分析 ~/Library/Developer:**
```bash
# Xcode 相关缓存分析
du -sh ~/Library/Developer/*/ 2>/dev/null | sort -hr

# 模拟器缓存（通常很大）
du -sh ~/Library/Developer/CoreSimulator 2>/dev/null

# Xcode 档案
du -sh ~/Library/Developer/Xcode/Archives 2>/dev/null

# 设备支持文件
du -sh ~/Library/Developer/Xcode/iOS\ DeviceSupport 2>/dev/null
```

**分析 ~/.npm 和 ~/.cache:**
```bash
# npm 缓存详情
du -sh ~/.npm/_cacache 2>/dev/null

# 各类开发工具缓存
du -sh ~/.cache/*/ 2>/dev/null | sort -hr
```

**分析 Docker:**
```bash
# Docker 空间使用情况
docker system df

# Docker 镜像列表
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# 未使用的 Docker 资源
docker system df -v
```

### 6. 生成分析报告

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

## 深度缓存分析
| 缓存目录 | 大小 | 风险等级 | 建议 |
|------|------|----------|------|
| ... | ... | ... | ... |

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
| iOS 模拟器 | ~/Library/Developer/CoreSimulator | iOS 模拟器数据 |
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

# 清理 iOS 模拟器（中等风险）
xcrun simctl delete unavailable

# 查看 Docker 空间使用情况
docker system df

# 清理 Docker 未使用资源（中等风险）
docker system prune
```

### 深度缓存清理命令

```bash
# 清理特定应用缓存（低风险）
rm -rf ~/Library/Caches/com.apple.dt.Xcode
rm -rf ~/Library/Caches/node-gyp
rm -rf ~/Library/Caches/electron
rm -rf ~/Library/Caches/electron-builder
rm -rf ~/Library/Caches/CocoaPods

# 清理 Gradle 缓存（低风险）
rm -rf ~/Library/Caches/gradle
rm -rf ~/.gradle/caches

# 清理 Go 模块缓存（低风险）
go clean -modcache

# 清理 Cargo 缓存（低风险，Rust）
rm -rf ~/.cargo/registry/cache
rm -rf ~/.cargo/registry/index

# 查找并清理大型缓存目录
du -sh ~/Library/Caches/*/ 2>/dev/null | sort -hr | head -10
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

🔍 深度缓存分析 (~/Library/Caches):

   🟢 低风险 (可安全清理):
   ┌─────────────────────────────────┬────────┬─────────────────────┐
   │ 缓存目录                        │ 大小   │ 说明                │
   ├─────────────────────────────────┼────────┼─────────────────────┤
   │ com.google.Chrome               │ 3.2 GB │ Chrome 浏览器缓存   │
   │ Homebrew                        │ 1.5 GB │ Homebrew 下载缓存   │
   │ node-gyp                        │ 800 MB │ Node 编译缓存       │
   │ electron                        │ 500 MB │ Electron 缓存       │
   └─────────────────────────────────┴────────┴─────────────────────┘

   🟡 中等风险 (请先确认):
   ┌─────────────────────────────────┬────────┬─────────────────────┐
   │ 缓存目录                        │ 大小   │ 说明                │
   ├─────────────────────────────────┼────────┼─────────────────────┤
   │ com.apple.Phojos                │ 2.1 GB │ 照片缓存            │
   │ com.jetbrains.intellij          │ 1.8 GB │ IntelliJ 缓存       │
   └─────────────────────────────────┴────────┴─────────────────────┘

🗑️ 可清理项目建议:

   低风险 (建议清理):
   ┌─────────────────────────────────┬────────┬─────────────────────┐
   │ 路径                            │ 大小   │ 说明                │
   ├─────────────────────────────────┼────────┼─────────────────────┤
   │ ~/.Trash                        │ 5 GB   │ 废纸篓              │
   │ ~/.npm                          │ 2 GB   │ npm 缓存            │
   │ ~/Downloads                     │ 15 GB  │ 下载文件（请确认）  │
   └─────────────────────────────────┴────────┴─────────────────────┘

📝 建议的清理命令 (请手动复制执行):

   # 清空废纸篓 (可释放约 5 GB)
   rm -rf ~/.Trash/*

   # 清理 npm 缓存 (可释放约 2 GB)
   npm cache clean --force

   # 清理 Chrome 缓存 (可释放约 3.2 GB)
   rm -rf ~/Library/Caches/com.google.Chrome

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
6. **深度分析**：当缓存目录较大时，自动展开详细分析

## 特殊目录说明

### macOS ~/Library/Caches 常见目录

| 目录名 | 应用/服务 | 安全等级 | 说明 |
|--------|-----------|----------|------|
| com.apple.Safari | Safari | 🟢 低 | 浏览器缓存，可清理 |
| com.google.Chrome | Chrome | 🟢 低 | 浏览器缓存，可清理 |
| com.microsoft.edgemac | Edge | 🟢 低 | 浏览器缓存，可清理 |
| com.apple.dt.Xcode | Xcode | 🟢 低 | 开发工具缓存 |
| Homebrew | Homebrew | 🟢 低 | 包管理器缓存 |
| com.apple.Phojos | 照片 | 🟡 中 | 照片缩略图缓存 |
| com.apple.music | Music | 🟡 中 | 音乐流媒体缓存 |
| com.jetbrains.* | JetBrains | 🟡 中 | IDE 缓存，清理后需重建索引 |
| com.adobe.* | Adobe | 🟡 中 | Adobe 应用缓存 |
| com.apple.coreservices | 系统 | 🔴 高 | 核心服务，不建议清理 |

### 开发相关目录

| 目录 | 说明 | 重建方法 |
|------|------|----------|
| node_modules | Node.js 项目依赖 | `npm install` |
| .venv / venv | Python 虚拟环境 | `python -m venv .venv` |
| __pycache__ | Python 字节码缓存 | 自动生成 |
| target (Maven) | Java 构建输出 | `mvn compile` |
| build (Gradle) | Java 构建输出 | `./gradlew build` |
| .gradle | Gradle 缓存 | 自动下载 |
| ~/go/pkg | Go 模块缓存 | `go mod download` |
| ~/.cargo | Cargo 缓存 | `cargo build` |

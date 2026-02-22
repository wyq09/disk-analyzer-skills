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

## 首次使用配置

### Webhook 推送配置（可选）

首次使用此技能时，会询问用户是否配置报告推送 webhook。用户可以选择：

1. **飞书机器人** - 配置飞书群机器人 webhook
2. **企业微信机器人** - 配置企业微信群机器人 webhook
3. **自定义 Webhook** - 配置自定义的 webhook 地址
4. **暂不配置** - 跳过配置，后续可通过自然语言设置

**配置引导话术：**

```
🔧 磁盘分析报告推送配置

是否需要将分析报告推送到即时通讯工具？可以选择：

1. 飞书机器人 - 输入飞书群机器人的 webhook 地址
2. 企业微信机器人 - 输入企业微信机器人的 webhook 地址
3. 自定义 Webhook - 输入自定义 webhook 地址
4. 暂不配置 - 之后再设置

请回复数字或直接说明你的选择。
```

**后续修改配置：**

用户可以随时通过以下自然语言修改配置：
- "把报告推送到飞书"
- "设置飞书 webhook"
- "配置企业微信推送"
- "修改 webhook 地址"
- "取消报告推送"

### 配置存储

配置信息存储在：`~/.disk-analyzer-config.json`

```json
{
  "webhook": {
    "type": "feishu | wecom | custom | none",
    "url": "https://...",
    "enabled": true
  }
}
```

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
- 深度分析某个目录
- 设置报告推送
- 配置 webhook

## 工作流程

### 0. 检查配置（首次使用）

```bash
# 检查配置文件是否存在
if [ ! -f ~/.disk-analyzer-config.json ]; then
  # 首次使用，引导配置
  echo "首次使用，请配置报告推送..."
fi
```

### 1. 确定分析目标

首先确认用户想要分析的路径：
- 如果用户指定了路径，分析该路径
- 如果用户未指定，询问用户想分析哪个目录（默认可分析用户主目录）
- **所有目录都支持深度分析**

### 2. 获取磁盘整体情况

**macOS / Linux:**
```bash
# 查看磁盘整体使用情况
df -h

# 查看指定目录所在分区的使用情况
df -h /path/to/directory
```

### 3. 基础目录大小分析

**macOS / Linux:**
```bash
# 分析指定目录下各子目录大小（按大小排序，显示前30个）
du -sh /path/to/directory/*/ 2>/dev/null | sort -hr | head -30

# 查找大文件（超过100MB）
find /path/to/directory -type f -size +100M 2>/dev/null | head -30

# 查找大文件并显示大小
find /path/to/directory -type f -size +100M -exec ls -lh {} \; 2>/dev/null | awk '{print $5, $9}' | sort -hr | head -30
```

### 4. 通用深度分析（适用于任何目录）

**所有目录都支持以下深度分析：**

#### 4.1 子目录逐层分析

```bash
# 深度分析指定目录的所有子目录（递归到第3层）
du -sh /path/to/directory/*/* 2>/dev/null | sort -hr | head -30

# 或者使用 find 进行更精细的分析
find /path/to/directory -maxdepth 3 -type d -exec du -sh {} \; 2>/dev/null | sort -hr | head -30
```

#### 4.2 文件类型分布分析

```bash
# 按文件扩展名统计大小
find /path/to/directory -type f -name "*.*" 2>/dev/null | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -20

# 按扩展名统计占用空间
find /path/to/directory -type f -name "*.*" -exec du -ch {} + 2>/dev/null | grep -E "\.[a-zA-Z0-9]+$" | awk '{sum[$NF]+=$1} END {for (ext in sum) print sum[ext], ext}' | sort -hr | head -20
```

#### 4.3 大文件定位

```bash
# 查找超大文件（>500MB）
find /path/to/directory -type f -size +500M -exec ls -lh {} \; 2>/dev/null | awk '{print $5, $9}' | sort -hr

# 查找超大文件（>1GB）
find /path/to/directory -type f -size +1G -exec ls -lh {} \; 2>/dev/null | awk '{print $5, $9}' | sort -hr
```

#### 4.4 过期/无用文件检测

```bash
# 查找超过30天未访问的文件
find /path/to/directory -type f -atime +30 -size +10M 2>/dev/null | head -30

# 查找超过90天未修改的文件
find /path/to/directory -type f -mtime +90 -size +10M 2>/dev/null | head -30

# 查找空文件和空目录
find /path/to/directory -empty 2>/dev/null | head -30

# 查找重复文件（通过文件大小初步判断）
find /path/to/directory -type f -printf '%s %p\n' 2>/dev/null | sort -n | uniq -D -w 10 | head -30
```

#### 4.5 隐藏文件分析

```bash
# 查找隐藏目录的大小
du -sh /path/to/directory/.* 2>/dev/null | sort -hr | head -20

# 查找隐藏文件
find /path/to/directory -name ".*" -type f -size +10M -exec ls -lh {} \; 2>/dev/null | awk '{print $5, $9}' | sort -hr | head -20
```

#### 4.6 符号链接分析

```bash
# 查找断开的符号链接
find /path/to/directory -type l -! -exec test -e {} \; -print 2>/dev/null | head -20

# 统计符号链接数量
find /path/to/directory -type l 2>/dev/null | wc -l
```

### 5. 特定目录深度分析

对于常见目录，提供更详细的分析建议：

#### 5.1 ~/Library/Caches 深度分析

**安全等级分类：**

**🟢 低风险 - 可安全清理（自动重建）**

| 缓存目录 | 说明 | 清理命令 |
|----------|------|----------|
| `com.apple.Safari/Webkit` | Safari 浏览器缓存 | `rm -rf ~/Library/Caches/com.apple.Safari/WebKit` |
| `com.google.Chrome` | Chrome 浏览器缓存 | 在 Chrome 设置中清理 |
| `com.microsoft.edgemac` | Edge 浏览器缓存 | 在 Edge 设置中清理 |
| `com.apple.dt.Xcode` | Xcode 临时缓存 | `rm -rf ~/Library/Caches/com.apple.dt.Xcode` |
| `Homebrew` | Homebrew 下载缓存 | `brew cleanup --prune=all` |
| `pip` | Python pip 缓存 | `pip cache purge` |
| `Yarn` | Yarn 包缓存 | `yarn cache clean` |
| `node-gyp` | Node.js 编译缓存 | `rm -rf ~/Library/Caches/node-gyp` |
| `electron` | Electron 缓存 | `rm -rf ~/Library/Caches/electron` |
| `electron-builder` | Electron 构建缓存 | `rm -rf ~/Library/Caches/electron-builder` |

**🟡 中等风险 - 清理前请确认**

| 缓存目录 | 说明 | 注意事项 |
|----------|------|----------|
| `com.apple.Phojos` | 照片缓存 | 清理后可能需要重新加载照片 |
| `com.apple.music` | Apple Music 缓存 | 清理后可能需要重新下载 |
| `com.jetbrains.*` | JetBrains IDE 缓存 | 清理后 IDE 需要重建索引 |
| `com.adobe.*` | Adobe 系列缓存 | 清理后可能需要重新登录 |

**🔴 高风险 - 不建议清理**

| 缓存目录 | 说明 | 原因 |
|----------|------|------|
| `com.apple.coreservices` | 核心服务缓存 | 系统关键组件 |
| `com.apple.security` | 安全相关缓存 | 系统安全组件 |

#### 5.2 ~/Library/Developer 深度分析

```bash
# Xcode 相关缓存分析
du -sh ~/Library/Developer/*/ 2>/dev/null | sort -hr

# iOS 模拟器（通常很大）
du -sh ~/Library/Developer/CoreSimulator 2>/dev/null

# Xcode 档案
du -sh ~/Library/Developer/Xcode/Archives 2>/dev/null

# 设备支持文件
du -sh ~/Library/Developer/Xcode/iOS\ DeviceSupport 2>/dev/null

# DerivedData 详细分析
du -sh ~/Library/Developer/Xcode/DerivedData/*/ 2>/dev/null | sort -hr | head -10
```

#### 5.3 开发工具目录深度分析

**Node.js 相关：**
```bash
# npm 全局包
du -sh $(npm root -g) 2>/dev/null
npm list -g --depth=0 2>/dev/null

# npm 缓存详情
du -sh ~/.npm/_cacache 2>/dev/null

# 各项目的 node_modules
find ~ -name "node_modules" -type d -prune -exec du -sh {} \; 2>/dev/null | sort -hr | head -10
```

**Python 相关：**
```bash
# pip 缓存
du -sh ~/Library/Caches/pip 2>/dev/null

# Python 虚拟环境
find ~ -name "venv" -o -name ".venv" -type d -prune -exec du -sh {} \; 2>/dev/null | sort -hr | head -10

# __pycache__ 目录
find ~ -name "__pycache__" -type d -prune -exec du -sh {} \; 2>/dev/null | sort -hr | head -10
```

**Docker 相关：**
```bash
# Docker 空间使用情况
docker system df 2>/dev/null

# Docker 镜像列表
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" 2>/dev/null

# Docker 卷
docker volume ls 2>/dev/null
```

**Go 相关：**
```bash
# Go 模块缓存
du -sh ~/go/pkg 2>/dev/null

# Go 构建缓存
du -sh ~/Library/Caches/go-build 2>/dev/null
```

#### 5.4 ~/Downloads 深度分析

```bash
# 按文件类型分析下载目录
find ~/Downloads -type f -name "*.*" 2>/dev/null | sed 's/.*\.//' | tr '[:upper:]' '[:lower:]' | sort | uniq -c | sort -rn | head -10

# 查找大型安装包
find ~/Downloads -type f \( -name "*.dmg" -o -name "*.pkg" -o -name "*.zip" -o -name "*.exe" \) -exec ls -lh {} \; 2>/dev/null | awk '{print $5, $9}' | sort -hr | head -10

# 查找旧文件（超过30天）
find ~/Downloads -type f -mtime +30 -exec ls -lh {} \; 2>/dev/null | awk '{print $6, $7, $8, $9}' | head -20
```

### 6. 生成分析报告

根据收集的信息，生成结构化的分析报告。

### 7. 推送报告（如已配置）

如果用户配置了 webhook，将报告推送到对应平台。

#### 7.1 飞书 Webhook

```bash
curl -X POST "${FEISHU_WEBHOOK}" \
  -H "Content-Type: application/json" \
  -d '{
    "msg_type": "interactive",
    "card": {
      "header": {
        "title": {"tag": "plain_text", "content": "📊 磁盘空间分析报告"},
        "template": "blue"
      },
      "elements": [
        {
          "tag": "div",
          "text": {"tag": "lark_md", "content": "**报告内容...**"}
        }
      ]
    }
  }'
```

#### 7.2 企业微信 Webhook

```bash
curl -X POST "${WECOM_WEBHOOK}" \
  -H "Content-Type: application/json" \
  -d '{
    "msgtype": "markdown",
    "markdown": {
      "content": "## 📊 磁盘空间分析报告\n\n报告内容..."
    }
  }'
```

#### 7.3 自定义 Webhook

```bash
curl -X POST "${CUSTOM_WEBHOOK}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "磁盘空间分析报告",
    "content": "报告内容...",
    "timestamp": "'$(date -Iseconds)'"
  }'
```

## Webhook 配置命令参考

### 读取配置

```bash
# 读取当前配置
cat ~/.disk-analyzer-config.json 2>/dev/null || echo "未配置"
```

### 保存配置

```bash
# 保存飞书配置
cat > ~/.disk-analyzer-config.json << 'EOF'
{
  "webhook": {
    "type": "feishu",
    "url": "https://open.feishu.cn/open-apis/bot/v2/hook/xxx",
    "enabled": true
  }
}
EOF

# 保存企业微信配置
cat > ~/.disk-analyzer-config.json << 'EOF'
{
  "webhook": {
    "type": "wecom",
    "url": "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxx",
    "enabled": true
  }
}
EOF

# 保存自定义配置
cat > ~/.disk-analyzer-config.json << 'EOF'
{
  "webhook": {
    "type": "custom",
    "url": "https://your-webhook-url",
    "enabled": true
  }
}
EOF

# 禁用推送
cat > ~/.disk-analyzer-config.json << 'EOF'
{
  "webhook": {
    "type": "none",
    "url": "",
    "enabled": false
  }
}
EOF
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

# 清理 Xcode 派生数据（中等风险）
rm -rf ~/Library/Developer/Xcode/DerivedData/*

# 清理 iOS 模拟器（中等风险）
xcrun simctl delete unavailable

# 清理 Go 模块缓存（低风险）
go clean -modcache

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
rm -rf ~/Library/Caches/gradle
rm -rf ~/.gradle/caches

# 清理 Cargo 缓存（低风险，Rust）
rm -rf ~/.cargo/registry/cache
rm -rf ~/.cargo/registry/index

# 清理所有 __pycache__
find ~ -name "__pycache__" -type d -exec rm -rf {} + 2>/dev/null
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

📁 分析目录: /Users/xxx/Projects
   总大小: 45 GB

🔍 深度分析结果:

   📂 子目录占用:
   ┌─────────────────────────┬────────┬─────────────────────┐
   │ 目录                    │ 大小   │ 占比                │
   ├─────────────────────────┼────────┼─────────────────────┤
   │ node_modules            │ 20 GB  │ 44%                 │
   │ .git                    │ 5 GB   │ 11%                 │
   │ dist                    │ 3 GB   │ 7%                  │
   │ build                   │ 2 GB   │ 4%                  │
   └─────────────────────────┴────────┴─────────────────────┘

   📄 文件类型分布:
   - .js: 8 GB (5000+ 文件)
   - .json: 3 GB (2000+ 文件)
   - .ts: 2 GB (1500+ 文件)

   📦 大文件 (>100MB):
   1. dist/bundle.js - 500 MB
   2. node_modules/... - 300 MB
   3. .git/objects/... - 200 MB

   ⏰ 过期文件 (>30天未访问):
   - 发现 15 个文件，共 2 GB

🗑️ 可清理项目建议:

   🟢 低风险 (建议清理):
   ┌─────────────────────────┬────────┬─────────────────────┐
   │ 路径                    │ 大小   │ 说明                │
   ├─────────────────────────┼────────┼─────────────────────┤
   │ node_modules (未使用)   │ 5 GB   │ 可通过 npm install  │
   │ dist (旧构建)           │ 3 GB   │ 可重新构建          │
   └─────────────────────────┴────────┴─────────────────────┘

📝 建议的清理命令 (请手动复制执行):

   # 清理未使用的 node_modules
   rm -rf /path/to/unused/project/node_modules

   # 清理旧构建产物
   rm -rf /path/to/project/dist

⚠️ 注意事项:
   - 以上命令仅供参考，请在执行前确认路径正确
   - 建议先备份重要数据
   - 如有疑问，请逐个确认后再执行

💡 预计可释放空间: 约 10 GB

📅 报告时间: 2026-02-22 15:30:00
```

## 注意事项

1. **绝对不自动删除文件**：只提供建议，所有删除操作由用户手动执行
2. **风险评估**：对每个可清理项目标注风险等级
3. **路径验证**：在给出建议前，先验证路径是否存在
4. **空间预估**：尽量提供可释放空间的预估值
5. **备份提醒**：提醒用户在清理前备份重要数据
6. **通用深度分析**：支持对任意目录进行深度分析
7. **配置灵活性**：支持随时修改 webhook 推送配置

## 特殊目录说明

### macOS ~/Library/Caches 常见目录

| 目录名 | 应用/服务 | 安全等级 | 说明 |
|--------|-----------|----------|------|
| com.apple.Safari | Safari | 🟢 低 | 浏览器缓存，可清理 |
| com.google.Chrome | Chrome | 🟢 低 | 浏览器缓存，可清理 |
| com.apple.dt.Xcode | Xcode | 🟢 低 | 开发工具缓存 |
| Homebrew | Homebrew | 🟢 低 | 包管理器缓存 |
| com.apple.Phojos | 照片 | 🟡 中 | 照片缩略图缓存 |
| com.jetbrains.* | JetBrains | 🟡 中 | IDE 缓存 |
| com.apple.coreservices | 系统 | 🔴 高 | 核心服务，不建议清理 |

### 开发相关目录

| 目录 | 说明 | 重建方法 |
|------|------|----------|
| node_modules | Node.js 项目依赖 | `npm install` |
| .venv / venv | Python 虚拟环境 | `python -m venv .venv` |
| __pycache__ | Python 字节码缓存 | 自动生成 |
| target (Maven) | Java 构建输出 | `mvn compile` |
| build (Gradle) | Java 构建输出 | `./gradlew build` |
| dist | 前端构建输出 | `npm run build` |
| .git | Git 仓库数据 | 无法重建（版本历史） |

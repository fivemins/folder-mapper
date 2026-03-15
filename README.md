# folder-mapper

<div align="center">
  <button onclick="document.getElementById('en').style.display='block';document.getElementById('zh').style.display='none'">English</button>
  <button onclick="document.getElementById('zh').style.display='block';document.getElementById('en').style.display='none'">中文</button>
</div>

---

<div id="en">

## folder-mapper

Temporary folder mapping tool with security features for AI agents.

### Features

- 🔗 **Mount external folders** into workspace via symlinks
- 🔒 **Read-only mode** by default
- 🛡️ **System directory protection**
- ⚙️ **User-configurable** forbidden/sensitive paths
- ⚠️ **Confirmation prompts** for sensitive operations

### Installation

```bash
npx skills add fivemins/folder-mapper
```

### Usage

```bash
# Map folder
python3 scripts/map_folder.py mount "/path"

# List mappings
python3 scripts/map_folder.py list

# Unmount
python3 scripts/map_folder.py unmount <name>

# Add forbidden path
python3 scripts/map_folder.py forbid "/path"

# Add sensitive path
python3 scripts/map_folder.py sensitive "/path"
```

### Security

Default forbidden: `/`, `/bin`, `/etc`, `/proc`, etc.

</div>

<div id="zh" style="display:none">

## folder-mapper

临时文件夹映射工具，为 AI Agent 提供安全访问外部目录的能力。

### 特性

- 🔗 将外部文件夹映射到工作空间
- 🔒 默认只读模式
- 🛡️ 系统目录保护
- ⚙️ 用户可配置禁止/敏感目录
- ⚠️ 敏感操作二次确认

### 安装

```bash
npx skills add fivemins/folder-mapper
```

### 使用方法

```bash
# 映射文件夹
python3 scripts/map_folder.py mount "/path"

# 查看映射
python3 scripts/map_folder.py list

# 取消映射
python3 scripts/map_folder.py unmount <name>

# 添加禁止目录
python3 scripts/map_folder.py forbid "/path"

# 添加敏感目录
python3 scripts/map_folder.py sensitive "/path"
```

### 安全机制

默认禁止: `/`, `/bin`, `/etc`, `/proc` 等

</div>

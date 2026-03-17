# folder-mapper 功能测试方案

## 测试环境准备

```bash
# 进入技能目录
cd ~/.openclaw/workspace/skills/folder-mapper

# 创建测试目录
mkdir -p /tmp/test_folder_mapper
mkdir -p /tmp/test_protected
mkdir -p /tmp/test_sensitive
mkdir -p /tmp/test_normal
echo "test content" > /tmp/test_normal/file.txt
```

---

## 一、配置管理测试

### 1.1 添加/移除禁止目录

```bash
# 测试添加禁止目录
python3 scripts/map_folder.py forbid "/tmp/test_protected"
# 预期: 已添加禁止目录: /tmp/test_protected

# 重复添加（应该提示已存在）
python3 scripts/map_folder.py forbid "/tmp/test_protected"
# 预期: 目录已在黑名单中

# 查看配置
python3 scripts/map_folder.py config
# 预期: 显示用户禁止目录列表包含 /tmp/test_protected

# 移除禁止目录
python3 scripts/map_folder.py allow "/tmp/test_protected"
# 预期: 已移除禁止目录: /tmp/test_protected

# 重复移除（应该提示不存在）
python3 scripts/map_folder.py allow "/tmp/test_protected"
# 预期: 目录不在黑名单中
```

### 1.2 添加/移除敏感目录

```bash
# 测试添加敏感目录
python3 scripts/map_folder.py sensitive "/tmp/test_sensitive"
# 预期: 已添加敏感目录: /tmp/test_sensitive

# 重复添加
python3 scripts/map_folder.py sensitive "/tmp/test_sensitive"
# 预期: 目录已在敏感列表中

# 移除敏感目录
python3 scripts/map_folder.py desensitive "/tmp/test_sensitive"
# 预期: 已移除敏感目录: /tmp/test_sensitive

# 重复移除
python3 scripts/map_folder.py desensitive "/tmp/test_sensitive"
# 预期: 目录不在敏感列表中
```

### 1.3 查看配置

```bash
python3 scripts/map_folder.py config
# 预期输出:
# 📋 当前配置:
# ----------------------
# 系统默认禁止目录 (39): /, /bin, /boot, /dev...
# 用户禁止目录 (0):
#   (无)
# 敏感目录 (0):
#   (无)
# ----------------------
```

---

## 二、映射功能测试

### 2.1 正常映射

```bash
# 映射普通目录
python3 scripts/map_folder.py mount "/tmp/test_normal"
# 预期:
# ✅ 已映射到 mnt/test_normal (安全映射模式)
# ⚠️ 警告：此为符号链接，删除/修改会直接影响原文件！

# 查看映射
python3 scripts/map_folder.py list
# 预期显示: test_normal -> /tmp/test_normal

# 验证符号链接创建
ls -la ~/.openclaw/workspace/mnt/
# 预期: 存在指向 /tmp/test_normal 的符号链接

# 验证可访问
cat ~/.openclaw/workspace/mnt/test_normal/file.txt
# 预期: test content
```

### 2.2 多次映射（同名处理）

```bash
# 再次映射同一目录
python3 scripts/map_folder.py mount "/tmp/test_normal"
# 预期: 自动命名为 test_normal_1

# 验证
python3 scripts/map_folder.py list
# 预期: 显示 test_normal, test_normal_1
```

### 2.3 取消映射

```bash
# 取消指定映射
python3 scripts/map_folder.py unmount test_normal
# 预期: ✅ 已解除映射: test_normal

# 验证
python3 scripts/map_folder.py list
# 预期: 不包含 test_normal

# 取消不存在的映射
python3 scripts/map_folder.py unmount not_exist
# 预期: error: 映射不存在: not_exist
```

### 2.4 清理所有映射

```bash
# 先创建多个映射
python3 scripts/map_folder.py mount "/tmp/test_normal"
python3 scripts/map_folder.py mount "/tmp/test_protected"

# 清理
python3 scripts/map_folder.py clean
# 预期: 已清理所有映射

# 验证
python3 scripts/map_folder.py list
# 预期: 当前映射 (0 个):
```

---

## 三、安全机制测试

### 3.1 系统目录保护（Linux）

```bash
# 尝试映射根目录
python3 scripts/map_folder.py mount "/"
# 预期: error: 禁止映射系统目录: /

# 尝试映射 /etc
python3 scripts/map_folder.py mount "/etc"
# 预期: error: 禁止映射系统目录: /etc

# 尝试映射 /proc
python3 scripts/map_folder.py mount "/proc"
# 预期: error: 禁止映射系统目录: /proc

# 尝试映射 /usr
python3 scripts/map_folder.py mount "/usr"
# 预期: error: 禁止映射系统目录: /usr
```

### 3.2 盘符挂载点保护

```bash
# 尝试映射 /mnt/a（假设存在）
python3 scripts/map_folder.py mount "/mnt/a"
# 预期: error: 禁止映射系统目录: /mnt/a

# 遍历所有盘符
for letter in {a..z}; do
  result=$(python3 scripts/map_folder.py mount "/mnt/$letter" 2>&1)
  if [[ "$result" != *"error"* ]]; then
    echo "FAIL: /mnt/$letter should be blocked"
  fi
done
# 预期: 所有 /mnt/[a-z] 都被阻止
```

### 3.3 用户禁止目录

```bash
# 添加禁止目录
python3 scripts/map_folder.py forbid "/tmp/test_protected"

# 尝试映射
python3 scripts/map_folder.py mount "/tmp/test_protected"
# 预期: error: 用户禁止映射: /tmp/test_protected

# 移除后重新映射
python3 scripts/map_folder.py allow "/tmp/test_protected"
python3 scripts/map_folder.py mount "/tmp/test_protected"
# 预期: ✅ 已映射到 mnt/test_protected
```

### 3.4 敏感目录检测

```bash
# 添加敏感目录
python3 scripts/map_folder.py sensitive "/tmp/test_sensitive"

# 映射
python3 scripts/map_folder.py mount "/tmp/test_sensitive"
# 预期输出包含: ⚠️ 警告: 该目录需要二次确认！

# 查看配置确认敏感标记
python3 scripts/map_folder.py config
# 预期: 敏感目录列表包含 /tmp/test_sensitive

# 清理
python3 scripts/map_folder.py unmount test_sensitive
python3 scripts/map_folder.py desensitive "/tmp/test_sensitive"
```

### 3.5 不存在的目录

```bash
python3 scripts/map_folder.py mount "/tmp/not_exist_folder"
# 预期: error: 文件夹不存在: /tmp/not_exist_folder
```

### 3.6 非目录文件

```bash
# 创建一个文件
echo "test" > /tmp/test_file.txt

python3 scripts/map_folder.py mount "/tmp/test_file.txt"
# 预期: error: 不是有效文件夹: /tmp/test_file.txt

rm /tmp/test_file.txt
```

---

## 四、Windows 路径测试（模拟）

### 4.1 Windows 盘符根目录检测

```python
# 测试正则匹配（可通过临时修改脚本打印测试）
# WINDOWS_DRIVE_ROOT_RE 应匹配: C:\, D:\
# WINDOWS_DRIVE_PATH_RE 应匹配: C:\path\to\folder

# 预期行为:
# - 映射 C:\ → 被阻止
# - 映射 D:\ → 被阻止
# - 映射 C:\Users → 允许（除非是盘符根目录）
```

---

## 五、错误处理测试

### 5.1 损坏的配置文件

```bash
# 手动破坏配置 JSON
echo "invalid json" > ~/.openclaw/workspace/folder_mapper_config.json

# 尝试加载配置（程序应能恢复）
python3 scripts/map_folder.py config
# 预期: 正常显示配置（自动恢复为空配置）

# 验证配置文件已修复
cat ~/.openclaw/workspace/folder_mapper_config.json
# 预期: 有效的 JSON 格式
```

### 5.2 损坏的映射文件

```bash
# 手动破坏映射 JSON
echo "broken" > ~/.openclaw/workspace/folder_mapping.json

# 尝试查看映射
python3 scripts/map_folder.py list
# 预期: 当前映射 (0 个):（自动恢复）

# 创建映射后再破坏
python3 scripts/map_folder.py mount "/tmp/test_normal"
echo "{}" > ~/.openclaw/workspace/folder_mapping.json
python3 scripts/map_folder.py list
# 预期: 当前映射 (0 个):（自动清理失效映射）
```

---

## 六、边界条件测试

### 6.1 符号链接目标不存在

```bash
# 映射后删除原目录
python3 scripts/map_folder.py mount "/tmp/test_normal"
rm -rf /tmp/test_normal

# 查看映射（应自动清理失效映射）
python3 scripts/map_folder.py list
# 预期: 当前映射 (0 个):
```

### 6.2 路径展开

```bash
# 使用 ~ 展开
python3 scripts/map_folder.py mount ~/test_folder
# 预期: 正确解析为 /home/xxx/test_folder
```

### 6.3 相对路径

```bash
# 相对路径应被解析为绝对路径
cd /tmp
python3 ~/.openclaw/workspace/skills/folder-mapper/scripts/map_folder.py mount test_normal
# 预期: 正确解析为 /tmp/test_normal
```

---

## 七、自动化测试脚本

```bash
#!/bin/bash
# test_folder_mapper.sh

SCRIPT="~/.openclaw/workspace/skills/folder-mapper/scripts/map_folder.py"
TEST_DIR="/tmp/test_folder_mapper"

# 颜色
GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m'

pass() { echo -e "${GREEN}✓ PASS${NC}: $1"; }
fail() { echo -e "${RED}✗ FAIL${NC}: $1"; }

# 初始化
mkdir -p "$TEST_DIR"
python3 $SCRIPT clean > /dev/null 2>&1

echo "=== 测试开始 ==="

# 测试1: 映射正常目录
python3 $SCRIPT mount "$TEST_DIR" > /dev/null 2>&1
if python3 $SCRIPT list | grep -q "$TEST_DIR"; then
    pass "映射正常目录"
else
    fail "映射正常目录"
fi

# 测试2: 系统目录被阻止
python3 $SCRIPT mount "/" 2>&1 | grep -q "error"
if [ $? -eq 0 ]; then
    pass "根目录被阻止"
else
    fail "根目录被阻止"
fi

# 测试3: 禁止目录
python3 $SCRIPT forbid "$TEST_DIR" > /dev/null 2>&1
python3 $SCRIPT mount "$TEST_DIR" 2>&1 | grep -q "error"
if [ $? -eq 0 ]; then
    pass "禁止目录生效"
else
    fail "禁止目录生效"
fi

# 清理
python3 $SCRIPT allow "$TEST_DIR" > /dev/null 2>&1
python3 $SCRIPT clean > /dev/null 2>&1
rm -rf "$TEST_DIR"

echo "=== 测试完成 ==="
```

---

## 八、测试检查表

| 类别 | 测试项 | 状态 |
|------|--------|------|
| **配置管理** | 添加禁止目录 | ⬜ |
| | 重复添加禁止目录 | ⬜ |
 | 移除禁止目录 | ⬜ |
 | 添加敏感目录 | ⬜ |
 | 移除敏感目录 | ⬜ |
 | 查看配置 | ⬜ |
| **映射功能** | 正常映射 | ⬜ |
 | 查看映射列表 | ⬜ |
 | 同名自动编号 | ⬜ |
 | 取消映射 | ⬜ |
 | 清理所有映射 | ⬜ |
| **安全机制** | Linux 系统目录阻止 | ⬜ |
 | /mnt 盘符阻止 | ⬜ |
 | 用户禁止目录 | ⬜ |
 | 敏感目录警告 | ⬜ |
 | 不存在目录错误 | ⬜ |
 | 非目录文件错误 | ⬜ |
| **错误处理** | 损坏的配置 JSON | ✅ 已修复 |
 | 损坏的映射 JSON | ✅ 自动恢复 | 
 | 失效符号链接清理 | ✅ 已修复 |
| **边界** | ~ 路径展开 | ⬜ |
 | 相对路径解析 | ⬜ |

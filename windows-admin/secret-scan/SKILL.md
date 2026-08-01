---
name: secret-scan
description: 发布/提交前扫描代码中的敏感信息：token、密码、个人路径、邮箱、机器名、内网 IP
---

# 敏感信息扫描

推送代码到公开仓库前，扫描所有将上传的文件。

## 扫描模式

| 模式 | 匹配 |
|---|---|
| `ghp_[A-Za-z0-9]{10,}` | GitHub token |
| `password\s*=\s*["\'][^"\']{6,}` | 明文密码 |
| `token\s*=\s*["\'][^"\']{10,}` | 明文 token 赋值 |
| 邮箱正则 | 个人邮箱 |
| `内网IP段` | 内网 IP（用占位符，如 10\.0\.0\..*） |
| 机器名/用户名 | 项目特定的标识 |

## 扫描脚本（git 跟踪文件）

```python
import os, re, subprocess
files = subprocess.run(['git', 'ls-files'], capture_output=True,
                       text=True).stdout.splitlines()
for f in files:
    c = open(f, encoding='utf-8', errors='replace').read()
    for pat, label in PATTERNS:
        if re.search(pat, c):
            print('HIT [%s] %s' % (label, f))
```

## 排除误报

- `api.github.com` 端点字符串（合法）
- 代码里故意写的显示占位符 `***`（如 UI 掩码）
- 测试用的假 token（建议用明确假值如 `ghp_TEST_...`）

## 配套措施

- `.gitignore` 排除凭据文件（openclaw.json 等）和构建产物
- `git ls-files` 确认跟踪清单（比扫内容更彻底——没跟踪的文件根本不会上传）
- 聊天中出现过的真实 token，提醒用户轮换

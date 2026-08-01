---
name: ssh-over-443
description: 国内网络访问 GitHub 的绕行方案：SSH over 443（ssh.github.com）与 HTTPS 代理
---

# GitHub 国内网络绕行（SSH over 443）

国内网络访问 GitHub 常见问题：ping 通（TCP 通）但 HTTPS 被掐（TLS 握手被干扰，curl HTTP 000）。

## 诊断方法

```powershell
# TCP 通但 HTTPS 不通 = TLS 被干扰
Test-NetConnection github.com -Port 443        # TCP True
curl.exe -s -o NUL -w "%{http_code}" https://github.com  # HTTP 000 = 被掐
```

## 方案 A：SSH over 443（最可靠，官方为被墙网络设计）

```ini
# ~/.ssh/config
Host github.com
    HostName ssh.github.com
    Port 443
    User git
    IdentityFile ~/.ssh/id_ed25519
```

```powershell
# 生成 key 并添加公钥到 GitHub（Settings → SSH and GPG keys）
ssh-keygen -t ed25519 -C "your@email.com" -f "$env:USERPROFILE\.ssh\id_ed25519" -N '""'

# 验证
ssh -T git@github.com
# → Hi xxx! You've successfully authenticated
```

之后所有 `git clone/push/pull` 走 ssh 协议（`git@github.com:USER/REPO.git`）即可穿透。

## 方案 B：HTTPS 代理

```powershell
# 工具内设置（或环境变量）
$env:https_proxy = "http://127.0.0.1:7890"
$env:http_proxy = "http://127.0.0.1:7890"
```

## 注意

- api.github.com 和 github.com 可能一个通一个不通，交替使用
- 匿名 API 有 IP 限流，用带 token 的认证请求
- SSH key 添加需要 `write:public_key` 权限（或网页手动加）

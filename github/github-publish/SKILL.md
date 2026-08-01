---
name: github-publish
description: GitHub 访问与开源发布一条龙：REST API 客户端、SSH over 443 网络绕行、安全红线扫描、git 发布与 Release
---

# GitHub 访问与发布一条龙

调 GitHub API、绕过国内网络、安全发布开源项目。

## 1. REST API 客户端模式

```python
def api_request(method, path, token, params=None, data=None, timeout=30, retries=2):
    url = "https://api.github.com" + path
    if params:
        url += "?" + urlencode(params)
    req = urllib.request.Request(url, method=method)
    req.add_header("Accept", "application/vnd.github+json")
    req.add_header("X-GitHub-Api-Version", "2022-11-28")
    req.add_header("User-Agent", "MyApp/1.0")
    if token:
        req.add_header("Authorization", "Bearer %s" % token)
    # ... urlopen；网络类错误重试2次；HTTPError 提取 message
```

关键设计：
- **自动重试**：网络类错误重试（连接/超时/SSL），认证类(4xx)不重试
- **友好错误分类**：网络→"连接失败/代理问题"；401→令牌无效；403→权限不足或限流；404→资源不存在
- **代理支持**：`os.environ["https_proxy"]="http://127.0.0.1:7890"`
- **Token 认证**：`Authorization: Bearer <token>`

常用端点：`/user`、`/rate_limit`、`/user/repos`、`/repos/{o}/{r}/issues`、
`PUT .../pulls/{n}/merge`、`POST .../releases`、上传资产走 `uploads.github.com`、
Actions 日志返回 zip 需解压。
注意：失败时错误是 str 非 dict，解析前先判断类型。

## 2. 国内网络绕行（SSH over 443）

诊断：ping 通但 curl HTTPS 超时（HTTP 000）= TLS 被干扰。

`~/.ssh/config`：
```ini
Host github.com
    HostName ssh.github.com
    Port 443
    User git
    IdentityFile ~/.ssh/id_ed25519
```
验证：`ssh -T git@github.com` → "Hi xxx! You've successfully authenticated"

之后 git 用 `git@github.com:USER/REPO.git` 即可穿透。备选：HTTPS 代理环境变量。

## 3. 安全红线扫描（发布前必做）

扫描待上传文件：token（`ghp_...`）、密码、邮箱、机器名、内网 IP、个人路径。
```python
import re, subprocess
for f in subprocess.run(['git','ls-files'],capture_output=True,text=True).stdout.splitlines():
    if re.search(r'ghp_[A-Za-z0-9]{10,}', open(f,encoding='utf-8',errors='replace').read()):
        print('HIT', f)
```
配套：`.gitignore` 排除 build/dist/exe/spec、开发脚本、测试、备份；`git ls-files` 确认清单。

## 4. 发布流程

```powershell
git init && git add -A && git commit -m "feat: v1.0.0"
git branch -M main
git remote add origin git@github.com:USER/REPO.git
git push -u origin main
git tag v1.0.0 && git push --tags   # 然后网页建 Release 传安装包
```
附带：README / LICENSE（MIT）/ CHANGELOG / 开发日志。

## 检查清单
- [ ] 敏感扫描零命中，git ls-files 干净
- [ ] SSH 认证成功
- [ ] 仓库公开/私有设置正确

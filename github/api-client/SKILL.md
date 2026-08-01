---
name: api-client
description: GitHub REST API 调用模式：带重试、友好错误分类、代理支持、Token 认证
---

# GitHub REST API 客户端模式

封装 GitHub REST API 调用，供桌面工具使用。

## 基础调用

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
    # ... urlopen，网络类错误重试，HTTPError 提取 message
```

## 关键设计

1. **自动重试**：网络类错误（连接/超时/SSL）重试 2 次，带递增间隔；认证类(4xx)不重试
2. **友好错误分类**：
   - 网络错误 → "网络连接失败，可能被墙/代理问题"
   - 401 → "令牌无效或过期"
   - 403 → "权限不足 或 速率超限"
   - 404 → "资源不存在或无权访问"
3. **代理支持**：`os.environ["https_proxy"] = "http://127.0.0.1:7890"`
4. **Token 认证**：`Authorization: Bearer <token>`（不是 Basic）

## 常用端点

| 功能 | 端点 |
|---|---|
| 用户信息 | `GET /user` |
| 速率余量 | `GET /rate_limit` |
| 我的仓库 | `GET /user/repos?per_page=100` |
| 仓库详情 | `GET /repos/{owner}/{repo}` |
| Issues | `GET/POST /repos/{o}/{r}/issues` |
| PR 合并 | `PUT /repos/{o}/{r}/pulls/{n}/merge` |
| Releases | `GET/POST /repos/{o}/{r}/releases` |
| 上传资产 | `POST https://uploads.github.com/.../assets?name=...`（Content-Type 按文件类型） |
| Actions 运行 | `GET /repos/{o}/{r}/actions/runs` |
| Actions 日志 | `GET .../actions/runs/{id}/logs`（返回 zip，需解压） |

## 注意

- 公开接口匿名调用有 IP 限流（403），认证请求配额高得多
- 失败时错误消息是 str 而非 dict，解析前先判断类型

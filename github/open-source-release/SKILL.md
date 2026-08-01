---
name: open-source-release
description: 项目开源发布到 GitHub 的完整流程：安全红线扫描、git 初始化、SSH 推送、tag 与 Release
---

# GitHub 开源发布流程

把项目安全地发布到 GitHub 开源。

## 安全红线（必须先做）

发布前扫描所有待上传文件，排除：
- 个人路径/用户名/机器名/内网 IP（如本机用户名、`C:\Users\xxx\`、内网地址段）
- 邮箱、token、密码
- 凭据文件（openclaw.json、exec-approvals.json 等）

```python
# 扫描示例（正则列表）
patterns = [r'ghp_[A-Za-z0-9]{10,}', r'用户邮箱', r'内网IP段',
            r'password\s*=\s*["\'][^"\']{6,}']
```

`.gitignore` 必须排除：
- 构建产物：`build/ dist/ *.exe *.spec`
- 开发脚本：`deploy*.ps1 diag.ps1 test_*.py`
- 备份/日志：`*.bak *.old *.log`

```powershell
git ls-files   # 确认实际跟踪的文件清单干净
```

## 发布步骤

```powershell
git init
git add -A
git commit -m "feat: v1.0.0"
git branch -M main
git remote add origin git@github.com:USER/REPO.git   # SSH（被墙网络也能推）
git push -u origin main
```

- 建仓：GitHub 网页 或 `POST /user/repos`（`private: false` 公开）
- 敏感扫描零命中后再推送

## Releases 发版

```powershell
git tag v1.0.0
git push --tags
```

网页 → Releases → Draft → 选 tag → 写说明 → 上传安装包 → Publish

## 附带文件

- `README.md`：简介/功能/安装/构建/FAQ + 截图
- `LICENSE`：MIT（推荐，最宽松）
- `CHANGELOG.md`：版本变更记录
- 开发日志：架构/踩坑/构建流程（二次开发必读）

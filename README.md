# Agent Skills 仓库

本仓库存放 OpenClaw Agent 可复用的技能（SKILL.md 格式），按分类组织。

## 分类结构

```
skills/
├── windows-dev/          # Windows 桌面工具开发
│   ├── tkinter-dark-ui/        # tkinter 暗色单窗口 UI 模式
│   ├── pyinstaller-packaging/  # PyInstaller 打包（含环境变量坑）
│   ├── cred-manager/           # Windows 凭据管理器安全存储
│   └── inno-setup-installer/   # Inno Setup 安装包制作
├── github/               # GitHub 开发与发布
│   ├── api-client/             # GitHub REST API 调用模式（重试/错误分类）
│   ├── ssh-over-443/           # 国内网络绕行（SSH over 443 + 代理）
│   └── open-source-release/    # 开源发布流程（安全扫描/tag/Release）
├── windows-admin/        # Windows 系统管理
│   ├── junction-migrate/       # 数据目录 junction 迁移
│   └── secret-scan/            # 发布前敏感信息扫描
└── workflow/             # 开发流程
    └── project-dev-log/        # 项目开发日志模板（二次开发参考）
```

## 使用方式

每个 skill 是独立目录，含 `SKILL.md`（YAML frontmatter + 指令）。
复制到 agent 的 skills 根目录即可加载：

```bash
# 本地加载示例（把某分类下的 skill 放进工作区）
mkdir -p ~/.openclaw/workspace/skills/
cp -r skills/windows-dev/pyinstaller-packaging ~/.openclaw/workspace/skills/
```

## 索引

| Skill | 分类 | 一句话说明 |
|---|---|---|
| tkinter-dark-ui | windows-dev | tkinter 暗色主题 + 单窗口多页面（不弹窗） |
| pyinstaller-packaging | windows-dev | PyInstaller 打包，含 TCL_LIBRARY 污染坑 |
| cred-manager | windows-dev | Windows 凭据管理器存 token（ctypes） |
| inno-setup-installer | windows-dev | Inno Setup 安装包（自定义目录/卸载） |
| api-client | github | GitHub REST API 调用（重试/友好错误） |
| ssh-over-443 | github | 国内访问 GitHub 绕行方案 |
| open-source-release | github | 开源发布流程（安全红线/tag/Release） |
| junction-migrate | windows-admin | Windows 数据目录 junction 迁移 |
| secret-scan | windows-admin | 发布前敏感信息扫描 |
| project-dev-log | workflow | 项目开发日志模板 |

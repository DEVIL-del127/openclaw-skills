# Agent Skills 仓库

OpenClaw Agent 可复用的技能库（SKILL.md 格式）。按核心内容合并为 4 个技能。

## 技能索引

| Skill | 分类 | 一句话说明 |
|---|---|---|
| [windows-desktop-dev](windows-dev/windows-desktop-dev/SKILL.md) | 桌面开发 | tkinter 暗色单窗口 UI + PyInstaller 打包 + 凭据管理器 + Inno Setup 一条龙 |
| [github-publish](github/github-publish/SKILL.md) | GitHub | REST API 客户端 + SSH over 443 网络绕行 + 安全扫描 + 开源发布一条龙 |
| [windows-data-migrate](windows-data-migrate/SKILL.md) | 系统管理 | Windows 数据目录 junction 迁移（应用无感知） |
| [project-dev-log](project-dev-log/SKILL.md) | 流程 | 项目开发日志模板（二次开发参考） |

## 结构

```
openclaw-skills/
├── windows-dev/windows-desktop-dev/SKILL.md   # 桌面工具开发一条龙
├── github/github-publish/SKILL.md             # GitHub 访问与发布一条龙
├── windows-data-migrate/SKILL.md              # 数据目录迁移
└── project-dev-log/SKILL.md                   # 开发日志模板
```

## 使用方式

每个 skill 是独立目录，含 `SKILL.md`（YAML frontmatter + 指令）。
复制到 agent 的 skills 根目录即可加载：

```bash
# 示例：加载"桌面工具开发"技能
cp -r windows-dev/windows-desktop-dev ~/.openclaw/workspace/skills/
```

验证加载：`openclaw skills list`

## 来源

2026-08-01 开发 OpenClawConsole / GitHubConsole 两个桌面工具实战沉淀。

---
name: pyinstaller-packaging
description: PyInstaller 打包 tkinter 桌面工具的完整流程，含 TCL_LIBRARY 环境变量污染坑与体积验证
---

# PyInstaller 打包

用 PyInstaller 把 Python/tkinter 程序打包成单文件 exe。

## 标准命令

```powershell
# ⚠️ 打包前必须清环境变量（最高频坑）
$env:TCL_LIBRARY=''
$env:TK_LIBRARY=''

pyinstaller --noconfirm --onefile --noconsole --name MyApp `
  --distpath dist --workpath build --specpath . main.py
```

## 坑 1：exe 缩水 + 自检崩溃 = 环境变量污染

- **症状**：exe 体积异常变小（如 11.8MB→8.7MB），`--check` 无输出 RC=-1，GUI 起不来
- **根因**：打包前环境变量被之前运行的 onefile exe 污染（`TCL_LIBRARY`/`TK_LIBRARY` 指向已删除的 `_MEIxxxx` 临时目录），tkinter 的 tcl/tk 数据收集不全
- **解决**：打包前清环境变量（见上）
- **验证**：打包后检查 exe 体积（tkinter 应用 11MB+ 为正常；缩水=打包失败）

## 坑 2：noconsole 打包后 stdout 不可见

- windowed exe 的 `print` 捕获不到（`--check` 无输出），但 cmd 下 RC=0 正常退出
- **这是正常行为**，不是 bug
- 验证方式：启动 GUI 看进程是否存活；或只看退出码

## 自检模式

```python
def main():
    if "--check" in sys.argv:
        sys.exit(core.selfcheck())   # 返回 0=OK
```

## 打包后验证流程

1. 检查 exe 体积（11MB+）
2. 运行 `MyApp.exe --check`（看退出码）
3. 启动 GUI，确认进程存活、无崩溃弹窗
4. 部署到目标位置（桌面等）

---
name: windows-desktop-dev
description: Windows 桌面工具开发一条龙：tkinter 暗色单窗口 UI、PyInstaller 打包、凭据管理器存储、Inno Setup 安装包
---

# Windows 桌面工具开发一条龙

用 Python tkinter 做桌面工具：UI → 打包 → 安全存储 → 安装包。

## 1. tkinter 暗色单窗口 UI

### 暗色配色
全部用 tk 组件自定义配色（**ttk 不走 `-background` 属性**，会报错）：

```python
BG="#0e1116"; CARD="#161a21"; CARD2="#1b202a"; BORDER="#232936"
TEXT="#e6e9f0"; MUTED="#7d8698"; GREEN="#3ddc97"; RED="#ff6b6b"
BLUE="#5b8cff"; ORANGE="#ffa94d"
```

### 单窗口多页面（用户偏好：不弹窗）
左侧导航 + 右侧内容区，多个 Frame 堆叠切换：

```python
def show_tab(self, key):
    for k, p in self._pages.items():
        p["page"].pack_forget()
    self._pages[key]["page"].pack(fill="both", expand=True)
```

列表页统一结构：`{"page","tree","count","btns","detail","dt_title","dt_meta","dt_body","form","f_title","f_bar","f_submit","f_entry_ref","f_body_ref"}`
- 选中表格行 → 内嵌详情面板（不弹窗）
- 新建 → 内嵌表单（不弹窗）
- 表格：`ttk.Treeview` + `style.theme_use("clam")` 配暗色

### 事件日志分级着色
```python
def log(self, msg, level="info"):
    color = {"info": EV_FG, "ok": GREEN, "err": RED, "warn": ORANGE}
    self.ev_text.tag_configure("lvl_%s" % level, foreground=color[level])
    self.ev_text.insert("end", "%s\n" % msg, ("lvl_%s" % level,))
```

### 线程安全
- 网络/IO 放后台线程，UI 更新用 `self.root.after(0, ...)`
- 后台回调 try/except 包住 after（窗口可能已关闭）

## 2. PyInstaller 打包（最高频坑）

### ⚠️ 打包前必须清环境变量
之前运行的 onefile exe 会污染 `TCL_LIBRARY`/`TK_LIBRARY`（指向已删除的 `_MEIxxxx`），导致 tkinter 数据收集不全 → exe 缩水（11.8MB→8.7MB）、GUI 崩溃：

```powershell
$env:TCL_LIBRARY=''; $env:TK_LIBRARY=''
pyinstaller --noconfirm --onefile --noconsole --name MyApp `
  --distpath dist --workpath build --specpath . main.py
```

**验证**：打包后检查体积（tkinter 应用 11MB+ 正常，缩水=失败）；GUI 能启动即 OK。

### noconsole 打包后 stdout 不可见
windowed exe 的 print 捕获不到，但 RC=0 正常——正常行为。验证用「启动 GUI 看进程存活」。

## 3. Windows 凭据管理器存储（token 不落盘）

ctypes 调 advapi32（免第三方库）：
- `CredWriteW`（Type=1 Generic, Persist=LOCAL_MACHINE）
- `CredReadW`（用完必须 `CredFree`）
- `CredDeleteW`；blob 用 **UTF-16LE**；目标名如 `AppName_PAT`

## 4. Inno Setup 安装包

```powershell
& "$env:LOCALAPPDATA\Programs\Inno Setup 6\ISCC.exe" installer.iss
```
要点：
- `DefaultDirName={code:GetDefaultDir}`（优先 D 盘）+ `DisableDirPage=no`（可选目录）
- `PrivilegesRequired=lowest`；`[Code]` 段 `CurStepChanged` 删旧文件/快捷方式
- 中文语言包：下载 `ChineseSimplified.isl` 放 ISCC 的 Languages 目录

## 检查清单
- [ ] exe 体积正常（11MB+）
- [ ] 无 Toplevel 弹窗（除确认框）
- [ ] 安装包能自定义目录 + 标准卸载
- [ ] 后台线程回调有 after 保护

---
name: tkinter-dark-ui
description: tkinter 暗色主题 + 单窗口多页面 UI 模式（左侧导航、标签切换、内嵌详情/表单、不弹窗）
---

# tkinter 暗色单窗口 UI

用 Python tkinter 构建桌面工具时，采用暗色主题 + 单窗口多页面布局。

## 暗色配色

全部用 tk 组件自定义配色（**不要用 ttk 的 `-background` 属性**，ttk 走样式系统会报错）：

```python
BG = "#0e1116"      # 窗口背景
CARD = "#161a21"    # 卡片
CARD2 = "#1b202a"   # 输入框/按钮
BORDER = "#232936"  # 边框
TEXT = "#e6e9f0"    # 主文字
MUTED = "#7d8698"   # 次要
GREEN = "#3ddc97"   # 成功
RED = "#ff6b6b"     # 危险
BLUE = "#5b8cff"    # 主按钮
ORANGE = "#ffa94d"  # 警告
```

## 单窗口多页面（用户偏好：不弹窗）

布局：左侧导航 + 右侧内容区；内容区多个 Frame 堆叠切换。

```python
# 建页：self._pages[key] = 统一结构字典
self._pages = {}
for key in ("detail", "issues", "pulls", "releases", "actions"):
    self._pages[key] = self._build_page(self.content, key)

# 切换：pack_forget 全部 + pack 目标页
def show_tab(self, key):
    for k, p in self._pages.items():
        p["page"].pack_forget()
    self._pages[key]["page"].pack(fill="both", expand=True)
```

列表页结构（表格 + 内嵌详情 + 内嵌表单）：

```python
page = {"page": Frame, "tree": Treeview, "count": Label,
        "btns": {}, "detail": Frame, "dt_title": Label,
        "dt_meta": Label, "dt_body": ScrolledText,
        "form": Frame, "f_title": Label, "f_bar": Frame,
        "f_submit": Button, "f_entry_ref": [None], "f_body_ref": [None]}
```

- 选中表格行 → 内嵌详情面板显示（`pack_forget` 收起）
- 新建 → 内嵌表单（不是 Toplevel 弹窗）
- 表格用 `ttk.Treeview` + `style.theme_use("clam")` 配暗色

## 事件日志分级着色

```python
def log(self, msg, level="info"):
    color = {"info": EV_FG, "ok": GREEN, "err": RED, "warn": ORANGE}
    tag = "lvl_%s" % level
    self.ev_text.tag_configure(tag, foreground=color[level])
    self.ev_text.insert("end", "%s\n" % msg, (tag,))
```

## 线程安全

- 网络/IO 放后台线程，UI 更新用 `self.root.after(0, ...)`
- 后台回调前 try/except 包住 `after`（窗口可能已关闭）

## 检查清单

- [ ] 无 Toplevel 弹窗（除极少数确认框）
- [ ] 选中即显示详情，不用二次点击
- [ ] 后台线程回调有 after 保护

---
name: junction-migrate
description: Windows 数据目录迁移方案：junction 符号链接让应用无感知，含占用处理与链接目录识别
---

# Windows 数据目录迁移（junction）

把用户数据目录从 C 盘迁到其他盘（如 D 盘），应用完全无感知。

## 原理

`mklink /J` 创建目录联接（junction）——原路径透明指向新位置，应用看到路径不变。

## 流程

```powershell
# 1. 复制到目标盘（保持目录结构）
robocopy "C:\src\data" "D:\target\data" /E /COPY:DAT

# 2. 校验文件数/字节一致
(Get-ChildItem "C:\src\data" -Recurse -File).Count
(Get-ChildItem "D:\target\data" -Recurse -File).Count

# 3. 原目录改名备份 -> 建 junction
Rename-Item "C:\src\data" "C:\src\data.old"
cmd /c mklink /J "C:\src\data" "D:\target\data"

# 4. 验证（经原路径可读 = 成功）
Get-Item "C:\src\data" -Force | Select LinkType, Target

# 5. 确认后删备份
Remove-Item "C:\src\data.old" -Recurse -Force
```

## 坑 1：目录被占用无法改名

- 症状：`Rename-Item` 报"正在使用中/访问被拒绝"
- 原因：进程持有目录句柄（如工作目录、正在写入的文件）
- 解决：停掉占用进程；脚本开头 `Set-Location 'C:\'` 避开 cwd 占用

## 坑 2：源目录里有 junction 子目录

- 症状：robocopy 后目标出现静态文件副本，但源里"看不到"文件
- 原因：子目录本身是 junction（指向别处，如 node_modules），robocopy 默认复制链接目标内容
- 解决：迁移前扫描，链接类目录**不迁移**：
  ```powershell
  Get-ChildItem $src -Recurse -Force | Where-Object {
    $_.Attributes -match "ReparsePoint" }
  ```

## 验证清单

- [ ] 经原路径可读文件数 = 备份文件数
- [ ] 应用运行正常（数据透明可用）
- [ ] junction 的 LinkType = Junction，Target 正确

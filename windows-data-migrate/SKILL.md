---
name: windows-data-migrate
description: Windows 数据目录迁移方案：junction 符号链接让应用无感知，含占用处理与链接目录识别
---

# Windows 数据目录迁移（junction）

把用户数据目录从 C 盘迁到其他盘，应用完全无感知（路径不变）。

## 流程

```powershell
# 1. 复制到目标盘
robocopy "C:\src\data" "D:\target\data" /E /COPY:DAT

# 2. 校验文件数/字节一致
(Get-ChildItem "C:\src\data" -Recurse -File).Count
(Get-ChildItem "D:\target\data" -Recurse -File).Count

# 3. 原目录改名备份 -> 建 junction
Rename-Item "C:\src\data" "C:\src\data.old"
cmd /c mklink /J "C:\src\data" "D:\target\data"

# 4. 验证
Get-Item "C:\src\data" -Force | Select LinkType, Target

# 5. 确认后删备份
Remove-Item "C:\src\data.old" -Recurse -Force
```

## 坑 1：目录被占用无法改名

- 症状：`Rename-Item` 报"正在使用中/访问被拒绝"
- 原因：进程持有目录句柄（工作目录、正在写入的文件）
- 解决：停占用进程；脚本开头 `Set-Location 'C:\'` 避开 cwd 占用

## 坑 2：源目录里有 junction 子目录

- 症状：robocopy 后目标出现静态副本，源里"看不到"文件
- 原因：子目录本身是 junction（指向别处如 node_modules），robocopy 复制了链接目标内容
- 解决：迁移前扫描 `Attributes -match ReparsePoint`，链接类目录**不迁移**

## 验证清单
- [ ] 经原路径可读文件数 = 备份文件数
- [ ] 应用运行正常
- [ ] LinkType = Junction，Target 正确

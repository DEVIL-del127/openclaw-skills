---
name: inno-setup-installer
description: Inno Setup 制作 Windows 安装包：自定义安装目录、桌面快捷方式、标准卸载、清理旧文件
---

# Inno Setup 安装包制作

把单文件 exe 封装成标准 Windows 安装包（Setup.exe）。

## 编译环境

```powershell
# ISCC.exe 位置（用户级安装）
& "$env:LOCALAPPDATA\Programs\Inno Setup 6\ISCC.exe" installer.iss
# 或 winget 安装
winget install --id JRSoftware.InnoSetup -e
```

## 中文语言包

官方安装版可能不带简体中文，需要下载 `ChineseSimplified.isl`：
https://raw.githubusercontent.com/jrsoftware/issrc/main/Files/Languages/ChineseSimplified.isl
放到 ISCC 同目录的 Languages 文件夹。

## 安装脚本要点

```ini
[Setup]
DefaultDirName={code:GetDefaultDir}   ; 支持自定义目录
PrivilegesRequired=lowest             ; 普通用户可装
DisableDirPage=no                     ; 显示"选择安装位置"页
OutputBaseFilename=MyApp-Setup
Compression=lzma2
CloseApplications=yes

[Tasks]
Name: "desktopicon"; Description: "创建桌面快捷方式"

[Files]
Source: "dist\MyApp.exe"; DestDir: "{app}"; Flags: ignoreversion

[Icons]
Name: "{autoprograms}\{#MyAppName}"; Filename: "{app}\MyApp.exe"
Name: "{autodesktop}\{#MyAppName}"; Filename: "{app}\MyApp.exe"; Tasks: desktopicon

[Code]
// 默认安装目录：优先 D 盘，否则 Program Files
function GetDefaultDir(Param: string): string;
begin
  if DirExists('D:\') then
    Result := 'D:\MyApp'
  else
    Result := ExpandConstant('{autopf}\MyApp');
end;

// 安装前清理旧版文件/快捷方式
procedure CurStepChanged(CurStep: TSetupStep);
begin
  if CurStep = ssInstall then begin
    DeleteFile(ExpandConstant('{userdesktop}\MyApp.exe'));
    DeleteFile(ExpandConstant('{userdesktop}\MyApp.lnk'));
  end;
end;
```

## 验证

1. 静默安装测试：`Setup.exe /VERYSILENT /SUPPRESSMSGBOXES /DIR=D:\TestDir`
2. 检查安装目录文件存在、GUI 能启动
3. 卸载走标准 Windows 通道（控制面板 → 程序）

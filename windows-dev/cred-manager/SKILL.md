---
name: cred-manager
description: 用 ctypes 调 Windows 凭据管理器（advapi32）安全存储 token/密码，不落明文文件
---

# Windows 凭据管理器安全存储

用 ctypes 调 advapi32 的 CredWrite/CredRead/CredDelete，免第三方库。

## 实现要点

```python
import ctypes

CRED_TYPE_GENERIC = 1
CRED_PERSIST_LOCAL_MACHINE = 2

class _CREDENTIAL(ctypes.Structure):
    _fields_ = [
        ("Flags", ctypes.c_uint32),
        ("Type", ctypes.c_uint32),
        ("TargetName", ctypes.c_wchar_p),
        ("Comment", ctypes.c_wchar_p),
        ("LastWritten", ctypes.c_int64),
        ("CredentialBlobSize", ctypes.c_uint32),
        ("CredentialBlob", ctypes.POINTER(ctypes.c_byte)),
        ("Persist", ctypes.c_uint32),
        ("AttributeCount", ctypes.c_uint32),
        ("Attributes", ctypes.c_void_p),
        ("TargetAlias", ctypes.c_wchar_p),
        ("UserName", ctypes.c_wchar_p),
    ]
```

- 写：`CredWriteW`，blob 用 **UTF-16LE** 编码，`Persist=LOCAL_MACHINE`
- 读：`CredReadW`，**用完必须 `CredFree`**（否则句柄泄漏）
- 删：`CredDeleteW`
- 目标名：如 `AppName_PAT`（唯一标识）
- 用户名：可填账号名做区分

## 参考实现

完整代码见 GitHubConsole 项目的 `core_gh.py`（`_cred_write/_cred_read/_cred_delete`）。

## 安全要点

- token 只存凭据管理器，**不写本地文件、不入代码、不入 git**
- 凭据管理器是 Windows 用户级加密存储
- 聊天/日志中出现的 token 要提醒用户轮换

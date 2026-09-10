# Task 4: The File System

## Overview
Covers the **NTFS** file system: how disks and volumes are organized, and how file/folder permissions work.

## Security-relevant concepts

### Access Control Lists (ACLs)
Every file and folder on NTFS carries a list defining who can read, write/modify, or execute it, broken down by user or group. This is the entire foundation of Windows' permission model — everything else (UAC, integrity levels) builds on top of it.

### Insecure file permissions
One of the most common privilege escalation paths on Windows: a standard, non-admin user has write or modify access to a sensitive file or folder — often a binary run by a service under SYSTEM. Swap that binary for a malicious one, wait for the service to restart (or trigger it), and the payload runs with far more privilege than the original user ever had. Checking for exactly this kind of misconfigured permission is one of the first things you do in any internal pentest or privesc-focused CTF box.

## Commands

### `icacls "C:\ExampleFolder"`
Shows the ACL for a given file or folder. The output lists each user/group next to a permission code:
- `F` = Full Control
- `M` = Modify
- `RX` = Read & Execute
- `W` = Write

This is the go-to command when hunting for exploitable permission misconfigurations.

```cmd
icacls "C:\ExampleFolder"
```

### `dir /q`
Lists the contents of the current directory with an extra column showing the **owner** of each item — useful for quickly figuring out who created or owns a given file.

```cmd
dir /q
```

### `Get-Acl -Path "C:\Windows" | Format-List`
The PowerShell equivalent of `icacls`, with more detail:
1. `Get-Acl -Path "C:\Windows"` pulls the full ACL object for the path — owner, primary group, and every individual access rule.
2. `Format-List` renders it one property per line instead of a compressed table, which matters here since an ACL object contains nested, multi-rule data that a table view flattens into something unreadable.

```powershell
Get-Acl -Path "C:\Windows" | Format-List
```

# Windows Fundamentals 1 (TryHackMe)

Notes and writeup from the TryHackMe **Windows Fundamentals 1** room, covering the core areas of the Windows operating system from a security perspective: file system permissions, the System32 directory, local accounts, UAC, the Control Panel/Settings split, and process monitoring through Task Manager.

Each task file below goes past the room's answers and explains the "why" behind the security angle, along with the commands used and what each one actually does.

## Structure

```
windows-fundamentals-1-writeup/
├── README.md
└── tasks/
    ├── 01-introduction.md
    ├── 02-windows-editions.md
    ├── 03-desktop-gui.md
    ├── 04-file-system.md
    ├── 05-system32-folder.md
    ├── 06-user-accounts.md
    ├── 07-user-account-control.md
    ├── 08-settings-control-panel.md
    └── 09-task-manager.md
```

## Room overview

Windows Fundamentals 1 is an introductory room, but treating it as "just basics" undersells it — most of what's covered here shows up again later in privilege escalation and incident response rooms. NTFS permissions, LOLBins, UAC's split-token model, and process masquerading are all concepts that keep reappearing once you move into offensive or defensive Windows work.

## Task summary

| # | Task | Main security angle |
|---|------|----------------------|
| 1 | Introduction | Isolated lab environment |
| 2 | Windows Editions | Feature gaps between editions (BitLocker, GPO, AD join) |
| 3 | Desktop (GUI) | Fast access to tools during incident response |
| 4 | The File System | NTFS permissions and ACLs |
| 5 | Windows\System32 | LOLBins and DLL hijacking |
| 6 | User Accounts | Least privilege and the SAM database |
| 7 | User Account Control | Split tokens and UAC bypass |
| 8 | Settings & Control Panel | Firewall rules and blocking lateral movement |
| 9 | Task Manager | Threat hunting and process masquerading |

Every file in `tasks/` follows the same layout: the concept, the room's question and answer (where one exists), and a command-by-command breakdown of anything run in CMD or PowerShell.

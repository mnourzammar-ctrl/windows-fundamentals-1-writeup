# Task 9: Task Manager

## Question
- **Q:** What is the keyboard shortcut to open Task Manager?
- **A:** `Ctrl+Shift+Esc`

## Security-relevant concepts

### Threat hunting basics
Watching running processes and matching each one's PID to its actual file path on disk. This matters because plenty of malware shares a name with a legitimate process but runs from a completely different location — and that mismatch is one of the fastest ways to spot something wrong.

### Process masquerading
Naming a malicious process after a well-known system one (`svchost.exe`, `explorer.exe`) to blend into the process list. Catching it means checking the actual file path, the digital signature, and the parent process — a legitimate `svchost.exe`, for instance, should be running from System32 with `services.exe` as its parent. Anything that deviates from that is worth a second look.

### Sysinternals
A free tool suite from Microsoft, and **Process Explorer** in particular goes well beyond what Task Manager offers by default — a full parent/child process tree, signature verification built directly into the process list, and visibility into loaded DLLs and handles per process. That level of detail is essential when dealing with rootkits or anything actively trying to hide from standard tools.

## Commands

### `taskmgr`
Launches Task Manager directly from the command line — an alternative to the `Ctrl+Shift+Esc` shortcut, useful in scripts or when the shortcut isn't working for some reason.

```cmd
taskmgr
```

### `tasklist`
Prints a text list of running processes with their PID and rough memory usage. A quick, GUI-free way to check processes — handy in remote or shell-only sessions where a graphical interface isn't available, like after landing a shell through an exploit.

```cmd
tasklist
```

### `taskkill /PID <PID_NUMBER> /F`
- `taskkill` — terminates a running process.
- `/PID <PID_NUMBER>` — targets it by process ID rather than name, which matters when multiple processes share the same name.
- `/F` — forces termination immediately, even if the process isn't responding.

Used to kill a suspicious or malicious process as soon as it's identified during incident response.

```cmd
taskkill /PID <PID_NUMBER> /F
```

### `Get-Process | Select-Object Id, ProcessName, Path, Company`
1. `Get-Process` returns every running process as a full object.
2. `Select-Object` narrows the output to four fields: PID, process name, **full file path on disk**, and the registered company name.

The `Path` column is the important one here — it's exactly what exposes process masquerading. A process named `svchost.exe` running from `C:\Users\Public\svchost.exe` instead of `C:\Windows\System32\svchost.exe` is close to a confirmed red flag on its own.

```powershell
Get-Process | Select-Object Id, ProcessName, Path, Company
```

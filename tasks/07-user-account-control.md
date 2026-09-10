# Task 7: User Account Control (UAC)

## Question
- **Q:** What does UAC mean?
- **A:** `User Account Control`

## Security-relevant concepts

### Split token architecture
When an admin logs in, Windows doesn't hand out full admin rights by default. Instead it creates two tokens:
- A **filtered/standard token**, used for everything by default — browsing, opening files, running ordinary programs — at standard-user privilege.
- An **elevated token**, only invoked when a UAC prompt is approved, and only for that specific action.

The split exists so that nothing runs with full privilege all the time, which limits how much damage a program can do if it turns out to be malicious.

### Mandatory Integrity Control (MIC)
A separate mechanism that stops lower-trust processes (Medium integrity, the default for most software) from tampering with higher-trust ones (High or System), even under the same user account. This is what prevents an ordinary-privilege malicious process from directly injecting into a sensitive system process.

### UAC bypass techniques
Some trusted Windows binaries are flagged `autoElevate = true`, meaning they get elevated automatically without triggering a UAC prompt, because Microsoft already trusts them. Attackers abuse this by hijacking or DLL-planting into one of these auto-elevating binaries, getting code to run with elevated privileges without the user ever seeing a prompt. This is one of the more common privilege escalation techniques on Windows precisely because it's silent.

## Commands

### `reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v EnableLUA`
1. `reg query` reads a value from the Windows registry.
2. The key path points at the registry location holding UAC policy settings.
3. `/v EnableLUA` targets the specific value that controls whether UAC is enabled system-wide (`1` = on, `0` = off).

Used to confirm UAC is actually enabled on a given machine. Finding it set to `0` during an audit is a red flag — it means every process runs at full privilege with no confirmation layer at all.

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v EnableLUA
```

### `[Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent() | Select-Object -ExpandProperty Claims`
This one works directly with .NET objects rather than a registry key:
1. `[Security.Principal.WindowsIdentity]::GetCurrent()` calls a static .NET method that returns the identity of the current session — the current token and username.
2. Casting it to `[Security.Principal.WindowsPrincipal]` wraps it in a broader object that includes role and permission info.
3. `Select-Object -ExpandProperty Claims` pulls out just the `Claims` property and expands it, rather than showing it as a collapsed nested field. The claims list contains the security-relevant attributes tied to the identity — group membership, integrity level, and so on.

More granular than the CMD-based checks above — useful when you need to inspect exactly what a session's token actually carries.

```powershell
[Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent() | Select-Object -ExpandProperty Claims
```

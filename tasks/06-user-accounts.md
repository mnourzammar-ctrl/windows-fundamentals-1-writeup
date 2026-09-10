# Task 6: User Accounts, Profiles, and Permissions

## Questions
1. **What is the name of the other user account?** → `tryhackmebilly`
2. **What groups is this user a member of?** → `Remote Desktop Users, Users`
3. **What built-in account is for guest access to the computer?** → `Guest`
4. **What is the account description?** → `Built-in account for guest access to the computer/domain`

## Security-relevant concepts

### Principle of least privilege (PoLP)
Keeping administrator and standard accounts separate limits blast radius. If a standard account gets compromised, the attacker is stuck with whatever that account can do. If it's an admin account, they get full control of the machine immediately — no extra steps needed.

### The SAM database
Local password hashes live in `C:\Windows\System32\config\SAM`. The file is locked while the OS is running and can't just be copied directly, but that hasn't stopped it from being a primary target — tools like **Mimikatz** pull hashes out of `lsass.exe`'s memory or from shadow copies instead, and those hashes then get used in pass-the-hash attacks or offline cracking.

### Built-in accounts
Recognizing Windows' default accounts matters for telling normal behavior apart from something suspicious:
- **Guest** — limited-access account, disabled by default on modern Windows.
- **WDAGUtilityAccount** — used by Windows Defender Application Guard to run the browser inside an isolated container when visiting untrusted sites.

## Commands

### `net user`
Lists every local user account on the machine. Basic enumeration step — useful whether you're an admin auditing accounts or an attacker who just landed on a box and wants to know what's there.

```cmd
net user
```

### `net user tryhackmebilly`
Same command, pointed at a specific account. Returns full detail: creation date, last logon, group memberships, and whether the account is enabled.

```cmd
net user tryhackmebilly
```

### `net localgroup Administrators`
Lists everyone in the local **Administrators** group. One of the highest-value commands for both defenders (confirming nobody unauthorized has admin) and attackers (identifying targets for impersonation or escalation).

```cmd
net localgroup Administrators
```

### `whoami /priv`
Shows every privilege attached to the current user's session (`SeDebugPrivilege`, `SeImpersonatePrivilege`, etc.). A single misconfigured privilege on an otherwise-standard account is a common privilege escalation route straight to SYSTEM.

```cmd
whoami /priv
```

### `Get-LocalUser | Select-Object Name, Enabled, LastLogon`
The PowerShell equivalent of `net user`, but returns structured objects instead of flat text:
1. `Get-LocalUser` pulls every local account as an object.
2. `Select-Object Name, Enabled, LastLogon` trims the output down to the three fields that matter most for a quick audit — is the account enabled, and when did it last log in.

```powershell
Get-LocalUser | Select-Object Name, Enabled, LastLogon
```

### `Get-LocalGroupMember -Group "Remote Desktop Users"`
Lists members of a specific local group — here, **Remote Desktop Users**, which controls who can RDP in. Worth auditing regularly, since an account added to this group without a clear reason (especially after a compromise) usually means someone set up persistent remote access.

```powershell
Get-LocalGroupMember -Group "Remote Desktop Users"
```

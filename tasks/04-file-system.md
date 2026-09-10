# Task 4: The File System

## Overview
This task introduces **NTFS (New Technology File System)** and focuses on one of its most important security functions: controlling access to files and folders.

For a security analyst, the important question is not only:

> **Where is the file?**

but also:

> **Who can access it, what can they do with it, and what security impact would that access have?**

---

## NTFS from a Security Perspective

NTFS supports security descriptors that Windows uses to define ownership, permissions, auditing information, and other access-control data for securable objects such as files and folders.

A simplified access-control view looks like this:

```text
File / Folder
     |
     v
Security Descriptor
     |
     +--> Owner
     |
     +--> DACL -> Who is allowed or denied access?
     |
     +--> SACL -> What access should be audited?
```

This distinction matters because an **ACL** is not simply a list of permissions. Windows commonly works with two important ACL types:

- **DACL (Discretionary Access Control List)** — controls who is allowed or denied access.
- **SACL (System Access Control List)** — specifies access attempts that can be audited when the appropriate auditing policy is enabled.

For basic permission analysis, the DACL is usually the part we are most interested in.

---

## ACLs and ACEs

An ACL is made up of individual entries called **ACEs (Access Control Entries)**.

A simplified example:

```text
File: report.txt
 |
 +-- DACL
      |
      +-- ACE: User_A          -> Read
      +-- ACE: SOC_Analysts    -> Read & Execute
      +-- ACE: Administrators  -> Full Control
```

The relationship is:

```text
ACL
 |
 +--> ACE
 +--> ACE
 +--> ACE
```

Each ACE can identify a user or group and define the access that applies to that identity.

### Why this matters

When investigating permissions, an analyst should not stop at:

> "This file has an ACL."

The useful questions are:

- Which users or groups appear in the access rules?
- Which permissions do they have?
- Were those permissions inherited?
- Is an Allow or Deny rule involved?
- Is that level of access appropriate for the object's purpose?

---

## Understanding Common NTFS Permissions

Common permissions include:

| Permission | Meaning |
|---|---|
| **Read** | View file contents and related information |
| **Write** | Write data or create content where permitted |
| **Read & Execute** | Read content and execute applicable files |
| **Modify** | Read, write, execute, and delete within the granted scope |
| **Full Control** | Modify permissions and perform the broad set of allowed file operations |

The security impact depends on **what object is writable**, not simply on seeing the word `Write` or `Modify`.

For example:

```text
User can modify:
C:\Users\User\Documents\notes.txt
```

may be completely normal.

But:

```text
Standard User
      |
      v
Modify Permission
      |
      v
Executable used by a privileged service
```

can represent a serious security weakness.

---

## Permission Inheritance

NTFS permissions can be inherited from parent folders.

A simplified example:

```text
C:\Application
      |
      +--> Permission granted here
      |
      +--> bin\
      |     |
      |     +--> app.exe
      |
      +--> logs\
```

Depending on the inheritance configuration, child files and folders may receive access rules from the parent.

### Why this matters

An insecure permission does not always have to be assigned directly to the sensitive file.

It may come from a parent directory.

Therefore, when analyzing an unexpected permission, ask:

```text
Where did this ACE come from?
        |
        +--> Explicit permission?
        |
        +--> Inherited permission?
```

This helps identify the actual source of the misconfiguration.

---

## Ownership vs Permissions

Ownership and permissions are related, but they are not the same thing.

```text
Owner
  !=
Current Effective Access
```

The owner identifies who owns the object and normally has special ability to manage its permissions. The DACL contains the access rules Windows evaluates when determining requested access.

Therefore, seeing who owns a file does **not** by itself tell you everything that every user can do with that file.

This is why `dir /q` and ACL inspection answer different questions.

---

## Insecure File Permissions and Privilege Escalation

A dangerous permission becomes especially important when a lower-privileged identity can modify something that will later be used by a higher-privileged process.

Consider this scenario:

```text
Standard User
      |
      | has Modify permission
      v
Service Executable
      |
      | executed by
      v
Windows Service
      |
      | runs as
      v
SYSTEM
```

The security boundary is broken because a lower-privileged user can influence code that is executed in a higher-privileged context.

Conceptually:

```text
Low Privilege
      |
      v
Control Over Privileged Resource
      |
      v
Privileged Execution
      |
      v
Privilege Escalation Risk
```

### Important nuance

A writable file does **not automatically mean** privilege escalation is possible.

The analyst must verify the complete context:

- Is the file actually used or executed by a privileged process?
- Which account runs that process or service?
- Can the lower-privileged user really modify or replace the relevant object?
- Are other protections or access checks involved?
- Can the privileged component be caused to load or execute the modified content?

The finding becomes meaningful when the permissions are combined with a privileged execution path.

---

## Blue Team Perspective

The same permission weakness that an authorized penetration tester may identify as a privilege-escalation path is something a defender should identify and remediate before it is abused.

A defensive investigation might look like this:

```text
Sensitive File / Directory
          |
          v
Inspect ACL
          |
          v
Unexpected Write / Modify Access?
          |
      +---+---+
      |       |
     No      Yes
              |
              v
      Identify User / Group
              |
              v
      Determine Object Purpose
              |
              v
      Check Privileged Usage
              |
              v
      Assess Security Impact
              |
              v
           Remediate
```

The core Blue Team question is:

> **Can a lower-privileged identity modify an object that influences a higher-privileged process?**

---

## Command: `icacls`

`icacls` can display and manage access-control information for files and directories.

```cmd
icacls "C:\ExampleFolder"
```

Example permission abbreviations you may encounter include:

```text
F   = Full Control
M   = Modify
RX  = Read & Execute
R   = Read
W   = Write
```

You may also encounter inheritance-related markers in `icacls` output, such as rules inherited from a parent object.

### What should an analyst look for?

Do not only read the permission letters.

Ask:

```text
Which identity has this permission?
        |
        v
What object does it apply to?
        |
        v
Was it inherited or explicitly assigned?
        |
        v
Can this identity modify something security-sensitive?
        |
        v
What privileged process depends on that object?
```

### Example

Suppose an application directory contains:

```text
C:\Program Files\ExampleApp\service.exe
```

and permission analysis shows that a broad, low-privileged group has unexpected modification rights over the relevant executable or directory.

That is not automatically proof of exploitation. It is a **security finding that requires context and remediation** because privileged software may depend on content that less-privileged users can alter.

---

## Command: `dir /q`

`dir /q` displays directory contents and includes ownership information.

```cmd
dir /q
```

### Why ownership is useful

Ownership provides additional context during an investigation.

For example, if a system file or application file has an unexpected owner, that may justify further investigation.

However:

```text
Unexpected Owner
       !=
Malicious File
```

and:

```text
Owner
       !=
Complete Permission Set
```

To understand access, inspect the ACL as well.

---

## PowerShell: `Get-Acl`

PowerShell can retrieve an object's security descriptor:

```powershell
Get-Acl -Path "C:\Windows" | Format-List
```

`Get-Acl` returns structured security information, and `Format-List` makes the selected object's properties easier to inspect interactively.

Useful properties include information such as:

```text
Path
Owner
Access
Audit
```

To focus specifically on access rules:

```powershell
(Get-Acl -Path "C:\ExampleFolder").Access
```

This can expose information such as:

```text
IdentityReference
FileSystemRights
AccessControlType
IsInherited
```

These properties answer useful investigative questions:

```text
IdentityReference -> Who?
FileSystemRights  -> What access?
AccessControlType -> Allow or Deny?
IsInherited       -> Where did the rule originate?
```

---

## Practical Permission Investigation

Imagine you discover:

```text
Object:
C:\ExampleApp\service.exe

Service account:
LocalSystem

Unexpected ACL:
Users -> Modify
```

Do not immediately conclude:

> "Privilege escalation occurred."

Instead investigate:

```text
Users -> Modify
       |
       v
Is the permission effective?
       |
       v
Is this the executable actually used by the service?
       |
       v
What account runs the service?
       |
       v
Can the object influence privileged execution?
       |
       v
Is there evidence that it was modified?
       |
       v
Security Finding / Incident Assessment
```

There are two different questions:

### Vulnerability / Misconfiguration Question
> Could a lower-privileged user abuse this permission?

### Incident Response Question
> Is there evidence that somebody actually abused it?

That distinction is extremely important in defensive security.

---

## Security Misconfiguration vs Active Compromise

A weak ACL is a **security weakness**.

It is not automatically evidence that an attacker exploited it.

```text
Weak Permission
      |
      +--> Potential Attack Path
      |
      v
Needs Remediation
```

If investigation also finds evidence such as an unexpected file modification or suspicious privileged execution, the situation may become an incident requiring deeper investigation.

This is another important Blue Team principle:

> **A vulnerability describes what could happen. Evidence tells us what did happen.**

---

## Lessons Learned

- NTFS provides access-control mechanisms for files and folders.
- A DACL controls allowed and denied access, while a SACL is related to auditing.
- ACLs contain individual ACEs associated with users or groups.
- Ownership and permissions are related but answer different questions.
- NTFS permissions may be explicitly assigned or inherited from parent objects.
- `icacls` and `Get-Acl` help analysts inspect access-control information.
- Write or Modify permissions are not automatically vulnerabilities; the security impact depends on the object and execution context.
- A lower-privileged user controlling content used by a higher-privileged process can create a privilege-escalation risk.
- A security misconfiguration is not the same thing as evidence of active exploitation.

## Key Takeaway

When reviewing Windows file permissions, do not ask only:

> **"Who can write to this file?"**

Ask:

> **"Who can modify this object, why do they have that access, and can that access influence a more privileged security context?"**

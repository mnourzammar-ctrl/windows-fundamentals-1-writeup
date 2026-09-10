# Task 7: User Account Control (UAC)

## Overview
This task introduces **User Account Control (UAC)**, Windows access tokens, elevation, and integrity levels.

For a security analyst, UAC should not be understood simply as:

> **"The popup that asks for administrator permission."**

The more useful security model is:

```text
User Identity
      |
      v
Access Token
      |
      v
Current Security Context
      |
      +--> Group information
      +--> Privileges
      +--> Integrity level
      |
      v
What can this process do?
```

---

## Question

- **Q:** What does UAC mean?
- **A:** `User Account Control`

---

# What Is UAC?

User Account Control helps reduce unnecessary administrative execution by requiring elevation for operations that need administrator-level rights.

The important concept is:

```text
Administrator Account
        !=
Every Process Runs Elevated
```

An administrator can normally use the desktop and ordinary applications without every process automatically running with the administrator's full elevated access.

When an operation requires elevation, Windows can request approval or credentials depending on the user and policy configuration.

---

# Access Tokens

When Windows authenticates a user, processes operate using **access tokens** that represent security information associated with that logon/session and process.

A simplified token model includes information such as:

```text
Access Token
    |
    +--> User SID
    +--> Group SIDs
    +--> Privileges
    +--> Integrity Level
    +--> Other security attributes
```

Windows uses this security context when deciding whether a process can access protected resources or perform privileged operations.

This gives us an important investigation question:

> **Which security context is this process actually running under?**

The username alone does not always tell the whole story.

---

# Split-Token Model

For an administrator operating under UAC, Windows commonly creates a filtered token and retains a linked elevated administrative token.

Simplified:

```text
Administrator Logs On
        |
        +----------------------+
        |                      |
        v                      v
Filtered Token          Elevated Token
        |                      |
        v                      v
Normal Desktop          Elevated Process
and Applications        when elevation occurs
```

The filtered token is normally used for ordinary desktop activity.

When an administrative action is approved through the elevation mechanism, the elevated process can run using the administrative token.

## Why This Matters

Without this separation, routine applications launched by an administrator could unnecessarily operate with elevated privileges.

Conceptually:

```text
Routine Application
       |
       v
Filtered Context
       |
       v
Reduced Immediate Privilege
```

This does not make an administrator account equivalent to a standard user, and UAC should not be treated as a complete security boundary against every local attack.

Its purpose is to reduce unnecessary administrative execution and make elevation an explicit security event or action.

---

# Standard Users and Elevation

The UAC experience is not identical for every account type.

Conceptually:

```text
Administrator
     |
     v
Elevation may request consent
```

while a standard user may need authorized administrator credentials for an operation requiring administrative rights.

The exact prompt behavior depends on Windows policy and configuration.

This distinction is useful because:

```text
Consent
   !=
Credential Entry
```

and because elevation behavior provides context about the identity and policy involved.

---

# Mandatory Integrity Control (MIC)

Windows also assigns **integrity levels** to security contexts.

Common integrity levels include:

```text
Low
Medium
High
System
```

A simplified desktop example:

```text
Normal User Process
       |
       v
Medium Integrity

Elevated Administrative Process
       |
       v
High Integrity

Many Core System Processes
       |
       v
System Integrity
```

## What MIC Does

Mandatory Integrity Control adds another access-control decision layer based on integrity labels.

A lower-integrity process can be restricted from performing certain write-like operations against higher-integrity objects, even when ordinary discretionary permissions might otherwise appear to allow access.

Simplified:

```text
Lower Integrity
      |
      | restricted write access
      v
Higher Integrity Object
```

### Important Nuance

Do not reduce MIC to:

> **"Medium processes can never interact with High processes."**

Windows security decisions involve multiple mechanisms, including access tokens, privileges, DACLs, integrity policy, process protections, and the specific operation being requested.

The safer mental model is:

> **Integrity levels add restrictions to what lower-integrity subjects can do to higher-integrity objects, particularly for write-style access.**

---

# UAC vs MIC

These concepts are related but different.

```text
UAC
 |
 +--> controls/effects elevation workflow
 +--> reduces routine administrative execution

MIC
 |
 +--> integrity labels
 +--> restricts lower-integrity write-style access to higher-integrity objects
```

Do not treat them as the same Windows security mechanism.

---

# UAC Bypass — Defensive Concept

A **UAC bypass** is a technique that attempts to obtain elevated execution without going through the expected visible UAC elevation interaction.

It is important to understand the concept without reducing all bypasses to one technique.

```text
UAC Bypass
     |
     +--> May involve trusted Windows components
     +--> May involve configuration or registry behavior
     +--> May involve application-loading behavior
     +--> Technique depends on Windows version and configuration
```

Some Windows components have elevation-related behavior intended for trusted operating-system workflows. Security researchers and attackers have historically found ways to abuse particular behaviors or configurations to influence elevated execution.

## Important Correction

Do not memorize:

```text
UAC Bypass = DLL Hijacking
```

That is incorrect.

DLL-loading weaknesses can be involved in some privilege/elevation scenarios, but **UAC bypass is a broader category of techniques**.

Likewise:

```text
auto-elevating component
       !=
automatic arbitrary-code execution
```

A bypass requires a specific abuse path and applicable system conditions.

---

# Is UAC a Security Boundary?

UAC is an important Windows security feature, but it should not be treated as the only control separating an administrator from full administrative execution.

For defensive thinking:

```text
UAC Enabled
     !=
Privilege Escalation Impossible
```

and:

```text
UAC Prompt Appeared
     !=
Activity Is Malicious
```

The analyst still needs context.

---

# Command: Check `EnableLUA`

The following registry query checks the `EnableLUA` policy value:

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v EnableLUA
```

A commonly expected enabled configuration is:

```text
EnableLUA    REG_DWORD    0x1
```

A value of `0` indicates that UAC-related behavior controlled by this setting is disabled.

## Security Perspective

Finding UAC disabled is security-relevant and should be compared against the organization's approved configuration.

However, avoid the oversimplified conclusion:

```text
EnableLUA = 0
      =
Every process automatically runs as SYSTEM/full privilege
```

That is not an accurate description of Windows access control.

The security concern is that disabling UAC removes important elevation and token-filtering behavior, weakening the expected Windows administrative security model.

---

# Inspecting the Current Security Context

Windows provides several useful ways to examine the current identity and token-related information.

A simple starting point is:

```cmd
whoami
```

To inspect group information:

```cmd
whoami /groups
```

To inspect token privileges:

```cmd
whoami /priv
```

Together, these help answer different questions:

```text
whoami
   |
   +--> Which identity?

whoami /groups
   |
   +--> Which groups / integrity-related information?

whoami /priv
   |
   +--> Which privileges?
```

This connects directly with Task 6.

---

# PowerShell and Identity Information

PowerShell and .NET can also expose identity-related information.

For example:

```powershell
[Security.Principal.WindowsIdentity]::GetCurrent()
```

This returns information about the current Windows identity.

The original lab notes also explored claims through Windows identity/principal objects. Claims can expose security-related identity attributes in applicable contexts, but they should not be treated as a universal replacement for direct token inspection.

For basic Windows investigation, commands such as:

```cmd
whoami /groups
whoami /priv
```

often provide a clearer starting point for understanding the current security context.

---

# Practical Investigation Example

Imagine a SOC analyst observes:

```text
User:
CORP\alex

Process:
powershell.exe

Integrity:
High
```

Should the analyst conclude that privilege escalation occurred?

**No.**

A high-integrity PowerShell process could have been legitimately elevated by an administrator.

The investigation should continue:

```text
High-Integrity Process
        |
        +--> Which user?
        +--> Is the user expected to have admin rights?
        +--> What was the parent process?
        +--> What was the command line?
        +--> When did elevation occur?
        +--> What activity followed?
        +--> Is there evidence of an unusual elevation path?
        |
        v
Correlate Evidence
        |
        v
Expected Administration
        OR
Suspicious Elevated Activity
```

Again:

> **Elevated does not automatically mean malicious.**

---

# UAC Bypass Investigation Mindset

If telemetry suggests an unusual process became elevated without the expected administrative workflow, an analyst may investigate:

```text
Unexpected Elevated Process
          |
          +--> Parent process
          +--> Command line
          +--> Integrity level
          +--> User / groups
          +--> Registry modifications
          +--> File / DLL activity
          +--> Related Windows components
          +--> Timeline
          |
          v
Was the elevation expected?
          |
      +---+---+
      |       |
     Yes      No
              |
              v
       Escalate Investigation
```

The goal is not to identify a UAC bypass from one indicator alone.

It is to reconstruct **how the process moved from its previous security context to elevated execution**.

---

# Connecting Tasks 4, 5, 6, and 7

The concepts from the previous tasks now begin to connect:

```text
Task 4
NTFS Permissions
      |
      v
Who can modify resources?
      |
      v
Task 5
System Binaries / DLL Loading
      |
      v
What code is executed and how?
      |
      v
Task 6
Users / Groups / Privileges
      |
      v
Which identity and rights?
      |
      v
Task 7
Tokens / Integrity / Elevation
      |
      v
Which security context is the process running under?
```

Together, these answer a broader Windows security question:

> **Who caused which code to execute, with what access, and under which security context?**

---

# Security Context Investigation Model

A useful analyst workflow is:

```text
Process
   |
   v
User Identity
   |
   v
Group Membership
   |
   v
Token Privileges
   |
   v
Integrity Level
   |
   v
Elevated?
   |
   v
Parent + Command Line
   |
   v
File / Registry / Network Activity
   |
   v
Compare With Expected Behavior
   |
   v
Verdict
```

This is much more useful than simply asking whether UAC is enabled.

---

# Lessons Learned

- UAC stands for User Account Control.
- An administrator account does not mean every process automatically runs elevated.
- Access tokens represent important parts of a process's Windows security context.
- UAC commonly uses filtered and elevated tokens for administrator sessions.
- Standard-user and administrator elevation prompts can behave differently.
- Mandatory Integrity Control and UAC are related to Windows privilege management but are different mechanisms.
- Integrity levels add restrictions between lower- and higher-integrity security contexts.
- UAC bypass is a broad class of elevation techniques and should not be equated with DLL hijacking.
- `EnableLUA = 0` weakens the expected UAC security model but does not mean every process automatically runs as SYSTEM.
- An elevated process is not automatically malicious; the elevation path and subsequent behavior must be investigated.
- User identity, token privileges, integrity level, parent process, and command line should be correlated when analyzing elevated activity.

# Key Takeaway

Do not ask only:

> **"Is UAC enabled?"**

Ask:

> **"Which security context is this process running under, how did it become elevated, and is that elevation consistent with expected user and system behavior?"**

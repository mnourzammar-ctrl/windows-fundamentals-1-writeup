# Windows Fundamentals 1 — Security & SOC Notes

A security-focused write-up based on the TryHackMe **Windows Fundamentals 1** room.

The room introduces core Windows concepts, but this repository goes beyond recording answers. Each task is used to build a basic **Blue Team / SOC investigation mindset** by connecting Windows fundamentals to access control, identity, privilege, process behavior, network access, and evidence-based analysis.

> **Core principle:** A suspicious indicator is a reason to investigate — not automatically proof of malicious activity.

---

## About This Project

The purpose of this repository is to document my Windows fundamentals studies while developing practical security-analysis skills.

Rather than only memorizing commands or TryHackMe answers, I focus on questions such as:

```text
What is this Windows component?
        |
        v
How does it normally work?
        |
        v
Why is it security-relevant?
        |
        v
What would a defender investigate?
        |
        v
What additional evidence is needed?
```

The notes gradually connect individual Windows concepts into a basic endpoint-investigation methodology.

---

## Skills Demonstrated

This project covers introductory experience with:

- Windows security fundamentals
- NTFS permissions, ACLs, ACEs, and inheritance
- Windows users, groups, and least privilege
- Local vs. domain identity concepts
- SAM and Windows credential-security concepts
- User Account Control (UAC)
- Access tokens and integrity levels
- Windows environment variables and system directories
- System32, SysWOW64, and WOW64 filesystem redirection
- LOLBins and process masquerading
- DLL-loading security concepts
- Windows Defender Firewall profiles and rules
- SMB and RDP security context
- Basic lateral-movement concepts
- Windows process investigation
- Parent-child process analysis
- Command-line analysis
- Basic behavior-based threat hunting
- PowerShell and CMD for host investigation

---

## Repository Structure

```text
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

---

## Task Summary

| # | Task | Main Security Focus |
|---|---|---|
| 1 | Introduction | Lab isolation, authorization, and learning normal Windows behavior |
| 2 | Windows Editions | Security/management capabilities and expected endpoint baseline |
| 3 | Desktop (GUI) | Locating Windows tools, evidence, and security-relevant filesystem locations |
| 4 | The File System | NTFS permissions, ACLs/ACEs, inheritance, and privilege-escalation risk |
| 5 | Windows\System32 | System binaries, WOW64, LOLBins, masquerading, and DLL-loading concepts |
| 6 | User Accounts | Identities, groups, least privilege, SAM, and credential-security concepts |
| 7 | User Account Control | Access tokens, integrity levels, elevation, and UAC security concepts |
| 8 | Settings & Control Panel | Windows Firewall, network access, SMB/RDP, and lateral-movement context |
| 9 | Task Manager | Process investigation, process trees, command lines, and behavior correlation |

---

# Security Concepts Developed Through the Tasks

## 1. Access Control

Tasks 4, 6, and 7 build a basic Windows access-control model:

```text
Identity
   |
   v
Groups / Privileges
   |
   v
Access Token
   |
   v
Integrity Level
   |
   v
ACL / Permissions
   |
   v
Allowed or Denied Operation
```

The important lesson is that access decisions cannot be understood from a username alone.

An analyst may need to consider the user's groups, token privileges, integrity level, object permissions, and the operation being attempted.

---

## 2. Trusted Does Not Automatically Mean Benign

Windows contains many legitimate Microsoft binaries that can appear in both normal and malicious activity.

Therefore:

```text
Microsoft-Signed Binary
        !=
Automatically Benign Behavior
```

For example, a legitimate Windows utility may still deserve investigation if its:

```text
Parent Process
Command Line
Loaded Content
Network Activity
File Activity
User Context
```

does not match expected behavior.

This is particularly important when investigating **Living-off-the-Land binaries (LOLBins)**.

---

## 3. Capability vs. Observed Activity

A recurring principle throughout the project is the difference between what a system **allows** and what actually **happened**.

Examples:

```text
User is in Remote Desktop Users
        !=
User actually logged in through RDP
```

```text
File has weak permissions
        !=
Privilege escalation actually occurred
```

```text
TCP 445 is reachable
        !=
Lateral movement occurred
```

Configuration identifies possible exposure or capability.

Logs and telemetry are needed to establish observed activity.

---

# Windows Process Investigation Methodology

Task 9 brings the previous concepts together into a basic process-investigation workflow.

```text
Process Name
     |
     v
PID
     |
     v
Executable Path
     |
     v
Digital Signature / Hash
     |
     v
User / Security Context
     |
     v
Integrity / Privileges
     |
     v
Parent Process
     |
     v
Command Line
     |
     v
Child Processes
     |
     v
File / Registry Activity
     |
     v
Network Activity
     |
     v
Timeline + Expected Baseline
     |
     v
Evidence-Based Verdict
```

Not every alert requires every field, but this provides a repeatable investigation model.

The objective is to move from:

> **"This process looks strange."**

to:

> **"This behavior is suspicious because multiple pieces of evidence do not match the expected system or user behavior."**

---

## Example: Process Triage

Consider:

```text
Process:
svchost.exe

Path:
C:\Users\Public\svchost.exe
```

The name resembles a legitimate Windows process, but the path is unusual for the expected Windows `svchost.exe`.

That does not automatically prove malware.

A stronger investigation would consider:

```text
Name
 |
 v
Path
 |
 v
Signature / Hash
 |
 v
User
 |
 v
Parent Process
 |
 v
Command Line
 |
 v
Child Processes
 |
 v
File / Registry Activity
 |
 v
Network Activity
 |
 v
Verdict
```

This reflects one of the central lessons of the repository:

> **Suspicious does not automatically mean malicious.**

---

# Network Investigation Mindset

Task 8 applies the same reasoning to network access.

Instead of:

```text
Port 445 = Bad
Port 3389 = Bad
```

the investigation asks:

```text
Source
   |
   v
Destination
   |
   v
Protocol / Port
   |
   v
User / Process
   |
   v
Firewall Rule / Network Policy
   |
   v
Expected Business Purpose?
   |
   v
Related Activity
   |
   v
Verdict
```

SMB and RDP are legitimate enterprise technologies, but they can also appear during unauthorized remote access or lateral movement.

The protocol provides context; it does not provide the final verdict.

---

# Commands Practiced

Examples of Windows commands and PowerShell cmdlets explored throughout the project include:

```cmd
systeminfo
net user
net localgroup Administrators
whoami
whoami /groups
whoami /priv
icacls
dir /q
tasklist
taskmgr
reg query
```

PowerShell examples include:

```powershell
Get-ComputerInfo
Get-Acl
Get-LocalUser
Get-LocalGroupMember
Get-NetFirewallProfile
Get-NetFirewallRule
Get-Process
Get-CimInstance Win32_Process
Get-ChildItem Env:
```

The goal is not to memorize commands in isolation.

For each command, the more important questions are:

```text
What information does it provide?
Why do I need that information?
How does it contribute to an investigation?
What additional evidence would I need?
```

---

# Blue Team Principles Reinforced

Several principles repeat throughout the repository:

```text
Suspicious
!=
Malicious
```

```text
Capability
!=
Observed Activity
```

```text
Valid Signature
!=
Benign Behavior
```

```text
Open Port
!=
Attack
```

```text
Elevated Process
!=
Privilege Escalation
```

```text
Security Misconfiguration
!=
Evidence of Exploitation
```

These principles encourage evidence-based analysis rather than conclusions based on a single indicator.

---

# Learning Outcomes

After completing and expanding these tasks, I can explain at an introductory level:

- How Windows uses users, groups, permissions, tokens, and integrity levels to control access.
- Why NTFS permission misconfigurations can create security risks.
- Why System32 binaries should be evaluated by behavior and context rather than name or signature alone.
- The difference between LOLBin abuse and process masquerading.
- The difference between stored local account credential data in the SAM and credential-related material associated with LSASS.
- The difference between password cracking and Pass-the-Hash.
- Why UAC, access tokens, and Mandatory Integrity Control are related but distinct concepts.
- Why SMB and RDP traffic requires context before being labeled malicious.
- How to perform basic Windows process triage using process name, PID, path, parent, command line, user context, and related activity.
- Why event correlation and timelines are stronger than isolated indicators.

---

# Investigation Mindset

The most important outcome of this project is a change in how Windows security information is evaluated.

Instead of:

```text
See Indicator
     |
     v
Make Immediate Conclusion
```

the goal is:

```text
Observe
   |
   v
Collect Context
   |
   v
Correlate Evidence
   |
   v
Compare With Baseline
   |
   v
Form Hypothesis
   |
   v
Validate
   |
   v
Evidence-Based Verdict
```

This repository represents foundational Windows security study and is intended to support continued learning toward **Blue Team and SOC analyst skills**.

---

## Platform

Training material: **TryHackMe — Windows Fundamentals 1**

This repository contains my own study notes, explanations, security observations, and investigation-focused extensions based on the concepts practiced in the room.

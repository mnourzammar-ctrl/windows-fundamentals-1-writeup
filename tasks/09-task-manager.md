# Task 9: Task Manager and Process Investigation

## Overview
This task introduces **Task Manager** and basic Windows process investigation.

For a SOC analyst, Task Manager is more than a way to close frozen applications. It provides a starting point for answering questions such as:

> **What is running, who started it, where did it come from, and does its behavior make sense in this environment?**

A process name alone is rarely enough to make a security decision.

---

## Question

- **Q:** What is the keyboard shortcut to open Task Manager?
- **A:** `Ctrl+Shift+Esc`

Task Manager can also be launched from the command line:

```cmd
taskmgr
```

---

# What Is a Process?

A **process** is a running instance of a program.

Simplified:

```text
Executable on Disk
        |
        v
Process Created
        |
        +--> PID
        +--> User / Security Context
        +--> Parent Process
        +--> Command Line
        +--> Loaded Modules
        +--> File Activity
        +--> Network Activity
```

For example:

```text
C:\Windows\System32\notepad.exe
        |
        v
notepad.exe
PID: 4321
```

The executable file and the running process are related, but they are not the same concept.

---

# Process ID (PID)

Each running process receives a **Process ID (PID)**.

Example:

```text
PID   Process
----  -------------
912   svchost.exe
1436  svchost.exe
2872  powershell.exe
```

Multiple processes can share the same executable name while having different PIDs.

This is why an analyst should avoid relying only on a process name when investigating activity.

---

# Process Investigation Methodology

A useful basic workflow is:

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
Digital Signature
     |
     v
User / Security Context
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
Compare With Expected Behavior
     |
     v
Verdict
```

Not every investigation will require every field, but this provides a strong mental model.

The goal is to move from:

> **"This process looks suspicious."**

to:

> **"This process is suspicious because multiple pieces of evidence do not match the expected behavior."**

---

# Process Name

The process name is useful for initial identification.

Examples:

```text
explorer.exe
svchost.exe
powershell.exe
rundll32.exe
```

But names are easy to imitate and legitimate binaries can also be abused.

Therefore:

```text
Process Name
      !=
Process Identity / Intent
```

A familiar name should never be the end of an investigation.

---

# Executable Path

The executable path tells the analyst where the process image exists on disk.

Example:

```text
Name:
svchost.exe

Path:
C:\Windows\System32\svchost.exe
```

compared with:

```text
Name:
svchost.exe

Path:
C:\Users\Public\svchost.exe
```

The second path is highly unusual for the legitimate Windows `svchost.exe` and deserves investigation.

However:

```text
Unusual Path
      !=
Confirmed Malware
```

The path is a strong clue that should be correlated with other evidence.

---

# Digital Signature

A digital signature can help establish information about the publisher and whether signed content has been altered since signing.

For Windows binaries, an expected Microsoft signature can increase confidence that the file is the expected signed binary.

But:

```text
Valid Signature
      !=
Benign Behavior
```

Task 5 already introduced why this matters with LOLBins.

A legitimate signed executable can still be used in suspicious activity.

---

# User and Security Context

A process runs in a security context.

Important questions include:

```text
Which user?
Which groups?
Which privileges?
Which integrity level?
Elevated or not?
```

This connects directly to Tasks 6 and 7.

For example:

```text
powershell.exe
User: CORP\alex
Integrity: Medium
```

has a different security context from:

```text
powershell.exe
User: CORP\alex
Integrity: High
```

Neither is automatically malicious. The difference helps the analyst understand what the process is capable of doing.

---

# Parent and Child Processes

Processes are commonly created by other processes.

This creates a **process tree**.

Example:

```text
explorer.exe
    |
    +--> cmd.exe
            |
            +--> whoami.exe
```

This could be completely normal interactive activity.

The relationship between processes can provide more context than individual process names.

## Suspicious Process Chains

Consider:

```text
WINWORD.EXE
     |
     v
powershell.exe
     |
     v
unknown.exe
```

This chain may deserve investigation because an Office document spawning a script interpreter that then launches another executable can be unusual depending on the environment and user activity.

Compare that with:

```text
explorer.exe
     |
     v
powershell.exe
     |
     v
Get-Process
```

which may represent ordinary administrative or user activity.

The key question is:

> **Does this parent-child relationship make sense for the application, user, and environment?**

---

# Parent-Process Heuristics Are Not Absolute Rules

It is useful to learn expected process relationships, but avoid turning them into rigid rules.

For example, analysts often examine the parent and context of `svchost.exe`.

A relationship that differs from what is normally expected can be a useful detection signal, but a single parent-process observation should not automatically determine the verdict.

Use process relationships as **heuristics**:

```text
Unexpected Parent
       |
       v
Investigate Further
```

not:

```text
Unexpected Parent
       |
       v
Automatically Malware
```

---

# Command-Line Analysis

The command line often explains **what a process was instructed to do**.

Compare:

```text
powershell.exe
```

with:

```text
powershell.exe -Command "Get-Process"
```

The second provides much more context.

A process name tells you:

> **What program is running?**

The command line can help answer:

> **What was that program asked to do?**

This is particularly important for interpreters and flexible Windows utilities such as PowerShell.

## Important Principle

```text
PowerShell
     !=
Malicious
```

Instead:

```text
PowerShell
     +
Command Line
     +
Parent Process
     +
User
     +
Subsequent Behavior
     =
Investigation Context
```

---

# Process Masquerading

**Masquerading** occurs when something attempts to appear legitimate in order to blend into an environment.

One possible form is using a name that resembles a legitimate Windows executable.

Example:

```text
Expected:
C:\Windows\System32\svchost.exe

Suspicious look-alike:
C:\Users\Public\svchost.exe
```

Useful checks include:

```text
Name
Path
Signature
Hash
Parent
Command Line
User
Behavior
```

## Masquerading vs LOLBin Abuse

Do not confuse the concepts:

```text
Masquerading
=
Something attempts to appear legitimate
```

while:

```text
LOLBin Abuse
=
A legitimate trusted binary is used in an unexpected or malicious way
```

This distinction was introduced in Task 5 and becomes especially useful during process investigation.

---

# `tasklist`

The command:

```cmd
tasklist
```

displays running processes and information such as their process IDs.

It is useful when a graphical interface is unavailable or when a quick command-line process inventory is needed.

### Analyst Perspective

`tasklist` is a starting point.

It can help answer:

```text
What processes are running?
What are their PIDs?
```

but deeper investigation usually requires additional context such as paths, command lines, users, process relationships, and telemetry.

---

# PowerShell: `Get-Process`

A useful basic process inventory is:

```powershell
Get-Process |
    Select-Object Id, ProcessName, Path, Company
```

This can provide:

```text
PID
Process Name
Executable Path
Company / Publisher-related metadata
```

### Limitation

This command does **not** provide the complete investigation context.

In particular, analysts often need additional sources to examine information such as:

```text
Command Line
Parent Process
User Context
Network Connections
File Activity
```

Therefore:

> **`Get-Process` is useful for triage, not a complete process investigation by itself.**

Also, access restrictions can prevent some process properties from being returned for every process.

---

# Retrieving Process Command Lines and Parent PIDs

PowerShell can query process information through CIM:

```powershell
Get-CimInstance Win32_Process |
    Select-Object ProcessId, ParentProcessId, Name, ExecutablePath, CommandLine
```

This provides several fields that are extremely useful during investigation:

```text
ProcessId
ParentProcessId
Name
ExecutablePath
CommandLine
```

This allows an analyst to begin reconstructing process relationships and execution context.

---

# Task Manager

Task Manager provides a quick graphical view of running applications and processes.

Useful information can include:

```text
Process name
PID
User
CPU
Memory
Disk
Network usage
```

Depending on the view and enabled columns, additional details can also be exposed.

## Security Perspective

Task Manager is useful for initial triage, but it should not be treated as a complete forensic or endpoint-detection platform.

A process that is not obviously suspicious in Task Manager may still require investigation through logs, EDR telemetry, Sysinternals tools, or other data sources.

---

# Sysinternals Process Explorer

Microsoft Sysinternals **Process Explorer** provides deeper process visibility than standard Task Manager for many investigative tasks.

Useful capabilities include:

- Parent/child process relationships
- Executable paths
- Digital-signature information
- Loaded DLLs
- Open handles
- Detailed process properties

Conceptually:

```text
Task Manager
     |
     v
Initial Process Triage
     |
     v
Process Explorer / EDR / Logs
     |
     v
Deeper Investigation
```

No single tool should be treated as sufficient for every investigation.

---

# Process Termination Is a Response Action

The command:

```cmd
taskkill /PID <PID_NUMBER> /F
```

can forcefully terminate a process by PID.

However, the original notes described killing a suspicious process immediately once identified. In professional incident response, that is not always the best first action.

Terminating a process can:

- Disrupt legitimate business activity.
- Remove volatile behavior that investigators were observing.
- Trigger malware recovery or persistence mechanisms.
- Affect evidence collection.
- Fail to remove the underlying persistence or root cause.

A stronger workflow is:

```text
Suspicious Process
       |
       v
Validate / Triage
       |
       v
Collect Relevant Evidence
       |
       v
Assess Impact and Urgency
       |
       v
Contain According to IR Procedure
       |
       v
Terminate / Remediate When Appropriate
```

If immediate harm is occurring, rapid containment may be necessary. The correct response depends on incident severity and organizational procedure.

---

# Process Investigation Example 1 — Suspicious Path

Observed:

```text
Name:
svchost.exe

PID:
4812

Path:
C:\Users\Public\svchost.exe
```

Do not stop at the name.

Investigate:

```text
svchost.exe
    |
    +--> Path unusual?
    +--> Signed?
    +--> Hash?
    +--> Parent?
    +--> Command line?
    +--> User?
    +--> Child processes?
    +--> Network connections?
    +--> File / registry activity?
    |
    v
Correlate Evidence
    |
    v
Verdict
```

The unusual path is a strong investigative lead, but the final conclusion should be evidence-based.

---

# Process Investigation Example 2 — Suspicious Parent/Child Chain

Observed:

```text
WINWORD.EXE
     |
     v
powershell.exe
```

This relationship may deserve investigation.

Now add the command line:

```text
powershell.exe -Command "Get-Process"
```

The command itself is ordinary.

The analyst should therefore continue investigating rather than labeling the event malicious solely because Word spawned PowerShell.

Questions include:

```text
Why did Word launch PowerShell?
Was a document or macro involved?
What user opened the document?
What happened afterward?
Did PowerShell create files?
Did it contact the network?
Did it spawn additional processes?
```

This demonstrates why:

> **Suspicious relationship + benign-looking command still requires context.**

---

# Process Investigation Example 3 — Behavior Chain

Consider:

```text
WINWORD.EXE
      |
      v
powershell.exe
      |
      v
Network Activity
      |
      v
New Executable
      |
      v
Child Process
```

This chain contains several related behaviors.

Instead of evaluating each event independently:

```text
Word
PowerShell
Network
File
Process
```

correlate them into a timeline:

```text
Document Opened
      |
      v
PowerShell Started
      |
      v
Network Connection
      |
      v
File Created
      |
      v
New Process Executed
```

The correlated sequence can be much more informative than any single event.

This is the beginning of **behavior-based threat hunting**.

---

# Basic Threat Hunting Mindset

Threat hunting is not simply:

> **"Look through Task Manager until something looks strange."**

A basic hunting mindset is:

```text
Hypothesis / Suspicious Pattern
          |
          v
Collect Relevant Telemetry
          |
          v
Find Deviations / Matches
          |
          v
Correlate Context
          |
          v
Validate
          |
          v
Escalate or Close
```

For this Windows fundamentals project, useful hunting questions include:

- Are system-like process names running from unexpected paths?
- Are unusual parent-child relationships present?
- Are script interpreters launched by unexpected applications?
- Are elevated processes appearing in unexpected user contexts?
- Are legitimate Windows utilities being used in unusual ways?
- Are suspicious process events followed by file or network activity?

---

# Connecting the Previous Tasks

Task 9 brings together many concepts from the repository:

```text
Task 3
Windows GUI / Security-Relevant Locations
        |
        v
Where can I find evidence?

Task 4
NTFS Permissions
        |
        v
Who can modify the executable?

Task 5
System32 / LOLBins / Masquerading
        |
        v
Is the binary legitimate and how is it being used?

Task 6
Users / Groups / Privileges
        |
        v
Which identity is involved?

Task 7
Tokens / Integrity / Elevation
        |
        v
What security context does the process have?

Task 8
Firewall / Network Access
        |
        v
Where is the process communicating?

Task 9
Process Investigation
        |
        v
Correlate Everything
```

The result is a basic investigation model:

```text
Process
   |
   +--> Name
   +--> PID
   +--> Path
   +--> Signature / Hash
   +--> User
   +--> Integrity / Privileges
   +--> Parent
   +--> Command Line
   +--> Children
   +--> Files / Registry
   +--> Network
   |
   v
Timeline + Context
   |
   v
Expected or Suspicious?
   |
   v
Evidence-Based Verdict
```

---

# Suspicious Does Not Mean Malicious

This principle appears throughout the project and is especially important in process analysis.

Examples:

```text
PowerShell running
       !=
Malware
```

```text
High-integrity process
       !=
Privilege escalation
```

```text
svchost.exe in an unusual path
       !=
Confirmed malware without investigation
```

```text
Connection to TCP 445
       !=
Lateral movement
```

The analyst's job is to combine evidence until there is enough context to make a defensible conclusion.

---

# Lessons Learned

- A process is a running instance of a program and has a PID and security context.
- Process names alone are weak evidence because names can be copied and legitimate tools can be abused.
- Executable path, signature, user, parent process, command line, child processes, file activity, and network activity provide stronger context.
- Parent-child relationships are useful heuristics, not absolute rules.
- Process masquerading and LOLBin abuse are different concepts.
- Task Manager and `tasklist` are useful for initial triage but do not provide complete investigative context.
- `Get-Process` is useful for process inventory, while CIM can expose fields such as parent PID and command line.
- Terminating a suspicious process is a response action and should be performed according to evidence, urgency, and incident-response procedure.
- Correlating multiple events into a timeline is more powerful than evaluating isolated indicators.
- Threat hunting is hypothesis- and behavior-driven rather than simply searching for strange process names.
- Suspicious behavior requires investigation; it is not automatically proof of malicious activity.

# Key Takeaway

Do not ask only:

> **"What processes are running?"**

Ask:

> **"Which process is running, from where, under which identity and privilege level, who launched it, what was it instructed to do, what did it do afterward, and does that complete behavior match the expected baseline?"**

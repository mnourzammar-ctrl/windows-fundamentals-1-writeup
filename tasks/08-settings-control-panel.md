# Task 8: Settings and the Control Panel

## Overview
This task introduces the Windows **Settings** application, the classic **Control Panel**, and Windows Defender Firewall.

For a security analyst, the important lesson is not simply knowing where a setting appears in the GUI. The goal is to understand:

> **Which security control is configured, what traffic or behavior it affects, and whether the configuration matches the organization's expected security baseline.**

---

## Question

- **Q:** In the Control Panel, change the view to Small icons. What is the last setting in the Control Panel view?
- **A:** `Windows Defender Firewall`

---

# Settings vs. Control Panel

Windows exposes configuration through multiple interfaces.

```text
Windows Configuration
        |
        +--> Settings
        |
        +--> Control Panel
        |
        +--> Management Consoles / Administrative Tools
        |
        +--> PowerShell / Command Line
```

The modern **Settings** application provides access to many common Windows configuration options.

The classic **Control Panel** continues to expose a number of traditional configuration interfaces.

More advanced administrative configuration may also be available through dedicated management consoles and PowerShell.

## Security Perspective

For an analyst, the important skill is not memorizing which GUI contains every option.

Instead:

```text
Security Question
      |
      v
Identify Relevant Control
      |
      v
Inspect Configuration
      |
      v
Compare With Expected Baseline
```

This is especially useful when reviewing firewall, networking, account, update, or system-security configuration.

---

# Windows Defender Firewall

A host firewall controls network traffic according to configured rules and profiles.

A simplified model is:

```text
Network Traffic
      |
      v
Windows Firewall
      |
   +--+--+
   |     |
 Allow  Block
   |
   v
Application / Service
```

Firewall decisions can depend on factors such as:

- Direction
- Protocol
- Local or remote port
- Local or remote address
- Application or service
- Active network profile
- Configured firewall rules

The firewall should therefore be understood as a **policy enforcement point**, not simply an ON/OFF switch.

---

# Firewall Profiles

Windows Firewall commonly uses three profiles:

```text
Domain
Private
Public
```

## Domain Profile

Used when Windows recognizes the network as associated with the organization's domain environment.

## Private Profile

Used for networks that are treated as trusted/private by configuration.

## Public Profile

Designed for less-trusted networks, such as public network environments.

### Why Profiles Matter

A firewall can behave differently depending on which profile is active.

Therefore, an analyst should not ask only:

> **"Is the firewall enabled?"**

Also ask:

```text
Which profile is active?
        |
        v
Which rules apply?
        |
        v
What traffic is allowed?
        |
        v
Is that expected for this endpoint?
```

---

# Inbound vs. Outbound Traffic

Understanding direction is fundamental to firewall analysis.

```text
Remote System
     |
     | inbound
     v
Your Windows Host
```

versus:

```text
Your Windows Host
     |
     | outbound
     v
Remote System
```

A rule allowing or blocking inbound traffic does not automatically apply to outbound traffic.

This distinction becomes important when investigating:

- Exposed services
- Remote administration
- Malware command-and-control activity
- Lateral movement
- Unexpected network connections

---

# Ports Do Not Equal Attacks

The original task highlights two security-relevant ports:

```text
TCP 445 -> commonly associated with SMB
TCP 3389 -> commonly associated with RDP
```

Both deserve attention in security monitoring, but an important correction is required:

```text
Port 445 open
      !=
Attack

Port 3389 open
      !=
Attack
```

SMB and RDP are legitimate technologies used in many enterprise environments.

The defensive goal is not:

> **"Block these ports everywhere."**

Instead:

> **"Allow required access only where it is justified, from appropriate sources, and according to the organization's network design and security policy."**

---

# SMB — TCP 445

SMB is commonly used for Windows file and resource sharing.

Examples of legitimate uses include:

```text
Workstation
     |
     v
File Server
     |
     v
SMB / TCP 445
```

SMB has also been involved in serious security incidents and can be relevant to lateral movement.

Therefore, the security question is:

```text
Who needs SMB?
     |
     v
Between which systems?
     |
     v
Is access appropriately restricted?
     |
     v
Is the observed SMB activity expected?
```

A workstation generally does not need unrestricted SMB connectivity to every other workstation simply because SMB is legitimate.

---

# RDP — TCP 3389

Remote Desktop Protocol allows remote interactive access to Windows systems when enabled and permitted.

Legitimate example:

```text
Authorized Administrator
          |
          v
Approved Administrative Host
          |
          v
RDP
          |
          v
Managed Server
```

Security problems arise when RDP is unnecessarily exposed, weakly protected, or used by unauthorized identities.

For investigation, useful questions include:

- Is RDP expected on this system?
- Which sources are allowed to connect?
- Which accounts are permitted?
- Was there a successful remote logon?
- Was the source system expected?
- What happened after the logon?

---

# Lateral Movement

**Lateral movement** describes activity where an attacker who has gained access to one system attempts to move to additional systems or resources in the environment.

Simplified:

```text
Initial Compromised Host
          |
          v
Credentials / Access / Remote Services
          |
          v
Second Host
          |
          v
Additional Systems
```

Remote services and protocols such as SMB and RDP can appear in lateral-movement scenarios, but their presence alone does not prove malicious movement.

The important defensive goal is to reduce unnecessary paths between systems.

---

# Network Segmentation and Access Restriction

Instead of thinking:

```text
445 / 3389
     |
     v
Block Everywhere
```

a stronger security model is:

```text
Business Requirement
        |
        v
Required Communication
        |
        v
Authorized Sources / Destinations
        |
        v
Firewall + Network Controls
        |
        v
Only Necessary Access
```

This follows the same least-privilege idea introduced in Task 6, but applied to network communication.

You can think of it as:

> **Least privilege for network access.**

---

# Command: Open Windows Defender Firewall

The classic firewall Control Panel interface can be opened with:

```cmd
control firewall.cpl
```

This is useful for quickly reaching the host firewall configuration.

For advanced firewall management, Windows also provides dedicated administrative interfaces such as Windows Defender Firewall with Advanced Security.

---

# Command: Open Control Panel

The Control Panel can be opened using:

```cmd
control /name Microsoft.ControlPanel
```

The security value is not the command itself. It simply provides a fast route to configuration interfaces that may be useful during administration or investigation.

---

# PowerShell: Inspect Firewall Profiles

PowerShell can inspect Windows Firewall profile configuration:

```powershell
Get-NetFirewallProfile |
    Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

This helps answer questions such as:

```text
Profile
   |
   +--> Enabled?
   +--> Default inbound behavior?
   +--> Default outbound behavior?
```

### Important Investigation Principle

Do not evaluate a firewall only from its default action.

Individual rules can create exceptions.

Conceptually:

```text
Default Policy
      +
Specific Rules
      +
Active Profile
      =
Effective Firewall Behavior
```

Therefore:

> **Firewall enabled does not automatically mean the endpoint is securely configured.**

---

# Inspecting Firewall Rules

A useful defensive next step is to inspect configured rules rather than immediately modifying them.

For example:

```powershell
Get-NetFirewallRule |
    Select-Object DisplayName, Enabled, Direction, Action
```

Rules can then be correlated with port filters and other configuration when deeper investigation is required.

The goal is to determine:

```text
What is allowed?
Why is it allowed?
For which direction/profile?
Is the rule expected?
```

---

# Why We Do Not Automatically Create a "Block SMB" Rule

A command can be used to create firewall rules programmatically, but automatically blocking TCP 445 on every endpoint is not a universal security recommendation.

In a real organization, SMB may be required for legitimate services.

A change made without understanding dependencies can disrupt business operations.

The safer administrative workflow is:

```text
Identify Exposure
      |
      v
Understand Business Requirement
      |
      v
Determine Authorized Communication
      |
      v
Design Restriction
      |
      v
Test
      |
      v
Deploy Through Approved Change Process
      |
      v
Monitor
```

This distinction is important for defensive professionals:

> **Security is not only about blocking things; it is about reducing risk while preserving required business functionality.**

---

# Practical SOC Investigation Example

Suppose an alert shows:

```text
Source:
WS-104

Destination:
WS-227

Destination Port:
445/TCP
```

Should the analyst immediately conclude that lateral movement occurred?

**No.**

Investigate:

```text
SMB Connection
      |
      +--> Is SMB expected between these hosts?
      +--> Which user/account was involved?
      +--> Was authentication successful?
      +--> What resource was accessed?
      +--> Is the source host already suspicious?
      +--> Were multiple hosts contacted?
      +--> What processes initiated related activity?
      +--> What happened before and after?
      |
      v
Correlate Evidence
      |
      v
Expected SMB Activity
      OR
Suspicious Lateral Movement
```

The port gives you **protocol context**.

It does not give you the final verdict.

---

# Practical RDP Investigation Example

Suppose telemetry shows a successful remote logon to a server.

Investigate:

```text
Successful Remote Logon
        |
        +--> Which account?
        +--> Source host / address?
        +--> Is the account allowed to use RDP?
        +--> Is the source an approved admin system?
        +--> What time did it occur?
        +--> What processes ran afterward?
        +--> Was there unusual account or file activity?
        |
        v
Expected Administration
        OR
Suspicious Remote Access
```

This connects Task 8 directly to Task 6:

```text
Remote Access
      +
User Identity
      +
Group Membership
      +
Authentication Activity
      =
Investigation Context
```

---

# Connecting Previous Tasks

The security model in the repository is now becoming connected:

```text
Task 4
File Permissions
      |
      v
Who can modify resources?

Task 5
System Binaries / DLLs
      |
      v
What code is running?

Task 6
Users / Groups
      |
      v
Who has access?

Task 7
Tokens / Elevation
      |
      v
With what privilege?

Task 8
Firewall / Network Access
      |
      v
Who can communicate with whom?
```

Together:

```text
Identity
   +
Privilege
   +
Host Access
   +
Network Access
   =
Attack Surface / Security Context
```

---

# Blue Team Investigation Model

When reviewing network access on a Windows endpoint:

```text
Connection / Service
        |
        v
Protocol + Port
        |
        v
Source + Destination
        |
        v
Firewall Profile / Rule
        |
        v
User / Process
        |
        v
Expected Business Purpose?
        |
        v
Correlate With Other Telemetry
        |
        v
Verdict
```

This is much stronger than simply memorizing:

```text
445 = SMB
3389 = RDP
```

---

# Lessons Learned

- Windows configuration can be accessed through Settings, Control Panel, management consoles, and PowerShell.
- Windows Defender Firewall controls traffic using profiles, rules, direction, protocol, ports, addresses, and other conditions.
- Domain, Private, and Public firewall profiles can apply different security policies.
- Inbound and outbound traffic are different directions and should be analyzed separately.
- TCP 445 is commonly associated with SMB and TCP 3389 with RDP, but an open or observed port does not automatically indicate malicious activity.
- SMB and RDP can be legitimate enterprise services as well as relevant protocols during attack investigations.
- Network access should be restricted according to business requirements rather than blocked blindly.
- Firewall state alone does not describe effective protection; active profiles and individual rules also matter.
- Lateral movement must be established from behavior and correlated evidence, not from a port number alone.
- Network segmentation and firewall restrictions can apply least-privilege principles to system-to-system communication.

# Key Takeaway

Do not ask only:

> **"Is this port open or blocked?"**

Ask:

> **"Which systems are allowed to communicate, why is that communication required, which identity and process are involved, and does the observed traffic match the expected network baseline?"**

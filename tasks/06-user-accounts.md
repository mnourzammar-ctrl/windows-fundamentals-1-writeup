# Task 6: User Accounts, Profiles, and Permissions

## Overview
This task introduces Windows local user accounts, local groups, account privileges, and the security principle of **least privilege**.

For a security analyst, identifying an account is only the beginning. The more important questions are:

> **Who is this user, what access do they have, is that access expected, and what has the account been doing?**

---

## Questions

1. **What is the name of the other user account?** → `tryhackmebilly`
2. **What groups is this user a member of?** → `Remote Desktop Users, Users`
3. **What built-in account is for guest access to the computer?** → `Guest`
4. **What is the account description?** → `Built-in account for guest access to the computer/domain`

---

# Users, Groups, and Privileges

These concepts are related, but they should not be treated as identical.

```text
User Account
     |
     +--> Group Memberships
     |
     +--> Assigned Rights / Privileges
     |
     +--> Access to Resources
```

A **user account** represents an identity.

A **group** allows permissions and rights to be assigned to multiple identities more efficiently.

A **privilege/user right** represents the ability to perform certain operating-system-level actions.

For example, knowing that an account belongs to `Remote Desktop Users` provides useful information about its intended access, but it does not by itself describe every permission, privilege, or action available to that user.

---

# Principle of Least Privilege (PoLP)

The **Principle of Least Privilege** means that users, services, and applications should receive only the access required to perform their legitimate functions.

Conceptually:

```text
Required Job Function
        |
        v
Minimum Necessary Access
        |
        v
Reduced Security Exposure
```

For example, a user who only needs normal workstation access should not automatically receive local administrator rights.

## Why It Matters

If an account is compromised, the attacker's immediate capabilities are influenced by the access available to that account.

```text
Compromised Account
        |
        v
Available Permissions / Rights
        |
        v
Potential Impact
```

A compromised administrator account generally creates a much more serious security situation than a compromised standard account.

However, a standard user compromise should **not** be considered harmless. Attackers may still be able to:

- Access data available to that user.
- Execute programs in the user's context.
- Steal user-accessible credentials or session material.
- Establish persistence within accessible locations.
- Attempt privilege escalation.
- Access network resources available to that identity.

Therefore, least privilege does not prevent compromise. It helps **limit what a compromised identity can immediately do**.

---

# Local Accounts vs Domain Accounts

Windows environments can contain different identity scopes.

### Local Account

A local account is managed by an individual Windows computer.

Conceptually:

```text
Computer A
   |
   +--> Local User
```

Its identity and local group memberships apply primarily to that machine.

### Domain Account

In an Active Directory environment, identities can be centrally managed through the domain.

```text
Active Directory Domain
          |
          +--> User
          |
          +--> Groups
          |
          +--> Managed Computers
```

This distinction matters during investigation because an analyst should determine whether an observed identity is:

```text
Local
  or
Domain / Centrally Managed
```

That affects where the account is managed, what access may be available, and which systems should be investigated.

---

# Built-In Accounts

Windows includes built-in accounts for specific operating-system purposes.

## Guest

The built-in `Guest` account exists for guest-style access and is normally disabled in modern Windows configurations.

During an investigation, an unexpectedly enabled or actively used Guest account may deserve review against the organization's expected configuration.

But:

```text
Guest account exists
       !=
Compromise
```

The existence of a built-in account is normal. Its **state and usage** provide the useful security context.

## WDAGUtilityAccount

Some Windows systems may contain `WDAGUtilityAccount`, associated with Windows Defender Application Guard functionality.

The important defensive lesson is broader than memorizing individual built-in account names:

> **Learn which accounts are expected on the system before treating an unfamiliar account as malicious.**

---

# The SAM Database

Windows stores information about local accounts in the **Security Account Manager (SAM)** database.

The SAM registry hive is associated with:

```text
C:\Windows\System32\config\SAM
```

Among other account-related information, local credential verification data includes password-derived hashes rather than plaintext passwords.

## Important Distinction: SAM vs LSASS

These concepts should not be mixed together.

### SAM

Think:

```text
SAM
 |
 +--> Local account database
 +--> Local credential/hash-related data
```

### LSASS

**LSASS (Local Security Authority Subsystem Service)** is a Windows security process involved in authentication and security-policy enforcement. Depending on the authentication activity and protections present, its memory may contain credential-related material.

Conceptually:

```text
SAM
= stored local account / credential data

LSASS
= running security/authentication process
```

They are related to Windows authentication, but they are **not the same credential source**.

This distinction matters when analyzing credential-access activity.

---

# Password Cracking vs Pass-the-Hash

A stolen password hash can be relevant to different attack techniques, but two concepts should be clearly separated.

## Password Cracking

The goal is to recover or guess the underlying password.

```text
Password Hash
      |
      v
Offline Guessing / Cracking
      |
      v
Possible Plaintext Password
```

## Pass-the-Hash

In environments and protocols where it is applicable, an attacker may attempt to authenticate using stolen NTLM credential material without first recovering the plaintext password.

```text
Stolen NTLM Hash
       |
       v
Authentication Attempt
       |
       v
No Plaintext Password Recovery Required
```

### Key Difference

```text
Password Cracking
= try to recover the password

Pass-the-Hash
= use applicable stolen hash material for authentication
```

For defenders, credential theft can therefore remain dangerous even when the attacker never learns the user's plaintext password.

---

# Local Group Membership

Group membership can significantly change what an account is allowed to do.

Security-relevant local groups can include groups such as:

```text
Administrators
Remote Desktop Users
Remote Management Users
```

The exact security impact depends on the environment, configuration, and other controls.

## Why Group Changes Matter

Suppose an ordinary account unexpectedly appears in:

```text
Administrators
```

or:

```text
Remote Desktop Users
```

The correct response is not immediately:

> "The attacker created persistence."

Instead:

```text
Unexpected Membership
        |
        v
Was the change authorized?
        |
        v
Who made the change?
        |
        v
When did it happen?
        |
        v
What activity followed?
        |
        v
Expected Administration
        OR
Suspicious Account Manipulation
```

An unauthorized group-membership change can be highly significant, but context and evidence are still required.

---

# Command: `net user`

Lists local user accounts:

```cmd
net user
```

### Analyst Questions

When reviewing the account list, consider:

- Are all accounts expected?
- Are there unfamiliar accounts?
- Are built-in accounts enabled unexpectedly?
- Do naming conventions match the organization?
- Are there accounts that should have been disabled or removed?

The command provides a starting point for account investigation rather than a verdict.

---

# Command: `net user <username>`

Displays information about a specific local account.

Example from this lab:

```cmd
net user tryhackmebilly
```

Useful information can include account status, password-related settings, logon information, and local group memberships.

For investigation, ask:

```text
Account
   |
   +--> Enabled?
   +--> Group memberships?
   +--> Last logon information?
   +--> Password/account settings?
   +--> Expected user?
```

---

# Command: `net localgroup Administrators`

Displays members of the local `Administrators` group:

```cmd
net localgroup Administrators
```

This is particularly useful during a security review because local administrator membership provides significant control over the endpoint.

### Blue Team Perspective

The important question is:

> **Does every account in this group have a legitimate reason to be there?**

Unexpected membership should be validated against the organization's approved administrative access.

---

# Command: `whoami /priv`

Displays privileges associated with the current access token:

```cmd
whoami /priv
```

The output can include privileges such as:

```text
SeChangeNotifyPrivilege
SeShutdownPrivilege
SeDebugPrivilege
SeImpersonatePrivilege
```

Do not assume that seeing the name of a powerful privilege automatically means it can be exploited.

An analyst should consider:

```text
Privilege
    |
    +--> Present?
    +--> Enabled / Disabled?
    +--> Which process/token?
    +--> Is it expected for this account?
    +--> Is there an applicable abuse path?
```

This reinforces an important principle:

> **A security-relevant privilege is context to investigate, not automatic proof of privilege escalation.**

---

# PowerShell: `Get-LocalUser`

PowerShell can return local accounts as structured objects:

```powershell
Get-LocalUser |
    Select-Object Name, Enabled, LastLogon
```

This is useful for a quick account inventory.

Structured PowerShell output also makes it easier to filter, sort, or automate account reviews later.

For example, the analyst can focus on:

```text
Name
Enabled
LastLogon
```

while remembering that these fields alone do not provide a complete account-activity history.

---

# PowerShell: `Get-LocalGroupMember`

To inspect membership of a local group:

```powershell
Get-LocalGroupMember -Group "Remote Desktop Users"
```

This can help identify which identities have been granted membership in that group.

### Security Perspective

An unexpected account in `Remote Desktop Users` can be an important investigative lead.

However:

```text
Remote Desktop Users membership
            !=
Proof that RDP was used
```

Group membership shows an access-related configuration.

To determine whether remote logon actually occurred, an analyst would need additional evidence such as relevant Windows logs and other endpoint/network telemetry.

This distinction is important:

```text
Permission / Capability
        !=
Observed Activity
```

---

# Practical SOC Investigation Example

Suppose an endpoint investigation reveals:

```text
Account:
backup_support

Account status:
Enabled

Local groups:
Users
Remote Desktop Users
```

The account name is unfamiliar to the analyst.

Do not immediately conclude that it is malicious.

Investigate:

```text
Unfamiliar Account
       |
       +--> When was it created?
       +--> Is it approved?
       +--> Who or what created it?
       +--> When was it enabled?
       +--> When were group memberships changed?
       +--> Has it logged on?
       +--> From where?
       +--> What processes ran under the account?
       +--> What resources did it access?
       |
       v
Correlate Evidence
       |
       v
Expected Account
       OR
Suspicious / Unauthorized Account
```

---

# Account Investigation Model

A useful mental model is:

```text
Identity
   |
   v
Account State
   |
   v
Group Membership
   |
   v
Privileges / Rights
   |
   v
Authentication / Logon Activity
   |
   v
Processes and Resource Access
   |
   v
Compare With Expected Baseline
   |
   v
Verdict
```

This is more useful than simply asking whether an account exists.

---

# Security Configuration vs Security Activity

Task 6 introduces another distinction that is important for SOC work:

```text
Account exists
Group membership exists
Privilege exists
        |
        v
Configuration / Capability
```

versus:

```text
Account logged on
Process executed
Resource accessed
Group membership changed
        |
        v
Activity / Evidence
```

For example:

> An account being a member of `Remote Desktop Users` tells you about a capability or configuration.

It does **not** by itself prove that the account successfully connected through RDP.

---

# Lessons Learned

- User accounts represent identities, while groups simplify the assignment of access and rights.
- Least privilege limits unnecessary access and can reduce the immediate impact of account compromise.
- Standard-user compromise can still be serious and may lead to credential theft, persistence, data access, or privilege-escalation attempts.
- Local and domain accounts have different identity scopes and management models.
- Built-in accounts should be evaluated based on expected state and usage rather than their existence alone.
- The SAM database stores local account information and password-derived credential data; it should not be confused with LSASS memory.
- Password cracking and Pass-the-Hash are different concepts.
- Group membership can expose important access capabilities but does not prove those capabilities were used.
- `whoami /priv` shows token privileges, but the presence of a privilege alone does not prove an exploitable path.
- Account investigations should combine configuration, authentication activity, process activity, and organizational context.

# Key Takeaway

Do not ask only:

> **"Which users exist on this computer?"**

Ask:

> **"Which identities exist, what access do they have, is that access expected, and is there evidence that those identities or permissions were used suspiciously?"**

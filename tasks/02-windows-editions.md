# Task 2: Windows Editions

## Overview
Covers the different Windows editions — Home, Pro, Enterprise, Server — and what actually separates them beyond marketing.

## Security-relevant differences
Home is missing a handful of features that matter a lot once you're managing more than one machine:
- **BitLocker** — full-disk encryption isn't available.
- **Group Policy Objects (GPO)** — no centralized policy management (password rules, permissions, update behavior, etc.).
- **Active Directory domain join** — Home machines can't join a domain at all.

Pro and Enterprise exist specifically to close that gap, since any organization managing more than a handful of endpoints needs centralized control rather than configuring each machine by hand.

### Why this matters in practice
A lot of "why is this machine unmanaged" incidents trace back to someone deploying a Home edition machine into an environment that assumed domain-joined, policy-managed endpoints. Knowing which edition you're dealing with tells you upfront what controls are even possible.

## Commands

### `systeminfo`
Prints a full system report — OS name, build number, install date, CPU, memory, and installed hotfixes. This is usually one of the first things run after landing on a box during post-exploitation recon, since the build number and patch level often point straight at known, unpatched vulnerabilities.

```cmd
systeminfo
```

### `Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsHardwareAbstractionLayer`
`Get-ComputerInfo` on its own returns dozens of properties, most of which you don't need. Piping it into `Select-Object` with specific property names trims that down to just the edition name, version, and HAL layer. This pipe-then-filter pattern (cmdlet → `|` → `Select-Object`) is the standard way to pull specific fields out of a large PowerShell object instead of scrolling through everything.

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsHardwareAbstractionLayer
```

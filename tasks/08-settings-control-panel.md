# Task 8: Settings and the Control Panel

## Question
- **Q:** In the Control Panel, change the view to Small icons. What is the last setting in the Control Panel view?
- **A:** `Windows Defender Firewall`

## Security-relevant concepts

### Settings vs. Control Panel
Modern Windows splits configuration across two interfaces: the newer **Settings** app (simpler, aimed at everyday users) and the legacy **Control Panel** (more advanced, and still the only place to reach some settings — advanced firewall configuration among them). Knowing both matters because some of the more critical security settings still live only in Control Panel or tools like `wf.msc`.

### Cutting off lateral movement
Blocking known high-risk ports at the firewall level:
- **445 (SMB)** — the protocol behind large-scale worm outbreaks like WannaCry and NotPetya, which spread automatically machine-to-machine.
- **3389 (RDP)** — a frequent target for brute-force attempts and unauthorized remote access.

Blocking these on internal network traffic (not just from the internet) is one of the more effective ways to stop a compromised machine from spreading to the rest of the network.

## Commands

### `control firewall.cpl`
Opens the classic **Windows Defender Firewall** control panel directly (`.cpl` files are Control Panel applet executables). Faster than navigating menus manually.

```cmd
control firewall.cpl
```

### `control /name Microsoft.ControlPanel`
Opens the Control Panel home page using its canonical name rather than jumping to a specific applet — a general starting point for navigating settings.

```cmd
control /name Microsoft.ControlPanel
```

### `Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction`
1. `Get-NetFirewallProfile` pulls the settings for each firewall profile: **Domain**, **Private**, **Public**.
2. `Select-Object` narrows it down to profile name, whether it's enabled, and the default behavior for inbound/outbound traffic.

A quick first check in any security review — confirming the firewall is actually on across all three profiles, and that inbound traffic defaults to blocked rather than allowed.

```powershell
Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

### `New-NetFirewallRule -DisplayName "Block SMB Inbound" -Direction Inbound -LocalPort 445 -Protocol TCP -Action Block`
Creates a new firewall rule programmatically:
- `-DisplayName` — a readable name for the rule.
- `-Direction Inbound` — applies only to incoming connections.
- `-LocalPort 445` — targets SMB specifically.
- `-Protocol TCP` — restricts it to TCP.
- `-Action Block` — drops matching traffic outright.

The net effect: any inbound TCP connection attempt on port 445 gets blocked at this machine — a direct defensive move against SMB-based exploits like EternalBlue, or against worms trying to spread via that protocol.

```powershell
New-NetFirewallRule -DisplayName "Block SMB Inbound" -Direction Inbound -LocalPort 445 -Protocol TCP -Action Block
```

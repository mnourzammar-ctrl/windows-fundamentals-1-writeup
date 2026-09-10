# Task 3: Desktop (GUI)

## Overview
Walks through the basic GUI elements: the Start menu, taskbar, Action Center, and the Run dialog.

## Why it's worth knowing cold
### Speed during incident response
Knowing shortcuts and quick-access paths saves real time when you're in the middle of investigating a live incident — pulling up Event Viewer, Task Manager, or a PowerShell console fast matters more than it sounds like it would, especially when you're trying to catch something before it moves or covers its tracks.

### Watching Action Center
Action Center notifications (Defender alerts, firewall prompts) are sometimes the first visible sign that something's wrong. A lot of infections get noticed first through a security balloon notification rather than any deliberate monitoring.

## Command

### `Win + R`
Opens the Run dialog directly, which lets you launch any program or command by name (`cmd`, `powershell`, `regedit`, `services.msc`) without going through Start.

It's also worth flagging from an attacker's perspective: Run is a common way to execute something quickly right after initial access — for example, launching `powershell -enc <base64 payload>` straight from Run instead of a full terminal session. Some hardened environments restrict or log Run dialog usage through GPO for exactly this reason.

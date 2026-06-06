---
title: "Processes and Their Misuse (csrss.exe, smss.exe, System Idle Process)"
date: 2026-06-06 12:00:00 +0200
categories: [Interesting Topics]
tags: [windows-internals, processes, malware-analysis, threat-hunting, csrss, smss, dfir, blue-team]
---

## 1. csrss.exe (Client Server Runtime Process)

The **Client Server Runtime Process (csrss.exe)** is a critical system process in Windows, responsible for handling the user-mode portion of the Win32 subsystem. Previously, it also managed window drawing and graphics rendering, but modern versions of Windows have moved most of these functions elsewhere. The executable is located at:

`%windir%\System32\csrss.exe`

### Role and Importance

Marked as a **critical system process**, **csrss.exe** is essential for the operating system. If terminated, it results in a system crash (BSOD). Each user session in Windows **has at least one instance of csrss.exe**, meaning at a minimum, there will be two instances running — one for **Session 0** (system services) and another for **Session 1** (first user session).

Instead of directly accessing kernel functions like **kernel32.dll**, Windows applications communicate with **csrss.exe** using "[**Inter-Process Communication (IPC)**](https://www.geeksforgeeks.org/inter-process-communication-ipc/)**"**. This improves security by reducing direct kernel exposure.

#### Example: Creating a Process (Notepad.exe)

1. When you open **Command Prompt (cmd.exe)** and type `notepad.exe`, Windows initiates the process.
2. Instead of directly invoking the kernel, the Win32 library (kernel32.dll) sends an **IPC request** to **csrss.exe**
3. **csrss.exe** handles the request and creates the **Notepad.exe** process, reducing security risks.

### Types of Process

- **Independent process:** An independent process is not affected by the execution of other processes. Independent processes are processes that do not share any data or resources with other processes. No inte-process communication required here.
- **Co-operating process:** Interact with each other and share data or resources. A co-operating process can be affected by other executing processes. Inter-process communication (IPC) is a mechanism that allows processes to communicate with each other and synchronize their actions. The communication between these processes can be seen as a method of cooperation between them.

### Misuse and Detection

Attackers often exploit **csrss.exe** to evade detection by imitating its behavior or creating processes with similar names.

#### Indicators of Malicious Activity:

- **Excessive CPU or RAM usage** without legitimate system activity.
- **Fake processes** such as `csrrs.exe` or `csrsr.exe`, which are not legitimate Windows processes.
- **Graphical glitches** or degraded GPU performance, indicating that a rogue **csrss.exe** is interfering with the system.

#### How to Check for Legitimacy

1. **Verify Digital Signatures**:

- Open **Task Manager** → Locate **csrss.exe** → Right-click → **Properties** → **Digital Signatures**.
- Check the **publisher** and cross-verify the hash on **VirusTotal** or any CTI platform.

**2. Identify Unusual Sessions**:

- The legitimate **csrss.exe** should only exist **once per user session**.
- Run `tasklist /v` in **Command Prompt** and check if any instance is running under a **non-SYSTEM** user.

More details on inspecting **csrss.exe** can be found in this article: [**Recoverit Guide**](https://recoverit.wondershare.com/windows-computer-tips/csrss-exe.html)**.**

## 2. smss.exe (Session Manager Subsystem)

The **Session Manager Subsystem (smss.exe)** is the first user-mode process executed by the system, located at:

`%SystemRoot%\System32\smss.exe`

### Role and Functions

Configures the **environment** during startup using the registry key:

- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager`

Maps **DOS devices** like `AUX`, `CON`, and `PIPE` based on:

- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\DOS Devices`

Loads subsystems configured under:

- `HKLM\System\CurrentControlSet\Control\Session Manager\SubSystems`

Launches essential processes such as **Winlogon.exe** and **csrss.exe**.

Manages **environment variables** at:

- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Environment`

Plays a role in **Remote Desktop Protocol (RDP) session creation**.

A **legitimate** smss.exe should have:

- **Only one instance running (much importnat)**
- **No child processes**
- **Parent Process ID (PPID) of 4**

### Misuse and Threats

Attackers can exploit **smss.exe** for malicious purposes, such as executing **Remote Access Trojans (RATs)**. A real-world example is documented in this [Any.Run report](http://any.run/report/24b0e23df17c77d44882a2e25ecbd4d3b07015af5d44cb325679a370b8304614/edaaef85-9936-42c3-a163-c217d4f8330f).

#### Indicators of Compromise (IoCs):

**1- Unexpected file locations:**

- `c:\users\admin\appdata\local\temp\smss.exe`

**2- Unusual parent processes:**

- If `smss.exe` is launched by `explorer.exe`, it is likely malicious (**User Execution**)

**3- Hashes of known malware:**

- MD5: `97A2185F37CDD11207322E349B344FB7`
- SHA1: `DE2933539CAAC225CD11768D192BB97467E67010`

### How to Detect Malicious smss.exe

- Use **Process Explorer** or **Task Manager** to check for multiple running instances.
- Run `tasklist /fi "imagename eq smss.exe"` in Command Prompt.
- Inspect unusual command-line executions (`cmd.exe`, `ping`, etc.).

## 3. System Idle Process

The **System Idle Process** is the **first** process executed by the kernel. Its primary purpose is to **occupy CPU cycles** when no other tasks are running.

### Role and Importance

- Prevents scheduler exceptions by ensuring there is always a **runnable** process.
- **Contains kernel threads** that run when no other threads are active.
- On **multiprocessor systems**, each CPU has its own **idle thread**.
- Uses **CPU time** without consuming actual resources.

#### Example:

If a process is consuming **30% CPU**, the System Idle Process will use the remaining **70%**, indicating **free CPU availability**.

### Security Considerations

- **Not typically targeted by malware**, as it does not interact with system security features.
- High CPU usage of **System Idle Process** (e.g., **99%**) is **normal** and does not indicate an issue.

## Resources:

- [Inter Process Communication (IPC) - GeeksforGeeks](https://www.geeksforgeeks.org/inter-process-communication-ipc/)
- <https://www.howtogeek.com/411569/what-is-system-idle-process-and-why-is-it-using-so-much-cpu/>
- <https://www.pcrisk.com/removal-guides/14914-csrss-exe-virus>
- <https://recoverit.wondershare.com/windows-computer-tips/csrss-exe.html>

---

> _Originally published on [Medium](https://medium.com/@mohamedabdullh116/processes-and-their-misuse-csrss-exe-smss-exe-system-idle-process-b20b956b1b4a)._
{: .prompt-info }

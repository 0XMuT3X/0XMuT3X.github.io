---
title: "MITRE ATT&CK \"Top 10\" Initial Access Techniques Explained Simply"
date: 2025-01-19 19:14:46 +0200
categories: [APT Hunting]
tags: [mitre-attack, initial-access, ttps, threat-intel, phishing, supply-chain, valid-accounts, soc, blue-team]
image:
  path: /assets/img/posts/mitre-top10-initial-access/01.jpg
  alt: "MITRE ATT&CK Top 10 Initial Access techniques"
---

In this blog, we will explore the different Initial Access techniques used by adversaries to gain a foothold in a target environment.

We'll break down how each attack technique works, provide real-world examples, and cover the mitigations and detection methods for each.

Let's dive into the most common and impactful Initial Access attack techniques.

ls go :)

![MITRE ATT&CK Initial Access overview](/assets/img/posts/mitre-top10-initial-access/02.png)

![Initial Access techniques map](/assets/img/posts/mitre-top10-initial-access/03.png)

## Technique #1: Content Injection

**Definition**:
Content injection is a form of cyberattack in which adversaries inject malicious data into a communication stream between a client and a server. This allows the attacker to manipulate or control the content being delivered, potentially compromising the victim's system.

**How It Happens**:

1. **Man-in-the-Middle (MITM)**: In this scenario, the attacker intercepts and modifies data between the client (user) and server (web application or website). For example, if you are visiting an online banking site, the attacker might change the transaction details you submit (such as altering the recipient's bank account number).
2. **Side-channel Injection**: Here, the attacker sends malicious responses to a client's request. For instance, an attacker could trick a victim's browser by sending fake files or resources in response to an HTTP request made by the victim's browser, leading them to unknowingly download malware.

**Example**:
A common example of content injection is the **Cross-Site Scripting (XSS)** attack, where an attacker injects malicious JavaScript into a website. When a user visits the page, the script executes and may steal their session cookies, redirect them to phishing sites, or perform unwanted actions on their behalf.

![Content injection example](/assets/img/posts/mitre-top10-initial-access/04.png)

**Mitigations**:

- **Encrypt Sensitive Information**: Always use HTTPS (SSL/TLS encryption) for communication between clients and servers to prevent attackers from intercepting data.

![HTTPS encryption](/assets/img/posts/mitre-top10-initial-access/05.jpg)

![TLS protected traffic](/assets/img/posts/mitre-top10-initial-access/06.jpg)

- **Restrict Web Content and Scan**: Use a **Web Application Firewall (WAF)** to filter out malicious scripts and disallow dangerous content from being loaded into websites.

Check This => [https://docs.aws.amazon.com/waf/latest/developerguide/waf-incident-response.html](https://docs.aws.amazon.com/waf/latest/developerguide/waf-incident-response.html)

**Detection Methods**:

- **File Monitoring**: Monitor files being downloaded or executed to detect any unexpected or unauthorized content.
- **Network Traffic Monitoring**: Look for signs of MITM activity, such as altered SSL/TLS certificates or inconsistent traffic patterns.
- **Process Monitoring**: Track browser processes or web servers for unusual outbound connections that may indicate a malicious script execution.

![Content injection detection](/assets/img/posts/mitre-top10-initial-access/07.png)

## Technique #2: Drive-by Compromise

**Definition**:
A drive-by compromise occurs when a user's system is compromised simply by visiting a malicious or compromised website. This typically happens without the user's knowledge or any user action, other than visiting a page.

**How It Happens**:

1. **Malicious Website**: A user unknowingly visits a website that hosts malware. The website may be a legitimate site that has been compromised, or a site specifically created to spread malware.

![Malicious website](/assets/img/posts/mitre-top10-initial-access/08.jpg)

![Drive-by download flow](/assets/img/posts/mitre-top10-initial-access/09.png)

![Compromised site delivering malware](/assets/img/posts/mitre-top10-initial-access/10.png)

Check this => [https://www.memcyco.com/7-ways-to-quickly-detect-malicious-websites/](https://www.memcyco.com/7-ways-to-quickly-detect-malicious-websites/)

**2. Exploiting Browser Vulnerabilities**: If the user's browser or plugins are outdated, attackers may exploit these vulnerabilities to install malware. A good example of this is **Flash-based** malware, which used to take advantage of security holes in Flash Player before it was phased out.

Check OWASP Top 10 => [https://owasp.org/www-project-top-ten/](https://owasp.org/www-project-top-ten/)

**Example**:
An example of a drive-by compromise is the **Flash Zero-Day** attacks, where attackers inject malware into a website. If the user's browser has an outdated version of Flash Player, it automatically runs the exploit, compromising the user's system.

![Flash zero-day exploit](/assets/img/posts/mitre-top10-initial-access/11.png)

**Mitigations**:

- **Sandboxing and Application Isolation**: Use technologies like **browser sandboxes** or security-focused browsers to isolate the exploitation from the rest of the system.
- **Exploit Protection**: Tools such as **Windows Defender Exploit Guard (WDEG)** or **EMET** help block known exploitation techniques.

**Detection Methods**:

- **Similar to Content Injection**: Monitor network traffic for abnormal data flows, check for processes related to web browsers making suspicious connections, and review logs for unusual web access patterns.

![Drive-by compromise detection](/assets/img/posts/mitre-top10-initial-access/12.png)

## Technique #3: Exploit Public-Facing Applications

**Definition**:
Public-facing applications are systems accessible from the internet, like websites, web applications, or even services like email or APIs. Attackers often exploit these to gain access to internal systems.

**How It Happens**:
Adversaries look for vulnerabilities in the code of publicly accessible applications. Common attack vectors include **SQL Injection (SQLi)**, **Cross-Site Scripting (XSS)**, and **Remote Code Execution (RCE)**. Exploiting these weaknesses can grant the attacker unauthorized access to the system.

**Example**:
In 2017, the **Equifax** breach occurred because attackers exploited a vulnerability in **Apache Struts**, a widely used web application framework. The vulnerability allowed attackers to execute commands remotely on Equifax's public-facing servers and steal sensitive personal data of over 147 million people.

**Mitigations**:

- **Sandboxing**: Limit the potential damage of successful exploits by isolating vulnerable applications.
- **Web Application Firewall (WAF)**: A WAF can detect and block known attack patterns, such as SQL injection or XSS.
- **Regular Vulnerability Scanning**: Regularly scan applications for known vulnerabilities and patch them promptly.

![Exploiting a public-facing app](/assets/img/posts/mitre-top10-initial-access/13.png)

![Web application exploitation](/assets/img/posts/mitre-top10-initial-access/14.png)

**Detection Methods**:

- **App Logs**: Monitor for unusual activity or error logs that could indicate exploitation attempts.
- **Network Traffic Analysis**: Identify unusual traffic patterns such as unexpected API calls or suspicious query strings in URLs.

![Public-facing app exploitation detection](/assets/img/posts/mitre-top10-initial-access/15.png)

## Technique #4: Exploit External Remote Services

**Definition**:
External remote services include APIs, VPNs, and cloud services that organizations use to facilitate remote access. Exploiting these services can give attackers entry into internal systems.

**How It Happens**:
Attackers exploit weak or compromised credentials, or take advantage of exposed services that don't require authentication, such as a **VPN** with weak or default credentials.

**Example**:
In **2017**, the **Cloudbleed** incident exposed sensitive data due to a bug in Cloudflare's reverse proxy service. Attackers could exploit this vulnerability to access internal data from websites using Cloudflare services.

**Mitigations**:

- **Multi-Factor Authentication (MFA)**: Use MFA for all remote access points to reduce the likelihood of credential theft.
- **Network Segmentation**: Limit the access of remote services to only necessary systems.

**Detection Methods**:

- **App Logs**: Look for signs of unauthorized access or failed login attempts for remote services.
- **Network Traffic Analysis**: Monitor VPN traffic for suspicious connections or unrecognized IP addresses.

![External remote services exploitation](/assets/img/posts/mitre-top10-initial-access/16.png)

## Technique #5: Hardware Additions

**Definition**:
This technique involves attackers physically introducing unauthorized hardware devices, such as USB sticks or other peripherals, into the target network. These devices can be used to deliver malware or enable further attacks.

Check this => [https://www.infosecinstitute.com/resources/mitre-attck/mitre-attck-hardware-additions/](https://www.infosecinstitute.com/resources/mitre-attck/mitre-attck-hardware-additions/)

**How It Happens**:
Attackers can plant malicious USB drives or network devices within an organization. Once plugged in, these devices can exploit vulnerabilities, steal data, or spread malware.

**Example**:
In the **Stuxnet** attack, a USB drive was used to spread the malware to Iran's nuclear program. The drive carried a payload that infected the industrial control systems when inserted into a machine within the network.

**Mitigations**:

- **Limit Hardware Installation**: Only allow authorized devices to connect to the network.
- **Endpoint Security**: Use software to monitor and block unauthorized devices.

**Detection Methods**:

- **Device Logs**: Monitor for new device connections or any attempt to access restricted areas of the network.
- **Network Traffic Monitoring**: Analyze unusual traffic that may indicate a new device sending out malicious data.

![Hardware additions](/assets/img/posts/mitre-top10-initial-access/17.png)

## Technique #6: Phishing

**Definition**:
Phishing is a form of social engineering where adversaries attempt to deceive individuals into disclosing sensitive information, clicking malicious links, or downloading malware, often through email.

**How It Happens**:
An attacker may send a fraudulent email that looks legitimate. The email may contain a malicious attachment or link, leading the victim to a fake login page to steal credentials or install malware on their device.

**Example**:
The **Google Docs Phishing Attack** in 2017 tricked users into granting an attacker access to their email contacts by sending emails that appeared to come from someone in their contacts list.

**Mitigations**:

- **Antivirus and Anti-Malware Tools**: These can block known phishing links and attachments.
- **Email Filtering**: Implement filtering rules to detect phishing messages using suspicious sender addresses or misleading URLs.

**Detection Methods**:

- **App Logs**: Monitor for any failed login attempts or signs that an employee's credentials have been compromised.
- **File Creation Logs**: Watch for files created or modified by phishing emails.

![Phishing](/assets/img/posts/mitre-top10-initial-access/18.png)

## Technique #7: Replication Through Removable Media

**Definition**:
Malware may be spread through **removable media** (e.g., USB drives), exploiting the **autorun** feature to execute malicious code automatically when connected to a system.

**How It Happens**:
The attacker places malware on a USB drive, which, when connected to a system with the autorun feature enabled, executes automatically, infecting the system.

**Example**:
In the **Conficker** worm attack, USB drives were used to spread the infection across networks, exploiting the autorun feature in older versions of Windows.

**Mitigations**:

- **Disable Autorun**: Disable the autorun feature on all systems to prevent automatic execution of files from USB drives.
- **Endpoint Protection**: Use endpoint security software to detect malicious files or processes triggered from external media.

**Detection Methods**:

- **File Access Logs**: Track file access from removable media devices.
- **Process Monitoring**: Watch for unusual processes spawned by USB devices.

![Replication through removable media](/assets/img/posts/mitre-top10-initial-access/19.png)

## Technique #8: Supply Chain Compromise

**Definition**:
Supply chain attacks target the systems or products that organizations depend on, compromising them before they reach the target, often through legitimate software or hardware.

**How It Happens**:
An adversary may manipulate the software or hardware during its development or distribution. This could involve altering code in development tools, injecting malware into software packages, or substituting legitimate software with compromised versions.

**Example**:
In 2020, the **SolarWinds** attack involved hackers inserting malicious code into a software update for the company's network management products. The compromised updates were then distributed to thousands of organizations, including U.S. government agencies.

**Mitigations**:

- **Regular Vulnerability Scanning**: Continuously scan software and hardware for known vulnerabilities.
- **Limit Software Installation**: Ensure only trusted, verified software is installed.

**Detection Methods**:

- **File Metadata**: Check file hashes and signatures to ensure files haven't been tampered with.
- **External Device Sensors**: Use sensors to monitor the health of incoming devices and software.

![Supply chain compromise](/assets/img/posts/mitre-top10-initial-access/20.png)

## Technique #9: Trusted Relationship

**Definition**:
In trusted relationships, third-party vendors or partners are granted access to a company's internal systems. Attackers may exploit these relationships to gain unauthorized access.

![Trusted relationship](/assets/img/posts/mitre-top10-initial-access/21.png)

**How It Happens**:
A third-party vendor may be compromised, allowing attackers to use their access to infiltrate the target network.

![Compromised third-party access](/assets/img/posts/mitre-top10-initial-access/22.png)

Check this => [https://harmj0y.medium.com/a-guide-to-attacking-domain-trusts-ef5f8992bb9d](https://harmj0y.medium.com/a-guide-to-attacking-domain-trusts-ef5f8992bb9d)

**Example**:
The **Target** breach in 2013 occurred when attackers compromised a third-party HVAC vendor. The attackers used this access to infiltrate Target's network and steal credit card data from millions of customers.

**Mitigations**:

- **MFA**: Use multi-factor authentication to secure third-party access.
- **Network Segmentation**: Restrict third-party access to only necessary systems.

**Detection Methods**:

- **App Logs**: Monitor logs for unusual activity by third-party accounts.
- **Network Traffic**: Track any suspicious data flows from third-party networks.

![Trusted relationship detection](/assets/img/posts/mitre-top10-initial-access/23.png)

## Technique #10: Valid Accounts

**Definition**:
Attackers often steal or use valid credentials to gain unauthorized access to systems. This can include compromised credentials, which can then be used for lateral movement, privilege escalation, or to evade detection.

Check this => [https://attack.mitre.org/techniques/T1003/](https://attack.mitre.org/techniques/T1003/)

![Valid accounts abuse 1](/assets/img/posts/mitre-top10-initial-access/24.png)

![Valid accounts abuse 2](/assets/img/posts/mitre-top10-initial-access/25.png)

![Credential dumping](/assets/img/posts/mitre-top10-initial-access/26.jpg)

**How It Happens**:
An attacker may steal or guess a valid username and password, often through phishing or brute-force attacks, to gain access to the target network.

**Example**:
In the **Equifax** breach, attackers exploited compromised credentials to access the company's systems and steal sensitive data.

**Mitigations**:

- **Account Use Policies**: Restrict access based on IP addresses or geographic locations to limit unauthorized access.

![Account use policy 1](/assets/img/posts/mitre-top10-initial-access/27.png)

![Account use policy 2](/assets/img/posts/mitre-top10-initial-access/28.jpg)

![MFA enforcement](/assets/img/posts/mitre-top10-initial-access/29.jpg)

- **MFA**: Enforce multi-factor authentication for all critical systems.

**Detection Methods**:

- **Logon Sessions**: Monitor for unusual logon attempts or login locations.
- **User Account Authentication Logs**: Track abnormal behavior such as failed login attempts or multiple logins from different locations.

---

> _Originally published on [Medium](https://medium.com/@mohamedabdullh116/miter-att-ck-top-10-initial-access-techniques-explanation-in-simple-format-33d1f6cb2f7c)._
{: .prompt-info }

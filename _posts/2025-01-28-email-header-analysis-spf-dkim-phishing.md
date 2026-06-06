---
title: "Email Header Analysis, Email Flow, SPF, DKIM, and Phishing Attack Preparation"
date: 2025-01-28 21:50:27 +0200
categories: [Phishing and Mail Analysis]
tags: [phishing, email-analysis, email-headers, email-flow, spf, dkim, dmarc, smtp, soc, blue-team]
image:
  path: /assets/img/posts/email-header-analysis/01.jpg
  alt: "Email header analysis, email flow, SPF, DKIM, and phishing attack preparation"
---

Email Flow: The email flow describes the various stages and systems involved in sending and receiving an email. Here's a breakdown with more detail on each component:

![Email flow overview](/assets/img/posts/email-header-analysis/02.png)

**-Mail User Agent (MUA):**

- A software application or interface that allows the user to create, send, and read email. Examples include email clients like Microsoft Outlook, Apple Mail, or web-based platforms like Gmail. The MUA sends the message to the Mail Submission Agent (MSA).

**-Mail Submission Agent (MSA):**

- A server or service that accepts outgoing emails from the MUA and ensures that the messages are properly formatted and ready for delivery. The MSA communicates with the Mail Transfer Agent (MTA) to route the email.

**-Mail Transfer Agent (MTA):**

- MTAs are the core of email transport, responsible for transferring emails between servers across the internet. They use the SMTP protocol to send emails to recipient servers. The MTA handles the routing and delivery of the message to the appropriate destination.

**-Mail Delivery Agent (MDA):**

- The MDA's responsibility is to deliver the message to the recipient's inbox. It stores the email in the recipient's mailbox (on the server) and makes it available for retrieval via IMAP or POP3.

### SPF (Sender Policy Framework):

SPF is an email validation system designed to detect and block email spoofing by verifying that incoming email from a domain comes from an IP address authorized by that domain's administrator. It ensures that the email is coming from a legitimate source, reducing the likelihood of spam and phishing.

![SPF validation flow](/assets/img/posts/email-header-analysis/03.jpg)

**Key concepts:**

- SPF checks the sender's IP address against a list of authorized IPs or domain names.
- If the SPF check passes, the message is verified as coming from an authorized sender.
- SPF failures (Softfail, Neutral, etc.) provide information about potential fraud or misconfigurations.

![SPF check states](/assets/img/posts/email-header-analysis/04.png)

**SPF Record States:**

- **Pass:** The email source is valid, and the email is from an authorized source.
- **Softfail:** Likely a fake source but not definitively invalid.
- **Neutral:** SPF cannot determine whether the source is valid or not.
- **None:** No SPF record exists for the domain.
- **Unknown:** SPF validation cannot be performed.
- **Error:** An error occurred during the SPF check.

You can also check any SPF record for any domain by using MXToolBox => [https://mxtoolbox.com/spf.aspx](https://mxtoolbox.com/spf.aspx)

![SPF record lookup in MXToolBox](/assets/img/posts/email-header-analysis/05.png)

### DKIM (Domain Keys Identified Mail):

DKIM uses cryptographic signatures to verify that the email content has not been altered in transit and that the sender is authorized. It provides a mechanism for authenticating the domain of the sender.

![DKIM signing and verification](/assets/img/posts/email-header-analysis/06.jpg)

**-DKIM Process:**

- The sender's mail server generates a digital signature based on the email's content and headers. This signature is then added to the email.
- The recipient's server retrieves the public key from the sender's DNS and uses it to verify the signature.
- If the signature matches, the email is verified as legitimate and untampered with.

**-DKIM Signature Breakdown:**

- **v:** Version of the DKIM standard used.
- **a:** The algorithm used for generating the signature (e.g., RSA, SHA-256).
- **c:** The canonicalization method used for preparing the message (e.g., relaxed or simple).
- **d:** The domain of the signer (the sender's domain).
- **h:** List of headers included in the hash for the signature.
- **b:** The body hash, which contains the cryptographic signature of the message body.

![DKIM signature header example](/assets/img/posts/email-header-analysis/07.jpg)

Lets get more deeper ;)

### Email Header Analysis:

Email headers provide a detailed log of how an email was processed as it moved through various mail servers. Here's a deeper look at key fields:

![Email header fields](/assets/img/posts/email-header-analysis/08.png)

- **Delivered-To:** The recipient's email address that received the message.
- **Received By:** This field shows the chain of SMTP servers the email passed through, containing:
- **Server's IP address:** The IP address of each server involved.
- **SMTP ID:** The identifier assigned by each server to track the email.
- **Date & Time:** The timestamp when the email was received.
- **Received-SPF:** Displays the result of the SPF check, showing whether the email passed or failed the SPF validation.
- **DMARC Authentication Results:** Indicates the results of SPF, DKIM, and DMARC checks, often included to help identify spoofed emails.
- **DKIM-Signature:** Lists the DKIM parameters, as previously detailed, to verify if the email has been signed and whether its integrity is intact.
- **Return-Path:** The address to which undeliverable messages are sent. This is useful for tracking bounce-backs or identifying mail spoofing if mismatched.
- **Message-ID:** A unique identifier for each email. It helps to track email chains and can indicate spoofing if two messages share the same ID.

![Full email header walkthrough](/assets/img/posts/email-header-analysis/09.png)

- Check This => [https://www.thesslstore.com/blog/how-to-read-an-email-header/](https://www.thesslstore.com/blog/how-to-read-an-email-header/)

### Identification of Phishing Attacks:

Monitoring and detection of phishing attacks is critical. Here's how to identify and monitor for them:

1. **Monitor Connection Points:**

. Keep an eye on all points of entry and exit in your network to track suspicious activities.

**2. Deploy Spam Traps:**

- Set up spam traps to collect potential phishing emails and identify patterns for further analysis.

**3. Active Monitoring:**

- Monitor known phishing repositories like PhishTank and Google Safe Browsing for newly reported phishing sites.

**4. Monitor Web Logs (very Important) :**

- Analyze your web logs for suspicious activities, such as a sudden increase in referral traffic or requests for unusual URLs.

### Phishing Attack Scope Assessment:

When assessing the scope of a phishing attack, consider the following:

1. **Targeted Users:**

- Determine how many users have been affected by the phishing attack and which systems they have accessed.

**2. Compromised Accounts:**

- Search for compromised machines and accounts and identify any secondary malicious activity (data exfiltration, lateral movement, etc.).

### Analyze the Phishing Attack:

Proper analysis of phishing attempts is crucial for containing and mitigating damage:

1. **Malware vs Credential Harvesting:**

- Analyze whether the phishing attack is attempting to deliver malware or harvest credentials (passwords, banking info, etc.).

**2. Message & Body Inspection:**

- Inspect the email message and body to look for signs of phishing (e.g., urgency, poor grammar, suspicious links).

**3. Sandboxing:**

- Use sandboxes to safely open any suspicious email attachments and extract Indicators of Compromise (IoCs).

**4. Link Analysis with CTI:**

- Analyze URLs and hostnames using Cyber Threat Intelligence (CTI) tools to identify malicious websites associated with phishing.

**5. Investigate Headers:**

- Analyze email headers for additional clues about the origin and authenticity of the email.

### Containment and Response Techniques:

After identifying a phishing attack, take these steps:

1. **Block IoCs:**

- Block any discovered IoCs, such as malicious attachments or URLs, on the network perimeter.
- Check IOC's Reputation of IoC's you collected by using VT or MALBAZ etc..

**2. Remediation Page:**

- Deploy a remediation page that alerts users to the current phishing attack and provides instructions for safeguarding their accounts.

**3. Black Hole DNS:**

- Use Black Hole DNS, where a DNS server provides no response (or a remediation page) when clients attempt to reach known malicious domains.

**4. Compromised Account Response:**

- Change passwords or block access to accounts that have been compromised by the phishing attack.

5. **Common Phishing Indicators:**

- The sender email name/address will masquerade as a trusted entity (email spoofing)
- The email subject line and/or body (text) is written with a sense of urgency or uses certain keywords such as Invoice, Suspended, etc.
- The email body (HTML) is designed to match a trusting entity (such as Amazon)
- The email body (HTML) is poorly formatted or written (contrary from the previous point)
- The email body uses generic content, such as Dear Sir/Madam. Hyperlinks (oftentimes uses URL shortening services to hide its true origin)
- A malicious attachment posing as a legitimate document

### Email Delivery Protocols:

1. **SMTP (Simple Mail Transfer Protocol):**

- Port 25: Used for sending emails from the client to the mail server and between mail servers.

**2. POP3 (Post Office Protocol):**

- Port 110: Allows users to download emails from the server to their local device, storing them locally.

**3. IMAP (Internet Message Access Protocol):**

- Port 143: Similar to POP3 but allows synchronization between multiple devices, so messages remain on the server.

### Differences Between POP3 and IMAP:

**-POP3:**

- Downloads emails to the local device, limiting access to the device where the email is downloaded.
- Emails may be lost if not configured to keep a copy on the server.

**-IMAP:**

- Stores emails on the server, accessible from any device.
- Sent messages are also stored on the server and synchronized across devices.

### Email Travel Journey:

The email journey from the sender to the recipient is complex and involves various steps and systems:

1. **Email Creation:** The message is written by the user in the MUA.
2. **SMTP Submission:** The MUA sends the email to the MSA, which submits it to the MTA.
3. **Routing via DNS:** The SMTP server queries DNS to determine the recipient's mail server.
4. **Multiple MTAs:** The email may traverse multiple MTAs until it reaches the final MTA.
5. **Delivery to MDA:** The recipient's MDA stores the email, making it accessible to the recipient via IMAP or POP3.
6. **Recipient Retrieval:** The recipient accesses their mailbox using IMAP or POP3 to retrieve the email.

Visualize examples:

![Email journey example 1](/assets/img/posts/email-header-analysis/10.png)

![Email journey example 2](/assets/img/posts/email-header-analysis/11.jpg)

![Email journey example 3](/assets/img/posts/email-header-analysis/12.png)

![Email journey example 4](/assets/img/posts/email-header-analysis/13.jpg)

## ############## =>>> Important Resources:

مهمه والله يسطا ابقا بص عليها

- [https://www.youtube.com/watch?v=QW62QqacC20&list=PLTfpxvprC-_y6XwXVbUj-CyKhk3lBNeUg](https://www.youtube.com/watch?v=QW62QqacC20&list=PLTfpxvprC-_y6XwXVbUj-CyKhk3lBNeUg)
- [https://tryhackme.com/r/room/parrotpost](https://tryhackme.com/r/room/parrotpost)
- [https://tryhackme.com/r/room/phishingemails1tryoe](https://tryhackme.com/r/room/phishingemails1tryoe)
- [https://tryhackme.com/r/room/phishingemails2rytmuv](https://tryhackme.com/r/room/phishingemails2rytmuv)
- [https://www.youtube.com/watch?v=0RbP9dU_ibw&list=PLdUDP-atVHBoDae43tcUZnW1YsjoPJRvP&index=5](https://www.youtube.com/watch?v=0RbP9dU_ibw&list=PLdUDP-atVHBoDae43tcUZnW1YsjoPJRvP&index=5)
- [https://www.youtube.com/watch?v=8JPEL7xGSxE&list=PLdUDP-atVHBoDae43tcUZnW1YsjoPJRvP&index=6](https://www.youtube.com/watch?v=8JPEL7xGSxE&list=PLdUDP-atVHBoDae43tcUZnW1YsjoPJRvP&index=6)
- [https://www.youtube.com/watch?v=fyAi7Tgdo0Y&list=PLdUDP-atVHBoDae43tcUZnW1YsjoPJRvP&index=4](https://www.youtube.com/watch?v=fyAi7Tgdo0Y&list=PLdUDP-atVHBoDae43tcUZnW1YsjoPJRvP&index=4)
- [https://www.youtube.com/watch?v=2C9pH-Vctlw&list=PLdUDP-atVHBoDae43tcUZnW1YsjoPJRvP&index=3](https://www.youtube.com/watch?v=2C9pH-Vctlw&list=PLdUDP-atVHBoDae43tcUZnW1YsjoPJRvP&index=3)
- [https://www.youtube.com/watch?v=Ex00XJnKHh4&list=PLG3zFBI_Jy2nusVjUJKYyjrqtzmCTHqNU](https://www.youtube.com/watch?v=Ex00XJnKHh4&list=PLG3zFBI_Jy2nusVjUJKYyjrqtzmCTHqNU)

---

> _Originally published on [Medium](https://medium.com/@mohamedabdullh116/email-header-analysis-email-flow-spf-dkim-and-phishing-attack-preparation-b49c5c57ff9a)._
{: .prompt-info }

---
title: "Detecting Web Attacks as a Proficient Investigator"
date: 2024-12-19 22:19:06 +0200
categories: [Interesting Topics]
tags: [web-attacks, log-analysis, soc, sqli, xss, brute-force, ddos, lfi-rfi, csrf, waf, threat-detection, blue-team]
image:
  path: /assets/img/posts/detecting-web-attacks/01.jpg
  alt: "Detecting web attacks as a proficient investigator"
---

**How to Leverage Logs for Web Application Security**

In today's digital landscape, understanding and analyzing logs is crucial for maintaining web application security. Logs not only help in troubleshooting issues but also provide valuable insights into your application's performance, security vulnerabilities, and potential attacks. Let's dive into how logs can be your best tool for keeping your application secure.

First Review your Knowledge about Web Service and how it works :

- What is the Components of Web Server infrastructure ?

![Web server infrastructure components](/assets/img/posts/detecting-web-attacks/02.png)

1- **Web Applications:** the top layer, and built from Scripting language like PHP, JS, HTML.CSS etc..
2- **Web Server:** contains OS of web server like Apache, Nginx, IIS which hosts web application
3- **Servers/Hardware :** the first Physical Layer which hosted all of the above, A virtual machine might also be used to host Web applications. Database Server and Application Server too can be added to this layer
**4- Network Hardware and Services**: is the last layer. It makes the communication possible between the servers (hosting Web application) and the client

## 1. Using Logs for Troubleshooting and Investigations

Logs are the first place to go when troubleshooting issues with your web application. Whether it's Apache, Nginx, or IIS, the error logs can point you to the root cause of problems. They contain key information like error messages, timestamps, and request details, which are essential for identifying and fixing issues.

- **Apache logs**: `apache.log`, `httpd.conf => check it below`

![Apache configuration](/assets/img/posts/detecting-web-attacks/03.png)

- **Nginx logs**: `nginx.log => check it below`

![Nginx configuration](/assets/img/posts/detecting-web-attacks/04.png)

- IIS logs `=> check it below`

![IIS log configuration](/assets/img/posts/detecting-web-attacks/05.png)

By reviewing these logs, you can not only identify bugs but also understand the root cause of performance bottlenecks or user access issues.

check also this resources :

- [https://httpd.apache.org/docs/2.4/configuring.html](https://httpd.apache.org/docs/2.4/configuring.html)
- [https://www.crowdstrike.com/en-us/cybersecurity-101/observability/access-logs/](https://www.crowdstrike.com/en-us/cybersecurity-101/observability/access-logs/)
- if u have some problems in understanding logs look at this room : [https://tryhackme.com/r/room/introtologs](https://tryhackme.com/r/room/introtologs)

## 2. The Power of Error Logs

Error logs can reveal a lot about your system's health and security. Here's why they matter:

- **Key Information**: They often contain critical information about failed requests, server issues, and misconfigurations.
- **Security Threats**: Logs can help identify potential attacks like SQL injection or brute force attempts by tracking failed login attempts or suspicious payloads.
- **Location Awareness**: Error logs are stored in different locations based on the server type:
- **Apache**: `/var/log/apache/error.log`
- **Nginx**: `/var/log/nginx/error.log`
- **IIS**: `%SystemDrive%\inetpub\logs\LogFiles\*.log`

> **_Note_**_: Logs can sometimes be faked, so it's important to correlate the data with other security tools._
>
> ركز يبني عشان متتغفلش كفايه الي الدنيا عاملاه فيك

## 3. Understanding URL Components

URLs are not just web addresses; they are windows into the behavior of your web application. A URL is composed of:

- **Scheme**: Defines the protocol (e.g., `http`, `https`).
- **Host**: The domain being addressed (e.g., `example.com`).
- **Port**: The port number (if specified).
- **Path**: The resource or file being requested.
- **Query**: Parameters sent to the server.

![URL components](/assets/img/posts/detecting-web-attacks/06.png)

[=====>>>>>>>>>> https://www.geeksforgeeks.org/components-of-a-url/](https://www.geeksforgeeks.org/components-of-a-url/)

You should know some facts about URL :
- **URL can** be sent over internet using only ascii characters
- **URL encoding** comes to stop that and make connection more safe
- **URL encoding** replace unsafe ascii character with % Followed by 2 hexadecimal digits
- **URL Canno**t contain any spaces as it replaced by %20
- It is common to see attacks using **encoded URLs**. Although a "%" in the URL doesn't mean always a malicious request

However, URLs can also be manipulated for malicious purposes. **URL encoding** replaces unsafe characters with encoded symbols (e.g., spaces become `%20`), but attackers may craft URLs with encoded data to exploit vulnerabilities. Always monitor for suspicious patterns like multiple SQL keywords in query strings—this could be a sign of an SQL injection attempt.

## 4. Detecting Vulnerabilities and Scanners

First you should ask your self How we can know if Web Application have a vulnerability ??
- we can review web application code but it not will be an effective way and time consuming process
- another way is to use vulnerability scanner
- it lunches some attacks on web application and investigate the response and inform you if there any existing attacks
- Used by attackers and non-attackers to find weaknesses on their targets: It can be used by attackers and defenders alike to identify the vulnerabilities. An attacker would use the scanner to exploit the vulnerabilities while a defender would use the scanner to identify the vulnerabilities and fix them
- Can be used to test your security tools like IDS, IPS and WAF: It can be used to test the defensive capabilities of the tool deployed in our Web application setup or infrastructure

![Vulnerability scanner activity in logs](/assets/img/posts/detecting-web-attacks/07.png)

- the easiest way to identify vulnerability scanner is to check logs generated from the connection

![Scanner request pattern 1](/assets/img/posts/detecting-web-attacks/08.png)

![Scanner request pattern 2](/assets/img/posts/detecting-web-attacks/09.png)

Vulnerability scanners, such as **sqlmap**, help detect weaknesses in web applications by simulating attacks. Both defenders and attackers use these tools, but it's essential to know when your server is being scanned. Indicators include:

- Multiple requests in a short time frame.
- Repeated requests to critical URLs like `/admin` or `/config`.
- Unusual user agents in the logs (e.g., tools like Hydra for brute force attacks).
- if you noticed a lot of requests generated that might be an alert for Vulnerability scanner activity

![High volume of scanner requests](/assets/img/posts/detecting-web-attacks/10.png)

If you notice high volumes of similar requests from a single IP, it's time to investigate further.

Additional notes from other recources :

" **Identifying a Vulnerability scan:**
**- Check the user agent can be vulnerability scanner software or something else weird**
**- If there are many requests in a short amount of time then it points to a vulnerability scan**
**- If there are lots of error returned for the requests or weird requests like PHP request on a page that doesn't have PHP. better understanding of the application helps in recognizing the attack**
**- Requests directed to admin or configuration pages are also a signature, as they are critical points to be exploited**

**- Commands or O_S words in requests_: ping, cat, shell, admin, config, and others are suspicious too.**
**- The best action would be to block the IP as it is performing a Vulnerability scan."**

### check this => [https://www.geeksforgeeks.org/how-to-analyze-threats-in-apache-logs/](https://www.geeksforgeeks.org/how-to-analyze-threats-in-apache-logs/)

## 5. Identifying Brute Force Attacks

Brute force attacks are attempts to gain unauthorized access by trying multiple username and password combinations. Logs reveal these attempts, and here's how to spot them:

- **Repeated failed login attempts** from the same IP address.
- **User agent analysis**: Brute force tools like Hydra can be identified by their user agents.
- **Multiple login attempts** on admin pages within a short period.

If you see a large number of login attempts from the same IP in a short time, block the IP and investigate further.

![Brute force attempts in logs 1](/assets/img/posts/detecting-web-attacks/11.png)

![Brute force attempts in logs 2](/assets/img/posts/detecting-web-attacks/12.png)

- Multiple username and password combinations are being attempted. Also, it is important to notice that these requests are being generated from same IP and within very short span which isn't possible for a human user

![Repeated logins from same IP](/assets/img/posts/detecting-web-attacks/13.png)

- From the above log we can see that someone is trying to login using username, Pablo. Behavior is common to a brute force attempt. Multiple requests are generating from the same IP within a short span of time

![Login attempts for user Pablo](/assets/img/posts/detecting-web-attacks/14.png)

- In a POST request, the username and the passwords won't be visible as they are contained in the payload.
- Hydra Proxy is being used as it is clear from the User Agent field. Hydra is a well known tool used to perform the Brute force attacks. Another important point to note is the large number of requests originating from the same IP and the attempts on the login page.

![POST brute force with Hydra user agent](/assets/img/posts/detecting-web-attacks/15.png)

- Here we have POST but the user agent looks normal. It can be a genuine attempt from a user trying to login. User types a username and a password, then the page refreshes. But, it is highly unlikely for someone to type the username and password in very short span, which points to a brute force attempt.
  — Indicators for Brute force attempt:
  —  Many requests to login pages in a short span of time: Same IP trying multiple login attempts in a very short span which might not be possible for a human user.
  —  GET = Different users or passwords: Different combinations being tried to brute force.
  —  POST = Many requests in a short amount of time or check other logs. Check if the attack or attempt is on a login page and the requests are sent within short intervals. User Agent: Check if the user agent is a tool used to perform Brute force attacks.
  —  Login Web Pages: The page where this unusual behavior is observed is a login page.
  —  Administrator username: The Administrator username is being attempted for cracking.
- **How to Detect via Logs**:
- **Frequent login attempts**: Multiple failed login attempts from the same IP address within a short period.
- **Usernames**: If attackers are targeting specific usernames (e.g., `admin`, `root`), it's a red flag.
- **User Agent and IP Analysis**: Multiple requests from the same IP or with the same user agent can indicate automated brute force tools like **Hydra** or **Burp Suite**.
- **HTTP Response Codes**: Multiple `401 (Unauthorized)` or `403 (Forbidden)` errors are strong indicators of a brute force attack.

## so now if I want to mitigate and detect some attacks what is the scope that i will focus on ???

## lets get some examples =>

## SQL Injection (SQLi) Attacks

SQL injection is one of the most common attack methods. Attackers inject SQL commands into input fields to manipulate the database. Detecting SQL injection attempts in logs is straightforward if you know what to look for:

- **SQL keywords** like `SELECT`, `DROP`, `UNION` in the request.
- **Encoded URLs**: Look for `%` symbols in the logs—an indicator of potential encoding used in SQL injection.
- **Suspicious user agents**: Tools like **sqlmap** are commonly used for such attacks.

The goal is to block malicious requests before they reach your application.

**How to Detect via Logs**:

- **Look for suspicious SQL keywords** such as `SELECT`, `DROP`, `UNION`, `OR 1=1`, or `--` in the request URL or payload.
- **Encoded requests**: Pay attention to `%` symbols in the URL, which may indicate encoded malicious SQL queries.
- **Error codes**: Status codes like `500` (internal server error) or `302` (redirection) could indicate that SQL queries were not processed successfully, potentially due to injection attempts.
- **Unusual User Agents**: Attack tools like **sqlmap** may appear in the user agent field.
- when you see multiple SQL Words in the request that might be an indicator to SQLI Attack

![SQL injection in logs 1](/assets/img/posts/detecting-web-attacks/16.png)

![SQL injection in logs 2](/assets/img/posts/detecting-web-attacks/17.png)

![SQL injection in logs 3](/assets/img/posts/detecting-web-attacks/18.png)

- It is important to note the common SQL words being used in the request. In the above logs, status code 302 is being returned which means redirection and the commands aren't successfully executed

![SQL keywords in request 1](/assets/img/posts/detecting-web-attacks/19.png)

![SQL keywords in request 2](/assets/img/posts/detecting-web-attacks/20.png)

![SQL keywords in request 3](/assets/img/posts/detecting-web-attacks/21.png)

- Identifying the SQL injection attacks from the logs: Check for various SQL commands in the logs. Below are some commonly used SQL commands :

![Common SQL commands](/assets/img/posts/detecting-web-attacks/22.png)

- Look for the encoded request with the % symbol.
- Look for the User Agents like sqlmap
- OS commands being used in requests might be suspicious.

**Mitigation Strategies**:

- **Input Validation**: Always validate and sanitize input before it reaches the database.
- **Prepared Statements/Parameterized Queries**: These prevent the injection of malicious code by treating user input as data rather than executable code.
- **Web Application Firewall (WAF)**: Deploy a WAF to detect and block SQL injection attempts.
- **Error Handling**: Ensure error messages do not reveal database information to the user.

## Cross-Site Scripting (XSS)

**Overview**:
XSS allows attackers to inject malicious scripts into web pages viewed by other users, potentially stealing session tokens, cookies, or redirecting users to malicious sites.

**How to Detect via Logs**:

- Here is one sample web access log entry that is a sign of an XSS attack.

> _192.168.0.252 — — [05/Aug/2009:15:16:42 -0400] "GET /%27%27;!–%22%3CXSS%3E=&{() } HTTP/1.1″ 404 310 "-" "Mozilla/5.0 (X11; U; Linux x86_64; en-US; rv:1.9.0.12) Gecko/2009070812 Ubuntu/8.04 (hardy) Firefox/3.0.12″_

**The part to look for is the GET /%27%27 command (there are several variants).**

- **Suspicious JavaScript**: Look for tags like `<script>`, `javascript:`, or event handlers like `onerror=`, `onclick=`, which could indicate injected scripts.
- **Unusual URL parameters**: Check for URL parameters that contain JavaScript code or encoded script content.
- **Error Codes**: Unusual errors related to the page rendering, such as **404 (Not Found)** for script files or **403 (Forbidden)** for certain actions, can hint at XSS payloads being rejected by the server.

**Mitigation Strategies**:

- **Input Escaping**: Ensure all user input is properly escaped before being rendered in the browser.
- **Content Security Policy (CSP)**: Implement CSP headers to restrict where scripts can be loaded from and prevent inline JavaScript execution.
- **Sanitize Input**: Use libraries that automatically sanitize HTML and JavaScript input, such as **OWASP Java Encoder** or **DOMPurify**.

## Denial of Service (DoS) and Distributed Denial of Service (DDoS) Attacks

**Overview**:
A DoS or DDoS attack overwhelms a server with traffic, causing it to crash and become unavailable to legitimate users.

![DoS / DDoS traffic spike 1](/assets/img/posts/detecting-web-attacks/23.png)

![DoS / DDoS traffic spike 2](/assets/img/posts/detecting-web-attacks/24.jpg)

![DoS / DDoS traffic spike 3](/assets/img/posts/detecting-web-attacks/25.png)

**How to Detect via Logs**:

- **High Request Rate**: A massive spike in incoming requests from a single IP or multiple sources in a very short period.
- **HTTP Status Codes**: Look for **503 (Service Unavailable)** or **504 (Gateway Timeout)** errors in logs, which could indicate that the server is struggling under heavy load.
- **Unusual Request Patterns**: Requests targeting a single endpoint or resource repeatedly might indicate an attack.

**Mitigation Strategies**:

- **Load Balancing**: Distribute traffic across multiple servers to prevent overload on a single server.
- **Traffic Filtering**: Use a DDoS protection service like **Cloudflare** or **AWS Shield** to mitigate high traffic volumes.
- **Rate Limiting**: Implement rate limiting to restrict the number of requests from a single IP address.

## Remote File Inclusion (RFI) and Local File Inclusion (LFI)

**Overview**:
RFI and LFI attacks involve including malicious files through user input, enabling attackers to execute arbitrary code or access sensitive files on the server.

**How to Detect via Logs**:

- **Suspicious File Paths**: Check for requests that include file paths like `/etc/passwd` (LFI) or URLs starting with `http://` (RFI).
- **Error Codes**: Status code **404 (Not Found)** or **500 (Internal Server Error)** for file access could indicate an attempt to include non-existent or malicious files.
- **User Agent and Parameters**: The presence of unusual user agents or suspicious parameters like `file=../../etc/passwd` can point to an attack attempt.

**Mitigation Strategies**:

- **Input Validation**: Sanitize and validate any input used for file paths to prevent directory traversal.
- **Disable URL fopen**: Disable the `allow_url_fopen` directive in PHP to prevent external URLs from being included in scripts.
- **Server Hardening**: Limit file permissions to prevent access to sensitive files and directories.

## Cross-Site Request Forgery (CSRF)

**Overview**:
CSRF exploits the trust a website has in a user's browser, tricking the user into making an unwanted request (such as changing account settings) without their knowledge.

**How to Detect via Logs**:

- **Unexpected Changes**: Look for unexpected or unauthorized changes, like password changes or profile updates, triggered by GET or POST requests that could have been forged.
- **Referer Header**: Check the `Referer` header in HTTP requests—if it's missing or from an untrusted site, it might indicate a CSRF attempt.

**Mitigation Strategies**:

- **Anti-CSRF Tokens**: Use tokens that are unique to each request to ensure that the request is coming from a legitimate source.
- **SameSite Cookies**: Set cookies with the `SameSite` attribute to prevent them from being sent in cross-site requests.
- **Validate HTTP Method**: Ensure sensitive actions (e.g., account updates) are only performed via POST requests and not GET.

## There other some event ids i prefer to check it on windows based web server (IIS) :

- **Event ID 4688**: A new process has been created. This can help identify unusual or unauthorized processes being launched.
- [https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4688](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4688)

![Event ID 4688](/assets/img/posts/detecting-web-attacks/26.png)

- **Event ID 4648**: A logon attempt was made using explicit credentials. This can help detect potential credential theft or brute force attempts.
- [https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4648](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4648)

![Event ID 4648](/assets/img/posts/detecting-web-attacks/27.png)

- **Event ID 4625**: Failed logon attempts. This is crucial for detecting brute force attacks and unauthorized access attempts.
- [https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4625](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4625)

![Event ID 4625](/assets/img/posts/detecting-web-attacks/28.jpg)

- **Event ID 5156**: The Windows Filtering Platform has allowed a connection. This event can be used to detect inbound traffic patterns and potential attacks, especially if large numbers of connections originate from suspicious IP addresses.
- [https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=5156](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=5156)

![Event ID 5156](/assets/img/posts/detecting-web-attacks/29.png)

- **Event ID 5158**: The Windows Filtering Platform has blocked a connection. This is useful for identifying denied connections from known malicious IPs.
- [https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=5158](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=5158)

![Event ID 5158](/assets/img/posts/detecting-web-attacks/30.png)

- and thanks for reading molo was here : )

---

> _Originally published on [Medium](https://medium.com/@mohamedabdullh116/detecting-web-attacks-as-proficient-investigator-553a966e561e)._
{: .prompt-info }

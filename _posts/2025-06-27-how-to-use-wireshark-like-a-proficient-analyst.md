---
title: "How to Use Wireshark Like a Proficient Analyst — Part 1"
date: 2025-06-27 05:58:50 +0200
categories: [Network Forensics]
tags: [network-forensics, wireshark, pcap, packet-analysis, tcp-ip, soc, dfir, blue-team]
image:
  path: /assets/img/posts/wireshark-proficient-analyst/01.jpg
  alt: "How to use Wireshark like a proficient analyst"
---

## What is a Packet?

A **packet** is a small unit of data that represents network activity at a specific point in time. It is the building block of network communication, containing crucial information about how data is transmitted between devices.

From a security perspective, packets are invaluable. They help:

- Analysts and SOC teams determine recent activities on a network
- Detect and investigate network-based attacks
- Reveal indicators of compromise (IOCs)
- Reconstruct attack steps to build a threat timeline or mindmap for further detection

عالسريع => Packets provide detailed insights into the behavior of users, services, and attackers on the network.

Lets Go:

![Lets go](/assets/img/posts/wireshark-proficient-analyst/02.gif)

## What is a PCAP File?

A **PCAP (Packet Capture)** file is a standard format for storing network traffic data. It contains packet-level evidence, which is vital for analyzing a wide range of performance or security issues.

## Why PCAP Files Matter:

- They store raw, unaltered packet data for forensics and troubleshooting
- They serve as digital evidence in incident response and legal investigations
- They are compatible with tools like Wireshark, tcpdump, NetworkMiner, and more

For more details, visit:
[What is a PCAP file? — Endace](https://www.endace.com/learn/what-is-a-pcap-file)

## Wireshark Tool: Interface and Setup

**Wireshark** is one of the most powerful and widely used tools for analyzing PCAP files. It allows you to view, filter, and dissect packets at every level of the network stack.

Download Wireshark:

- [https://www.wireshark.org/download.html](https://www.wireshark.org/download.html)
- [https://www.wireshark.org](https://www.wireshark.org)

## Opening a File

Upon launching Wireshark, you can:

- Open a PCAP file from your file system using the **"Open"** button

![Opening a PCAP file](/assets/img/posts/wireshark-proficient-analyst/03.png)

- Access **recently opened files** directly from the interface

![Recently opened files](/assets/img/posts/wireshark-proficient-analyst/04.png)

Once opened, the interface will display a large volume of traffic, which might seem overwhelming at first — but don't worry, filtering and analysis will make it manageable.

## What Does Each Packet Contain?

Each packet includes several layers of data based on the **TCP/IP model**, starting from the application layer down to the data link layer:

1. **Application Layer** — Protocol name (e.g., HTTP, DNS, FTP)
2. **Transport Layer** — TCP or UDP protocol
3. **Network Layer** — IP (e.g., IPv4)
4. **Data Link Layer** — Ethernet
5. **Frame** — Metadata about how the packet was captured and stored

![Packet layers in Wireshark](/assets/img/posts/wireshark-proficient-analyst/05.png)

> _Note: Each layer's content depends on the protocol being used. It's important to understand how each protocol functions during analysis.
> For more information:_
> [_Top 40 Protocols — CERTCube_](https://blog.certcube.com/top-40-protocols-a-comprehensive-guide/)

## Do You Have to Analyze Every Packet?

**Good news homie :) No, you don't.** And here's why:

- Many packets are encrypted and contain limited readable data
- Some packets may not carry valuable content (e.g., keep-alives, acknowledgments)
- Not every communication is significant for your case

![Not every packet matters](/assets/img/posts/wireshark-proficient-analyst/06.png)

As an analyst, you should focus on:

- Which protocols indicate meaningful actions
- What hosts are targeted and why
- How the communication started and evolved
- Where to extract valuable data to build your case

Think of your analysis as progressing through a story — not every page matters equally, but key parts reveal the full picture.

## Step 1: Understand the Scope of the Environment

Before diving into specific packets, it's important to get a **high-level overview** of the environment represented in the PCAP.

Navigate to:

- **Statistics** → **IPv4 Statistics** → **All Addresses**

![Statistics → IPv4 Statistics → All Addresses](/assets/img/posts/wireshark-proficient-analyst/07.png)

This section reveals all IPs involved in at least one packet, providing an overview of your network scope.

## Insights Provided:

1. List of all IP addresses in the capture
2. Number of packets sent/received by each IP (important)
3. Average, minimum, and maximum packet counts
4. Rate of communication (ms)
5. Percentage of total traffic per IP

![IPv4 statistics breakdown](/assets/img/posts/wireshark-proficient-analyst/08.png)

This gives you visibility into who is talking the most and who might be behaving abnormally.

## Step 2: Identify Who Communicated with Whom

To understand **which IPs are communicating with each other**, go to:

- **Statistics** → **Conversations** → **IPv4 Tab**

![Statistics → Conversations → IPv4 Tab](/assets/img/posts/wireshark-proficient-analyst/09.png)

Here, you will find:

- **Address A:** Source IP
- **Address B:** Destination IP
- **Packets:** Number of exchanged packets
- **Bytes:** Total data exchanged
- **Duration:** Time span of the communication

This view is critical to identifying active sessions, suspicious peer-to-peer connections, or beaconing behavior.

> **Quick Hint: if there any public IPs found in Conversations check its reputation, it will be a good move to check how communication goes**

After Knowing This data you should know what is data provided in the packets in the interface

## 1. Time

This column displays the **timestamp** of when the packet was captured — essentially, the exact moment it arrived during the capture session.
Wireshark allows you to change the time format (e.g., relative time, UTC, local time), but for most investigations, the default setting is sufficient.

## 2. Source

This shows the **source IP address**, which is the originator of the traffic. It tells you which system initiated the connection or sent the data.

## 3. Destination

This is the **destination IP address**, identifying the host that received the packet. Tracking destination IPs helps identify targeted systems or services.

## 4. Protocol

The **protocol column** displays the network protocol used in the packet — typically the application layer protocol (HTTP, DNS, FTP), but sometimes it may show transport or network layer protocols like TCP, UDP, or ICMP.

## 5. Length

Indicates the **total size of the packet** in bytes, including all protocol headers and payload data. This helps estimate data flow sizes and can highlight unusually large or small packets.

## 6. Info

This column provides a **brief description of the packet's content or behavior**, depending on the protocol. It may include things like:

- HTTP request methods (GET, POST)
- TCP flags (SYN, ACK, FIN)
- DNS queries and responses
  This field gives quick insight into what's happening in the traffic without needing to expand each packet.

![Packet list columns in Wireshark](/assets/img/posts/wireshark-proficient-analyst/10.png)

## Why Understanding Protocols Matters

At this stage of analysis, the real value comes from **knowing which protocols are in use** and **how they behave**. The more familiar you are with network protocols, the better you'll be at spotting unusual activity or signs of compromise.

When you're analyzing traffic in Wireshark, **each protocol behaves differently** and reveals specific types of data. For example:

- **HTTP** traffic can show URLs, request methods, and responses — useful for identifying malicious web activity
- **DNS** requests can reveal domain name lookups — often abused in Command & Control (C2) communication
- **FTP/SMB** may indicate file transfers — potentially used for data exfiltration or lateral movement
- **TLS/SSL** indicates encrypted sessions — worth checking for certificate anomalies or suspicious endpoints
- **ICMP** might be used for reconnaissance (ping sweeps)

By understanding protocol behavior, you gain the ability to:

- Focus on meaningful traffic
- Identify protocol misuse (Such Important)
- Correlate traffic patterns with attack stages
- Filter traffic efficiently using Wireshark's display filters

Continue in Part 2 Saan u seen guys

![See you in Part 2](/assets/img/posts/wireshark-proficient-analyst/11.gif)

---

> _Originally published on [Medium](https://medium.com/@mohamedabdullh116/how-to-use-wireshark-like-a-proficient-analyst-5f0766e1b94b)._
{: .prompt-info }

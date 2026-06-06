---
title: "Getting Started with Memory Forensics"
date: 2026-06-01 12:00:00 +0200
categories: [Memory Forensics]
tags: [memory-forensics, volatility, ram-analysis, malware-analysis, incident-response, dfir, blue-team]
---

# Memory Forensics

Memory forensics is the analysis of a system's volatile memory (RAM) to uncover
evidence that never touches disk — running processes, network connections,
injected code, decrypted payloads, credentials, and rootkit artifacts.

This category collects notes and walkthroughs on:

- **Acquisition** — capturing RAM with tools like WinPmem, DumpIt, AVML, and LiME.
- **Analysis frameworks** — Volatility 3 and MemProcFS for triaging a memory image.
- **Hunting techniques** — finding hidden processes, code injection, malicious
  drivers, and persistence living only in memory.
- **Credential & artifact recovery** — pulling secrets, command history, and
  network state out of a capture.

More detailed, hands-on posts will land here over time.

---
title: "Getting Started with Reverse Engineering"
date: 2026-06-04 12:00:00 +0200
categories: [Reverse Engineering]
tags: [reverse-engineering, malware-analysis, ghidra, x64dbg, static-analysis, dynamic-analysis, blue-team]
---

# Reverse Engineering

Reverse engineering, from a blue-team perspective, is about taking a suspicious
binary apart to understand what it does — its capabilities, indicators, and
behavior — so you can detect and respond to it.

This category collects notes and walkthroughs on:

- **Triage** — file typing, strings, hashing, and PE/ELF header inspection.
- **Static analysis** — disassembly and decompilation with Ghidra, IDA, and
  Cutter.
- **Dynamic analysis** — debugging with x64dbg/GDB and detonating samples in a
  sandbox to observe behavior.
- **Unpacking & deobfuscation** — defeating packers, encryption, and
  anti-analysis tricks to reach the real payload.

More detailed, hands-on posts will land here over time.

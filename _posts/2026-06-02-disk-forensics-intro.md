---
title: "Getting Started with Disk Forensics"
date: 2026-06-02 12:00:00 +0200
categories: [Disk Forensics]
tags: [disk-forensics, file-system, autopsy, timeline-analysis, artifacts, dfir, blue-team]
---

# Disk Forensics

Disk forensics focuses on the non-volatile evidence left behind on storage
media — file systems, deleted files, registry hives, browser history, and the
many OS artifacts that reconstruct *what happened and when*.

This category collects notes and walkthroughs on:

- **Imaging & integrity** — creating forensically sound images and verifying
  hashes with FTK Imager, `dd`, and `dc3dd`.
- **File systems** — NTFS, FAT, and ext internals: `$MFT`, journals, and slack space.
- **Windows artifacts** — registry, Prefetch, Shimcache, Amcache, LNK files,
  Jump Lists, and the Event Logs.
- **Timeline analysis** — building a super timeline with Plaso and reviewing it
  in Autopsy or Timeline Explorer.

More detailed, hands-on posts will land here over time.

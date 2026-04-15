---
draft: false
comments: true
date: 2026-02-27
tags: [wireshark]
---

# Wireshark Lua Dissector Debugging Experience

Recently, I discovered that my Lua script for Wireshark fails on PCAP files captured in unit tests—the RTP dissector stops without any error message. After spending considerable time debugging, I identified the issue: the RTP header version was incorrect.

<!-- more -->

## Root Cause
By comparing the script with an LLM-generated one (which manually parses the RTP header), and the LLM version revealed that the RTP header version in my test PCAP differed from standard captures.

## Debugging Lessons Learned
Prior attempts at debugging and LLM queries proved ineffective until I manually reviewed the raw data. My key takeaway: **Always parse raw data directly for deeper Wireshark Lua dissection.**
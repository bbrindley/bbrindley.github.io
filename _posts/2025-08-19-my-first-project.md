---
layout: post
title: "My First Cybersecurity Project"
date: 2025-08-19
---

This is my first post!  
Over the past couple of weeks, I followed along with Gerald Auger’s SOC Analyst project to build my own red-team/blue-team lab. The goal was simple: simulate attacker activity using a C2 framework (Sliver) and observe how an endpoint detection and response (EDR)-style tool (LimaCharlie) logs and detects it.

The process was anything but simple. In fact, the struggles became the most valuable part of the journey. Here’s how it went.  

**Setting Up The Lab**  
I started with two virtual machines in VMware:

Ubuntu as my attacker box (running Sliver C2).

Windows 10 as the victim endpoint (running Sysmon + LimaCharlie agent).

This sounds straightforward until you’re actually in VMware. Disk sizes, ISO mounting, and boot order tripped me up multiple times. I had to resize the Windows VM’s hard disk and confirm the installer booted from the right drive before it finally worked.

**Hardening vs. Attacker Simulation**
Before playing attacker, I had to set the stage like one. That meant turning off defenses inside my Windows VM:
Microsoft Defender
SmartScreen
Real-time protection
Controlled folder access

I also installed Sysmon to get deeper process logging and configured the LimaCharlie agent to send events to the cloud.

**Installing and Running Sliver C2**
On my Ubuntu attacker box, I installed Sliver C2. This required setting up both the server and the client. Early on, I hit issues where the sliver-server binary wasn’t being found, which turned out to be a PATH problem. Eventually I had both running and connected.

I then spun up an HTTP listener and generated Windows payloads — EXEs meant to call back to my Sliver server.
This is where things got real. Microsoft Defender and SmartScreen hated these files. They were flagged as malicious the moment I tried to download them. Even renaming to .txt or zipping didn’t always work. The only consistent fix was turning off protections entirely.

**Implant Delivery and Callbacks**
After a lot of trial and error, I managed to get payloads like CLEVER_SLIPPER.exe and WET_ZEBRAFISH.exe onto my Windows VM. Once executed, they attempted to call back to my Ubuntu Sliver server.
Sometimes the sessions appeared in Sliver, sometimes they didn’t. I ran into port conflicts and job crashes more than once. But the point wasn’t to have a flawless C2 session — it was to understand the workflow.

**LimaCharlie Integration**
With Sysmon and the LimaCharlie agent running, I started to see events in the Live Feed:
NEW_PROCESS
TERMINATE_PROCESS
DNS_REQUEST

I could drill into events, expand JSON, and see things like parent processes, command-line arguments, and hashes.
At one point, I spent a while trying to enable FILE_CREATE and FILE_DELETE logs by adjusting exfil rules. Even when I couldn’t always get them to appear, I learned what defenders rely on: process trees, command-line telemetry, and hash lookups.

**Big Takeaways**
Here’s what I walked away with:
Environment setup is half the battle. VMware disk sizes, ISOs, and boot order gave me just as much trouble as C2 payloads. That’s part of the work.
Defenses are noisy but powerful. Defender and SmartScreen blocked me again and again. Attackers work around this; SOCs rely on it.
Not every attack works. Payloads crashed, listeners died, and callbacks failed. That’s real life — and defenders need to recognize failed attempts as much as successful ones.
Telemetry is the win. Even when my payload didn’t call back, Sysmon + LimaCharlie logged process creation and network attempts. Analysts don’t need malware to succeed to know something’s wrong.
Adversary mindset sharpens defense. Playing attacker gave me empathy for defenders. Now when I see a random whoami.exe in logs, I know exactly what it means in context.

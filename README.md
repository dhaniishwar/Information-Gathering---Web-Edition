**Platform:** Hack The Box Academy 

**Module:** Information-Gathering---Web-Edition

**Difficulty:** Medium

**Target OS:** Linux

**Attacker OS:** Kali Linux

**Target IP:** 154.57.164.74

**Target Port:** 30886

**Date:** 2026-05-17

---
***GOAL:***

 The goal of this lab was to perform full web reconnaissance on the target domain inlanefreight.htb to answer 5 questions:
<br>

1. Find the IANA ID of the registrar of `inlanefreight.com`.
2. Find the HTTP server software powering `inlanefreight.htb`.
3. Find the API key in the hidden admin directory.
4. Find the email address by crawling `inlanefreight.htb`.
5. Find the API key the developers were planning to change.

---
***Tools Used:***

| **Tool** | **Purpose** |
|:------:|:---------:|
| whois | Domain registration lookup |
| nmap | Port scanning and service detection |
| ffuf | Virtual host and subdomain enumeration |
| ReconSpider.py | Web crawling to extract emails and comments |
| Firefox | Manual browsing of discovered pages |

---
***WalkThrough :***

***WHOIS Lookup:***

&nbsp;&nbsp;&nbsp;&nbsp; The first step was to gather public registration information about the real domain `inlanefreight.com`.
<br>

```bash
whois inlanefreight.com
```

<img width="582" height="600" alt="1" src="https://github.com/user-attachments/assets/ef0704c6-0870-40eb-a072-f8e1f5d642e9" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; We identified IANA ID: 468 from WHOIS lookup, completing our first goal.

---
***Port Scanning:***

<br>
&nbsp;&nbsp;&nbsp;&nbsp; Before scanning the port, we need to add the target to /etc/hosts because public internet DNS servers don't know they exist.
<br>

```bash
sudo nano /etc/hosts
```

<img width="245" height="50" alt="2" src="https://github.com/user-attachments/assets/17f7a90d-e6af-40e0-9c71-b220be748c9d" />
<br>
<img width="354" height="133" alt="3" src="https://github.com/user-attachments/assets/fea888b4-0cf1-4d91-9d1d-c7d72e18041a" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; Let's run an nmap scan against the provided port to identify the running service.
<br>

<img width="612" height="170" alt="4" src="https://github.com/user-attachments/assets/ce58e0f0-2868-4b75-afc3-d3a388a791c6" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; Nmap scan revealed that the port was running nginx version 1.26.1, second goal was completed.

---
***HTTP Enumeration:***

&nbsp;&nbsp;&nbsp;&nbsp; 


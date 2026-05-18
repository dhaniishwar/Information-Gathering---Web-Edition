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
<br>

<h4 align = center>First goal was completed</h4>

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
&nbsp;&nbsp;&nbsp;&nbsp; Nmap scan revealed that the port was running nginx version 1.26.1.
<br>

<h4 align = center>Second goal was completed</h4>

---
***HTTP Enumeration:***

&nbsp;&nbsp;&nbsp;&nbsp;  Let's visit `http://inlanefreight.htb:30886` in Firefox to see we can get anything.
<br>

```bash
http://inlanefreight.htb:30886
````

<img width="483" height="248" alt="8" src="https://github.com/user-attachments/assets/0d9e1337-c56f-4d0a-90c8-ffa4b74c906f" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; We found a basic placeholder page saying "Welcome to inlanefreight.htb" with no useful content. This was a signal that the real content was likely hosted on subdomains rather than the main domain. Now, we have to use ffuf tool to discover hidden subdomains.
<br>

```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u 'http://154.57.164.74:30886' -H "Host: FUZZ.inlanefreight.htb" -fs 120
```

<img width="788" height="375" alt="9" src="https://github.com/user-attachments/assets/ac15031a-c0bb-4109-9514-67baf6319fbf" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; ffuf discovered web1337.inlanefreight.htb as a valid virtual host. Before visiting the subdomain in Firefox, we need to add the subdomain in /etc/hosts as we did for inlanefreight.htb.
<br>

<img width="255" height="194" alt="7" src="https://github.com/user-attachments/assets/ffaeaa01-2a2c-479f-b765-498afb49f5ba" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; Now, we can visit the subdomain in Firefox.
<br>
<br>

```bash
http://web1337.inlanefreight.htb:30886
```

<img width="483" height="248" alt="8" src="https://github.com/user-attachments/assets/ba944404-bb62-43d0-bd2b-60e8484778bf" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; We did not get any information from the subdomain also. Let's Brute force again by ffuf to identify there is any hidden directory or file are present in subdomain.
<br>

```bash
ffuf -w /usr/share/wordlists/dird/common.txt -u 'http://web1337.inlanefreight.htb:30886/FUZZ' -ic
```

<img width="788" height="375" alt="9" src="https://github.com/user-attachments/assets/cb92f329-642a-465d-86ce-6c8c41228933" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; We have discover robots.txt file was present in the subdomain. Let's navigat to robots.txt
<br>

```bash
http://web1337.inlanefreight.htb:30886/robots.txt
```

<img width="478" height="230" alt="10" src="https://github.com/user-attachments/assets/17632022-0926-4b4e-a92d-8bcba4fbfb40" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; The robots.txt file revealed a disallowed path: /admin_h1dd3n. Let's navigat to it
<br>

```bash
http://web1337.inlanefreight.htb:30886/admin_h1dd3n/
```

<img width="480" height="245" alt="11" src="https://github.com/user-attachments/assets/b45de6d9-6a4e-4bbb-9c3f-5837557529e6" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; The page displayed a message stating the admin panel was under maintenance but that the API was still accessible along with its key in plain text.
<br>

<h4 align =center>Third goal is completed</h4>

---
***Crawling dev with ReconSpider:***

<br>
&nbsp;&nbsp;&nbsp;&nbsp; We can ran ffuf again, this time fuzzing for subdomains of web1337.inlanefreight.htb specifically.
<br>

```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u 'http://154.57.164.74:30886' -H "Host: FUZZ.web1337.inlanefreight.htb" -fs 120
```

<img width="1118" height="368" alt="12" src="https://github.com/user-attachments/assets/7eaa350f-1e80-4b9c-91c3-d641190985c3" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; This revealed a second subdomain: dev.web1337.inlanefreight.htb. After adding the dev subdomain to /etc/hosts, visit the div subdomain in Firefox.
<br>

```bash
http://dev.web1337.inlanefreight.htb:30886
```

<img width="479" height="247" alt="13" src="https://github.com/user-attachments/assets/4d9a914b-181b-4ce1-a01e-a383fe708419" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; Again we didn't get any information from Firefox. This time, let's use ReconSpider against dev subdomain. ReconSpider is a Python-based web crawler that extracts emails, links, JavaScript files, images and importantly — HTML comments.
<br>

```bash
python3 ReconSpider.py http://dev.web1337.inlanefreight.htb
```

<img width="526" height="41" alt="15" src="https://github.com/user-attachments/assets/ec01f4da-9ac1-431d-a939-48259e3b7def" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; The output is saved in a file named 'results.json'. Let's see want we got
<br>

```bach
cat results.json
```

<img width="530" height="263" alt="16" src="https://github.com/user-attachments/assets/5663680d-9363-4dcd-bec3-11c59feb1d68" />
<br>
<img width="554" height="343" alt="17" src="https://github.com/user-attachments/assets/89e2bab0-ad50-47a1-ab90-de08e0ca27cc" />

<br>
&nbsp;&nbsp;&nbsp;&nbsp; The results.json output revealed an email address 1337testing@inlanefreight.htb and an HTML comment left by a developer saying "Remember to change the API key to ba988b835be4aa97d068941dc852ff33".
<br>

<h4 align =center> fourth and fifth goals are completed </h4>


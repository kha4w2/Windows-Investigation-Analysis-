# Effectively Using Splunk – Scenario 1 (LAB 8)

## Overview

This lab is based on the **Boss of the SOC (BOTS) v1** dataset and focuses on investigating an **Advanced Persistent Threat (APT)** incident using **Splunk SIEM**. The goal is to understand how to leverage Splunk search capabilities to identify attacker activity across different stages of the **Cyber Kill Chain**.

The organization under investigation is **Wayne Enterprises**, and the targeted public-facing website is:

```
imreallynotbatman.com
```

Splunk ingests multiple log sources including:

* Windows Event Logs
* Sysmon Logs
* Fortinet Firewall Logs
* Suricata IDS Logs

---

## What is an APT (Advanced Persistent Threat)

An **APT** is a targeted attack carried out by an organized group or entity. These attackers:

* Maintain long-term access to the network (months or more)
* Operate stealthily
* Perform suspicious and goal-oriented actions over time

---

## Log Sources Used

* **Sysmon Logs**: Monitor system-level activity such as running processes on Windows systems.
* **Windows Event Logs**: Record all activities occurring on Windows machines.
* **Firewall Logs**: Track permitted and denied traffic through the firewall.
* **Suricata Logs**: Host-based Intrusion Detection System (HIDS) logs based on signatures. Alerts are generated when traffic matches malicious signatures.

As SOC Analysts / Incident Responders, the task is to utilize Splunk searches to investigate and respond to the incident.

---

## Splunk Components


<img width="858" height="323" alt="image" src="https://github.com/user-attachments/assets/130c2ac4-a50b-457e-9a0e-f30fd67ecaff" />


### Search Head (Splunk User Interface)

* Used to define time ranges
* Executes **SPL (Splunk Processing Language)** queries
* Enables searching, filtering, and analyzing logs

### Indexer

* Responsible for data storage and processing
* Core Splunk component
* Indexes, categorizes, and organizes data based on time and content

### Forwarder

* Installed on endpoints and network components
* Collects logs and sends them to the Indexer

**Flow:**

```
Forwarder → Indexer → Search Head
```

---

## Scenario Context

Wayne Enterprises has been hit by an APT group. The SOC manager requested a full investigation using Splunk to identify attacker activity across the Cyber Kill Chain.

---

## Cyber Kill Chain Explanation


<img width="645" height="737" alt="image" src="https://github.com/user-attachments/assets/5c793b45-e76a-4792-a93d-4a240d8362ce" />


The **Cyber Kill Chain** represents the steps an attacker follows to conduct an attack:

1. **Reconnaissance**
   Gathering public information such as email addresses, domains, and services.

2. **Weaponization**
   Coupling exploits with backdoors into a deliverable payload.

3. **Delivery**
   Sending the payload via email, web traffic, or removable media.

4. **Exploitation**
   Exploiting vulnerabilities to execute code.

5. **Installation**
   Installing malware on the target system.

6. **Command & Control (C2)**
   Establishing remote communication with the victim.

7. **Actions on Objectives**
   Achieving the attacker’s final goals with hands-on access.

---

## Task 1 – Reconnaissance Detection

### Objective

Identify reconnaissance activities targeting the organization using Splunk searches.

---

## Step 1: View All Available Events

<img width="975" height="517" alt="image" src="https://github.com/user-attachments/assets/44dc6e43-faa2-477a-9abf-4dad3afc7c8b" />


```spl
index="botsv1" earliest=0
```

* `index="botsv1"`: Search all events from the BOTS dataset
* `earliest=0`: Start from the oldest available timestamp

---

## Step 2: Search for the Target Domain


<img width="975" height="516" alt="image" src="https://github.com/user-attachments/assets/0c3dac91-c748-42d4-bbcd-269314daf53d" />


```spl
index="botsv1" earliest=0 imreallynotbatman.com
```

This search returns all events where the target domain appears.

---

## Step 3: Identify Relevant Sourcetypes

<img width="975" height="496" alt="image" src="https://github.com/user-attachments/assets/73cd9d9b-c9c5-412b-a434-758607a11bbc" />


By analyzing the `sourcetype` field, the following sources were identified:

* **suricata** (IDS alerts)
* **stream:http** (HTTP traffic)
* **fgt_utm** (Fortinet firewall logs)
* **iis** (Windows web server logs)

This confirms that multiple security and network components observed traffic related to the domain.

---

## Step 4: Identify Top Source IP Addresses


<img width="975" height="443" alt="image" src="https://github.com/user-attachments/assets/b119dcf2-6362-4773-a039-0485d84f0a79" />


```spl
index="botsv1" earliest=0 imreallynotbatman.com
```

Top source IPs interacting with the domain:

| Source IP      | Request Count |
| -------------- | ------------- |
| 40.80.148.42   | 30,166        |
| 192.168.250.70 | 11,493        |
| 23.22.63.114   | 2,884         |

The IP **40.80.148.42** shows significantly higher activity.

---

## Step 5: Focus on HTTP Reconnaissance Traffic


<img width="975" height="445" alt="image" src="https://github.com/user-attachments/assets/23d9825f-aabd-4a7c-804f-ba1d0d026609" />


```spl
index="botsv1" earliest=0 imreallynotbatman.com sourcetype=stream:http
```

This limits the results to HTTP traffic, which is commonly used during reconnaissance and scanning.

---

## Step 6: Identify Most Active Source IP


<img width="975" height="471" alt="image" src="https://github.com/user-attachments/assets/5e554aca-c4a6-4371-b700-5168bbcf0401" />


```spl
index="botsv1" earliest=0 imreallynotbatman.com sourcetype=stream*
| stats count(src_ip) as requests by src_ip
| sort -requests
```

Results confirm:

* **40.80.148.42** generated ~95% of HTTP requests

---

## Step 7: Validate Using Suricata IDS Alerts


<img width="975" height="466" alt="image" src="https://github.com/user-attachments/assets/d721d9ac-24f5-4935-ac75-51048d40d303" />


```spl
index="botsv1" earliest=0 imreallynotbatman.com src=40.80.148.42 sourcetype=suricata
```

Suricata generated **17,484 alerts** related to this source IP.

---

## Step 8: Analyze Triggered Signatures

<img width="975" height="447" alt="image" src="https://github.com/user-attachments/assets/d436fb95-dce8-4214-abf3-f5095526d49e" />

<img width="975" height="447" alt="image" src="https://github.com/user-attachments/assets/09a21446-d353-464a-87b7-7cd6ecfd028d" />


<img width="975" height="469" alt="image" src="https://github.com/user-attachments/assets/3890240e-a5ae-4520-b501-bbb1e07cc679" />

A total of **46 Suricata signatures** were triggered, including:

* Cross-Site Scripting (XSS) attempts
* SQL Injection attempts
* XXE attacks
* PHP injection patterns
* Known CVE exploit attempts

This confirms malicious scanning behavior rather than normal browsing.

---

## Step 9: Identify Scanning Tool via HTTP Headers


<img width="975" height="470" alt="image" src="https://github.com/user-attachments/assets/b8179c26-01f6-496d-8820-be6e5fc6e1ce" />


<img width="975" height="468" alt="image" src="https://github.com/user-attachments/assets/12c537ac-b167-49f5-960e-230fc3436306" />



```spl
index="botsv1" src_ip=40.80.148.42 sourcetype=stream:http
```

By inspecting the `src_headers` field, the following User-Agent was observed:

```
Acunetix Web Vulnerability Scanner - Free Edition
```

---

## Conclusion – Task 1 Answer

The reconnaissance phase was performed using an automated vulnerability scanning tool.

**Final Answer:**

> The APT group utilized an instance of the reputable **Acunetix Web Vulnerability Scanner** during the reconnaissance phase.

---

*End of Task 1 – Reconnaissance*


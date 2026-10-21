<img width="1350" height="541" alt="image" src="https://github.com/user-attachments/assets/b3b8f13a-1cb9-449a-bee4-753e2e448652" />

# Network Enumeration with Nmap — HTB Academy

## Overview

This repository documents my hands-on learning journey through the **Network Enumeration with Nmap** module on Hack The Box Academy.

The module focuses on network reconnaissance, host discovery, TCP and UDP port scanning, service enumeration, operating system identification, and interpreting network responses.

The primary objective is to develop a structured enumeration methodology, understand how Nmap interacts with network protocols, and accurately identify potential attack surfaces during authorized penetration testing engagements.

This repository will be updated progressively as I complete each section and practical exercise.

## Learning Objectives

- Understand the fundamentals of network enumeration.
- Discover active hosts within a network.
- Identify open, closed, and filtered ports.
- Understand TCP SYN scanning and the TCP three-way handshake.
- Perform full TCP port scans.
- Identify running services and software versions.
- Analyze ICMP responses and TTL values.
- Understand ARP-based and ICMP-based host discovery.
- Explore operating system detection and service fingerprinting.
- Use Nmap results to guide further security assessments.
- Document findings using a structured penetration testing methodology.

## Tools and Technologies

| Tool / Technology | Purpose |
|---|---|
| Nmap | Network discovery and enumeration |
| Linux Terminal | Command-line interaction |
| TCP/IP | Network communication and transport protocols |
| ICMP | Host discovery and connectivity analysis |
| ARP | Local network host discovery |
| NSE | Nmap Scripting Engine |
| SSH | Remote access service identification |
| SMB | Network service and hostname enumeration |

## Module Topics

The documentation is organized into individual Markdown files to keep each topic clear, practical, and easy to review.

| Topic | Description |
|---|---|
| Introduction to Nmap | Nmap fundamentals, scanning techniques, and TCP SYN scanning |
| Host Discovery | Discovering active hosts using ARP, ICMP, and target lists |
| Host and Port Scanning | Identifying TCP ports and enumerating target hosts |
| Service Enumeration | Identifying network services and their versions |
| Advanced Scanning | Additional Nmap scanning techniques and options |
| Nmap Scripting Engine | Using NSE scripts for further enumeration |
| Firewall and IDS Evasion | Understanding filtering behavior and scan limitations |

Topics will be documented as I progress through the module.

## Practical Commands

The following commands represent techniques studied and practiced during the initial sections of the module.

### TCP SYN Scan

```bash
sudo nmap -sS <target>
```

Performs a TCP SYN scan to identify the state of target ports without completing the standard TCP three-way handshake.

### Full TCP Port Scan

```bash
sudo nmap -sS -p- --open <target>
```

Scans all 65,535 TCP ports and displays ports identified as open.

### Network Host Discovery

```bash
sudo nmap -sn <network>/24
```

Discovers active hosts without performing a port scan.

### ICMP Echo Discovery

```bash
sudo nmap -sn -PE --disable-arp-ping <target>
```

Requests ICMP Echo-based discovery while disabling ARP ping discovery.

### Packet Analysis

```bash
sudo nmap -sn -PE --packet-trace <target>
```

Displays packets sent and received during the discovery process.

### Scan Result Explanation

```bash
sudo nmap -sn --reason <target>
```

Displays the reason Nmap assigned a particular host state.

### Save Scan Results

```bash
sudo nmap -sn -oA host-discovery <target>
```

Saves scan results in Nmap's normal, XML, and grepable output formats.

### Service and Operating System Detection

```bash
sudo nmap -sV -O <target>
```

Attempts to identify running service versions and the target operating system.

## Understanding Port States

| State | Description |
|---|---|
| Open | A service is accepting connections on the port |
| Closed | The port is reachable, but no service is accepting connections |
| Filtered | Packet filtering prevents Nmap from determining the port state |
| Open/Filtered | Nmap cannot determine whether the port is open or filtered |

Understanding these states is essential for interpreting scan results and identifying potential network security controls.

## Understanding TTL Values

The Time to Live (TTL) field can provide useful clues about a target's operating system.

Common initial TTL values include:

| Initial TTL | Common Association |
|---|---|
| 64 | Linux / Unix |
| 128 | Windows |
| 255 | Network devices and other systems |

TTL-based operating system identification is an estimate, not definitive proof. Network hops and system configurations can affect the observed value.

## Enumeration Methodology

My approach to network enumeration follows a structured process:

1. Identify the authorized target or network range.
2. Discover active hosts.
3. Perform initial TCP port scanning.
4. Conduct full-port enumeration when appropriate.
5. Identify running services and software versions.
6. Analyze responses, protocols, and possible filtering.
7. Perform additional service-specific enumeration.
8. Document findings and potential security risks.

The goal is not simply to run scanning tools, but to understand the information they provide and use it to make informed decisions.

## Repository Structure

```text
06-Network-Enumeration-with-Nmap/
│
├── README.md
├── 01-Introduction-to-Nmap.md
├── 02-Host-Discovery.md
├── 03-Host-and-Port-Scanning.md
├── 04-Service-Enumeration.md
└── ... Additional topics as completed
```

The topic files shown above represent the planned documentation structure and will be created progressively.

## Key Takeaways

Network enumeration is one of the most important stages of penetration testing.

Nmap provides powerful capabilities for discovering hosts, identifying exposed services, and understanding network behavior. However, accurate interpretation requires knowledge of TCP/IP, ICMP, ARP, operating systems, and service-specific protocols.

A successful enumeration process combines automated scanning with manual investigation and technical analysis.

**Enumeration is the foundation of effective penetration testing.**

## Learning Path

**Platform:** Hack The Box Academy  
**Module:** Network Enumeration with Nmap  
**Focus:** Network Reconnaissance and Enumeration  
**Learning Goal:** HTB Certified Penetration Testing Specialist (CPTS) preparation  
**Progress:** In Progress

## Disclaimer

All activities documented in this repository are performed in authorized cybersecurity training environments.

This material is intended for educational purposes, ethical hacking, and professional cybersecurity skill development.

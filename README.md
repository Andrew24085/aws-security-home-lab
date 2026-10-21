# AWS Security Home Lab

A small corporate-style network built in AWS where I attack my own machines, detect the activity in logs, fix the weaknesses, and attack again to confirm the fixes work.

**Status:** In progress
**Started:** October 2026
**Author:** Andrew Chigbu

## Goal

Practice the daily work of a SOC analyst in a safe environment: run an attack, find it in the logs, write an alert or rule, apply a fix, and retest. Everything is documented so each step can be repeated.

## Tools

| Purpose | Tool |
|---|---|
| Cloud platform | AWS (VPC, EC2, security groups, CloudTrail, VPC Flow Logs) |
| SIEM | Splunk |
| Attacker | Kali Linux |
| Targets | Ubuntu Server, Windows |
| Diagramming | draw.io |

## Network Diagram

![Network diagram](screenshots/network-diagram.png)

*Add the diagram after the VPC and instances are built.*

## Safety Rules

- All attacks are run only against machines I own in this lab.
- Security groups allow inbound traffic only from my own IP address and from the attacker machine.
- No access keys, passwords, or account IDs are published in this repo.
- Instances are stopped or terminated when not in use, and a billing alert is set.
- Testing follows the AWS penetration testing policy (no denial-of-service attacks).

## Phase 1: Account Setup

- Enabled MFA on the root account
- Created a $10 monthly cost budget with an email alert
- Worked in one region (us-east-2) so nothing is left running elsewhere
![Budget alert](screenshots/01-budget-alert.png)

**Notes / problems and fixes:**
- The IAM user I created had no console sign-in option in my console, so I used the root account with MFA for this lab. In a production account I would use IAM Identity Center or a dedicated admin user.
## Phase 2: Network (VPC)

- VPC CIDR: `10.0.0.0/16`
- One public subnet, no NAT gateway (to avoid cost)
- Access controlled with security groups

![VPC created](screenshots/02-vpc.png)

**Design decisions:**
- No Nat gateway, because it costs about $30 a month and isn't needed
- One subnet keeps the lab simple
- Two security groups:
-   kali-sg: SSH allowed only from my IP
-   targets-sg: SSH from my IP, and all traffic from kali-sg so attacks can reach targets

## Phase 3: Machines

| Machine | Role | OS | Notes |
|---|---|---|---|
| kali-attacker | Attacker | Kali Linux | t3.small |
| ubuntu-target | Linux server | Ubuntu Server | t3.micro |
| Target 2 | Windows host | Windows | |
| SIEM | Log collection | Ubuntu + Splunk | |

![Instances running](screenshots/03-instances.png)

## Phase 4: Logging and Detection

- VPC Flow Logs enabled on lab-vpc (filter: all traffic, 1-minute aggregation) and sent to CloudWatch Logs
- Searched the logs with CloudWatch Logs Insights
- Splunk to be added later

Detection query (count distinct ports touched per source):
fields srcAddr, dstAddr, dstPort
| filter dstAddr = "10.0.3.157"
| stats count_distinct(dstPort) as ports_touched by srcAddr
| sort ports_touched desc

### Scenario 1: Port scan

| Attack | nmap -sV from Kali (10.0.7.114) against the target (10.0.3.157) |
| What the logs showed | About 2,000 connection records in a few minutes. Kali touched 866 distinct ports on the target, while every other source touched 54 or fewer. All records were ACCEPT because targets-sg allowed all traffic from kali-sg |
| Detection | Logs Insights query counting distinct destination ports per source. A source touching hundreds of ports in minutes indicates a scan |
| Fix | Replaced the "all traffic from kali-sg" rule in targets-sg with SSH (port 22) only, following least privilege |
| Retest | Same scan: 999 ports now filtered (no response) instead of closed, and scan time rose from 0.45s to 5.07s. Flow logs show 1,999 REJECT records at the target's interface and only SSH accepted |

![Before](screenshots/05-portscan-before.png)
![After](screenshots/05-portscan-after.png)

### Scenario 2: SSH brute force

| Step | Details |
|---|---|
| Attack | |
| What the logs showed | |
| Detection | |
| Fix | |
| Retest | |

### Scenario 3: Windows failed logons

| Step | Details |
|---|---|
| Attack | |
| What the logs showed | |
| Detection | |
| Fix | |
| Retest | |

## Problems and Fixes

- Powershell path with spaces needed qoutes
- SSH "permission denied (publickey)": the Kali image uses the username kali, not ubuntu
- Ran a Linux ping command in PowerShell by mistake; learned to double check the prompt before running commands

## What I Learned

- Flow logs record traffic at each network interface, so one packet appears twice (outgoing at the attacker, incoming at the target)
- "Filtered" (no response) and "closed" (reset) ports behave differently, and a firewall drop is silent
- A server with a public IP gets probed by internet scanners within minutes. My security group rejected them
- [What a security group can and cannot stop]
- [What I would add next]

## Next Steps

- [ ] Add GuardDuty findings
- [ ] Build the same lab locally with pfSense
- [ ] Write more detection rules

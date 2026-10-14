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
- Created a monthly cost budget with an email alert
- Chose one region and used it for all resources

![Budget alert](screenshots/01-budget-alert.png)

**Notes / problems and fixes:**
- [Example: IAM user could not be given console access, so I used root with MFA for this lab.]

## Phase 2: Network (VPC)

- VPC CIDR: `10.0.0.0/16`
- One public subnet, no NAT gateway (to avoid cost)
- Access controlled with security groups

![VPC created](screenshots/02-vpc.png)

**Design decisions:**
- [Why one subnet, why no NAT gateway, how security groups separate the machines.]

## Phase 3: Machines

| Machine | Role | OS | Notes |
|---|---|---|---|
| Kali | Attacker | Kali Linux | |
| Target 1 | Linux server | Ubuntu Server | |
| Target 2 | Windows host | Windows | |
| SIEM | Log collection | Ubuntu + Splunk | |

![Instances running](screenshots/03-instances.png)

## Phase 4: Logging and Detection

- Log sources connected to Splunk: [Linux auth logs, Windows event logs, VPC Flow Logs, CloudTrail]
- [Describe how logs get to Splunk.]

![Splunk receiving data](screenshots/04-splunk.png)

## Phase 5: Attack, Detect, Fix, Retest

Repeat this section for each scenario.

### Scenario 1: Port scan

| Step | Details |
|---|---|
| Attack | [Nmap command and target] |
| What the logs showed | [Events, fields, and search used to find it] |
| Detection | [Splunk search or alert written] |
| Fix | [Security group change or hardening applied] |
| Retest | [Result after the fix] |

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

## What I Learned

- [How the attacks appeared in the logs]
- [What a security group can and cannot stop]
- [What I would add next]

## Next Steps

- [ ] Add GuardDuty findings
- [ ] Build the same lab locally with pfSense
- [ ] Write more detection rules

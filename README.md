# Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS

## Introduction

Security teams often struggle with slow response times to compromised devices in enterprise environments due to manual alerting and remediation processes. To address this, I designed and deployed an automated threat response workflow leveraging cloud, SIEM, SOAR, and messaging integrations on AWS.

The system detects suspicious activity on a managed endpoint, triggers a notification workflow requiring human confirmation, and then automatically isolates the compromised device or user account through directory services and cloud orchestration.

This project highlights my skills in administering directory services, configuring secure cloud network communications, ingesting and analyzing security telemetry, and building automated workflows for incident response.

---

## Table of Contents

- [Setup and Prerequisites](#setup-and-prerequisites)
- [Architecture](#architecture)
- [Firewall Setup](#firewall-setup)
- [🖥️ Active Directory Setup](#active-directory-setup)
  - [Domain Controller Configuration](#domain-controller-configuration)
  - [DNS Roles & GPO Policies](#dns-roles--gpo-policies)
  - [User Creation in AD & Machine Join](#user-creation-in-ad--machine-join)
  - [Joining Test Machine to Domain](#joining-test-machine-to-domain)
- [📊 Splunk Installation & Configuration](#splunk-installation--configuration)
  - [EC2 Setup for Splunk](#ec2-setup-for-splunk)
  - [Universal Forwarder on AD & Test Client](#universal-forwarder-on-ad--test-client)
  - [Inputs for AD Logs & Windows Events](#inputs-for-ad-logs--windows-events)
  - [Alert Creation](#alert-creation)
- [🔁 SOAR Integration with Shuffle & Slack](#soar-integration-with-shuffle--slack)
  - [Setup of Shuffle Playbooks](#setup-of-shuffle-playbooks)
  - [Slack Notification Configuration](#slack-notification-configuration)
  - [User Input & Automated Account Disable via AD](#user-input--automated-account-disable-via-ad)
  - [Confirmation & Final Notification](#confirmation--final-notification)
- [🔐 Automated Response Workflow](#automated-response-workflow)
  - [Detection > Slack Alert > User Account Disable via AD](#detection--slack-alert--user-account-disable-via-ad)
  - [Workflow Diagrams & Screenshots](#workflow-diagrams--screenshots)
- [Incident Simulation](#incident-simulation)
- [🧠 Key Learnings & Takeaways](#key-learnings--takeaways)
  - [Skills Demonstrated](#skills-demonstrated)
  - [Challenges Faced](#challenges-faced)
  - [Future Enhancements](#future-enhancements)
- [Automated Phishing Analysis Workflow with n8n](#automated-phishing-analysis-workflow-with-n8n)
- [📚 References & Documentation](#references--documentation)
- [🧑‍💻 Author & Contact](#author--contact)

---

## Setup and Prerequisites

### Prerequisites

- **Cloud Environment**: Access to AWS with permissions to deploy and manage EC2 instances and configure networking components
- **Windows Server Administration**: Proficient in managing Windows Server and Active Directory environments
- **Domain Management**: Experience in joining and administering domain-joined Windows clients
- **Log Aggregation**: Skilled in centralized log collection and processing from diverse sources
- **Automation & Orchestration**: Ability to design and implement automated workflows with multi-channel alerting mechanisms
- **Network Security**: Expertise in configuring secure remote access and managing firewall/security group policies for inter-instance communication

### Setup Diagram

Below is the architecture and setup diagram for this Automated Threat Response Lab:

![Setup Layout Diagram](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/raw/main/Setup/Layout%20Diagram.jpg)

---

## Architecture

This lab leverages AWS to simulate a production-like environment for automated threat response workflows. The core components are:

- **Active Directory Domain Controller** on Windows Server EC2.
- **Windows Test Client** joined to the AD domain.
- **Splunk SIEM** hosted on a dedicated EC2 instance.
- **Shuffle SOAR** for automating incident response and integrations (Shuffle SOAR is deployed as a managed service accessible over the internet, so no AWS EC2 instance is required for SOAR in this lab).
- **Slack** for real-time alerting and human-in-the-loop confirmation.
- **AWS Security Groups & VPC** for network segmentation and control.

**EC2 Instance Specifications:**

| Component                          | OS/Platform      | Instance Type    | vCPUs | RAM   | Notes                                  |
|-------------------------------------|------------------|------------------|-------|-------|----------------------------------------|
| Active Directory Domain Controller  | Windows Server   | t3.xlarge or up  | 4     | 16GB  | Recommended for responsive directory services |
| Windows Test Client                 | Windows 10/11    | t3.large or up   | 2     | 8GB   | For basic domain-joined operations     |
| Splunk Server                       | Ubuntu Server    | t3.xlarge or up  | 4     | 16GB  | Splunk runs on Ubuntu for this lab     |
| **General Note**                    |                  |                  |       |       | Adjust as needed for cost or scale     |

**High-level flow:**
1. Suspicious activity is detected on the Windows client.
2. Logs are forwarded to Splunk for analysis and correlation.
3. Splunk triggers an alert, sending details to Shuffle.
4. Shuffle orchestrates a notification to Slack, requesting human confirmation.
5. Upon approval, Shuffle automates disabling the compromised user account in Active Directory.

---

[...existing content remains unchanged up to "Future Enhancements" section...]

## Automated Phishing Analysis Workflow with n8n

### Introduction

Email remains one of the most common attack vectors for phishing campaigns. Security teams often face challenges in rapidly analyzing suspicious emails due to manual link inspection and delayed notifications. To address this, I designed and deployed an automated phishing analysis workflow using n8n.

The system ingests incoming Outlook emails, parses messages for indicators of compromise, and checks embedded URLs against URLscan.io and VirusTotal using their APIs. Results are merged into a consolidated phishing report, which is automatically delivered to a designated Slack channel for real-time security visibility.

This project highlights my skills in:

- Integrating cloud-based APIs (Outlook, Slack, VirusTotal, URLscan.io)
- Automating phishing analysis workflows using n8n
- Writing custom detection logic with JavaScript for indicator extraction
- Building scalable security reporting pipelines that support rapid incident response

### Table of Contents

- Setup and Prerequisites
- Architecture
- Workflow Layers
  - Step 1: Trigger & Scheduling
  - Step 2: Email Processing
  - Step 3: IOC Detection & URL Extraction
  - Step 4: Threat Intelligence Checks
  - Step 5: Report Generation
  - Step 6: Slack Notifications
- Configuration Details
- Sample Report
- Error Handling & Limitations
- Future Enhancements

### Setup and Prerequisites

- n8n (Cloud or self-hosted)
- Microsoft 365 / Outlook account with API access
- API Keys:
  - URLscan.io
  - VirusTotal (v3)
  - Slack Bot Token + Channel ID
- Slack App with chat:write scope enabled
- Basic JavaScript for IOC detection logic

### Architecture

The phishing analysis workflow follows a layered design:

- **Trigger & Scheduling:** Workflow runs at scheduled intervals to poll Outlook for unread emails.
- **Email Processing:** Unread emails are retrieved via Outlook API and marked as read.
- **IOC Detection & URL Extraction:** JavaScript node scans message content for indicators of compromise (IOCs) such as suspicious links.
- **Threat Intelligence Checks:** URLs are submitted to URLscan.io and VirusTotal for analysis.
- **Report Generation:** Results are normalized, merged, and scored for severity.
- **Slack Notifications:** A formatted phishing analysis report is posted to a dedicated Slack channel.

📸 *Insert workflow diagram screenshot here*

### Workflow Layers

#### Step 1: Trigger & Scheduling

- Configure Schedule Trigger to run every X seconds/minutes.
- Ensures continuous monitoring without manual checks.
- 📸 *Insert screenshot of Schedule Trigger node here*

#### Step 2: Email Processing

- Outlook Node: Retrieves unread messages.
- Mark as Read Node: Prevents reprocessing of the same email.
- 📸 *Insert screenshot of Outlook nodes here*

#### Step 3: IOC Detection & URL Extraction

- Split In Batches: Processes one email at a time.
- JavaScript Node: Extracts URLs from email body/headers.
- 📸 *Insert screenshot of IOC Detection logic here*

#### Step 4: Threat Intelligence Checks

- URLscan.io Node: Submits URL and retrieves scan results.
- VirusTotal Node: Submits URL and checks reputation.
- 📸 *Insert screenshot of URLscan.io node here*
- 📸 *Insert screenshot of VirusTotal node here*

#### Step 5: Report Generation

- Merge Node: Combines results from both APIs.
- Code Node: Scores severity and prepares structured report.
- 📸 *Insert screenshot of Merge + Report Builder here*

#### Step 6: Slack Notifications

- Slack Node: Sends report to a security channel.
- Provides real-time visibility for security teams.
- 📸 *Insert screenshot of Slack node here*

### Configuration Details

- Environment variables for API keys
- Rate limits for URLscan.io and VirusTotal
- Slack bot token setup and required scopes (`chat:write`)

### Sample Report

```
✉️ Phishing Scan
Subject: Verify your account
From: it-support@example.com
Received: 2025-09-05

Findings:
• https://example.bad/phish — Severity: High | VT(mal:7/sus:2) | URLscan: malicious
```

📸 *Insert screenshot of Slack message output here*

### Error Handling & Limitations

- API rate limits (URLscan.io & VirusTotal)
- Requires network connectivity to external APIs
- Only scans links, not attachments (future improvement)

### Future Enhancements

- Add attachment scanning (sandbox integration)
- Forward results into SIEM for long-term storage
- Expand detection to include suspicious keywords and domains
- Add caching to avoid resubmitting known safe URLs

---

[...remaining content: references, author, contact unchanged]
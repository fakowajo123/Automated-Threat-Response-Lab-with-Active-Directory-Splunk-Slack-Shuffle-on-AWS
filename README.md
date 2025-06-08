# Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS

## Introduction

Security teams often struggle with slow response times to compromised devices in enterprise environments due to manual alerting and remediation processes. To address this, I designed and deployed a lab that demonstrates an end-to-end automated threat response solution using Active Directory, Splunk, Slack, Shuffle (SOAR), and AWS infrastructure.

The system detects suspicious activity on a managed endpoint, triggers a notification workflow requiring human confirmation, and then automatically isolates the compromised device or user account to prevent further spread—all in a cloud lab environment.

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

**[Place Screenshot Here: AWS Resource Group/Network Architecture]**

---

## Architecture

This lab leverages AWS to simulate a production-like environment for automated threat response workflows. The core components are:

- **Active Directory Domain Controller** on Windows Server EC2.
- **Windows Test Client** joined to the AD domain.
- **Splunk SIEM** hosted on a dedicated EC2 instance.
- **Shuffle SOAR** for automating incident response and integrations (Shuffle SOAR is deployed as a managed service accessible over the internet, so no AWS EC2 instance is required for SOAR in this architecture).
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

**[Place Screenshot Here: High-level Flow/Process Diagram]**

---

## Firewall Setup

- Create a dedicated VPC or use an isolated subnet for the lab.
- Define Security Groups for:
  - Domain Controller (RDP, DNS, AD ports open only to test client and admin IPs).
  - Splunk Server (allow management and forwarder ports).
  - Test Client (limit RDP/WinRM, allow outbound to Splunk and DC).
- Use Network ACLs for extra segmentation.
- Ensure no public IP exposure except for admin access (ideally use VPN or jumpbox).

**[Place Screenshot Here: Security Group Rules/Network ACLs]**

---

## 🖥️ Active Directory Setup

### Domain Controller Configuration

- Deploy Windows Server EC2 (`t3.xlarge` or higher: 4 vCPUs, 16GB RAM).
- Install "Active Directory Domain Services" role.
- Promote server to Domain Controller (e.g., domain: `lab.local`).

**[Place Screenshot Here: AD DS Installation and Promotion Steps]**

### DNS Roles & GPO Policies

- Allow log forwarding to Splunk.

**[Place Screenshot Here: DNS Role Setup/GPO Policy Editor]**

### User Creation in AD & Machine Join

- In Active Directory, create a user account from the AD Users and Computers console. Make this user a member of the "Domain Admins" group so that it has administrative privileges.
- When deploying the Windows Test Client, join it to the `lab.local` domain using this user account.
- Log into the Test Client with the domain user credentials (now with admin rights) so you can install required software and perform administrative tasks on the test machine.

**[Place Screenshot Here: Creating User in AD & Assigning Domain Admin Rights]**

### Joining Test Machine to Domain

- Deploy a Windows EC2 as the test client (`t3.large` or higher: 2 vCPUs, 8GB RAM).
- Set DNS to the DC's private IP.
- Join to `lab.local` domain using the domain admin user you created.
- Test authentication and GPO application.

**[Place Screenshot Here: Joining Test Machine to Domain]**

---

## 📊 Splunk Installation & Configuration

### EC2 Setup for Splunk

- Launch a separate EC2 (Ubuntu Server, `t3.xlarge` or higher: 4 vCPUs, 16GB RAM).
- Download Splunk Enterprise from the official Splunk site and install it on Ubuntu. Use sudo as needed.
- To start Splunk for the first time, go to the Splunk installation directory's `bin` folder and run `./splunk start`.
- Set the username and password as prompted.
- Access the Splunk web interface via port 8000 and log in.
- Create a new index in Splunk that will be used for log ingestion from the forwarders.
- For further Splunk usage, users should refer to Splunk documentation and tutorials to learn about configuring searches, alerts, dashboards, and advanced features.

**[Place Screenshot Here: Splunk Installation, Initial Login, and Index Creation]**

### Universal Forwarder on AD & Test Client

- On both the Active Directory Domain Controller and the Windows Test Client, download and install the Splunk Universal Forwarder from the official Splunk website.
- Use the domain admin account to perform the installation so you have the necessary privileges.
- After installation, navigate to the forwarder directory: `C:\Program Files\SplunkUniversalForwarder\etc\system\default`.
- Copy the `inputs.conf` file from the `default` directory to the `local` directory (`C:\Program Files\SplunkUniversalForwarder\etc\system\local`) and edit the copy in the `local` directory.
- Add or modify the following stanza in `inputs.conf` for Windows security logs:
  ```
  [WinEventLog://Security]
  disabled = 0
  index = <your-index-name>
  ```
- Replace `<your-index-name>` with the name of the index you created earlier in Splunk.

**[Place Screenshot Here: Splunk Universal Forwarder Installation and Configuration]**

### Inputs for AD Logs & Windows Events

- In Splunk, configure data inputs for:
  - Windows Security, Application, System logs.
  - Directory Service logs (on DC).
- Use Splunk Add-on for Microsoft Windows for parsing and CIM compliance.

**[Place Screenshot Here: Data Inputs Setup in Splunk]**

### Alert Creation

- Create correlation searches in Splunk (e.g., multiple failed logins, privilege escalation).
- Configure alert actions to send events to Shuffle via webhook or app integration.
- For detailed instructions or advanced configurations, users are encouraged to learn more via Splunk's official documentation.

**[Place Screenshot Here: Splunk Alert Configuration]**

---

## 🔁 SOAR Integration with Shuffle & Slack

### Setup of Shuffle Playbooks

- In Splunk, under the alert rule’s webhook action, configure Shuffle as the webhook receiver.
- Shuffle receives the alert and parses the information about what triggered the alert (user, machine, event, etc.).
- This information is then sent from Shuffle to Slack via Shuffle’s Slack integration.
- Ensure Slack is added as an application to Shuffle and authenticated.

**[Place Screenshot Here: Splunk Alert Webhook to Shuffle]**

**[Place Screenshot Here: Shuffle Playbook Logic/Workflow]**

### Slack Notification Configuration

- Slack receives the alert with the details from Shuffle.
- The notification includes buttons or options for the analyst to approve or deny disabling the user account.
- The user’s decision (input) is captured by Shuffle for the next step.

**[Place Screenshot Here: Example Slack Alert with Action Buttons]**

### User Input & Automated Account Disable via AD

- If the analyst chooses to disable the user, Shuffle—using its Active Directory application and authenticated connection to the domain controller—issues a command to disable the user account in AD.
- Shuffle then queries the domain controller to confirm the status of the user account (check if it is disabled).

**[Place Screenshot Here: Shuffle AD Action/Confirmation of User Disabled]**

### Confirmation & Final Notification

- Once the user is confirmed as disabled, Shuffle sends a final notification to Slack stating that the user account has been successfully disabled.

**[Place Screenshot Here: Slack Final Confirmation Message]**

---

## 🔐 Automated Response Workflow

### Detection > Slack Alert > User Account Disable via AD

- End-to-end flow:
  1. Suspicious event logged and detected.
  2. Splunk triggers alert, sends to Shuffle.
  3. Shuffle notifies Slack and/or email.
  4. Analyst reviews and approves in Slack/email.
  5. Shuffle uses its AD integration to disable the affected user account via the domain controller.
  6. Shuffle confirms the user's account is disabled, then notifies Slack of the result.

**[Place Screenshot Here: End-to-End Workflow Diagram or Timeline]**

### Workflow Diagrams & Screenshots

- Include images of:
  - Splunk alert configuration.
  - Shuffle playbook graph.
  - Slack incident notification example.
  - Confirmation of user disablement in AD.

---

## Incident Simulation

- Example scenario:
  - Simulate brute-force attack on test client.
  - Observe detection in Splunk.
  - Confirm alert propagation to Shuffle and Slack.
  - Approve user disablement and verify account is disabled in AD.

- Steps for reproduction:
  1. Generate test event (manual logon failure, PowerShell execution).
  2. Follow alert through pipeline.
  3. Confirm automated response and restoration steps.

**[Place Screenshot Here: Incident Simulation Steps/Results]**

---

## 🧠 Key Learnings & Takeaways

### Skills Demonstrated
- AWS cloud & networking
- Windows AD deployment & GPOs
- Splunk SIEM data ingestion & alerting
- SOAR automation with Shuffle & Slack
- Automated and human-approved incident response

### Challenges Faced
- Ensuring reliable log forwarding from Windows to Splunk
- Orchestrating automation without losing human oversight
- Seamless integration between cloud, on-prem, and SaaS tools

### Future Enhancements
- Add MFA/JIT access for critical actions
- More automated playbooks (phishing, privilege escalation, etc.)
- Expand to other SIEM/SOAR platforms and cloud services

---

## 📚 References & Documentation

- [Active Directory Domain Services](https://docs.microsoft.com/en-us/windows-server/identity/active-directory-domain-services)
- [Splunk Install Guide](https://docs.splunk.com/Documentation/Splunk/latest/Installation/InstallonWindows)
- [Splunk Forwarder](https://docs.splunk.com/Documentation/Forwarder/latest/Forwarder/Deploytheuniversalforwarder)
- [Shuffle Docs](https://shuffler.io/docs/)
- AWS Security Blog, Reddit r/sysadmin, GitHub security labs

---

## 🧑‍💻 Author & Contact

- **Author:** fakowajo123
- **GitHub:** [fakowajo123](https://github.com/fakowajo123)
- **Email:** [fakowadeo@gmail.com](mailto:fakowadeo@gmail.com)
- **LinkedIn:** [fakowajo-oluwabukunmi-5144381a9](https://www.linkedin.com/in/fakowajo-oluwabukunmi-5144381a9/)

Feel free to reach out for questions, collaboration, or feedback!

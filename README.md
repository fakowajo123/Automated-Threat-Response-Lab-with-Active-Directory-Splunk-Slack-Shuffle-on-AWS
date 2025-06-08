# Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS

## Introduction

Security teams often struggle with slow response times to compromised devices in enterprise environments due to manual alerting and remediation processes. To address this, I designed and deployed a[...]

The system detects suspicious activity on a managed endpoint, triggers a notification workflow requiring human confirmation, and then automatically isolates the compromised device to prevent furthe[...]

This project highlights my skills in administering directory services, configuring secure cloud network communications, ingesting and analyzing security telemetry, and building automated workflows [...]

---

## Table of Contents

- [Setup and Prerequisites](#setup-and-prerequisites)
- [Architecture](#architecture)
- [Firewall Setup](#firewall-setup)
- [🖥️ Active Directory Setup](#active-directory-setup)
  - [Domain Controller Configuration](#domain-controller-configuration)
  - [DNS Roles & GPO Policies](#dns-roles--gpo-policies)
  - [Joining Test Machine to Domain](#joining-test-machine-to-domain)
- [📊 Splunk Installation & Configuration](#splunk-installation--configuration)
  - [EC2 Setup for Splunk](#ec2-setup-for-splunk)
  - [Universal Forwarder on Test Client](#universal-forwarder-on-test-client)
  - [Inputs for AD Logs & Windows Events](#inputs-for-ad-logs--windows-events)
  - [Alert Creation](#alert-creation)
- [🔁 SOAR Integration with Shuffle](#soar-integration-with-shuffle)
  - [Setup of Shuffle Playbooks](#setup-of-shuffle-playbooks)
  - [Slack Notification Configuration](#slack-notification-configuration)
  - [Email-Based Confirmation to Trigger Isolation](#email-based-confirmation-to-trigger-isolation)
- [🔐 Automated Response Workflow](#automated-response-workflow)
  - [Detection > Slack Alert > Email Action > EC2 Isolation](#detection--slack-alert--email-action--ec2-isolation)
  - [Use of AWS CLI/API or Systems Manager for response](#use-of-aws-cliapior-systems-manager-for-response)
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

# Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS

## Introduction

Security teams often struggle with slow response times to compromised devices in enterprise environments due to manual alerting and remediation processes. To address this, I designed and deployed a[...]

The system detects suspicious activity on a managed endpoint, triggers a notification workflow requiring human confirmation, and then automatically isolates the compromised device or user account t[...]

This project highlights my skills in administering directory services, configuring secure cloud network communications, ingesting and analyzing security telemetry, and building automated workflows [...]

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
- **Shuffle SOAR** for automating incident response and integrations (Shuffle SOAR is deployed as a managed service accessible over the internet, so no AWS EC2 instance is required for SOAR in thi[...]
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

---

## 🖥️ Active Directory Setup

### Domain Controller Configuration

- Deploy Windows Server EC2 (`t3.xlarge` or higher: 4 vCPUs, 16GB RAM).
- Install "Active Directory Domain Services" role.
- Promote server to Domain Controller (e.g., domain: `lab.local`). 
![Active Directory Installation Screenshot](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/Active%20Directory%20installation.jpg)

**For step-by-step instructions to install Active Directory, follow this guide:**  
[How to Install Active Directory Domain Services on Windows Server](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-)

- After installing Active Directory, restart the PC.
- Next, open "Active Directory Users and Computers" and proceed to create the user account as shown below.



### User Creation in AD & Machine Join

- In Active Directory, create a user account from the AD Users and Computers console. Make this user a member of the "Domain Admins" group so that it has administrative privileges.
![User Creation Screenshot](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/User%20Creation.jpg)
- When deploying the Windows Test Client, join it to the `lab.local` domain using this user account.
- Log into the Test Client with the domain user credentials (now with admin rights) so you can install required software and perform administrative tasks on the test machine.

**Automation Account Creation:**  
- For the automation workflows (such as those orchestrated by Shuffle), you will need to create a separate domain account in Active Directory that will be used specifically for authentication by automation tools.  
- This account can be named anything you choose (e.g., `automation_user`), but ensure you document its credentials securely as they will be required for automated actions such as disabling user accounts via SOAR integrations.

### Joining Test Machine to Domain

- Deploy a Windows EC2 as the test client (`t3.large` or higher: 2 vCPUs, 8GB RAM).
- Set DNS to the DC's private IP.
- Join to `lab.local` domain using the domain admin user you created.
- Test authentication and GPO application.
![User Adding to Domain Screenshot](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/User%20Adding%20.jpg)

- Once you have successfully added the user to the PC and joined it to the domain, you should see confirmation like the screenshot below:

![Account Joined to Domain Proof](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/account%20in%20.jpg)

---

## 📊 Splunk Installation & Configuration

### EC2 Setup for Splunk

- Launch a separate EC2 (Ubuntu Server, `t3.xlarge` or higher: 4 vCPUs, 16GB RAM).
- To install Splunk on Ubuntu, follow the official guide: [Splunk Install Guide for Ubuntu](https://docs.splunk.com/Documentation/Splunk/latest/Installation/InstallonLinux)
- During installation, you will be prompted to create a Splunk username and password.
- Once Splunk is installed and started, you can access the Splunk web interface by navigating to `http://your-ec2-ip:8000` in your browser.
- The login page should look like this:
  
  ![Splunk GUI Screenshot](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/splunk%20GUI.jpg)

- Create a new index in Splunk that will be used for log ingestion from the forwarders.
- For further Splunk usage, users should refer to Splunk documentation and tutorials to learn about configuring searches, alerts, dashboards, and advanced features.

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

#### Splunk Forwarding & Receiving Setup

- In the Splunk GUI, navigate to **Settings > Forwarding and receiving**.
- Under **Receive data**, click **Add new** and set the listening port to `9997` (this is the default and recommended port for Splunk indexers to receive data from forwarders).
- Ensure this port is open in your security group/firewall for the relevant sources.

- On the Domain Controller and Test Client, configure the Universal Forwarder to send data to the Splunk server's private IP on port `9997`.

#### Windows Add-on for Splunk

- To properly parse and enrich Windows event logs in Splunk, you need to install the Windows Add-on for Splunk.
- In the Splunk GUI, navigate to **Apps > Find More Apps** and search for "Windows Add-on".
- Install the Windows Add-on for Microsoft Windows as shown below:

  ![Windows Add-on for Splunk Screenshot](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/windows%20add%20on.jpg)

#### Testing Telemetry

- To confirm that telemetry is coming through, create a Splunk search rule that captures unauthorized successful logins.
- Sort the results by `IP address`, `time`, `computer name`, `source`, `user`, and `count`.
- This search rule can be used as the basis for an alert in Splunk which will trigger actions via the Shuffle webhook integration.

Example SPL (Search Processing Language) rule:
```
index=<your-index-name> EventCode=4624 LogonType=3 NOT (Account_Name="ANONYMOUS LOGON")
| stats count by src_ip, _time, ComputerName, source, Account_Name
| sort by count desc
```
- Adjust the field names as per your actual log fields and requirements.
- Use this rule to create a Splunk alert and configure a webhook action to integrate with Shuffle for automated response.

- Once you have telemetry successfully coming from both the domain controller and test PC, you should see data like the example below:

![Telemetry from Both Hosts](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/Telemetry%20from%20both%20hosts%20.jpg)

The screenshot below shows a simulated alert in Splunk for an unauthorized successful login, which serves as a trigger for the automated incident response in this lab:

![Alert From Splunk](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/Alert%20From%20splunk.jpg)

---

## 🔁 SOAR Integration with Shuffle & Slack

### Setup of Shuffle Playbooks

- In Splunk, under the alert rule’s webhook action, configure Shuffle as the webhook receiver.
- Shuffle receives the alert and parses the information about what triggered the alert (user, machine, event, etc.).
- This information is then sent from Shuffle to Slack via Shuffle’s Slack integration.
- Ensure Slack is added as an application to Shuffle and authenticated.

### Automated Response Workflow & Analyst Interaction

The automation workflow proceeds as follows:

1. **Automation Workflow Layout**

   ![Automation Layout](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/Automation%20Layout.jpg)

2. **Email Notification (Optional)**

   Email notifications can also be configured for critical alerts and workflow steps. In this workflow, the email notification is sent immediately after the alert is triggered and right before analyst input. This means the analyst or designated recipients are notified by email (or other channels) that an action is required, before any analyst input is provided.

   ![Email Notification Screenshot](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/Email%20Notification.jpg)

3. **Analyst Input to Disable Account or Not**

   After the email is sent, the analyst receives an actionable message (usually in Slack, optionally via email) with options to disable the account or not.

   ![Analyst Option to Disable Account](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/click%20option.jpg)

4. **Account Disabled by Active Directory Automation**

   If the analyst chooses to disable the account, the automation proceeds and disables the user in Active Directory.

5. **Slack Alert Notification**

   The analyst and/or team receives a Slack alert confirming the action taken.

   ![Slack Alert Screenshot](https://github.com/fakowajo123/Automated-Threat-Response-Lab-with-Active-Directory-Splunk-Slack-Shuffle-on-AWS/blob/main/Screenshots/Slack%20Alert.jpg)

## 🔐 Automated Response Workflow

### Detection > Slack Alert > User Account Disable via AD

- End-to-end flow:
  1. Suspicious event logged and detected.
  2. Splunk triggers alert, sends to Shuffle.
  3. Shuffle notifies Slack and/or email.
  4. Analyst reviews and approves in Slack/email.
  5. Shuffle uses its AD integration to disable the affected user account via the domain controller.
  6. Shuffle confirms the user's account is disabled, then notifies Slack of the result.

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
- [Splunk Install Guide for Ubuntu](https://docs.splunk.com/Documentation/Splunk/latest/Installation/InstallonLinux)
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

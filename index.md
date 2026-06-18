---
layout: default
---

[Back to main page](https://ornaka.github.io)

### Overview

This project focuses on building a small cybersecurity homelab to develop and strengthen hands-on skills in networking, administration and cybersecurity. The main focus areas include Active Directory design and management, network configuration, and security monitoring using Splunk and Sysmon. The goal of the lab was to simulate a small enterprise environment where identity management, endpoint security, and log collection are centrally managed and observable.

### Used software and hardware specifications

*   Windows 10 desktop with 2TB SSD/64GB RAM as the host machine
*   GNS3 & Oracle Virtualbox for virtualization
*   Windows 2022 Server acting as Domain Controller with Active Directory, DNS and DHCP
*   2 x Windows 11 clients
*   Ubuntu 26.04 running Splunk enterprise
*   Splunk Universal Forwarders and Sysmon for endpoint telemetry
*   pfSense firewall for network routing and internet access

### Setting up core network services and Active Directory

The lab was built around the 192.168.3.0/24 subnet. A Windows Server 2022 domain controller was configured with a static IP address (192.168.3.20), while Windows 11 clients received network configuration dynamically via DHCP. After deploying the domain controller both client machines were joined to the domain to validate DNS resolution, DHCP leasing, and Active Directory functionality. 

![Setting up DC and hosts](/images/Topology.png)

Connectivity was verified by testing domain logins and confirming that client machines could communicate with the domain controller and vice versa.

![Pinging DC](/images/pings.jpg)

Verifying clients have joined the domain

![Hosts](/images/computers.png)

Verifying that DHCP leases are working as expected

![DHCP-leashes](/images/leases.png)

To structure the environment properly, an Organizational Unit (OU) was created along with two standard user accounts and a domain administrator account for later administrative tasks, such as installing Splunk Forwarders and Sysmon on clients. A shared network folder was also configured for the OU to simulate basic file sharing in a domain environment. 

Testing access to the shared folder with a regular user part of an OU. Domain admin, who was not part of the OU was not able to access the file. 

![Testing access to the shared foleder on DC1](/images/testing_access.png)

Group Policy Objects (GPO's) were impletemented to introduce basic endpoint restrictions. Command prompt and software installation were restricted from regular users to simulate basic endpoint hardening.

Testing confirmed restrictions applied correctly. Admin account was still able to use CMD on client machines.

![Testing GPO](/images/disabling_cmd_gpo.png)

### Setting up pfSense, Splunk and security monitoring.

After stabilizing the Active Directory environment, pfSense was introduced to provide internet connectivity for the internal network. This was required to install security tools such as Sysmon and Splunk forwarders. An Ubuntu 26.04 server was deployed to host Splunk Enterprise and assigned a static IP address (192.168.3.30) via DHCP reservation to ensure consistency for forwarder configuration. The server was also joined to the domain for centralized management.

![Lab after Ubuntu & FW](/images/topology2.png)

During deployment, several configuration issues were encountered. Initially, insufficient virtual disk allocation prevented Splunk from installing correctly. Additionally, Splunk was first installed as the root user, which caused permission issues later when attempting to manage files from a non-privileged domain admin account. These issues were resolved by reinstalling Splunk with proper storage allocation and correct user permissions from the start. Official Splunk documentation proofed to be extremely helpful during troubleshooting and configuration.

After successful installation Splunk was confirmed to be running

![Splunk](/images/Splunk_works.png)

Sysmon was installed on the domain controller and Windows 11 clients to generate detailed security telemetry. A configuration file based on [SwiftOnSecurity](https://github.com/swiftonsecurity/sysmon-config) Sysmon template was used to standardize logging across endpoints. Splunk Universal Forwarders were then deployed to each machine to forward logs to the Splunk instance. Initially, only internal Splunk logs were visible, and expected Windows security events were missing. After some research, it was determined that a Splunk Sysmon add-on was required to properly parse and ingest Sysmon telemetry to Splunk.

Using Splunk Deployment Server, the add-on was distributed across all forwarders, after which security event logs became available in Splunk.

![Sysmon_add-on](/images/UF_agent.png)

To validate the logging, multiple failed login attempts were generated on an endpoint. These events were successfully ingested into Splunk and identified as Event ID 4625 (Failed Logon), confirming that security telemetry collection and indexing were functioning correctly.

![Event4625](/images/event4625.png)

To further improve visibility into authentication-related events, a Splunk alert was configured to monitor failed login attempts within the domain environment. For demonstration and validation purposes, the alert threshold was intentionally set to a low value to ensure that alert generation could be tested reliably. Domain controller also had a GPO which locks the account after five failed login attempts. The action set for this alert was simply triggering an alert just for testing purposes. 

![alert](/images/alert.png)

Succesful alert generation was verified through the Triggered Alerts dashboard. This confirmed that the alert was functioning as expected. 

![Alert dashboard](/images/triggered_alerts.png)

To complement the alerting functionality, a basic security monitoring dashboard was created in Splunk to provide centralized visibility into authentication activity across the lab environment. The dashboard includes visualizations for failed login attempts by host, failed login attempts by user account, and successful logins by host. These views make it easier to identify unusual authentication patterns and quickly determine which systems or accounts generate the most security-relevant events.

![Dashboard](/images/dashboard.png)

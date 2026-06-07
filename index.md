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

Verifying that DHCP leases working as expected

![DHCP-leashes](/images/leases.png)

I also created two regular users and created organizational unit and a shared folder for the said unit on the DC. Since Group Policy Objects are big part of AD's security features I created and tested couple which were tied to the regular users and their OU. One domain-admin was created for administrative tasks that would come later when installing and configuring Splunk forwarders on hosts. 

Testing access to the shared folder on DC with a regular user

![Testing access to the shared foleder on DC1](/images/testing_access.png)

First GPO I implemented was disabling access to CMD for regular users, since there shouldn't be any need for it in a regular office environment. In the next picture ran a test of GPO, which worked. After testing that I logged on to the domain admin account and tested that CMD was working fine for that user. Another GPO I implemented was disabling installation of software from regular users. This was tested when installing forwarders on the host machines. 

![Testing GPO](/images/disabling_cmd_gpo.png)

### Phase 2

After getting the AD environment to a point where I was happy was time to implement pfSense FW and Splunk server to the lab. I decided to run Splunk on an Ubuntu 26.04 machine, which got a static IP address of 192.168.3.30 from DHCP server since I suspected that having a leashed DHCP address would mess up with configuration of Splunk Forwarders on the host machines. The server was also joined into the domain and implementing the changes topology looked like the following image. All of the machines also got an internet connection through the FW which was crucial for downloading Sysmon for security logs and Splunk Forwarders to feed data into the Splunk. 

![Lab after Ubuntu & FW](/images/topology2.png)

At first I ran into a trouble while installing Splunk. At first while installing Splunk I ran into some troubles. The first one being I gave the image too low VDI, which resulted in not being able to download Splunk Enterprise completely. The first installation I did was also done as root, which resulted my domain-admin (who wasn't in the sudoers file at the time) not being able to modify the Splunk files I wanted. Eventually I just reinstalled Splunk properly and all started running as supposed. Splunk had really good documentation about installations and configurations, which I referred to while doing the installations. 

Splunk running on the server

![Splunk](/images/Splunk_works.png)

After the installation of the Splunk, it was time to install forwarders on DC and host machines for log ingestion. I downloaded Sysmon to get Windows security logs on each host and installed forwarders to the machines. For Sysmon configuration file I used ![SwiftOnSecuritys](https://github.com/swiftonsecurity/sysmon-config) config file which I had previously stumbled upon while watching Sysmon-related videos on YouTube. This was pretty straight forward process and I got the host logs going to the Splunk Enterprise. 

At this point I ran into trouble. While Splunk was getting data from the hosts, all of them were internal logs from Splunk itself, not the security logs I wanted. After some research I found out that I needed to install add-on for Sysmon-logs to flow into the server. I utilized Splunks deployment server-feature to do this and after a while I started seeing security-related logs in Splunk. 

Deployment of Splunk-Sysmon add-on
![Sysmon_add-on](/images/UF_agent.png)

As a test I failed login onto one of the host couple times and queried it in Splunk. Event 4625, which stands for failed login was found.

Security events coming in

![Event4625](/images/event4625.png)


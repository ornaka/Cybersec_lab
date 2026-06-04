---
layout: default
---

[Back to main page](https://ornaka.github.io)

# Creating a small homelab environment

This project focuses on building a small cybersecurity homelab to develop and strengthen my practical, hands-on skills. The main focus areas include network configuration and fundamentals, designing and managing an Active Directory environment, and integrating security monitoring and analysis tools such as Splunk within the domain.

### Used software and hardware specifications

*   Desktop running Windows 10 as OS, 2TB SSD/64GB RAM
*   GNS3 & Oracle Virtualbox for virtualization
*   Windows 2022 Server hosting DNS, DHCP and Active Directory
*   2 x Windows 11 hosts
*   Ubuntu 26.04 acting as server for SIEM
*   SPLUNK
*   pfSense

### Approach 

The project was divided into phases. First phase was getting Windows server with DNS, DHCP and Active Directory up and running and connecting two hosts to the server to ensure that the services running on it are running properly. In the second phase, I added a pfSense firewall to get internet connection to the hosts in the network, added Ubuntu to act as a server for Splunk and configured Splunk forwarders on the Windows hosts to feed data to Splunk. 

### Phase 1 

The network I decided to use for the machines was 192.168.3.0/24. Domain Controller got a static IP address 192.168.3.20 and the two W11 hosts would get their IP's as a DHCP leash. After setting the machines up into the GNS3 the network looked as follows: 

![Setting up DC and hosts](/images/Topology.png)

Set up the domain controller first and after that joined both hosts to the network to test interconnectivity and local domain working properly. 

Testing connectivity to DC from Host 1 on a local user

![Pinging DC](/images/pings.jpg)

Hosts in Domain Controllers AD's users and computers

![Hosts](/images/computers.png)

DHCP leases working as expected

![DHCP-leashes](/images/leases.png)

I also created two regular users and created organizational unit and a shared folder for the said unit on the DC. Since Group Policy Objects are big part of AD's security features I created and tested couple which were tied to the regular users and their OU. One domain-admin was created for administrative tasks that would come later when installing and configuring Splunk forwarders on hosts. 

Testing access to the shared folder on DC with a regular user

![Testing access to the shared foleder on DC1](/images/testing_access)

First GPO I implemented was disabling access to CMD for regular users, since there shouldn't be any need for it in a regular office environment. In the next picture ran a test of GPO, which worked. After testing that I logged on to the domain admin account and tested that CMD was working fine for that user. Another GPO I implemented was disabling installation of software from regular users. This was tested when installing forwarders on the host machines. 

![Testing GPO](/images/disabling_cmd_gpo.png)



### Header 3

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

### Large image

![Branching](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```

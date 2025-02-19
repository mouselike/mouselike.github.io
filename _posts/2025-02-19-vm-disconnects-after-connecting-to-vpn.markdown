---
layout: post
title:  "VM disconnects after connecting to VPN"
categories: power automate
---


## Issue
I was connected to a VM via RDP (Remote Desktop using the Windows app on my Mac). On the VM, I needed to establish a VPN connection to access another machine running in the local network. However, as soon as the VPN connection was established (from the VM to the local network), I lost access to the VM. I was unable to reconnect from my Mac and had to restart the VM before I could regain access.


## Solution
I resolved the issue of losing connection to the VM by disabling the “Use default gateway on remote network” option in the IPv4 properties of the VPN network adapter settings. This prevented the VPN from overriding the VM’s default route, allowing me to maintain my RDP session while still accessing the remote network.

After resolving the VM connection issue, I encountered another problem: I couldn’t connect to the machine running on the local network. The issue was that my VM was on a different subnet. I discovered this by running the following command in PowerShell:

```shell 
$ ipconfig /all
```

I got the following result which revealed that my VPN connection assigned an IP address in a different subnet, preventing direct communication with the target machine.
PPP adapter AKI-VPN:

```shell
   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : VPN
   Physical Address. . . . . . . . . :
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::809a:2fff:b549:7fe2%55(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.3.6(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.255
   Default Gateway . . . . . . . . . :
   DHCPv6 IAID . . . . . . . . . . . : 898501757
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-2E-B1-1D-C1-60-45-BD-9F-BB-6B
   DNS Servers . . . . . . . . . . . : 172.20.0.10
                                       8.8.8.8
   NetBIOS over Tcpip. . . . . . . . : Enabled
```


To resolve the subnet issue and enable communication with the machine on the local network, I manually created a route for the required IP address using the following command in PowerShell:

```shell
$ route add 172.20.0.10 mask 255.255.255.255 192.168.3.6 -p
```

This ensured that traffic destined for 172.20.0.10 was correctly routed through the VPN connection while maintaining my RDP session.
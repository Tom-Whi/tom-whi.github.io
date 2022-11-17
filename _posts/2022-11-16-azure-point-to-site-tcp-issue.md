---
title: Azure Point to Site tcp issues
author: TomWhi
date: 2022-11-16
categories: [Azure]
tags: [azure,troubleshooting]
---

## Overview 

I was approached with a weird issue where a client on the Azure Point-to-Site (P2S) VPN, connecting to an Azure Virtual Gateway, wouldn't open SMB file shares from the on-prem network. 

Rough topology: 

- Azure Virtual Gateway in UK South
  - Site to Site configured with On-prem firewall 
  - Point to Site configured to allow clients to connect using Azure AD

- PC1 with Azure VPN Client installed & all services working fine 
- PC2 with Azure VPN Client installed & some services are not working 

Noticed behavior: 

- PC1 could connect to the Azure P2S
- PC1 could ping the SMB file server on-prem
- PC1 could connect to the SMB shares on-prem in Explorer 

- PC2 could connect to the Azure P2S
- PC1 could ping the SMB file server on-prem
- PC1 could NOT connect to the SMB shares on-prem in Explorer 

## Troubleshooting

### On-Prem firewall 

We started taking Wireshark captures on the On-Prem firewall thinking it was a firewall issue. 

On the on-prem firewall we could see the traffic coming in for:

- PC1 - DNS, ICMP and SMB traffic, and confirmed that PC was working normally
- PC2 - only DNS and ICMP traffic - no SMB traffic! 

At this point we knew the client was connecting successfully to devices on-prem because DNS and ICMP were working (DNS was resolving and pings were being returned).  However SMB traffic still wasn't working for PC2. 

I found the following packets to be interesting: 

- 57 ICMP / ping request from PC2 to on-prem SMB server 
- 60 ICMP reply from on-prem SMB server to PC2
- 76 DNS request for A record for smbserver.domain.local
- 84 DNS reply for A record for smbserver.domain.local

What's missing - SMB traffic. 

![cloudshell](/assets/images/2022-11-16_onprem-firewall.png){: .shadow }
_On-Prem Firewall Wireshark_


We also took traces on the Azure Virtual Network Gateway - however found the information to be the same, we weren't seeing SMB traffic from PC2 but were seeing other types of traffic being allowed.  


At this point we were sure there was an issue with the client not sending the data:

- We checked the Windows firewall to ensure that SMB traffic was allowed out (and found nothing)
- We checked for any other firewalls on the client (and found nothing)

## Penny drops 

We Wiresharked the PC2 (the not working PC) to see what was happening, we traced the Azure VPN interface first (not the LAN port) - and the trace we took looked pretty much like the one on the on-prem firewall, DNS and ICMP - no SMB traffic.  

We ran another trace on PC2 but this time on the LAN port - we saw all the SMB requests going out of the LAN port, not down the Azure P2S VPN port. 

We then realise that it's only TCP traffic that's being dropped, ICMP (not TCP) and DNS (not TCP) were allowed out and was getting to the intended target.  So we questioned the knower of all.  Google. 

## Resolution 

We got one hit back <https://techcommunity.microsoft.com/t5/windows-10/split-tunnel-vpn-does-not-route-tcp-traffic-with-release-21h2/m-p/3486104>

The last comment gave us the answer. 

"The problem was caused by the Dell Optimiser SW that came pre-installed on out laptops, but it only affected users that did NOT have local administrator rights. Uninstalling the SW fixed the issue for all our endpoints."

The reason it worked on PC1 and not PC2 was because PC1 didn't have the DELL Optimiser software installed.  We removed that and after a reboot the client was able to connect to the file share via the Azure VPN! 

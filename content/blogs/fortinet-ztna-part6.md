---
title: "Fortinet ZTNA Part VI"
date: 2025-03-10T04:03:33Z
draft: false
github_link: "https://github.com/abed1985"
author: "Abood Alakhras"
tags:
  - security
  - fortinet
  - zerotrust
image: /images/fortinet/ztnap.png
description: "ZTNA Destinations|Tags|Rules"
toc:
---
In this sixth installment of our Fortinet ZTNA series, we delve into the configuration of ZTNA destinations in FortiEMS. These destinations play a crucial role in enabling secure access to internal resources through the access proxy. This guide provides a step-by-step overview of setting up ZTNA destinations, including key configuration elements and best practices.

## ZTNA Destinations

We have a ZTNA Destination profile named "ENDPOINTPOLICY-OFF-NETWORK" created under Endpoint Profiles > ZTNA Destinations.

Before connecting, ZTNA destinations to the access proxy need to be created. ZTNA destinations are a part of the Endpoint Profiles that need to be assigned to the endpoint policy.
Refer to the following snapshots for the POC (Creating ZTNA Destinations): 

We have 4 ZTNA Destinations configured in our Proof of Concept (POC):

- Destination Host: Refers to the real internal IP address/FQDN and port of the server. RDP & SSH connections are securely proxied through the gateway.
- Proxy Gateway: Refers to the FortiGate device access IP address and port number (usually WAN).
- Mode: Always set to Transparent.

![My Image](/images/fortinet/ztna_dest1.PNG)
![My Image](/images/fortinet/ztna_dest2.PNG)
![My Image](/images/fortinet/ztna_dest4.PNG)



### ZTNA TCP Forwarding Access Proxy:

- For non-web applications like SSH, you can configure ZTNA TCP forwarding access proxy without encryption. The connection still begins with a TLS handshake.
- The client uses an HTTP 101 response to switch protocols and remove the HTTPS stack.
- End-to-end communication between the client and server is encapsulated in the specified TCP port but not encrypted by the access proxy.
- This improves performance by reducing encryption overhead when the underlying protocol is already secure (e.g., SSH or HTTPS).
- For insecure protocols like Telnet or RDP, encryption must still be enabled to ensure security.
- SAML Authentication: The feature in the destination rule allows using an external web authentication for SAML for certain applications.

![My Image](/images/fortinet/ztna_dest3.PNG)
![My Image](/images/fortinet/ztna_dest5.PNG)

### Assigning the ZTNA Destination Profile

Once configured, the ZTNA Destination profile needs to be assigned to the endpoint policy to take effect.

![My Image](/images/fortinet/ztna_dest6.PNG)


## Conclusion

With ZTNA destinations configured and assigned to the endpoint policy, secure access to internal resources is now established. 
In the next section, we will explore ZTNA tags and tagging rules, which further refine and control access based on user and device attributes.
---
title: "Fortinet ZTNA Part I"
date: 2025-03-06T00:58:01Z
draft: false
github_link: "https://github.com/abed1985"
author: "Abood Alakhras"
tags:
  - security
  - fortinet
  - zerotrust
image: /images/fortinet/ztnap.png
description: ""
toc:
---

## Solution Overview

<p align="middle">
  <img src="/images/fortinet/ztnap11.png" alt="My Image">
</p>
<br>

### Definition & Overview

Zero Trust Network Access (ZTNA) is an advanced access control method that leverages client device identification, authentication, and Zero Trust tags to provide role-based access to applications. ZTNA enables administrators to manage network access for both on-fabric local users and off-fabric remote users. Access is granted only after a series of checks: device verification, user identity authentication, user authorization, and context-based posture checks, all using Zero Trust tags.

In traditional networks, users and devices have distinct sets of rules for on-fabric (local) access and off-fabric (remote) access, often through VPNs. With the rise of distributed workforces and the need to access resources across diverse environments such as company networks, data centers, and the cloud, managing these access rules becomes increasingly complex. Additionally, user experience can suffer when multiple VPNs are required to access different resources.

Traditionally, a user and device have different sets of rules for on-fabric access and off-fabric VPN access to company resources. With a distributed workforce, and access that spans company networks, data centers, and the cloud, ,managing the rules can be complex. User experience is also affected when an organization needs multiple VPNs to access various resources. 

#### ZTNA operates in two primary modes:

1. Full ZTNA: Allows users to securely access resources through an SSL-encrypted access proxy, removing the need for traditional VPNs and simplifying remote access.
2. IP/MAC Filtering: Combines ZTNA tags with IP/MAC-based filtering to enhance device identification, providing an additional security layer for role-based, Zero Trust access.

### Key Components & Workflow
 
The solution has the following components:

#### FortiClient EMS (Endpoint Management System):

- Issues and signs client certificates containing the FortiClient UID, certificate serial number, and EMS serial number.
- Synchronizes client certificates with FortiGate for secure communication.
- Shares its EMS ZTNA Certificate Authority (CA) certificate with FortiGate.
- Uses tagging rules to apply ZTNA tags to endpoints.
- Shares ZTNA tag information with FortiGate via the Security Fabric integration.

#### FortiClient (Endpoint Agent):

- Communicates directly with FortiClient EMS to continuously provide device status updates via ZTNA telemetry.
- Supplies detailed endpoint information such as device type, logged-on users, and security posture (including on-fabric or off-fabric status, antivirus software, vulnerability status, etc.).
- Obtains a client certificate from the FortiClient EMS ZTNA Certificate Authority (CA) on the first connection attempt to the access proxy.
- Uses the certificate to authenticate the device with FortiGate, which uses the shared ZTNA CA to verify the client.

#### FortiGate:

- Maintains a continuous connection to FortiClient EMS, synchronizing endpoint information such as FortiClient UID, client certificate serial number, EMS serial number, and network details (IP and MAC addresses).
- Acts as an SSL-encrypted access proxy, securing remote access to applications.
- Updates device information as changes occur, with FortiClient EMS notifying FortiGate.
- Uses ZTNA tags to enforce access control rules, ensuring that incoming traffic through the ZTNA access proxy complies with the Zero Trust framework. FortiGate's WAD daemon processes this information during ZTNA traffic handling. 

---
*Some content and images in this post are sourced from [Fortinet](https://www.fortinet.com).  
All rights belong to their respective owners.*
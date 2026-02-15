---
title: "Fortinet ZTNA Part V"
date: 2025-03-09T02:18:29Z
draft: false
github_link: "https://github.com/abed1985"
author: "Abood Alakhras"
tags:
  - security
  - fortinet
  - zerotrust
image: /images/fortinet/ztnap.png
description: "ZTNA Certificate Management"
toc:
---

## Introduction

In this fifth installment of our Fortinet ZTNA series, we explore ZTNA certificate management and its role in securing client authentication and access control.

## ZTNA Certificate Management

FortiClient EMS plays a central role in managing certificates for ZTNA authentication. Below are key points regarding ZTNA CA and certificate handling:

- FortiClient EMS has a default root CA certificate (default_ZTNARootCA), which is assigned to the default customer site. Each customer site, such as CUSTOMER-TEST, has a unique root CA.
- The ZTNA CA uses this root certificate to sign CSRs from FortiClient endpoints.
- Root CA updates and revocations automatically propagate to FortiGate and FortiClient endpoints, enforcing certificate renewal.
- FortiClient EMS manages individual client certificates, ensuring each endpoint is securely authenticated.
- Revoking a client certificate is necessary when private keys are compromised. This can be done in FortiClient EMS:
  - Navigate to Endpoint > All Endpoints.
  - Select the client and click Action > Revoke Client Certificate.
- Important: Do not confuse the FortiClient EMS ZTNA CA certificate with the SSL certificate:
  - The ZTNA CA certificate is used for client authentication.
  - The SSL certificate is used for FortiClient EMS HTTPS access and FortiGate fabric connectivity.

![My Image](/images/fortinet/ztna_cm1.png)

The certificate issued by EMS looks like the following at the endpoint certificate store.

![My Image](/images/fortinet/ztna_cm2.png)
![My Image](/images/fortinet/ztna_cm3.png)

## Certificate Storage and Verification
- Windows Endpoints: FortiClient automatically installs the client certificate into the Windows certificate store.
- Certificate Attributes:
  - The certificate UID and serial number (SN) should match the records on FortiClient EMS and FortiGate.
  - FortiGate validates these attributes before granting access.
- Verification on FortiGate:
  - Use the following CLI command to list endpoint records:
```python
diagnose endpoint record list
```
  - This command displays information such as client UID, certificate SN, and EMS certificate SN.
  - If a mismatch or missing data occurs, client authentication may fail due to the inability to locate the corresponding endpoint entry.

![My Image](/images/fortinet/ztna_cm4.png)

## SSL Certificate-Based Authentication

SSL certificates play a crucial role in client authentication. Below is the typical workflow:

1. Client Registration:
    - When an endpoint registers with FortiClient EMS, it obtains a client certificate.
2. Certificate Signing Process:
    - FortiClient automatically submits a CSR (Certificate Signing Request).
    - FortiClient EMS signs the CSR and returns the client certificate.
3. Certificate Storage:
    - The certificate is stored in the OS certificate store for secure authentication.
4. FortiGate and EMS Synchronization:
    - Endpoint information (UID, SN) is synchronized between FortiGate and FortiClient EMS.
5. Client Disconnects or Unregisters:
    - The certificate is removed from the OS store.
    - The certificate is revoked on FortiClient EMS.
6. Client Reconnects:
    - A new certificate is obtained when the client re-registers.

## Client Authentication Process
By default, client certificate authentication is enabled on the FortiGate access proxy. The authentication process works as follows:
- When FortiGate receives an HTTPS request, the WAD process challenges the client to present its certificate.
- FortiGate validates the certificate using the following logic:
  1. If the client presents a valid certificate:
      - FortiGate extracts the client UID and certificate SN.
      - If these match an entry in FortiGate’s endpoint record, the client proceeds with ZTNA proxy rule processing.
      - If they do not match, access is denied.
  2. If the client cancels authentication or submits an empty certificate:
      - FortiGate processes the request based on the empty-cert-action setting:
        - If empty-cert-action is set to accept, the client is allowed to continue ZTNA proxy rule processing.
        - If empty-cert-action is set to block, the client is denied access.

## Conclusion

ZTNA certificate management is fundamental to ensuring secure, certificate-based authentication in a Zero Trust environment. Proper handling of FortiClient EMS root CA, client certificates, and authentication workflows enhances security and prevents unauthorized access.

By implementing certificate-based ZTNA authentication, organizations can:

- Automate client certificate management through FortiClient EMS.
- Enhance endpoint security with seamless revocation and renewal processes.
- Ensure accurate authentication records between FortiGate and FortiClient EMS.
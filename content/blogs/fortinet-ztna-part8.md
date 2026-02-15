---
title: "Fortinet ZTNA Part VIII"
date: 2025-03-18T23:40:02Z
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

## Finalizing Fortinet ZTNA: Rules, Verification, and Testing

In this final installment of the Fortinet ZTNA series, we will explore FortiGate ZTNA rules, conduct final verifications, and test the proof of concept (POC) from an end-user perspective.

### ZTNA Rules

ZTNA rules define access policies using proxy policies to enforce Zero Trust role-based access control. These policies can include ZTNA tags, which are synchronized from FortiClient EMS, and security profiles for additional protection.

### Key Points from the POC:

- SAML Authentication: When saml is set as the source, external SAML authentication is required.
- ZTNA Tags Matching Logic: Multiple ZTNA tags within a rule follow an "OR" logic, meaning a device must have at least one tag to match. This behavior can be adjusted via CLI using set ztna-tags-match-logic.
- Deep Inspection for SSH Proxy: SSH proxy terminates the session at FortiGate, creating a new session to the SSH server using a PKI certificate, enhancing security. This can be combined with SAML authentication for an extra layer of protection.

### Configuration in GUI:

![My Image](/images/fortinet/ztna_rules.PNG)


### Equivalent CLI Configuration:

```python
config firewall proxy-policy
    edit 4
        set uuid ecc725e0-1870-51ed-151e-538adec6e364
        set name "ZTNA_RULE_TEST"
        set proxy access-proxy
        set access-proxy "ABOOD-DC"
        set srcintf "WAN"
        set srcaddr "all"
        set dstaddr "all"
        set ztna-ems-tag "FCTEMS88XXXX_net-sec-ops-domain" "FCTEMS88XXXX_got-the-file"
        set action accept
        set schedule "always"
        set logtraffic all
        set utm-status enable
        set ssl-ssh-profile "certificate-inspection"
        set av-profile "default"
        set ips-sensor "default"
    next
    edit 5
        set uuid e19f5202-2746-51ed-54d1-fe116338332a
        set name "ZTNA_RULE_TEST_2"
        set proxy access-proxy
        set access-proxy "ABOOD-DC2"
        set srcintf "WAN"
        set srcaddr "all"
        set dstaddr "all"
        set ztna-ems-tag "FCTEMS88XXXX_net-sec-ops-domain" "FCTEMS88XXXX_got-the-file"
        set action accept
        set schedule "always"
        set logtraffic all
        set groups "saml"
        set utm-status enable
        set ssl-ssh-profile "certificate-inspection"
        set av-profile "default"
        set ips-sensor "default"
    next
    edit 6
        set uuid b70acbee-2e48-51ed-e02b-a3cba7347105
        set name "ZTNA_RULE_TEST_3"
        set proxy access-proxy
        set access-proxy "ABOOD-DC3"
        set srcintf "WAN"
        set srcaddr "all"
        set dstaddr "all"
        set ztna-ems-tag "FCTEMS88XXXX_net-sec-ops-domain" "FCTEMS88XXXX_got-the-file"
        set action accept
        set schedule "always"
        set logtraffic all
        set groups "saml"
        set utm-status enable
        set ssl-ssh-profile "certificate-inspection"
        set av-profile "default"
        set webfilter-profile "default"
        set ips-sensor "default"
        set application-list "default"
    next
    edit 7
        set uuid c19b52a4-28df-51ed-48cd-3d27773970f0
        set name "ZTNA_RULE_TEST-4"
        set proxy access-proxy
        set access-proxy "ABOOD-DC4"
        set srcintf "WAN"
        set srcaddr "all"
        set dstaddr "all"
        set ztna-ems-tag "FCTEMS88XXXX_net-sec-ops-domain" "FCTEMS88XXXX_got-the-file"
        set action accept
        set schedule "always"
        set logtraffic all
        set groups "saml"
        set ssl-ssh-profile "custom-deep-inspection"
    next
    edit 3
        set uuid 076cc7a0-9dac-51ec-3827-e47760967d9d
        set name "ZTNA_DENY_ALL"
        set proxy access-proxy
        set access-proxy "ABOOD-DC" "ABOOD-DC2" "ABOOD-DC4"
        set srcintf "WAN"
        set srcaddr "all"
        set dstaddr "all"
        set ztna-ems-tag "MAC_FCTEMS88XXXX_High"
        set schedule "always"
        set logtraffic all
    next
end
```

### Final Endpoint Verification

1. Verify FortiClient EMS Connection

![My Image](/images/fortinet/ztna_ver1.PNG)

2. Confirm ZTNA Destinations Learned via Endpoint Policy

![My Image](/images/fortinet/ztna_ver2.PNG)

3. Verify Correct ZTNA Tag Assignment

![My Image](/images/fortinet/ztna_ver3.PNG)

![My Image](/images/fortinet/ztna_ver4.PNG)

4. Confirm ZTNA Certificate Assignment

![My Image](/images/fortinet/ztna_ver5.PNG)

5. Validate Endpoint Information in FortiGate
```bash
diagnose endpoint record list
```
![My Image](/images/fortinet/ztna_ver6.PNG)

## Access Testing

1. Web Server Access Without SAML Authentication

![My Image](/images/fortinet/ztna_ver7.PNG)

![My Image](/images/fortinet/ztna_ver8.PNG)

2. Web Server Access with SAML Authentication

- Redirected to SAML IdP for authentication.
- Successful login with MFA.

![My Image](/images/fortinet/ztna_ver9.PNG)

- Directed to SAML IdP for authentication :  Completing login and MFA 

![My Image](/images/fortinet/ztna_ver10.PNG)
![My Image](/images/fortinet/ztna_ver11.PNG)

Landing on the web server post authentication with SAML .

![My Image](/images/fortinet/ztna_ver12.PNG)


3. Testing SSH Proxy with TCP Forwarding
- Accessing machine 192.168.2.3 via SSH.
- FortiGate handles proxying and SAML authentication.

![My Image](/images/fortinet/ztna_ver13.PNG)


## Conclusion 

This concludes the Fortinet ZTNA series. Below are the key takeaways:

- ZTNA Enhances Security: By enforcing granular, identity-based access control with minimal trust assumptions.
- Flexible Authentication: Supports external SAML authentication for seamless integration.
- Deep Inspection & Proxying: Ensures secure access to both web and non-web applications like SSH and RDP.
- ZTNA Tags & EMS Integration: Simplifies endpoint classification and access policy enforcement.
- Comprehensive Logging & Visibility: FortiGate provides full monitoring and traffic analysis for security teams.

ZTNA is a powerful addition to Fortinet’s security stack, providing robust protection while maintaining user experience. With this series, you now have a complete understanding of implementing, verifying, and testing ZTNA within your Fortinet environment.

Thank you for following along this journey! Stay secure.
---
title: "Fortinet ZTNA Part III"
date: 2025-03-07T01:45:48Z
draft: false
github_link: "https://github.com/abed1985"
author: "Abood Alakhras"
tags:
  - security
  - fortinet
  - zerotrust
image: /images/fortinet/ztnap.png
description: "SAML Authentication with MS Azure"
toc:
---
In this third installment of our Fortinet ZTNA series, we explore integrating SAML authentication with Microsoft Azure as the Identity Provider (IdP) and FortiGate as the Service Provider (SP). By leveraging SAML, authentication is offloaded from the access proxy to the IdP, allowing organizations to enforce advanced authentication policies such as Multi-Factor Authentication (MFA)

To learn more about SAML authentication, check out my by blog post: [SAML Authentication](http://net-sec-ops.com/blogs/saml-authentication/)


## Configuring a SAML Application (IdP) on Microsoft Azure


#### Step 1: Create an Enterprise Application 
 

1. Log in to the Azure Administration Console under your customer tenant.
2. Navigate to Enterprise Applications > New Application.

![My Image](/images/fortinet/ztna_saml1.PNG)
3. Choose "Create your own application", enter a name (e.g., "fortigate-saml"), and select "Integrate any other application you don't find in the gallery (Non-gallery)".
![My Image](/images/fortinet/ztna_saml2.PNG)
4. Click Create

#### Step 2: Assign Users and Groups

Once the application is created, the overview page will display key configuration steps.

1. Under Users and Groups, assign Active Directory users and groups that will use this application.
2. Ensure the required users and groups already exist under Azure Active Directory.
3. For this Proof of Concept (PoC), we created a group called ZTNA.
4. Note the Object ID of the group, as it will be used in FortiGate "User Groups" to control access through SAML attributes.

![My Image](/images/fortinet/ztna_saml3.PNG)
![My Image](/images/fortinet/ztna_saml6.PNG)


#### Step 4: Configure SAML Parameters

##### Sections Breakdown:

- Sections 1 and 2 require configuration; Sections 3 and 4 are automatically generated.
- Section 1: Identifiers and Reply URLs
    - Use the FortiGate proxy VIP as the SP URL (e.g., aboodshouse.fortiddns.com).
    - Any proxy VIP port can be used (e.g., 8443, 7443, 9443, 6443).
- Section 2: User Attributes & Claims
    - Ensure the correct attributes are mapped:
        - username → userprincipalname
        - group → user.groups (group claim)
- Important: Azure allows only one group claim by default. Delete the existing claim and create a new group claim named groups with value user.groups.
- This step is also documented by Microsoft [for using SAML with FortiGate SSL-VPN: Microsoft Docs](https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/fortigate-ssl-vpn-tutorial)

#### Step 5: Retrieve Certificate and Configuration Details

##### Section 3: Download the Certificate (Base64)—this will be imported into FortiGate.
##### Section 4: Note all the generated URLs, as they will be required when configuring SAML on FortiGate.
 
 
![My Image](/images/fortinet/ztna_saml4.PNG)


This completes the initial setup of the SAML application on Azure. In the next section, we will cover how to configure FortiGate to integrate with this SAML IdP.

## Configuring a SAML Application (SP) on Fortigate

#### Step 1: Import the SAML Certificate
1. Import the exported SAML Certificate (Base64) into FortiGate.
2. Rename the certificate in the CLI using: 
![My Image](/images/fortinet/ztna_saml7.PNG)
```python
config vpn certificate remote
set name "azure-saml"
end
```
- This serves as the Remote Certificate, containing only the IdP’s public key, which FortiGate uses to verify signed SAML responses from Azure.

#### Step 2: Configure SAML on FortiGate

1. Navigate to User & Authentication > Single Sign-on.
2. Copy and paste all URLs from Sections 1 and 4 of the Azure SAML configuration.
3. Ensure the Local Certificate option is disabled, as FortiGate does not require a private key to sign SAML requests.
![My Image](/images/fortinet/ztna_saml8.PNG)
The equivalent CLI Configuration: 
![My Image](/images/fortinet/ztna_saml9.PNG)

#### Step 2: Configure SAML on FortiGate

1. Navigate to User & Authentication > Single Sign-on.
2. Copy and paste all URLs from Sections 1 and 4 of the Azure SAML configuration.
3. Ensure the Local Certificate option is disabled, as FortiGate does not require a private key to sign SAML requests.


#### Step 3: Create a User Group

1. Create a User Group that references the SAML IdP.
2. Set the group type to Firewall.
3. Add a remote server named azure-saml (configured in Step 2).
4. Use the Object ID of the Azure ZTNA group as the group name.
![My Image](/images/fortinet/ztna_saml10.PNG)
The equivalent CLI Configuration: 
![My Image](/images/fortinet/ztna_saml11.PNG) 


#### Step 4: Configure Authentication Scheme

1. Go to Policy & Objects > Authentication > Authentication Schemes.
2. Create a scheme named azure-saml-scheme.
3. Select SAML as the authentication method.
4. Choose azure-saml as the SAML SSO server.
![My Image](/images/fortinet/ztna_saml12.PNG)
The equivalent CLI Configuration: 
![My Image](/images/fortinet/ztna_saml13.PNG)

#### Step 5: Create an Authentication Rule

1. Go to Policy & Objects > Authentication > Authentication Rules.
2. Create a rule named azure-saml-rule.
3. Set source address to All.
4. Set incoming interface to Any (or the WAN interface).
5. Set protocol to HTTP.
6. Select azure-saml-scheme as the authentication scheme.
![My Image](/images/fortinet/ztna_saml14.PNG)
The equivalent CLI Configuration: 
![My Image](/images/fortinet/ztna_saml15.PNG) 

#### Step 6: Enable Captive Portal

1. Go to User & Authentication > Authentication Settings.
2. Enable Authentication Scheme and select azure-saml-scheme.
3. Enable Captive Portal and set the FortiGate Access Proxy IP or FQDN (e.g., aboodshouse.fortiddns.com).
4. Ensure an FQDN/IP address object is created for this setting.
5. Enable HTTP Redirect and select the FortiGate SSL certificate (e.g., aboodfwcert).
![My Image](/images/fortinet/ztna_saml16.PNG) 
The equivalent CLI Configuration: 
![My Image](/images/fortinet/ztna_saml17.PNG)
6. Ensure proxy-captive-portal is enabled on the WAN interface.  
![My Image](/images/fortinet/ztna_saml18.PNG)  

 
#### Next Steps

Your SAML authentication is now set up! In the following sections, we’ll apply these settings to ZTNA policies.

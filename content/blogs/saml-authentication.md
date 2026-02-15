---
title: "Saml Authentication"
date: 2025-03-04T04:51:13Z
draft: false
github_link: "https://github.com/abed1985"
author: "Abood Alakhras"
tags:
  - security
image: /images/fortinet/saml_1.png
description: "SAML Protocol Operation"
toc:
---

## What is SAML

SAML (Security Assertion Markup Language) is an XML based (but Base64 encoded) protocol.It is a standard protocol for web Single Sign-On (SSO)  & Federated identity.
SAML allows apps and services to offload "authentication" to a trusted 3rd party. SSO intended to be easy for the user and transparent after the user has logged in the first time.

### Key terms

- Federated Identity:  Linking a user's identity across different identity management systems or domains, such as between two different organizations, like a service porvider & a customer.
- IdP - Identity Provider: Authenticating users. It is a user directory for Azure AD or G suite, FortiAuthenticator, Cisco ISE, etc...
    - Entity ID :
        - Idp and SP have an EntityID which is a globally unique name for a SAML entity. This must be a URI, typically in the form of a URL. However, It does not need to be resolvable as it is an identifier rather than a web location.
    - Single SignOn Service endpoint:
        - It must be HTTPS URL, which is where the user will be redirected to the SP with SAML authntication request (Base64 encoded XML). Then the IDP will respond by challenging  users to log in by presenting  them a login page and apply MFA if configured. This is also reffered to as the IDP login URL.
    - Single Logout Service ( SLS) endpoint:
        - The user is redirected to the IdP SLS URL to log out of the IdP when they have chosen to logout of the SP. If configered tp do so, the Idp may send a logout request directly to the other SP's that are also participating in this user's SSO session. This is also reffered to as the Idp logout URL.        
- Service Provider (SP) is an entity that offers services to users such as an access proxy or platform like  salesforce, AWS, and Egnyte. An assertion is a statement made by the Idp regarding a user's identity, which may include attributes such as username, groups, and email address. For example: the Idp has authenticated the user Abood, confirming that their email address is abood@gmail.com. Abood also belongs to the following groups: immortals, actionheros, and super_admin. the Idp asserts that Abood's first name is Abood and their surename is Alakhras. 
        

<br>

### SAML Generic SP Initiated Auth Flow
<br>

![My Image](/images/fortinet/saml_2.PNG)

Service provider (SP) that uses SAML authentication should not support IdP initiated SAML authentication due being less secure.
IdP initiated has been described as like a CSRF/XSRF (Cross Site Request Forgery) attack.

![My Image](/images/fortinet/saml_3.PNG)

---
*Some content and images in this post are sourced from [Fortinet](https://www.fortinet.com).  
All rights belong to their respective owners.*

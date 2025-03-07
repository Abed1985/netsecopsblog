---
title: "Saml Authentication"
date: 2025-03-04T04:51:13Z
draft: false
github_link: "https://github.com/abed1985"
author: "Abood Alakhras"
tags:
  - security
image: /images/fortinet/saml_1.png
description: ""
toc:
---

## What is SAML

SAML (Security Assertion Markup Language) is an XML based (but Base64 encoded) protocol.It is a standard protocol for web Single Sign-On (SSO)  & Federated identity.
SAML allows apps and services to offload "authentication" to a trusted 3rd party. SSO intended to be easy for the user and transparent after the user has logged in the first time.

### Key terms

- Federated Identity:  Linking a user's identity across different identity management systems/domains. eg. between two different organizations like a service porvider & a customer.
- IdP - Identity Provider: Authenticates the user. It is the user directory. Like Azure AD or G suite, FortiAuthenticator, Cisco ISE, or any other user directory.
    - Entity ID :
        - Both Idp and SP have an EntityID which is a globally unique name for a SAML entity.
        - Must be a URI, typically is a URL
        - Does not need to be resolvable as it is an identifier, not a web location
    - Single SignOn Service endpoint:
        - Must be HTTPS URL
        - This is the URL the user is redirected to by the SP with the SAML auth request ( Base64 encoded XML).
        - The IdP will respond by challenging the user to login ( i.e present a login page to the user, and do MFA if configured).
        - Also referred to as the IdP login URL.
    - Single Logout Service ( SLS) endpoint:
        - The user is redirected to the IdP SLS URL to log them out of the IdP when they have chosen to logout of the SP.
        - The IdP may ( if configured to do so) then send a logout request directly to other SP's that are also participants in this user's SSO session.
        - Also referred to as the IdP logout URL
- Service Provider (SP): Provides services to the user. An access proxy or anything like Salesforce, AWS, Egnyte..
- Assertion: A statement about a user's identity made by the Idp.
    - May contain attributes such as username,groups, email address.
        1. I have authenticated this user as abood
        2. abood email address is abood@gmail.com
        3. abood belongs to these groups: immortals, actionheroes, super_admin
        4. abood first name is Abood
        5. abood surname is Alakhras.

<br>

### SAML Generic SP Initiated Auth Flow
<br>

![My Image](/images/fortinet/saml_2.PNG)

Service provider (SP) that uses SAML authentication should not support IdP initiated SAML auth, due to it being less secure.
IdP initiated has been described as like a CSRF/XSRF (Cross Site Request Forgery) attack.

![My Image](/images/fortinet/saml_3.PNG)

---
*Some content and images in this post are sourced from [Fortinet](https://www.fortinet.com).  
All rights belong to their respective owners.*

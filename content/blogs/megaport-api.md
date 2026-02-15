---
title: "Megaport API"
date: 2025-03-20T07:39:51Z
draft: false
github_link: "https://github.com/abed1985"
author: "Abood Alakhras"
tags:
  - Networking
  - NetworkAutomation
image: /images/fortinet/megaport_1.png
description: "Megaport API"
toc:
---


## Introduction

In this blog post, I will discuss utilizing the Megaport API from the end customer's perspective. As you may already know, Megaport works with SDN (Software-Defined Networking) and leverages extensive automation in the backend. It provides both production and development environments to test your API queries with the provisioned circuits. You can find the API reference guide and additional details at :[Megaport API Documentation](https://docs.megaport.com/api/).

As an end client, I would like to use the API to run various calls against my circuits. In this post, I will show how I used Python to interact with the Megaport API. By the end of this post, we will discuss a potential use case for automating circuit upgrades.

### Step 1: Retrieve API Key and Secret

To get started, you first need to grab an API key and secret in order to generate an access token for subsequent API calls. 
You can generate your API key and secret in your :[Megaport account at Megaport Portal](https://portal.megaport.com/services).

Once you have your credentials, let's move on to the next steps.

#### Storing API Keys Securely

Store your API keys and secret securely in an environment variable file. Here's an example of what the .env file should look like:

```bash
cat .env
MP_STAGE_API_KEY_SECRET=someapikeysecret
MP_STAGE_API_KEY=someapikey
```

Next, I’ve written a Python function with basic error handling to fetch an access token using the API key. Megaport follows good security practices for API security, which is beyond the scope of this discussion.

The endpoint URL is https://auth-m2m.megaport.com/oauth2/token, and the Python method to get the token looks like this:

```python
import requests
import base64
import os
import json
from dotenv import load_dotenv

# Load environment variables from the .env file
load_dotenv()

def get_megaport_token():
    API_KEY = os.getenv('MP_STAGE_API_KEY')
    API_KEY_SECRET = os.getenv('MP_STAGE_API_KEY_SECRET')
    credentials = f"{API_KEY}:{API_KEY_SECRET}"
    credentials_b64 = base64.b64encode(credentials.encode("utf-8")).decode("utf-8")
    url = "https://auth-m2m.megaport.com/oauth2/token"
    headers = {
        "Authorization": f"Basic {credentials_b64}",
        "Content-Type": "application/x-www-form-urlencoded",
    }
    payload = 'grant_type=client_credentials'
    response = requests.post(url, headers=headers, data=payload)
    
    if response.status_code == 200:
        return response.json()['access_token']
    else:
        print(f"Could not get access token. API request failed with status code {response.status_code}")
        return None
```

Now let's run the method to retrieve the access token:

```python
In [9]: access_token = get_megaport_token()

In [10]: print(access_token)
someobfusicated token 
```

### Step 2: Get Active Megaport Circuits

Now that we have the access token, let's retrieve all the active Megaport circuits. 
The following Python script uses the access token to get the active Megaport product list in your account:

```python
def get_active_megaport_product_list(access_token):
    url = "https://api.megaport.com/v2/products"
    payload = {}
    headers = {
        "Authorization": f"Bearer {access_token}",
        "Content-Type": "application/json",
    }
    response = requests.get(url, headers=headers, data=payload)
    
    product_uids = []
    if response.status_code == 200:
        products = response.json()['data']
        for product in products:
            if product["provisioningStatus"] in ["LIVE", "CONFIGURED"]:
                for vxc in product["associatedVxcs"]:
                    product_uids.append(vxc["productUid"])
        return list(set(product_uids))
    else:
        print(f"Could not get active products. API request failed with status code {response.status_code}")
        return None
```

Running the method will return the list of active circuits:

```python
In [11]: active_megaport_product_list = get_active_megaport_product_list(access_token)

In [12]: from pprint import pprint
```

The output should look like this (note that all the product UIDs are replaced with dummy values):

```python
In [13]: pprint(active_megaport_product_list)
['a1vvvvvv-c9ww-43xx-8ayy-dd2b7c951d27',
 '9cvvvvvv-eaww-40xx-bbyy-61ccbe3a5023',
 '69vvvvvv-adww-4exx-81yy-acbc001d6e46',
 'f5vvvvvv-d9ww-47xx-89yy-a2b56bef950a',
 'e1vvvvvv-4aww-49xx-a6yy-b861cbc079c2',
 '45vvvvvv-4bww-4dxx-8ayy-0ba6bcedd3f4',
 '02vvvvvv-e0ww-4cxx-b3yy-bab09a4ed768',
 'cevvvvvv-fdww-4bxx-92yy-fczzzzzzzzzz',
 'bfvvvvvv-57ww-46xx-b4yy-5ezzzzzzzzzz',
 '48vvvvvv-f6ww-41xx-abyy-5ezzzzzzzzzz',
 '36vvvvvv-84ww-43xx-b5yy-ebzzzzzzzzzz',
 '83vvvvvv-3cww-4fxx-94yy-15zzzzzzzzzz',
 '61vvvvvv-5bww-44xx-afyy-b6zzzzzzzzzz',
 'a1vvvvvv-89ww-4axx-85yy-f3zzzzzzzzzz',
 '63vvvvvv-72ww-4bxx-b0yy-b7zzzzzzzzzz',
 '36vvvvvv-3fww-42xx-9cyy-bdzzzzzzzzzz',
 '41vvvvvv-b7ww-4exx-a6yy-d7zzzzzzzzzz',
 '80vvvvvv-f7ww-40xx-bdyy-89zzzzzzzzzz',
 '89vvvvvv-e7ww-45xx-b5yy-23zzzzzzzzzz',
 '25vvvvvv-f5ww-4dxx-afyy-1ezzzzzzzzzz',
 '79vvvvvv-05ww-4dxx-96yy-51zzzzzzzzzz',
 '2bvvvvvv-f8ww-43xx-a7yy-92zzzzzzzzzz',
 '41vvvvvv-75ww-42xx-93yy-aezzzzzzzzzz',
 '62vvvvvv-99ww-4exx-b1yy-fbzzzzzzzzzz',
]
```

From this output, we can see that there are 12 active circuits in our account.

```python
In [41]: len(active_megaport_product_list)
Out[41]: 12
```

### Step 3: Retrieve Product Details for a Specific Circuit

Next, let's retrieve the details for a specific Megaport product (circuit) by its product UID. 
The following method accepts the access token and the product UID to retrieve the circuit details in JSON format:

```python
def get_megaport_product_detail(access_token, productUid):
    url = f"https://api.megaport.com/v2/product/{productUid}"
    headers = {
        "Authorization": f"Bearer {access_token}",
        "Content-Type": "application/json",
    }
    response = requests.get(url, headers=headers)
    
    if response.status_code == 200:
        product_details = response.json()['data']
        return product_details
    else:
        print(f"Could not get product details. API request failed with status code {response.status_code}")
        return None
```

Running the method for a specific product:

```python
In [14]: megaport_product_detail_1 = get_megaport_product_detail(access_token,active_megaport_product_list[12])
```

This will output the details of the selected circuit, such as its bandwidth, provisioning status, and location:

```python
In [19]: pprint(megaport_product_detail_1)
{'aEnd': {'connectType': 'DEFAULT',
          'diversityZone': 'red',
          'innerVlan': None,
          'location': 'NextDC B1',
          'locationDetail': {'city': 'Brisbane',
                             'country': 'Australia',
                             'metro': 'Brisbane',
                             'name': 'NextDC B1'},
          'locationId': 5,
          'ownerUid': 'a-b-c-d',
          'productName': 'BNE-MEGAPORT-LINK',
          'productUid': 'e-f-g-h',
          'secondaryName': None,
          'vlan': 1200},
 'adminLocked': False,
 'attributeTags': {},
 'autoRenewTerm': False,
 'bEnd': {'connectType': 'DEFAULT',
          'diversityZone': 'blue',
          'innerVlan': None,
          'location': 'NextDC M1',
          'locationDetail': {'city': 'Melbourne',
                             'country': 'Australia',
                             'metro': 'Melbourne',
                             'name': 'NextDC M1'},
          'locationId': 4,
          'ownerUid': 'a-b-c-d',
          'productName': 'COMPANY-CORE-M1-10G',
          'productUid': 'i-j-k-l',
          'secondaryName': None,
          'vlan': 1200},
 'cancelable': True,
 'companyName': 'Comany name under the megaport portal',
 'companyUid': 'm-n-o-p',
 'contractEndDate': 1622037600000,
 'contractStartDate': 1619501590224,
 'contractTermMonths': 1,
 'costCentre': 'COM001',
 'createDate': 1619501558536,
 'createdBy': 't-r-e-w',
 'dealUid': 'z-x-c-v',
 'distanceBand': 'INTERZONE',
 'liveDate': 1619501590908,
 'locked': False,
 'maximumRate': 10000,
 'productId': 85050,
 'productName': 'COMPANY-CORE-B1-M1-Megaport-10M (BW:variable)',
 'productType': 'VXC',
 'productUid': '61vvvvvv-5bww-44xx-afyy-b6zzzzzzzzzz',
 'provisioningStatus': 'LIVE',
 'rateLimit': 10,
 'resources': {'vll': {'a_vlan': 1200,
                       'b_vlan': 1200,
                       'rate_limit_mbps': 10,
                       'resource_name': 'vll',
                       'resource_type': 'vll',
                       'shutdown': False,
                       'up': 1}},
 'secondaryName': 'COMPANY-CORE-PToP-B1-M1-Megaport-10M',
 'shutdown': False,
 'up': True,
 'usageAlgorithm': 'POST_PAID_HOURLY_SPEED_LONG_HAUL_VXC',
 'vxcApproval': {'message': None,
                 'newSpeed': None,
                 'status': None,
                 'type': None,
                 'uid': None}}
```

### Step 4: Use Case for Automated Bandwidth Upgrade

Now that we have the circuit details, let’s explore a use case. Imagine you have multiple redundant circuits between two data centers, and the Megaport-provisioned link serves as a backup. You may be running IGP with metrics or BGP with local AS path, preferring the circuit with higher bandwidth. However, you don’t want to pay extra fees for the backup circuit unless it's needed.

With Megaport’s SDN fabric, you can upgrade the circuit’s bandwidth on-demand, and I’ve written a Python method to automate this process.

For example, if you want to upgrade a circuit to 500Mbps based on certain triggers (e.g., SNMP traps or syslog events), you can do so without manual intervention. This eliminates the need to wake up an engineer in the middle of the night to click buttons on the Megaport portal and reduces the risk of human error, making the entire process more efficient and reliable. 

Here’s method to check the current bandwidth value for a given circuit:

```python
def get_megaport_product_bandwidth(access_token, productUid):
    url = f"https://api.megaport.com/v2/product/{productUid}"
    payload = {}
    headers = {
    "Authorization": f"Bearer {access_token}",
    "Content-Type": "application/json",
    }
    response = requests.request("GET", url, headers=headers, data=payload)
    if response.status_code == 200:
        rate_limit_mbps = response.json()['data']['resources']['vll']['rate_limit_mbps']
        return rate_limit_mbps
    else:
        return(print(f"Could not get Vxc product details. API request has failed with status code {response.status_code}")) 
```
Here's a method to upgrade the Megaport circuit's bandwidth for a specific circuit: 

```python
def upgrade_megaport_product_bandwidth(access_token, productUid, new_bandwidth):
    url = f"https://api.megaport.com/v2/product/{productUid}"
    payload = {
        "rateLimit": new_bandwidth
    }
    headers = {
        "Authorization": f"Bearer {access_token}",
        "Content-Type": "application/json",
    }
    response = requests.put(url, headers=headers, json=payload)
    
    if response.status_code == 200:
        print(f"Successfully upgraded the circuit to {new_bandwidth} Mbps.")
    else:
        print(f"Failed to upgrade the circuit. Status code: {response.status_code}")
```


lets confirm and get the existing bandwidth for a circuit we are interested in 

```python
In [42]: current_bandwidth_megaport_product_1 = get_megaport_product_bandwidth(access_token, active_megaport_product_list[12])
Out[23]: 10
```

lets upgrade the bandwidth of the circuit using the "upgrade_megaport_product_bandwidth" method to 500 Mbps. 

```python
In [22]: update_bandwidth_product_detail_1 = update_megaport_product_bandwidth(access_token, active_megaport_product_list[12], 500)
Out[23]: COMPANY-CORE-B1-M1-Megaport-10M (BW:variable) VXC 85050 has been updated to a new rate limit of 500 Mbps.
```

Notice that the PUT request has successfully upgraded the bandwdith parsing the name and the VXC along with the new speed. 

We can run the "get_megaport_product_bandwidth" again to confirm 

```python
In [20]: current_bandwidth_megaport_product_1 = get_megaport_product_bandwidth(access_token, active_megaport_product_list[12])
Out [21]: pprint(bandwidth_megaport_product_1)
500
```

## Conclusion

In this blog post, we've explored how to interact with the Megaport API to retrieve product details and automate bandwidth upgrades. Automating tasks like circuit bandwidth upgrades can save time, reduce operational costs, and prevent human error in critical situations. By leveraging the power of APIs and SDN, you can streamline your network operations and enhance your infrastructure's responsiveness to changes, ensuring a more agile and cost-effective environment for your business.

I will write in the near future a blog post on how I utilized the opMantek opEvents (Firstwave) platform that support an automatic script execution on any event received (whether that’s via SNMP/SNMP traps/Syslog, etc.) to upgrade bandwidth for a certain link upon adjacency link failures, store those values in an SQLite database, and open/close ServiceNow tickets. Stay tuned in this space! 🙂
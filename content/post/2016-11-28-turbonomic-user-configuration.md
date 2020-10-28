---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- Reviews
date: "2016-11-28T23:44:55Z"
id: 509
image: /wp-content/uploads/2016/11/turbonomic-black.png
tags:
- Guide
- Turbonomic
title: 'Turbonomic: User Configuration'
url: /BrettsITBlog/2016/11/turbonomic-user-configuration/
---

We are going to cover the basics of user configuration that Turbonomic makes available. Authentication methods and RBAC are the two main focus areas. We will step through connecting to AD and providing users access.

#### Authentication Methods

Turbonomic provides two platforms authentication platforms. You can use local authentication and AD. AD authentication provides improved flexibility and management.

Managing local authentication comes with significant management overhead. Generally speaking, local authentication is best used for times when AD is unavailable.

Whether using AD or Local authentication, user configuration is the same.

#### Default User Account

The default credentials for Turbonomic are. Username: Administrator and Password: vmturbo.

You are not able to remove the default administrator account. If using AD authentication, it';s a good practice to only use this account if the connection to AD fails.

#### Connecting to AD

All account configuration takes place by going to Admin > Workflows > User Configuration.

Connecting to AD is straight forward. Provide the Domain and if required, specify a domain controller. At the bottom of the page, there is a check box to enable secure communication with AD.

[![Config](/assets/images/2016/11/AD-config.png)]({{site.url}}/assets/images/2016/11/AD-config.png)

Enabling secure connection to AD is critical in a production environment. The image below is a Wireshark capture of my login to Turbonomic. That';s some tasty creds.

[![Plain Text](/assets/images/2016/11/LDAP-plain-text.png)]({{site.url}}/assets/images/2016/11/LDAP-plain-text.png)

There are prerequisites for configuring secure AD communication. The following link provides instructions: <https://greencircle.vmturbo.com/community/products/blog/2014/05/18/setting-up-vmturbo-to-communicate-to-secure-ldap-ldaps>

### Adding Accounts

Adding accounts is simple enough. Click on the Add button and provide the username. You will enter the password for a local user during this process. This is not required for AD users.

[![User Add](/assets/images/2016/11/User-add.png)]({{site.url}}/assets/images/2016/11/User-add.png)

You can select Dedicated or Shared for the account type. Shared accounts can only have the Observer or Advisor roles assigned. Dedicated accounts do not have this limit. Shared accounts are also unable to work on physical infrastructure.

**Observer**
  
The user can use the Dashboard, Inventory, Supply Chain, and Reports views. This is the most restrictive role.
  
**Advisor**
  
The user can use the Dashboard, Inventory, Supply Chain, Reports, and Plan views, but cannot accept recommended actions.
  
**Automator**
  
The user can use all the views except Admin and Policy. This user can accept recommended actions, but cannot perform administrative tasks.
  
**Administrator**
  
The user can use all Turbonomic views.

The account scope allows you to limit what the user can see. You can limit a user to only have control of their VM';s. Or the supporting infrastructure for their VMs.

#### Adding Groups

Groups only work with AD authentication. The group name does not need to be the full distinguished name. You only enter the group name.

[![Group Add](/assets/images/2016/11/Group-Add.png)]({{site.url}}/assets/images/2016/11/Group-Add.png)

Roles and scope are the same was what you configure for user accounts.

When a user is a member of a group in AD, they are able to access Turbonomic with the role assigned to the group.

#### Wrapping up

The process of configuring user access is straightforward. Integration with AD allows for simplified user management. This is especially true with proper use of security groups.

One feature that is absent, is the ability to pass credentials thru on the login page. From a user experience view, it is nice to click a button and pass my credential thru. This removes the need to type passwords more often than required.
---
author: Brett Johnson
categories:
- Vmware
date: "2015-01-22T21:49:20Z"
id: 26
title: Initial Setup of SSO
url: /BrettsITBlog/2015/01/initial-setup-of-sso/
---
SSO is an integral part of providing access rights to your Vcentre server. You can assign permissions to people based on their user account, group memberships and link it with various authentication methods.

After installing Vcentre for the first time log onto the Web Client with the username administrator@vsphere.local and the password you used during the install. The address for the Web Client will be
  
https://_Server Name_:9443/web-client
  
Though if you selected a different port to 9443 during the installation then use that.

[![Initial Login](/assets/images/2015/01/1-Initial-logon.png)]({{site.url}}/assets/images/2015/01/1-Initial-logon.png)

On the Left hand side go to Administration

[![Administration](/assets/images/2015/01/2-Vcentre-home.png)]({{site.url}}/assets/images/2015/01/2-Vcentre-home.png)

Click Configuration > Identity Sources > Plus symbol

[![Administration Menu](/assets/images/2015/01/3-Administration.png)]({{site.url}}/assets/images/2015/01/3-Administration.png)

There are a number of authentication options. As my Vcentre server is part of a Windows domain, I will be using AD (Integrated Windows Authentication) in this example.
  
Enter the domain name if needed and if your Vcentre server is part of the Windows domain, select Use machine account.
  
If you want to use SPN please refer to http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2058298

[![Add Id Source](/assets/images/2015/01/8-Add-Identity-Source1.png)]({{site.url}}/assets/images/2015/01/8-Add-Identity-Source1.png)

From the left hand menu go to Users and Groups, then select the group you would like to change. In the lower box select the Add member button.

[![Users and groups](/assets/images/2015/01/4-Users-and-groups.png)]({{site.url}}/assets/images/2015/01/4-Users-and-groups.png)

Select your domain from the drop down menu and select the user / group you would like to add, then OK.

[![Add user](/assets/images/2015/01/5-Adding-user.png)]({{site.url}}/assets/images/2015/01/5-Adding-user.png)

We are close to finished with adding the new users to the Vcentre server. We now need to add the newly assigned users and groups to specific Vcentre servers.

From the Left hand menu go

Home > Vcentre > Vcentre > Vcentre Servers > _Server Name_ > Manage > Permissions

[![Permissions](/assets/images/2015/01/6-Permissions.png)]({{site.url}}/assets/images/2015/01/6-Permissions.png)

Click on the Plus symbol

Select the role you would like to assign and then click Add
  
Select your user / group and OK and OK again.

[![Add Permissions](/assets/images/2015/01/7-adding-permissions.png)]({{site.url}}/assets/images/2015/01/7-adding-permissions.png)

You have now given a domain member permissions to use the Vcentre server.
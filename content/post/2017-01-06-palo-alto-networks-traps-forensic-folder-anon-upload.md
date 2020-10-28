---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- Security
date: "2017-01-06T12:35:22Z"
id: 582
image: /wp-content/uploads/2017/01/Traps-by-Palo-Alto-Networks-Advanced-Endpoint-Protection.jpg
tags:
- EndPoint
- Palo Alto
- Security
- Traps
title: Palo Alto Networks Traps Forensic Folder Anon Upload
url: /BrettsITBlog/2017/01/palo-alto-networks-traps-forensic-folder-anon-upload/
---

During my time deploying Palo Alto Traps. I noticed something a bit concerning with the Forensics Folder security. After verifying the discovery, I raised a ticket with PA. Which confirmed my findings.

The PA Traps Forensics Folder, cannot support any authentication.

To visit the offical Traps site, click [here](https://www.paloaltonetworks.com/products/secure-the-endpoint/traps)

#### Traps Terms Overview

ESM &#8211; Windows Servers which provide services to run PA Traps

Endpoints &#8211; Windows devices (Laptop, Desktop, Server)

Agent &#8211; PA Traps agent installed on Endpoint

#### Forensics Folder

When a process triggers a prevention event, the endpoint agent can capture forensic information. This is then uploaded to the Forensics Folder.

The location of the Forensics Folder is a URL. https://FF.company.com/BitsUpload. ESM servers provide agents with that URL.

The BITS protocol is used for uploads to the Forensics Folder. Thus, we configure IIS to provide the service.

During installation of the ESM Console, you are able to install a Forensics Folder. ESM will configure IIS for you. ESM console is not supported in the DMZ. Thus, you must configure IIS by hand. This is simple and documented.

#### Anonymous Upload to Forensics Folder

If you choose to do a manual configuration of the Forensics Folder. You should note authentication must be anonymous.

Through my testing, enabling 401 Windows auth, generates an error in the agent log.

Enabling client certificate verification does not error in the agent logs. but, the upload status in ESM will show as &#8216;failed';.

In short. To use this capability, you cannot use IIS to control who uploads to the Forensics Folder. As the agent does not support authentication.

On an internal network, this might be ok. Unless, you subscribe to the &#8216;Zero Trust'; approach.

It is desirable to have agents upload forensic data. Without a public-facing folder, external endpoints must connect to the internal network to upload. Remember, these endpoints have detected local malware. A public facing folder is open to anyone who wishes to upload.

The client uploading must trust the SSL certificate provided by IIS. Or, have a flag to ignore.

#### Support for Forensics Folder in the DMZ

We have established that to work, the Forensics Folder must allow anonymous access. If public facing, this could be quite the issue. Yet according to the Admin guide, this is a supported configuration.

[![No VPN](/assets/images/2017/01/WithoutVPN.png)]({{site.url}}/assets/images/2017/01/WithoutVPN.png)

[![With VPN](/assets/images/2017/01/WithVPN.png)]({{site.url}}/assets/images/2017/01/WithVPN.png)

Securing uploads to the Forensics Folder will require non-IIS authentication methods. These will depend on the capabilities of your environment. This may increase deployment complexity.

Placing the Forensics Folder in AWS could be an option. This is not something I have tested, but am planning on doing.

#### Raising with PA

In light of this, I raised a ticket with PA. After some back and forth, the fine response was.

The upload to forensic folder with BITS is working as designed. The authentication option will be a feature request.

Let';s hope that it leads to something.

#### Final Thoughts

While I have been trying to keep this post to the facts. Limiting personal influence. I would like to share a final thought.

Not providing a method to authenticate uploads feels like a big oversight. Anyone can upload to this folder.

I';m hoping that PA will provide agents the capability to authenticate against IIS. Securing the Forensics Folder, especially public facing.
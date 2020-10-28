---
author: Brett Johnson
categories:
- VCAP7-CMA-Design
date: "2018-11-10T00:00:00Z"
description: Build Manageability Requirements into the Logical Design
tags:
- VMware
- vRA
title: VCAP7-CMA Objective 2.4
---

Manageability requirements cover monitoring and maintaining a solution from both complexity and capability perspective. 

Other requirements can impact how the manageability requirements are met as part of the design. To provide HA functionality, redundant components need to be deployed plus load balancing, this increases the complexity of management. 

A management requirement might be to monitor workflows. With an HA deployment, this could be met by using a centralised logging platform which all components log to, making it easier to search logs and monitoring workflows start to finish.

Looking at this specifically within the scope of the CMA exam, consider what options are available to monitor the system as someone with an administrative role or a general end user. Remembering that requirements could be to limit the number of alerts or e-mails, not just send or see more.

A tenant administrator can configure the notification settings within a vRA tenant, controlling which e-mails are sent at certain events. Do business group managers need to get all the e-mails or only those where an issue has occurred.

Consider at what points would a fabric or IaaS administrator needs to get alerts; such as capacity limits.

How would you integrate vROPs to vRA so a consumer can see the health of VM's that they have deployed? What requirement would make you use this option?

From a holistic system administration perspective, there are some areas where external logging can be configured, such as Syslog. It is important not to forget about the IaaS servers, they are Windows-based and have different options available for exporting logs. 

As vRA is part of the vRealize suite, it is helpful to know a bit about Log Insight and Operations Manager. This is not to say you have to dig deep on those products, but both do have content/management packs available for vRA and having an understanding of those can be helpful.

This post has focused primarily on logging as the method of system monitoring. However, being able to track the cost of VM's and infrastructure is another management requirement which is likely to be in the scope of the exam. vRealize Business provides the functionality mentioned above. 

Consider what influence configurations in vRB could have on managing the system and the requirements it might be used to meet, an example being cost based approval policies.
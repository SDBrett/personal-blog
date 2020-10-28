---
author: Brett Johnson
categories:
- Vertiv
date: "2017-05-13T10:58:21Z"
tags:
- Vertiv
title: 'Vertiv: UPS Discussion'
url: /BrettsITBlog/2017/05/vertiv-ups/
---

During Dell EMC World 2017, I had the chance to chat to Vertiv about who they are and what they do. For the conversation, they chose to discuss their UPS and the benefits it brings, which frankly was underwhelming. Vertiv offers a range of products and services, which we did not discuss at the time.

The first question is &#8216;Who are Vertiv?';. Vertiv provides infrastructure and services for mission critical systems. The official company overview is:

> At Vertiv, your vision is our passion. It’s why we design, build and service mission critical technologies that enable vital applications for data centers, communication networks, and commercial and industrial environments. Formerly Emerson Network Power, we support today';s growing mobile and cloud computing markets with our portfolio of power, thermal and infrastructure management products, software and solutions, all complemented by our global service network. We help strengthen the world’s most vital applications by bringing together global reach and local knowledge, and our decades-long heritage including brands like ASCO, Chloride, Liebert, NetSure and Trellis. It’s an uncommon combination of resources and expertise and one that allows you to truly realize what’s possible

#### On to the UPS discussion

The value proposition for a Virtue UPS is that if the controller board, there is no interruption in power supply. The UPS will run in bypass mode, but the supply is not interrupted. On the surface, these seems decent, but I wonder about the true value as a good power design includes handling of UPS failure. Regardless of vendor claims, it is not reasonable to only place one UPS instead of two.

Additional to failure handling, the UPS feature the ability to dial home. This means, Vertiv can be alerted to ship an UPS in the event of failure. This does not work by telling Vertiv there is a failure, instead by going dark. This is not a model I';m a fan of, as it';s the on board controller that sends the alert. I have the impression that there is not a seperate card which handles this.

Additionally, it is the UPS which calls home. To utilise the service, you must provide connectivity from the UPS to reach Vertiv on an external network. Providing the OOB internet access. This is quite an oversight. In addition, there is a central management server a client can use on their internal network. Why is this service not sending alerts back to Vertiv? Instead of many sources sending data out, it could be limited to one. Also on a system which could be better patched and monitored.

The conversation left me feeling like Vertiv have a good idea, but need to rethink their security approach and value proposition.

If you want to learn more about Vertiv you can visit their website <https://www.vertivco.com/>
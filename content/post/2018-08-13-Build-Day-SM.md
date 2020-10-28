---
author: Brett Johnson
categories:
- Opinion
- vBrownBag
date: "2018-08-13T00:00:00Z"
image: /assets/images/build-day-sm-big-twin/bdl-sm-crew.jpg
summary: Analysis of Supermicro products from Build Day Live!
tags:
- Opinion
- vBrownBag
- Supermicro
title: Build Day Live! Supermicro - Big Twin
---

The Build Day Live! for Supermicro where they built and configured a vSAN cluster on a Supermicro BigTwin.

This post is not a review of the build day video, instead focuses on products discussed throughout the build day video to provide thoughts and opinions on Supermicro's offerings.

[Build Day Live! Videos](https://www.youtube.com/watch?v=RJv9XwyUZog&list=PL2rC-8e38bUXJnvfrQ15Kivo6AYLCpixv)

[Supermicro](https://www.supermicro.com/index.cfm)

#### What is a vBrownBag Build Day Live! ?

Build Day Live! is a live-streamed build event targeted at engineers aimed to show to provide an open and honest insight into the build process. If there are errors in the build process, you're going to see them.

Pre-recorded video is used to fill in the loading wait periods, as the process is streamed live there is no cutting of load times. These videos are with engineers from the vendor to discuss some of the products offered.

[![Courtesy of Jeffery Powers](/assets/images/build-day-sm-big-twin/bdl-sm-crew.jpg)]({{site.url}}/assets/images/build-day-sm-big-twin/bdl-sm-crew.jpg)


#### Who are Supermicro

Supermicro is known for their server hardware, with a focus on achieving maximum performance and efficiency allowable for a given form factor. The brand is most commonly known by engineers who need a more custom hardware build or home lab enthusiasts. 

Generically speaking it would be fair to say that they are a white box hardware vendor. In that, you can build component specific x86 hardware solutions for your needs. Their products are usually very competitive on price.

Supermicro hardware is also sold on the wholesale market to vendors such as Nutanix or Rubrik, meaning that you may already be running their hardware and not know it. 

The feature which made the x86 architecture so famous is also what can make buying hardware from Supermicro difficult. Solutions are designed to be modular, therefore to purchase hardware you need to select each component. For the niche solution requirements, such as high-performance computing, this is great. For the general hardware purchaser, it does make things very difficult.

Consulting services are available as part of the purchasing process to simplify hardware selection. However, this approach is still a bit much for most general use case purchases. It would be an advantage to have several predefined use case builds with several customisation options. Making the process easier for general purchasing, while still providing the high degree of modularity for those niche case customers.

#### vSAN Build

Build Day Live! highlighted the simplicity of deploying vSAN on Supermicro's Big Twin hardware. Many of the build scripts are from the AutoLab project, simplifying the build process.

There were some script errors during the build process which appeared to be minor hiccups without causing a negative impact.

vSAN did display compatibility health warnings which are caused by device and firmware identification on the controller used. The error is benign and to be resolved in a future release.

Overall the build was, well, boring. Which is for a scalable solution from both Supermicro and VMware is a good thing.

#### Big Twin

Supermicro's Big Twin platform received the most attention for this Build Day Live! video. The platform supports up to 4 Nodes per 2U chassis, with each node supporting 2 processes, up to 205W TDP and 24 DIMMs.

A key design factor in providing 4 full powered nodes in the 2U form factor is the physical size of the power supply. The use of a thinner power supply allows more space on the motherboard to increase component capacity. Power supplies are N+1 redundant, meaning a single power supply can power the entire system.

Increase compute density translates into an increase of heat generation. To make this solution work Supermicro have put significant engineering effort into solving the heat removal problem.

A range of boot device options is configurable through the use of an adaptor board on each node. Customers can select a boot device media suitable to their needs and not any of the 6 front disk slots.

Networking is provided by a Supermicro IO Module, which allows the customer to choose the networking speed and physical connection type for their needs. Using a SIOM for networking takes advantage of the building networking capabilities of the CPU.

Each node also provides 2 x 16xPCIe slots to further expand flexibility options.

All of the above highlights an essential factor which Supermicro aims to achieve, modularity. With flexible configuration options, customers purchase a well-engineered hardware solution for their specific needs.

The flexibility of choice can be seen as a double-edged sword, providing too many options becomes overwhelming. By creating validated designs, customers can see examples of hardware configurations.

The other downside to having too many options is when a customer needs to ensure that they can buy the same BOM at a later date if the hardware solution needs to scale. Particularly with HCI, there is value in knowing you can buy the same 24 months later to ensure consistent performance and configuration across the cluster.

I am a big fan of the 4 Node, 2U form factor. It provides more flexibility than a blade solution, often being more straightforward to manage than blade platforms, but is more space efficient than standard rack mount servers. 

[![Courtesy of Build Day Live](/assets/images/build-day-sm-big-twin/bigtwin-switch.jpg)]({{site.url}}/assets/images/build-day-sm-big-twin/bigtwin-switch.jpg)


#### Supermicro now does networking

While watching the event, I learnt that Supermicro has a networking product line with Switches and Adapters. Before this, I honestly did not know that this product line existed.

The switch range includes L2 and L3 switches, ToR datacenter switches and SDN capable switches. With speed options to 100Gbit.

Judging by the user guides the non-SDN ranges of switches have a Quagga interface for configuration and are configured by the CLI and some having a UI as well. I cannot see mention of API or programmatic configuration of these switches.

The SDN switches are what many people refer to as 'white box', are sold with or without a Networking Operation system. These switches are fully compatible with Cumulus Linux which is great to see.

What was disappointing to hear, particularly with the SDN switch, is that there is currently no RedFish or bare metal API support. They did mention that it on the roadmap, but even if this is the case it means that the switch hasn't been build with programmatic interaction as the prefered configuration method from Day 0.


#### Hardware management

Hardware management was briefly touched on throughout the video, with mentions of RedFish API and Supermicro Server Manager (SSM), neither were covered in depth and it would have been nice to learn about bit more about managing Supermicro hardware.

Information on the Supermicro site for SSM and RedFish is lacking so I've been having trouble finding out enough information to being to understand the solutions. A stable and efficient management platform is required to operate at scale, mainly providing crucial global information upfront and the ability to drill into more detail are required.

I was happy to hear about RedFish API support though I have not been able to find examples of solutions which consume this for Supermicro. Much like REST API, the value isn't having the API available; it has solutions which are improved because they can consume the API.

It does feel like Supermicro is on the right track with their management solutions, but without examples of management and external system integrations, it's hard to back this up.

Supermicro has a full suite of hardware products, storage, compute and networking and are in a prime position to push an Infrastructure as Code management strategy. Partnering and creating a configuration and monitoring solutions could give Supermicro an edge.

[![Courtesy of Build Day Live](/assets/images/build-day-sm-big-twin/lab.jpg)]({{site.url}}/assets/images/build-day-sm-big-twin/lab.jpg)

#### Summary

Supermicro hardware is exceptionally well engineered and priced well against the market. With a focus on flexibilty customers can work with Supermicro to purchase hardware specific to their needs regardless of the size.

With such a broad range of hardware they should be pushing a stronger message around Day 2 operations which includes IaC approaches. This is a missed oppurtunity when it comes to clients who are after fast and repeatable deployments.

I would like to see more infomation on their management solutions including videos covering common management operations and automated approaches.
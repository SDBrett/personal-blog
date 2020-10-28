---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- Cloud
date: "2016-11-30T10:14:15Z"
id: 522
tags:
- Cloud
- Data Gravity
- Hybrid Cloud
- SDDC
title: 'Cloud Considerations: Data Gravity'
url: /BrettsITBlog/2016/11/cloud-considerations-data-gravity/
---

Data gravity imposes a significant constraint on assessing where a workload should run. Workloads perform better when close to the source of data.
  
Data latency has a tangible cost associated. The timing might be in milliseconds, but the impact can be significant. A business loses money when employees are waiting. Customers are not patient and will look elsewhere if services are slow.
  
Storage vendors make significant investments to lower storage latency. Sometimes to the point of sub-millisecond. A major use case for HCI is data locality. Which means to get data as close to the workload as possible.
  
For years, storage design has placed workloads close to data storage. Cloud is quite disruptive to this traditional line of thinking. This is especially true for hybrid cloud.
  
Public cloud reduces the visibility of storage locality. There is no guarantee that workload and data are in the same rack, let alone the same room. Data gravity does play a role even when all services are located with one public cloud provider.
  
Hybrid cloud exacerbates, the complexity of data gravity. At the time of writing, I have 22ms latency to AWS Sydney. That is a significant increase from the 1 &#8211; 5ms latency you might achieve on premises. Correct application placement is vital. Another consideration is how to compensate for link degradation. That 22ms may jump to 100ms.

#### Case study

We work at a public transport company, PTC. Customers can view timetables and distributions through our website. The website is a typical 3 tier application. The web server in the DMZ, Application server and DB.
  
Currently, all 3 tiers are within the same DC. 10Gbit connectivity 1ms latency.
  
PTC wants to look at moving this application to the cloud.
  
Scenario 1: Move it all to public cloud
  
Everything is going well with this approach. All services have reasonable latency. Data transfer rate is ok. Due to the nature of public cloud, we are on shared infrastructure. These metrics are not guaranteed.
  
PTC pays a premium for performance configurations. This still doesn';t guarantee that PTC is not affected by noisy neighbours. This premium increases the running cost, especially when all 3 tiers are on 24/7.
  
Due to the moving to cloud, on-prem applications that rely on this DB, have an added 20ms latency. Ouch.

#### Scenario 2: DB on Prem, Application and Web public cloud

PTC have seen a negative business impact from the DB being off-prem. The increase of 20ms latency is reducing productivity. They have decided to move the DB back on-prem.
  
Internal applications are performing much better. But not all is well in the land of transport. Customer start complaining of poor user experience. The website is slow and at times not responsive.
  
The connection between the application server and DB has incurred a 20ms latency penalty. When a customer places a request through the website, many transactions take place. Each DB transaction takes 20ms, plus processing time. This model brings a risk of cost increase. DB transactions are data transfers, often charged for.

#### Scenario 3: DB and Application on-prem and Web public cloud

With their twitter feed on fire from the angry customers. PTC have moved the application server back on-prem.
  
For a customer to view a timetable, a majority of transactions are between the application and DB. Between web server and application, the transaction is minimal.
  
Customer complaints are lower and internal systems are running in a performant manner.

#### Case Study summary

The point of the case study is not to shine a bad light on cloud services. It is to highlight the impact of placement. As workloads that consume data move away, experience degrades. There are considerations to be made and no right way.
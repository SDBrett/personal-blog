---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- Cloud
date: "2016-11-30T14:10:37Z"
id: 529
tags:
- Cloud
- Hybrid Cloud
- Operations Management
- SDDC
title: Operations Managers Role in Hybrid Cloud
url: /BrettsITBlog/2016/11/operations-managers-role-hybrid-cloud/
---
#### Operations managers

Intelligent workload placement used to be a challenge. Systems such as VMware';s DRS provided simplicity. Operations managers take workload placement to another level.

Ops Managers maintain a holistic view of DC performance and health. They have a large base of information to make decisions on. The more sources of data the better the output. Detection of possible failure conditions can trigger an automatic evacuation.

OPs managers provide a range of capabilities outside of workload placement. Through, intelligent placement is the focus of this article.

### On-Premises Placement

Workload placement on premises is somewhat straightforward. Historic and current performance characteristics provide the basis for a decision. These decisions can be reactive or proactive.

At  the end of each month, a finance server experiences heavy load for 2 days. This is easy to identify a pattern. An Ops manager can free up resources, or move the server to higher tier storage. After the 2 days, the server can go back to lower tier storage or move to a host with more VM density.

A simple PowerShell script could also achieve the above. But that';s a static decision. It caters for the environment as was, not as it is.

Infrastructure has a finite capacity. Workloads that have higher demands, cost more to run. This capacity can be the network, disk or compute. As integration increases, we see further improvement in decisions.

### Public Cloud Placement

Hybrid cloud is a new kettle of fish. There are more factors to consider, some of which are outside our control.

The cost model changes as well. The capacity of the cloud provider is unknown. Workloads are on shared infrastructure. We have no visibility into performance metrics.

Workloads have varying running costs, dependent on how they consume resources. As your business consumes more services, the financial cost becomes significant.

We face the challenge, of per second billing on some services. Monitoring the time it takes to consume FaaS is difficult. Understanding the best place to run this function day to day is even more difficult.

For an OPs manager, retrieving current pricing from a cloud provider is a simple GET away. The challenge is understanding the best placement for a workload. Conditions are changing, patterns are not always as clear.

This doesn';t mean an Ops manager is less effective with cloud-based workloads. There is still valuable. It placements more emphasis on correct policy construction.

### Dependency on Policy

Ops managers contain a lot of intelligence. This intelligence is somewhat generic. Out of the box, the software will have a baseline configuration. This baseline fits many purposes. But there are always snowflakes.

In our Software Defined world, policies are critical. We are trusting software to make changes without constant interaction. In a way, the SDDC is more akin to a Policy Defined Data Centre. We use policy to build the framework on how workloads should run.

In the scope of this article, as engineers we use policy to guide our Ops manager. We can make sure it always places two workloads together. We can';t expect our Ops manager to know an application requires 2ms latency or less. Or that it crashes during a migration.

There are conditions known to application owners, which could cause failure. We would construct policy to prevent these conditions.

With policy comes complexity. Policies are powerful and required. But introduce too many and you might reduce the effectiveness of an Ops manager. Each policy is a constraint on the platform. This isn';t bad, just that too many constraints lower functionality.

### Container Use Case

Back on the old PTC (can you tell I write while commuting?). We are running our web servers on containers. We have an on-premises DC, with AWS and Azure providing cloud services.

If latency between our Web servers and on-prem application services exceeds 50ms. User experience degrades. We configure a policy that the containers must be within 50ms of application servers. This policy also states to run on the cheapest location. This second part does not carry as much weight as the first.

Current AWS is the cheapest for our Web Server containers. Latency is at 20ms. Both conditions are met.

During normal operation, our link to AWS sees a latency increase. Latency is now past our 50ms threshold. Our OPs management platform records the change and flags the condition. The action of raising a condition starts an action.

The action checks pricing of other cloud providers and on prem. Azure is the next cheapest option. Latency to Azure is within policy spec.

Now that the action has a location which meets policy. Our Ops manager places a recommendation. Move web server containers to Azure. Using the magic of API, the orchestration platform carries out the task.

Web server containers get spun up in Azure and torn down in AWS.

There is more behind the scenes to make a scenario like this to happen. As the containers are public facing. The action is likely to have contacted the load balancer to redirect traffic.

The purpose of the use case is to demonstrate the increasing value of Ops manager platforms. With the proper use of policy, we are able to ensure the best placement, even with hybrid cloud.
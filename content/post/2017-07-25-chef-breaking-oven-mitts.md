---
author: Brett Johnson
categories:
- Chef
date: "2017-07-25T08:07:19Z"
tags:
- Chef
title: 'Chef: Breaking out the oven mitts'
url: /BrettsITBlog/2017/07/chef-breaking-oven-mitts/
---

Configuration management systems are used to deploy and ensure a standard environmental state. You might use a configuration manager to ensure that a file exists, or a certain setting is always applied to a system. As your IT infrastructure grows or becomes more distributed ensuring a consistent state becomes more difficult. Through deploying and ensuring consistent state across multiple systems based on central policies configuration managers help to lower the administrative overhead, improve security and reliability.

In terms of core functionality, Chef is much like any other configuration managers. It will provide a central location to store and distribute policies and settings to a node (server, network device, etc.). These policies are used to ensure that state of a node is consistent, if there is a variation it will attempt to bring the node into line.

#### **Choosing a configuration manager**

When choosing a configuration manager, the starting point is to understand your use cases, both current and expected; then assess the strongest for your use case. As mentioned above, core functionality between configuration manages is very similar, they have the same goal but different methods to achieve that. Below are some of the key high-level differences between configuration managers, or at least the key ones I have encountered when assessing.

  * Deployment 
      * If an agent is required
      * Methods to deploy an agent
  * Management architecture 
      * If a management server required
      * Policy update methods (push or pull)
  * Language Policies are written in 
      * Ruby
      * YAML
      * XML
  * Node interoperability 
      * OS support strengths and weakness
      * Ability to manage appliances (Switches and routers for example)

#### Architecture and Core Components

Chef requires a central server which manages the Chef environment such as; permissions, policies and nodes. When learning Chef, a great start is to look at their hosted Chef server solution, available at <https://api.chef.io>. This is a free Chef server provided by Chef and provides full application functionality.

The hosted Chef server is not suitable for production deployments instead, you should build your own internal Chef server. By building an in-house server you have extra configuration flexibility and can make topological choices to provide HA and suite environmental topology.

Managed endpoints are known as Nodes, these are added to the Chef server when the Chef agent is installed. The agent installation process is referred to as bootstrapping, which provides all the settings and configurations for the agent. After a node is bootstrapped it will run the chef-client software with parameters providing through the bootstrap command.

##### Cookbooks

Cookbooks contain configurations which are to be applied to nodes, these are downloaded onto the node from the Chef server through chef-client. Individual configuration files are called recipes and describe the desired configuration state. Recipes are applied to a node individually or grouped together in a role.

As scale increases, attaching individual run lists to servers can incur unnecessary management overhead. By grouping recipes together in a role, a single role can provide multiple configurations to the nodes which they are attached. This improves management and consistency as updating a single role will push the update to all applicable nodes.

There are more components to the architecture, the above is just to provide a high-level view, further information can be found at <https://docs.chef.io/>. When starting out, remember that the server is the central repository of nodes, configurations and policies; Nodes are devices or end points, an agent is a software package that checks into the server and configurations are defined in recipes.

#### Assessing Chef

The use of an agent and central management server can be seen as a benefit or a weakness, depending on who you ask. In truth, it all depends on what you’re trying to accomplish. Or as the consultant would say ‘it depends’.

Agents are strong with servers but weak on appliances and network devices. A switch running Cumulus Linux provides a full Debian installation and therefore can have an agent installed. However, a switch running Cisco IOS does not. Therefore, if automating a Cisco network is your goal, Chef is not likely going to be a good choice.

When working on servers, especially Windows servers, many actions require the server to reboot. Agents have a strength in that after the OS is booted, they will attempt to check in and continue where the workflow left off. Removing the need for times or polling to see when the server is online.

A central server does mean additional management overhead, there are considerations to be made regarding availability and monitoring. Even with a single server, you will need to monitor services, patching and security. The benefit is a single point of management and reporting. Reports can be pulled from a central location, providing simple up to date monitoring of the platform.

#### Summary

** **I have been using Chef for a few months now as part of my personal education. While the initial learning curve was steep, it has been proving to be a reliable platform with excellent community support. As a server infrastructure person, it fits very well. I don’t think this would be the case if I was looking for a solution for network devices.

If you want to learn more, there are excellent training resources at <https://learn.chef.io/> covering a number of different scenarios across different node platforms.
---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- Cloud
date: "2016-11-30T10:50:27Z"
id: 525
tags:
- Cloud
- Hybrid Cloud
- Microservices
- SDDC
title: 'Cloud Considerations: Application Architecture'
url: /BrettsITBlog/2016/11/cloud-considerations-application-architecture/
---

Traditional application architecture is not optimal for cloud services. They are big, singular and designed to be always on. Cloud requires small, mobile applications which handle failure without impact. Cloud is constantly failing. Systems are crashing, misconfigurations etc. We see it as stable through application design.

When we interact with cloud systems, we are interacting with systems designed for failure. When there is a failure, the application is able to remain functional.

Cloud application developers build applications with failure in mind. The goal is not to create an application that does not fail. Instead, one that fails with little to no impact.

#### A cloud application characteristics

Mobility: Cloud abstracts location. Applications that have light weight components are able to move fast. Cloud services fail often, this is part of the design. You can minimise this impact by using applications which can move fast.

Stateless: When an application with state fails, the state fails to. Using online shopping, for example, this means customers shopping cart is empty. This is bad for user experience. Applications which are stateless or share state provide resiliency.

Ephemeral: Why pay for something that you';re not using? Using an application that is ephemeral can provide cost savings. As demand drops, instances of the application spin down. When demand increases, instances spin up.

Resiliency: In a cloud world, resiliency doesn’t mean failure don';t occur. It means that we expect failures and plan accordingly. An application needs to fail without impact.

Segmented: Small segments allow scaling and cost reduction. More segments also provide the advantage of reducing failure domains.

As of today, finding applications that meet the above, is a bit of a utopia. Rearchitecturing applications, especially large ones is like steering a massive ship. It takes time. This is a significant reason for the direction of cloud being hybrid.

#### Containers

Containers provide a key component of a cloud-native application, Microservices. Microservices provide the framework to meet a number of cloud application characteristics.

Microservices are the art of dividing an application into small pieces. These pieces are small and lightweight. This, in turn, means that they are fast to spin up. As demand increases, instances can exist in less than a second.

Application mobility is not gained through traditional means, such as vMotion. Due to fast provisioning, it';s not required. Application mobility is the provisioning of a new instance. Why copy when you can build and destroy?

#### Orchestration

Orchestration and configuration management tools are key to containers. Containers are small and there are likely to be many. Management by hand is not workable.

Engineers create policies which define the behaviour. Such as, load thresholds. When load surpasses the threshold, a new container gets created. Orchestration tools consume the policy and manage containers as such.

You want to know that every container is the same. No variance. This is the job of configuration management. Engineers define the desired state. When the orchestrator creates a container, it also talks to the configuration management system. This relationship is how all containers are the same.

This transient nature is not without challenges. We will discuss those in another article.

Summary

Cost is important when considering cloud. Many people look at the price of an instance. But you should understand the impact on user experience. If application architecture delivers poor user experience, there is a loss potential. This, in turn, increases cloud cost.

#### Further Reading

[What are microservices? &#8211; Opensource.com](https://opensource.com/resources/what-are-microservices)
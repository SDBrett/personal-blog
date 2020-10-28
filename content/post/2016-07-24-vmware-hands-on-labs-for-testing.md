---
author: Brett Johnson
categories:
- Vmware
date: "2016-07-24T08:49:26Z"
id: 307
title: VMware Hands on Labs for testing
url: /BrettsITBlog/2016/07/vmware-hands-on-labs-for-testing/
---
VMwares Hands on Labs (HOL) are a great way to explore products and features, without needing to have your own lab. When you launch a lab, the environment is spun up, when you exit  it is torn down. An entire lab is built in a built in a matter of minutes, sometimes less.

Each lab has an intended objective, with a manual on the right to guide you through.

Going through the a lab and following the designated steps is great, but that doesn';t mean you have to stay on track. One of the best parts of the labs is being about to go completely off track and do your own thing.  Once a lab is launched there is nothing stopping you from hiding the built in manual treating it like a personal lab.

Recently I needed confirm the effect of admission control when using reservations. I needed to confirm if a VM had a vCPU reservation set in GHz was set, if admission would stop it booting. I wasn';t sure at the time due the way vCPU reservations only allocate the CPU cycles to that VM when it needs it.

A quick look to find a HOL that might provide what is needed to test. In this case HOL-SDC-1604 vSphere Performance Optimization. This lab has a number of VM';s that have an application to generate load. Great for this test.

Through this lab I was able to test the effect of admission control on assigning resources, booting the VM without any load on the host and when there is load. This provided enough data to confirm the limitations of VM sizing and reservations specified by a vendor. With in 15 minutes of looking at the vendors document I was able to have a lab spun up and concept tested. It took longer to write the response then it did to test.

Sometimes HOL doesn';t support the features required for certain concepts. Due to labs being nested some CPU features aren';t available to the HyperVisor. An example of this is fault tolerance, in a HOL environment the CPU doesn';t support FT.

Next time you have something to test and don';t have a lab ready to go, check out HOL at Https://Hol.vmware.com
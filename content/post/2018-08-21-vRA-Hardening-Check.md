---
author: Brett Johnson
categories:
- vRA
date: "2018-08-21T00:00:00Z"
image: /assets/images/vRA-Product-Icon-Mac_0-150x150.png
summary: Compare vRA appliance configuration against the hardending guide
tags:
- VMware
- Automation
- Chef
- Inspec
title: vRA Hardening Check
---

![vRA Logo](/assets/images/vRA-Product-Icon-Mac_0-150x150.png)
![Inspec Logo](/assets/images/inspec-by-chef-logo.svg)

VMware does a great job of releasing hardening guides for its products, helping us to improve the security level of our deployments. If you're in a position where you need to implement the hardened settings, then it's likely there's an audit process to follow.

I have been working on a project to make the validation process easier by converting settings stated the hardening guide into a DSL which is used by Chef Inspec to validate the environment. The goal is to provide a fast and repeatable validation tool to reduce the operational overhead.

Project: [https://github.com/SDBrett/vra-7-hardening](https://github.com/SDBrett/vra-7-hardening)

Using a code based approach to validation enables the process to be run frequently, perhaps as a scheduled task, further improving security as configuration drift can be detected closer to the time it occurred.

In a test environment, the total time to run the test is 18 seconds.

![Command time](/assets/images/vra-hardening-runtime.png)

As part of making this as easy to use as possible, the project has been designed to work with the chef/inspec docker image. This decision was made to remove the need for Ruby environment management and to deal with dependencies on an air-gapped network.

The container uses the `inspec` command as the entry point, which means standard inspec command options work. For example, the output can be formatted in a more friendly way to your requirements using the --reporters argument.

At the time of writting most appliance settings have been defined, but there is still work to be done.

Also big shout out to [JJ](https://twitter.com/jjasghar) for helping me with some of the sticking points throughout.

#### Links
- [https://github.com/SDBrett/vra-7-hardening](https://github.com/SDBrett/vra-7-hardening)
- [https://docs.vmware.com/en/vRealize-Automation/7.2/vrealize-automation-72-hardening.pdf](https://docs.vmware.com/en/vRealize-Automation/7.2/vrealize-automation-72-hardening.pdf)
- [https://hub.docker.com/r/chef/inspec/](https://hub.docker.com/r/chef/inspec/)
- [https://www.inspec.io/](https://www.inspec.io/)
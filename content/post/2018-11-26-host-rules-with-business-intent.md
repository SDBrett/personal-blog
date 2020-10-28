---
author: Brett Johnson
categories:
- VMware
- vROPS
date: "2018-11-26T00:00:00Z"
tags:
- VMware
- vROPS
title: Host rules with Business Intent
---

vROPS 7.0 launched with a feature called Business Intent which controls workload placement based on vSphere tags [Announcement](https://blogs.vmware.com/management/2018/08/whats-new-in-vrealize-operations-7-0.html). This feature tackles several use cases one being controlling the host placement of workloads which have physical licencing requirements.

Business Intent settings are configured at the data centre (or custom datacenter) level within vROPS 7.0 and can be configured at the per cluster or per host level.

For the scope of this post, I am going to cover a customer use case to combine clusters, increaseing availability for some workloads while maintaining licencing compliance for others.

This workloads within the environment are about 98% RHEL and only a couple are Windows-based. Both RHEL and Windows operating systems are licenced by the physical host that the VM can run on. The more hosts within a cluster, the more OS licences required.

In this scenario, there were 2 clusters, 
- AppL: 4 Physical hosts, all licenced for RHEL
- AppLW: 2 Physical hosts, all licenced for both RHEL and Windows

[![Current Clustering](/assets/images/vROPS-BI-Existing-Clusters.svg)]({{site.url}}/assets/images/vROPS-BI-Existing-Clusters.svg)

The customer was experiencing sub-optimal resource utilization with the existing cluster configuration. With the launch of vROPS 7.0, we decided to see if improvements could be make with using Business Intent.

The goal of implementing Business Intent rules was to consolidate the two clusters, increasing the availability of the RHEL VMs while maintaining the host alignment restrictions of the Windows VMs.

[![New Clustering](/assets/images/vROPS-BI-New-Cluster.svg)]({{site.url}}/assets/images/vROPS-BI-New-Cluster.svg)

Under the hood, Business Intent uses vSphere tags on hosts and VMs to create DRS affinity groups. Those groups are used to create DRS 'must run on' rules to enforce the policy.

The initial plan was to create a vSphere tag category called 'Licence' and two tags within; the first tag was 'RHEL' and the second was 'Windows'. I would assign RHEL only hosts the RHEL tag and on hosts which could run Windows and RHEL I would assing both.

During the setup, I learnt that a single host could not have multiple tags from the same selected category. My initial plan would not work.

After some e-mails, a new way to look at the solution clicked. Lightbulbs and everything.

Business Intent rules only apply DRS 'must rules' to VMs and Hosts which have the required tag. It does not impact VMs or Hosts which do not have these tags. I could have the RHEL workloads run on all 6 hosts within the cluster by doing nothing. The Windows workloads were limited to specific hosts by applying the tags.

[![New Clustering](/assets/images/vROPS-BI-New-Cluster-Tags.svg)]({{site.url}}/assets/images/vROPS-BI-New-Cluster-Tags.svg)

Example configuration and results:

[![Tags](/assets/images/vrops-bi-catagory.png)]({{site.url}}/assets/images/vrops-bi-catagory.png)

[![Host Tags](/assets/images/vrops-bi-host-tag.png)]({{site.url}}/assets/images/vrops-bi-host-tag.png)

[![VM Tags](/assets/images/vrops-bi-vm-tag.png)]({{site.url}}/assets/images/vrops-bi-vm-tag.png)

[![vROP Setup](/assets/images/vrops-bi-setup.png)]({{site.url}}/assets/images/vrops-bi-setup.png)

[![Host Group](/assets/images/vrops-bi-affinity-hostgroup.png)]({{site.url}}/assets/images/vrops-bi-affinity-hostgroup.png)

[![Vm Group Setup](/assets/images/vrops-bi-affinity-vm-group.png)]({{site.url}}/assets/images/vrops-bi-affinity-vm-group.png)

[![Affinity Rule](/assets/images/vrops-bi-affinity-rule.png)]({{site.url}}/assets/images/vrops-bi-affinity-rule.png)

From the above affinity rule images, you can see that the settings are there to specifically restrict the hosts a VM can run on, not control VMs which are do not have the required vSphere tag.

When considering host based Business Intent rules, remember that they are a 'this' construct; not 'this OR that'.
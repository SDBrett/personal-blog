---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- Reviews
date: "2016-11-01T00:01:01Z"
id: 407
tags:
- Turbonomic
title: Turbonomic First impression
url: /BrettsITBlog/2016/11/turbonomic-first-impression/
---
A few days ago, I spun up Turbonomic on my home lab and have had a bit of a look around. I thought I would share my experience and thoughts on what I have seen from the product.

Installing Turbonomic is about as easy and straight forward as you can get. Download the OVA from Turbonomics website and deploy. After deployment log on to the console and configure the IP addressing. Done, time for a beer.

All work is done through the web UI which is functional, but not that pretty. There are a number of dashboards, and you can build your own as well. Below is an image of the default Dashboard "Assure Service Performance&#8221;

[![Dashboard](/assets/images/2016/10/Dashboard.png)]({{site.url}}/assets/images/2016/10/Dashboard.png)

When looking through the inventory of my vCenter server, there are numerous about of options to find an object and get performance data on it. As you drill down you notice the same entries appearing more then once. To me, this appears that the full inventory isn';t intended to be used often. The reliance should be more weighted to dashboards.

[![Inventory](/assets/images/2016/10/Inventory.png)]({{site.url}}/assets/images/2016/10/Inventory.png)

The UI is quite responsive and considering my home lab build that is saying something. I have 3 ESXi hosts nested in vCenter using the 1 physical SSD to provide storage to all three, which is then configured to run VSAN where all my VM';s run. Getting very quick responses from the UI and no errors or issues has been awesome and I think it could only be better in an environment there the configure makes sense.

Systems that Turbonomic ingests data from are called "Targets&#8221;. There are many options for Targets and honestly more than I expected. There are options for Hypervisors, PaaS, Databases, Windows Applications, Cloud and event Load Balancers. I am quite impressed by the list of options that I can configure Turbonomic to work with and provide feedback on.

[![HV Entry](/assets/images/2016/10/HV-entry.png)]({{site.url}}/assets/images/2016/10/HV-entry.png)

After the initial configuration,Turbonomic needs time to gather data.  After a few days and thanks to the nature of a home lab, I came back to a dashboard with some nice recommendations on how I can improve my environment. Based on a recommendation I was able to move a VM from within the Turbonomic UI, I didn';t need to touch vCenter. There was a latency warning about VSAN, that';s not a surprise as I can spike past 14 seconds (yes seconds). My NSX manager VM had a recommendation that memory should be decreased from 4 to 3 GB.

Like any good capacity manager system, there is a planning section to help evaluate the impact changes will have to the environment. I will be writing about  what it can do later on, prior to that I feel I need to understand the product better.

From a first impression POV I like what I have seen. I am looking forward to spending some time with the product to see what it can do. There is a lot more to Turbonomic that my initial assumptions told me. To get the most out of this product there will be tuning required on a per deployment basis, using the default recommendations will only get you so far, properly tuning the baselines etc. will provide better ROI.

From here I';m going to actually open the manual and understand the product and posting, explaining more about the capabilities and how I am finding the product, good, bad or indifferent.

If you want to check out Turbonomic, it can be downloaded from their website. You will need to fill out a form and expect a follow-up call. The documentation I';m happy to say does not require and sort of sign up.
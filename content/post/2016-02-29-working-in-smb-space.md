---
author: Brett Johnson
categories:
- Career
date: "2016-02-29T20:01:23Z"
id: 178
title: Working in SMB Space
url: /BrettsITBlog/2016/02/working-in-smb-space/
---
One thing I have been hearing a bit of recently is that doing a task at small scale is easier than at large scale. This simply isn';t true, though it';s not harder either. Both small and large scale problems have their limitations and difficulties. While not unique to SMB, budget constraints have a huge impact, spending enough to keep it running and stopping there is pretty common.

Let';s face it, pretty no matter who you work for, there is never enough money for IT. When working with SMB clients one of the hardest things to work with is the limited budget for a solution or a fix, and demonstrating value of going beyond the essentials.

Warranties and maintenance contracts are a great example here. I see a fairly typical refresh cycle of 5 &#8211; 7 years for SMB, during this time the biggest constraint isn';t the need for more performance but disk storage space the workload doesn';t really change but data grows. Generally speaking when an SMB client does can upgrade it';s everything at once.

Now it';s great practice to keep hardware under warranty to ensure that you can get parts in a reasonable time, especially as the infrastructure ages or you get a tech out in the next 6 hours. However when extended warranties make up a significant percentage of the IT budget it';s hard for the client to justify the spend.

One of the saving graces for aging hardware with no warranty has been virtualisation. Abstracting the hardware away, making DR restores easier has really helped a lot. An MSP can grab a few servers on special and use them for testing internally until a client';s hardware crashes. Loan the server out and run up a DR restore. This can be done pretty quickly, usually within a day or two.

Redundancy, surely that can help prevent the above from happening? Right?? You go to a client with a 25K hardware quote for a new server, new switch and some labour, all good, that';s what the client was looking at and budget is met. Now let';s put HA into the equation, we now need to provide shared storage, another pizza box, licencing for the solution, more warranty, more labour. Most of the time this will just shoot well past the client';s budget limits and it';s not that they don';t see how it';s helpful, it just adds up to 2.5 X the original quote. The most common SMB client build I come across is 1 host and 2 &#8211; 4 VM';s, this is backed up to a NAS and replicated to a USB drive for off site backup. The reality is that with a significant percentage there is only DR.

With budget';s incredibly tight on hardware, comes tight budget on hours. Let';s face it MSP';s need to bill. It';s common for SMB';s to have regular maintenance checks on a monthly basis. The hours allotted for this are enough to cover the basics, updates, events and maybe a request here and there. One of the biggest things cut due to the limitation of hours is security. A firewall is setup at infrastructure update and after that it';s rarely checked or updated. Security training practices for clients is not easy to &#8216;sell';. Log';s are there for when things break and audits are not commonly done on them.

What I';m trying to get across is that while the scale is small, the restrictions are high. As an MSP we need to be able to provide as much value for the time we spend and the dollar the client spends with us and when working to a budget there have to be hard calls.

On a positive note on of the biggest changes to the SMB market recently has been Office 365. To me this is probably best thing for SMB';s. No exchange server to manage, they now have a redundant email system and the cost is incredibly reasonable. Even in Australia where we could almost shout the bits down a string joining 2 cans and get more reliable internet, the service is delivered exceptionally well. Removing exchange from the maintenance saves enough time to bring in better practices such as verifying firewall policies.

I have seen a few hardware vendors start to offer maintenance agreements that offer hardware upgrades after X number of years. This to me is great, because it can help the client see something at the end of the agreement instead of throwing money at something that they believe shouldn';t break because it cost 25k.

With all of the above it';s probably not surprising that another missing area is test. A dedicated test environment is very rare for an SMB. For things such as updates -Both OS and Application &#8211; it';s done live, depending on the server, maybe there is a snapshot.  There are some basic work arounds for individual circumstances. The above mentioned loan servers can be deployed to a client to test a major change and backups can be DR tested in a similar method.

When working with SMB clients you will need to have a very broad skillset. As clients like to have a single point of contact, you might find that you will end up working on the networking, infrastructure, doing break fix and designing the next refresh. It';s hard to be in a silo in this space, a good way to develop a skillset is like this letter T. The vertical line is a skillset that you know deeply and the horizontal are the rest of the skills that you need to maintain. So you might have deep virtualisation knowledge and good knowledge in AD, Exchange, 0365 etc. If your essentially a SME on one topic and can';t cover the rest you will have a hard time.

The point of this post is simply to point out how tight budgets can be for SMB and hopefully help shed some light on why things such as 2 DC';s or HA aren';t commonplace despite being good practice and even things such as WSUS have to be sacrificed due to the cost of disk space.
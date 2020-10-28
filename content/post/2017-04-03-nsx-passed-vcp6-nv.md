---
author: Brett Johnson
categories:
- NSX
date: "2017-04-03T09:17:35Z"
tags:
- NSX
- VMware
title: 'NSX: Passed VCP6-NV'
url: /BrettsITBlog/2017/04/nsx-passed-vcp6-nv/
---

In October 2016, I took the VCP6-NV exam and failed. While this was not the first exam I have failed, it was eye opening. After seeing the question set, I realized my understanding of the exam was incorrect. I wrote a post 

In February, I went to retake the exam but due to an issue with the Pearson systems I couldn';t sit on the day.

On the 31st of March I sat the exam for the second time and passed. During my journey to get this cert I have had some leanings I would like to share.

##### It's not an NSX Exam 

The biggest change I had to make in thinking of this exam, is that it is not an NSX exam. Now, I know the product it';s based on is NSX. But the exam is on Network Virtualization.

I failed the first attempt at the exam, because I only studied NSX. I did not deepen my knowledge on vSphere networking and the role physical networking plays
  
Objective 3 on the blueprint should not be overlooked. To meet this objective, you will need to know these topics well. Do not skimp.

There is an upgrade path from vCNS to NSX. As a result, you should understand the differences between the two.

##### Know your exam version

Software defined anything enables rapid change between versions. When first attempting the exam, I was using version 6.2 documentation. As it contains more detail and content compared to the 6.0 documents. There have been significant changes between these versions. Changes that does impact your ability to correctly answer the question asked.

If you';re familiar with any VCP exam, you';ll know that VMware loves to test on configuration maximums. There are many changes between 6.0 and 6.2. A small example is with the ESG, where the maximum number of VIPs is not 1024. You';re answer might be right for 6.2 but wrong for what the question is asking.

There are differences in the version of vSphere ESXi that 6.0 and 6.2 support. As there are upgrade and installation topics, you';ll want to make sure you have the correct versions for that exam

#####  Study Material

I used 3 primary sources of material for my studies; The official study guide, VMware documentation and Jason Nash'; Pluralsight course.

I spoke about the official guide on my failure post. The guide is aimed towards the 2V0-641 exam. Not the new 6.2 version 2V0-642.

The product documentation contains all the information required for the exam. It';s long and a bit dry. Do not neglect the vSphere networking PDF.

Jason Nash did an excellent course on Pluralsight for NSX. The course is not up to date for the latest exam. However if you are new to NSX it still can provide value by providing a general understanding.

##### Summary

The exam is Network Virtualization, not NSX. Keep this in mind and make sure you don';t skimp on the vSphere networking knowledge.

I did the 2V0-641 exam, which is the first of the NV exams. This exam will be retired on 30/04/2017. The replacement is the 2V0-642. This is good as newer releases have better documentation. The new exam is based on NSX 6.2 so stick to those documents.

Good luck to those looking to sit this exam.
---
author: Brett Johnson
categories:
- Reviews
date: "2016-02-15T14:18:42Z"
id: 160
title: Canary Token
url: /BrettsITBlog/2016/02/canary-token/
---
On the 28th of Jan, I got a notification on my phone that episode 396 of the Risky Business podcast was ready to be streamed for my drive home commute. The sponsored interview with Haroon Meer on a free service called Canary Tokens from Thinkst really got my attention. It was said that Canary Tokens were simple to implement service to produce an alert when activated, they could be placed into directory structures, DNS lookup, Word documents etc.

What got my attention is how easy this could be to set and forget, just generate the token and wait for a designated mailbox to got &#8216;bing';.

Setting up the tokens couldn';t be easier. All you need to do is enter an email address and a description, set the type of token that you want and hit generate. Afterwards you get a number of options for your token. The token generation and activation is bloody quick. As fast as I could, I copied the DNS of my new token and ran it through _nslookup_ as soon as the second time out occurred I had an email that my token was hit.

{{< highlight text >}}
One of your canarydrops was triggered.

Channel: DNS
Time : 2016-02-15 02:23:58.035387
Memo : DNS Test token
Source IP:
{{< / highlight >}}

The key with the creation is to have a good description, since it could be a while before the token is hit. Tokens can be turned off with a management link on the bottom of the email.

The implementation of tokens is very straightforward. The directory option for creating a token is a desktop.ini file with

<pre class="lang:default decode:true">[.ShellClassInfo]
IconResource=\\%USERNAME%.%USERDOMAIN%.INI.kdh1.canarytokens.com\resource.dll</pre>

Very simple to scatter

In the word document it';s part of the footer2.xml file contained as part of the .docx file. Again this is something that is simple to place throughout your network. There is no need for macros or adjusting the security settings in office.

There is a great deal of flexibility, some great examples are given on the podcast. For those of you who are concerned about supplying your email address and getting spammed. So far I have not received one email from Thinkst asking me to join a webex, attend an &#8216;exclusive'; event or look at a single product, it really has been great, in fact the only emails I have received have been from when I triggered my tokens.

I';m not a security expert and I think that this actually makes tokens more valuable to me as they are an easy detection system to scatter throughout my network. They cost nothing and don';t consume any resources.

If you have event 15 minutes free I highly suggest that you check out Canary Tokens.

A few links for you to check out

<a href="http://canarytokens.org/generate" target="_blank">Canary Tokens</a>

<a href="http://thinkst.com/" target="_blank">Thinkst</a>

<a href="http://risky.biz/" target="_blank">Risky Business</a>

<a href="https://twitter.com/riskybusiness" target="_blank">Patrick Gray Twitter</a>

<a href="https://twitter.com/haroonmeer" target="_blank">Haroon Meer Twitter</a>

&nbsp;
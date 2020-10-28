---
author: Brett Johnson
categories:
- Python
date: "2017-02-08T22:39:23Z"
tags:
- API
- Python
- Cloud
title: 'Python: Getting Started with HTTP Requests'
url: /BrettsITBlog/2017/02/python-http-requests/
---

If you';re looking at Python to interact with API';s, it';s likely that you';ll use the Requests module. Many platforms also offer SDK';s to help. Such as, Boto3 which is the AWS Python SDK.

Requests is very well documented, both official and community documentation.

The official site for Requests is <http://docs.python-requests.org/en/master/>

In this post, we are going to cover the basics of performing a HTTP GET and working with the data.

#### Installing Requests

Requests can be installed in a couple of ways. The first is using pip. For Linux and Mac this can be installed using the command

{{< highlight bash >}}
$ pip install requests
{{< / highlight >}}

For Windows

{{< highlight bash >}}
$ Python -m pip install requests
{{< / highlight >}}

Another option is to download requests from GitHub and run the setup.py file

{{< highlight bash >}}
$ git clone git://github.com/kennethreitz/requests.git
$ cd requests
$ python setup.py install
{{< / highlight >}}

Note: Installation may require elevated privileges.

#### Performing a HTTP GET

Performing a HTTP GET is one of the first things you';re likely to do. A GET will attempt to retrieve (GET) data from a resource. The operation is simple enough. To help with learning, there are a number of sites which you can test basic operations. For this post we are going to use [JSON Placeholder](https://jsonplaceholder.typicode.com/).

The resource that you intend to retrieve data from could require authentication. This is out of the planned scope for this post.

To get started you will need to import the module.

{{< highlight python >}}
>>> import requests
{{< / highlight >}}

Use the following command to make a GET request. This retrieves the data and saves it to a variable.

{{< highlight python >}}
>>> response = requests.get('https://jsonplaceholder.typicode.com/users')
{{< / highlight >}}

In order to verify the request was successful run the following.

{{< highlight python >}} 
>>> response.ok
{{< / highlight >}}

This is a boolean value, returning True for a valid response. Otherwise False. It provides a simple method for controlling the scripts behavior. Response also contains the status code.

To see the status code run

{{< highlight python >}} 
>>> response.status_code
{{< / highlight >}}

Response will contain a number of attributes. Such as, header, content and as we have seen ok. For this post we will focus on the content. Which is the actual data retrieved.

#### Parsing Content

The content of a request can be accessed through a number of attributes. Each attribute return the content with a specific class.

{{< highlight python >}} 
>>> type(response.text)
class str
{{< / highlight >}}

{{< highlight python >}} 
>>> type(response.content)
class byte
{{< / highlight >}}

Requests includes a built in JSON encoder, which makes list much easier for parsing data. As a side note: This is something I should have read up on before using requests. I would have saved a lot of time for myself.

{{< highlight python >}} 
>>> type(response.json())
class list
{{< / highlight >}}

Each item in the list is a dict. Which means to parse, we iterate through the list and look for a key value.

{{< highlight python >}} 
>>> for x in response.json():
  print(x)
{{< / highlight >}}

Using the same data, lets look for the phone name and contact name for kale.biz.

We will iterate the content and look for an entry where the key "website&#8221; matches "kale.biz". If it does match we will print the phone number and contact name.

{{< highlight python >}} 
>>> for x in response.json():
  if x['website'] == 'kale.biz':
    print(x['name'])
    print(x['phone'])
{{< / highlight >}}

#### Summary

The requests module is a key part of interacting with API using Python. Take your time to understand how it works and what scenarios you can apply it to.

For practicing there a number of sites which allow you to make HTTP requests, have a look and experiment.
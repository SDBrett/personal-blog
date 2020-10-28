---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- Python
date: "2017-01-26T13:09:04Z"
id: 624
image: /wp-content/uploads/2017/01/python-logo-master-v3-TM.png
tags:
- API
- Automation
- Python
- Scripting
title: 'Python: Parsing values from API Response'
url: /BrettsITBlog/2017/01/python-parsing-values-from-api-response/
---

Boto3 was my first real attempt to work with an API interface. At the start, I had difficulty using the API response. This was partly due to only light exposure to Python. Also, an incorrect understanding of what the response was.

When people talk about APIs, it';s hard to go a minute without hearing "JSON format';. I had seen JSON formatted text before. Combining this, with documentation displaying API call response in JSON formation, lead to a 2+2=5 scenario. I thought that was the object type returned.

JSON is a text formatting standard. That';s it. It does not describe the object type which is returned. For a high level "What is JSON&#8221; visit this [page](http://developers.squarespace.com/what-is-json/).

#### API Response Object Types

When you make an API call, whether it is a GET, PUSH or PUT, you will get a response. The response is in a structured format, using Keys and Values. In Python, that';s a dictionary (dict).

The dict structure is what provides the flexibility and searchability. The response could be a standard dict. However, if a key contains multiple values, you are likely to see a list in the dict.

This distinction matters, because it helps get more accurate search results when you';re stuck. You';re not looking how to parse JSON. Instead, how to parse a dict. This is also true for how you might store values from the response.

#### Example: Retrieve data from response

The below code snippet is an example of a response from Boto3 documentation. This response is a dict and contains lists.

{{< highlight python >}}
response={
    'InternetGateway': {
        'InternetGatewayId': 'string',
        'Attachments': [
            {
                'VpcId': 'string',
                'State': 'attaching'
            },
        ],
        'Tags': [
            {
                'Key': 'string',
                'Value': 'string'
            },
        ]
    }
}{{< / highlight >}}

Dicts are contained within a pair of curly braces. From that, we can see "InternetGateway&#8221; is a dict. It contains key values. In order to retrieve the internet gateway ID, we would use the following:

{{< highlight python >}}response['InternetGateway']['InternetGatewayId']{{< / highlight >}}

Retrieving data stored in "Attachments&#8221; is a bit different. The square brace after "Attachments&#8221; tells us that we need to parse a list as well. For added fun, the list contains a dict. As the list only contains the one dict, we know our data is in position 0.

To get the state of this internet gateway, we would run:

{{< highlight python >}}response['InternetGateway']['Attachments'][0]['State']{{< / highlight >}}

This will go to the value of the key "State&#8221; from index 0 of the key "Attachments&#8221;

As a side note on lists, the index starts at 0, not 1. The first entry will be index 0. Below example image is courtesy of [Carl Niger](https://twitter.com/carl_niger).

[![Index](/assets/images/2017/01/list-index.png)]({{site.url}}/assets/images/2017/01/list-index.png)

#### Summary

If you';re going to be working with API calls, understanding how to parse lists and dictionaries will be a foundation skill. Take the time to understand different methods and where to use them.

JSON is great for reading, but it';s not the object type that you will be actually working with.
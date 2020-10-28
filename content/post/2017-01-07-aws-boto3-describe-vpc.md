---
academia_post_display_home:
- ""
author: Brett Johnson
categories:
- AWS
- Python
date: "2017-01-07T22:48:43Z"
id: 596
image: /wp-content/uploads/2017/01/AWSLOGO.png
tags:
- AWS
- Boto3
- Cloud
- Python
- Script
title: 'AWS Boto3: Using Boto3 to Describe VPC'
url: /BrettsITBlog/2017/01/aws-boto3-describe-vpc/
---

To describe a VPC is to retrieve the values of it attributes. A task we might perform to validate configuration.

This article will demonstrate the following:

  * Find VPC ID using filters
  * Retrieve VPC configuration values

Information on Boto3 can be found [here](http://boto3.readthedocs.io/en/latest/index.html).

This post assumes that you already have a working Boto3 installation. Including IAM configuration to perform the task. If you have not, click [here](https://boto3.readthedocs.io/en/latest/guide/quickstart.html) for the install document.

#### Modules and EC2 connection

We require the JSON and Boto3 modules. Thus, they will be imported at the start of the script. The reason for Boto3 should be fairly straight forward. To make the responses readable, JSON is required.

The third line connects to EC2 for our region. Adjust the region name as required.

{{< highlight python >}}import json
import boto3
ec2 = boto3.resource('ec2', region_name='ap-southeast-2')
client = boto3.client('ec2')
{{< / highlight >}}

#### Retrieving VPCs

Filters are used to scope our results. These filters are based on tags. Such as, Name, VPCID etc. You can use multiple entries for the value or a wildcard "*&#8221;.

{{< highlight python >}}
filters = [{'Name':'tag:Name', 'Values':['VPN*']}]
{{< / highlight >}}

<{{< highlight python >}}
filters = [{'Name':'tag:Name', 'Values':['VPN01', 'VPN02']}]
{{< / highlight >}}

Now to perform the query using the ec2.vpcs resource.

<{{< highlight python >}}vpcs = list(ec2.vpcs.filter(Filters=filters)){{< / highlight >}}

The variable vpcs now contains a list of VPCIDs for the VPCs which matched our filter.

<{{< highlight python >}}[ec2.Vpc(id='vpc-1b7e'), ec2.Vpc(id='vpc-dcb7')]{{< / highlight >}}

#### Describing the VPCs

We have searched for VPCs and their IDs are now in the list vpcs. It';s now time to see some information on them.

We will utilise a for loop to achieve this.

{{< highlight python >}}for vpc in vpcs:
    response = client.describe_vpcs(
        VpcIds=[
            vpc.id,
        ]
    )
    print(json.dumps(response, sort_keys=True, indent=4)){{< / highlight >}}

vpc.id provides the ID number for each entry.

I have ended this with a print. You can of course change that to store the output for further use.

The formatted response should be similar to this:

{{< highlight json >}}
{
    "ResponseMetadata": {
        "HTTPStatusCode": 200,
        "RequestId": "5bed6fab-fc86"
    },
    "Vpcs": [
        {
            "CidrBlock": "192.168.0.0/25",
            "DhcpOptionsId": "dopt-8",
            "InstanceTenancy": "default",
            "IsDefault": false,
            "State": "available",
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "VPN1"
                }
            ],
            "VpcId": "vpc-1b7e"
        }
    ]
}{{< / highlight >}}

#### Summary

The script provides a framework to audit VPCs and their configurations. I hope that you found it helpful.

The script in full can be found at https://github.com/oversizedspoon/DescribeVPC-BOTO3
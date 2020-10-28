---
author: Brett Johnson
categories:
- Python
date: "2016-05-08T21:38:39Z"
id: 245
title: Creating AWS Instances with Boto3
url: /BrettsITBlog/2016/05/creating-aws-instances-with-boto3/
---
Yesterday I decided that I would like to create an AWS instance using python. After not very much searching, I came across Boto3 which is the python SDK for AWS and set to work. Being fairly green with both python and using APIs I felt like this was a bit of learning curve, but worth undertaking.

The code for this task is located on [GitHub](https://github.com/oversizedspoon/NewAWSInstance)

For testing, I have been using Python 3 and the latest Boto3 build as of the 8/05/2016. If youre using a version of Boto prior to 3, you will most likely find that the details below will not work.

Boto3 can be installed through pip or by cloning the GitHub repo, I chose the GitHub repo. The Installation was very simple, navigate to the directory that you cloned Boto3 into and run _python setup.py install_. Thats all there is to getting Boto3.

To use Boto3 our script needs to import the modules, this is done by using

{{< highlight python >}}import boto3{{< / highlight >}}

After importing the Boto3 module we need to connect to the EC2 region that the instances are to be created on. This is achieved through the below snippet.

{{< highlight python >}}ec2 = boto3.resource('ec2', region_name="ap-southeast-2"){{< / highlight >}}

A list of regions with codes can be found [here](http://docs.aws.amazon.com/ElasticMapReduce/latest/DeveloperGuide/emr-plan-region.html)
  
The first section of code creates a KeyPair to be assigned to the created instances. If using an existing KeyPair, this can be commented out. The request to create a KeyPair returns key_material which is the private key, this is placed as a string into KeyPairOut and saved to a file.

{{< highlight python >}}outfile = open('TestKey.pem','w')
key_pair = ec2.create_key_pair(KeyName='TestKey')
KeyPairOut = str(key_pair.key_material)
outfile.write(KeyPairOut){{< / highlight >}}

The next major code section creates the instances.

{{< highlight python >}}
instances = ec2.create_instances(
	ImageId='ami-e0c19f83', 
	MinCount=1, 
	MaxCount=5,
	KeyName="TestKey",
	InstanceType="t2.micro"
)
{{< / highlight >}}

There are a lot of attributes that can be used when creating and instance, for this task I have tried to keep it as simple as possible.

ImageID: This specifies the instance we want to create. In this case, we will create a RHEL 7.2 instance

MinCount and MaxCount: Specify the number of instances to establish

KeyName: The name of the KeyPair to use. If KeyName is not used then you will not be able to access the instance

InstanceType: The size of the instance to create.

The file section of code provide output to the instance IDs created with the script

{{< highlight python >}}
for instance in instances:
    print(instance.id, instance.instance_type)
{{< / highlight >}}

This is a script that I plan on developing over time, making it more useful and seeing what functionality I can put into it.
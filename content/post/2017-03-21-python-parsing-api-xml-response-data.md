---
author: Brett Johnson
categories:
- Python
date: "2017-03-21T21:46:32Z"
tags:
- Python
title: 'Python: Parsing API XML Response Data'
url: /BrettsITBlog/2017/03/python-parsing-api-xml-response-data/
---

Recently I have started to look at the Turbonomic API. Due to my current skill level in Python, I quickly hit a roadblock. The response from an API is in in XML format. Parsing the response in XML slowed things down a little. Which is the focus of the post. How to parse the XML response with Python.

After some time I put the pieces together can work with this product through the API.

You will need two modules. Requests and ElementTree. Links to the documentation is below.

  * Requests: <http://docs.python-requests.org/en/master/>
  * ElementTree: <https://docs.python.org/3.6/library/xml.etree.elementtree.html#module-xml.etree.ElementTree>
  * If you';re unfamilar with XML terminology, I';d recommend having a quick look [here](https://www.w3schools.com/xml/xml_tree.asp).

#### Getting Started

To practice with this, we need XML formatted text. Below is a copy of the response I have been testing against. I performed a GET to retrieve the users.

{{< highlight xml >}}TopologyElements
	TopologyElement creationClassName="User" displayName="testuser" isScoped="false" loginProvider="Local" name="testuser" userType="DedicatedCustomer" uuid="_iXQoYAaeEeeT5YCo6TtTyA"
		TopologyRelationship childrenUuids="_4T_7lQY-Ed-WUKbEYSVIDw" name="role"/
	/TopologyElement
	TopologyElement creationClassName="User" displayName="Administrator User" isScoped="false" loginProvider="Local" name="administrator" userType="DedicatedCustomer" uuid="_4T_7kwY-Ed-WUKbEYSVIDw"
		TopologyRelationship childrenUuids="_4UAioQY-Ed-WUKbEYSVIDw" name="role"/
	/TopologyElement
/TopologyElements{{< / highlight >}}

For these examples, the content isn';t a concern. We will be looking at process.

#### The API Call and Data Parsing

The specifics of the API call will change depending on the system you';re accessing. Due to that, lets keep it generic.

{{< highlight python >}}r = requests.get('https://sdbrett.com/api/users')
{{< / highlight >}}

We have some data in a variable. It';s in XML format. Which means looking at the content object isn';t that helpful.

{{< highlight python >}}r.content
b'?xml version="1.0" encoding="ISO-8859-1"?TopologyElements\nTopologyElement creationClassName="User" displayName="testuser" isScoped="false" loginProvider="Local" name="testuser" userType="DedicatedCustomer" uuid="_iXQoYAaeEeeT5YCo6TtTyA"\nTopologyRelationship childrenUuids="_4T_7lQY-Ed-WUKbEYSVIDw" name="role"/\n/TopologyElement\nTopologyElement creationClassName="User" displayName="Administrator User" isScoped="false" loginProvider="Local" name="administrator" userType="DedicatedCustomer" uuid="_4T_7kwY-Ed-WUKbEYSVIDw"\nTopologyRelationship childrenUuids="_4UAioQY-Ed-WUKbEYSVIDw" name="role"/\n/TopologyElement\n/TopologyElements\n'
{{< / highlight >}}

These is were the module ElementTree comes in. Using ElementTree, we parse the data into a variable. This will use the root of the structure. Essentially, we create a dictionary.

{{< highlight python >}}root = ElementTree.fromstring(r.content){{< / highlight >}}

Now our data is in the root variable, we can work with it.

We will use the method &#8216;iter'; to access data within the variable.

To view all elements (tags) we can use a wildcard.

{{< highlight python >}}for child in root.iter('*'):
    print(child.tag)

TopologyElements
TopologyElement
TopologyRelationship
TopologyElement
TopologyRelationship{{< / highlight >}}

The output lists the elements.

Each element contains attributes. These attributes can be used to access the returned data in a structured format.

{{< highlight text >}}for child in root.iter('TopologyElement'):
    print(child.tag, child.attrib)

TopologyElement {'creationClassName': 'User', 'displayName': 'testuser', 'isScoped': 'false', 'loginProvider': 'Local', 'name': 'testuser', 'userType': 'DedicatedCustomer', 'uuid': '_iXQoYAaeEeeT5YCo6TtTyA'}
TopologyElement {'creationClassName': 'User', 'displayName': 'Administrator User', 'isScoped': 'false', 'loginProvider': 'Local', 'name': 'administrator', 'userType': 'DedicatedCustomer', 'uuid': '_4T_7kwY-Ed-WUKbEYSVIDw'}
{{< / highlight >}}

Looking at this from a Key:Value pairing POV. The key is an attribute and the value is a value. With this in mind, we can use a similar method to access data.

{{< highlight python >}}for child in root.iter('TopologyElement'):
    print(child.attrib['displayName'], child.attrib['loginProvider'])
{{< / highlight >}}

This provides us with usernames and where they are located.

##### Putting it Together

With knowing everything above, we can make practical use of the data.

Lets say, we want to minimize the use of local accounts. Rely on AD for authentication. A first step would be to verify how many local accounts.

{{< highlight python >}}root = ElementTree(r.content)

local_users = []

for user in root.iter('TopologyElement'):
    if user.attrib['loginProvider'] == 'Local':
        local_users.append(user.attrib['displayName']){{< / highlight >}}

And there we go. Local user accounts are stored in a list. Ready to be exported as required.
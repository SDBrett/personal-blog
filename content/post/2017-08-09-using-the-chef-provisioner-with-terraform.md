---
author: Brett Johnson
categories:
- Chef
- Terraform
date: "2017-08-09T20:14:57Z"
tags:
- AWS
- Chef
- Terraform
title: Using the Chef provisioner with Terraform
url: /BrettsITBlog/2017/08/using-the-chef-provisioner-with-terraform/
---

Terraform is an awesome tool used to manage infrastructure using the Infrastructure as Code philosophy. Modules called Providers enable Terraform to communicate with a number of different cloud providers.

Post deployment tasks are performed through a separate set of modules called &#8216;Provisioners';. A provisioner is used to execute commands locally on an instance after it’s been created. One such provisioner enables the Chef client to be installed on the newly provisioned instance and the instance to be added as a node to the Chef Server.

#### Using Provisioners

To use a provisioner, you place the resource for your specific provisioner into the code block for the resource you';re deploying. In the examples below, the provisioned a resource is an AWS EC2 instance. But, the same principals can be applied to a VM in Azure.

{{< highlight hcl >}}
resource “aws_instance” “lamp_server”{
  Instance_type: “type”
  //Instance settings
  Provisioner “chef” {
    Provisioner options{}
  }
}
{{< / highlight >}}

By using the code above, you have stated that you want to have an AWS instance and that you would like to use the Chef provisioner for post deployment configuration. For more information on the EC2 instances settings check out <https://blog.gruntwork.io/an-introduction-to-terraform-f17df9c6d180> by Josh Padnick.

Settings for the provisioner are required in each resource code block. It';s a good idea to store repeated settings in another file such as, &#8216;provisioner.tf';. This provides you with a single point to update required settings.

{{< highlight hcl >}}
variable "chef_provision" { 
  type                      = "map"
  description               = "Configuration details for chef server"

  default = {
    server_url              = "https://api.chef.io/organizations/org"
    user_name               = "username"
    user_key_path           = "~/.chef/userkey.pem"
    recreate_client         = true
    }
}
{{< / highlight >}}

This makes life much easier when working on many resources in a single configuration.

To use the above variables in the provisioner code blocks you need to reference them. Below is an example of my configuration from the chef_mcsa project.

{{< highlight hcl >}}
provisioner "chef" {
  server_url      = "${var.chef_provision.["server_url"]}"
  user_name       = "${var.chef_provision.["user_name"]}"
  user_key        = "${file("${var.chef_provision.["user_key_path"]}")}"
  node_name       = "${var.file_server.["hostname_prefix"]}-${count.index}"
  run_list        = ["role[fileserver]"]
  recreate_client = "${var.chef_provision.["recreate_client"]}"
  on_failure      = "continue"
  attributes_json = -EOF
  {
    "tags": [
      "fileserver"
    ]
  }
  EOF
}
{{< / highlight >}}

As you can see, the provisioner variables are used in conjunction with others to describe how the state of this node for my Chef server.

#### The Chef Provisioner

The Chef provisioner provides a number of options, many which I have not used in my projects to date. The full list is available at <https://www.terraform.io/docs/provisioners/chef.html>.

The key options that you will need to specify are:

  * Server_url – The URL for your organization on the Chef server
  * User_name – The user name for the account connecting to the Chef server
  * User_key – The path to the SSH key for the above user account
  * Run_list – the initial run list to be applied to the node

Some helpful optional ones:

  * Node_name – If specify the hostname/node name
  * Recreate_client – overwrite the node on the Chef server if it exists, helpful for tear down and spin up. Otherwise, you will need to clean up the Chef server manually
  * On_failure – Error handling
  * Attributes_json – attributes to apply to the node at creation

The Chef provisioner will call the bootstrap procedure to install the client on the node. To make this connection it uses the connection option specified within the resource code block.

**Windows:**

{{< highlight hcl >}}
connection {
  type        = "winrm"
  user        = "Administrator"
  password    = "${var.administrator_pw}"
}
{{< / highlight >}}

**Linux:**

{{< highlight hcl >}}
connection {
  type        = "ssh"
  user        = "ec2-user"
  key    = "${file("keypair.pem")}"
}
{{< / highlight >}}

With the correct information, provisioning an instance now allows immediate connection to the configuration management system for well…. Configuration.

It’s important to note that the computer you run the command _terraform apply_ on, needs to remain connected to both the provider and the provisioner for the duration of the process. There is some recovery if the connection drops, but my experience with that has been hit and miss.

##### Additional Steps for Windows Servers

Initially, I had trouble when deploying Windows EC2 instances on AWS due to the random password set at provisioning. I was unable to place that in the connection string To resolve this, I used the user data functionality of EC2 to set the administrator password at creation time.

The password is stored in the variable "${var.administrator_pw}&#8221; which is used in the connection argument reference and within the template used to render the user data file. Together, the administrator password can be set when the instance is provisioned and used for the connection.

To perform the above-mentioned steps in a secure manner, you should have a look at password storage options such as Vault, where you can encrypt the password but retrieve and place into the workflow at runtime.

Another option for domain joined computers is to use a static password, then use a password manager, be it Secret server or MS LAPS to change the password.

##### Summary

Using Terraform to deploy an EC2 instance and have it connect to Chef Server is simple, but it’s a task that requires hands on to get properly.  It’s a great way to spin out test labs which can be destroyed at will and spun up again.

In practice, best results are achieved when the initial Chef run list is minimal and simple. Use logic as part of the first run or another task to apply run lists which are more complex and time-consuming. The reason for this is because if there is an error for the Chef run, the run list will not be applied to the node on your Chef Server.

EDIT: After posting this on the Chef slack channel, [Noah Kantrowitz](https://twitter.com/kantrn) raised the point that using single AWS instances is a bad practice. Instead, use auto scaling groups to prevent issues. This is a very good tip, so here it is 🙂
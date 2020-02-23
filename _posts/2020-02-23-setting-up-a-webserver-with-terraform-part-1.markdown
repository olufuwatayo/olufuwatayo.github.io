---
title: Setting up a webserver with terraform part 1
date: 2020-02-23 12:56:00 Z
tags:
- Terraform
- Aws
- Security groups
- VPC
- Subnet
- Cidr
- Nginx
- Web Server
---

In the last post, we did set up an AWS ec2 instance in terraform but it's just an instance with nothing running on it.

Today we would work on how to set up the ec2 instance with ssh keys, Security group, user data and install NGINX (Install a web server )

To achieve this we need the following components

**VPC**,  Our instance needs to be in a VPC with a subnet and an Internet gateway so we can access it over the internet
**SSH key**, We need this ssh key so we can ssh into our instance using terminal or putty
**Security Group**, This is important as by default all access to our instances are blocked by amazon virtual firewall unless we explicitly allow ports and ip address to access our instance
Userdata. This is the step that installs apps that we need on our instance when it runs for the first time.

**User data** : When you launch an instance in Amazon EC2, you have the option of passing user data to the instance that can be used to perform common automated configuration tasks and even run scripts after the instance starts

\*\*VPC: \*\*

Amazon Virtual Private Cloud ([Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html#default-vpc-components)) lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including a selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.

A default VPC is suitable for getting started quickly, and for launching public instances such as a blog or simple website. You can modify the components of your default VPC as needed.

When you deploy a new terraform instance without specifying the default VPC terraform deploys it in whatever your default VPC would be.

In this case, we would just deploy our instance in a default VPC and create an ssh key for it.

There are two ways to we can launch our instance in a VPC in aws using terraform. We can use the default VPC which comes with all accounts, or we can create our own vpc with all needed components I wrote another article about creating AWS networking components in terraforming if you want to learn how you can check it here

To do that we simply ignore the subnet field when creating the aws instance and terraform would deploy in the default subnet.

\*\*#SSH key \*\*

There are multiple ways to create ssh keys for your ec2 instance in aws using terraform.
You can import your pub key to aws manually from the console or using aws cli
You can generate a key pair and upload your pub key using terraform
You can make terraform create the public key pair and reference it as a variable

As a beginner, you should import an ssh key to AWS and reference it as a key for our AWS ec2 instance.

To generate your ssh key do this on your mac type this command `ssh-keygen` and enter the name of your key-pair, by default your key pairs would be saved to your \`\~/.ssh/id_rsa\`.


Once you have that you can login to aws, click on ec2, click on key pairs and import your key pair. Paste the content of your keyfile that ends with the .pub

![Screenshot 2020-02-22 at 19.47.07.png](/uploads/Screenshot%202020-02-22%20at%2019.47.07.png)

And now we have a key pair in aws that we can reference in terraform. Under the aws_instance terraform resource we can add the key here like this key_name      = "ty"

You can also let terraform  import the public key to aws for you like this and reference it using terraform.

To do this generate your ssh key with this command  `ssh-keygen` and copy the content of the public key to `“public_key = "copy_content_of_public_key_ssh_here”` section of your aws key pair.

{% highlight terraform %}

`resource "aws_key_pair" "my_key" {
key_name   = "my_key_name"
public_key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD3F6tyPEFEzV0LX3X8BsXdMsQz1x2cEikKDEY0aIj41qgxMCP/iteneqXSIFZBp5vizPvaoIR3Um9xK7PGoW8giupGn+EPuxIA4cDM4vzOqOkiMPhz5XK0whEjkVzTo4+S0puvDZuwIsdiW9mxhJc7tgBNL0cYlWSYVkz4G/fslNfRPW5mYAM49f4fhtxPb5ok4Q2Lg9dPKVHO/Bgeu5woMc7RY0p1ej6D4CKFE6lymSDJpW0YHX/wqE9+cfEauh7xZcG0q9t2ta6F6fmX0agvpFyZo8aFbXeUBr7osSCJNgvavWbM/06niWrOvYX2xwWdhXmXSrbX8ZbabVohBK41 email@example.com"
}`

{% endhighlight %}

1. You can use terraform generate the public and private key pair with the help of open ssh and use it as a public key pair but that is beyond the scope of this article

\#security group

Once we have a key pair the next thing is to create a security group.
According to aws definition of [security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) 
A security group acts as a virtual firewall that controls the traffic for one or more instances.
When you launch an instance, you can specify one or more security groups; otherwise, we use the default security group.

To do that we can add this block of codes to terraform, what this does is that it allows all tcp traffic from only our IP address and can allow all outgoing connection from the instance to any ip address
resource "aws_security_group" "allow_from_my_ip" {
name        = "allow_from_my_ip"
description = "Allow all inbound traffic from my ip "
\#vpc_id      = "${aws_vpc.main.id}"

ingress {
from_port   = 0
to_port     = 0
protocol    = "-1"
cidr_blocks = \["100.23.34.12/32"\]  #add your IP address here to get your IP address type curl ifconfig.co in your terminal
}

egress {
from_port       = 0
to_port         = 0
protocol        = "-1"
cidr_blocks     = \["0.0.0.0/0"\] #we want to open the outgoing connections to the world
}
}

\#userdata to install Nginx
sudo apt-get update -y
sudo apt-get istall nginx -y

In the ec2.tf add this block of code below
data "template_file" "myuserdata" {
template = "${file("${path.cwd}/myuserdata.tpl")}"
}

This is calling a data resource called template_file and we assigned the resource name myuserdata

The location of the file is "path.cwd" which is current working directory / file name  with the extension

terraform would load this file up and parse it as an ec2 userdata. You can also use cloud-init but for the sake of eimplicity let us use userdata.

Create a new file named `myuserdata.tpl` and paste this shell script
This is just a simple shell script that does two things update and install nginx webserver
\#!/bin/bash
sudo apt-get update -y
sudo apt-get install nginx -y

After this we can launch our ec2 insatnce with an nginx web server running on it.

Lastly we would need to get the public ip address of out instance using output variable
When building potentially complex infrastructure, Terraform stores hundreds or thousands of attribute values for all your resources. But as a user of Terraform, you may only be interested in a few values of importance, such as a load balancer IP, VPN address, etc.

Outputs are a way to tell Terraform what data is important. This data is outputted when apply is called, and can be queried using the terraform output command.

create a file called outputs.tf

output "public_ip" {
description = "List of public IP addresses assigned to the instances, if applicable"
value       = aws_instance.web.\*.public_ip
}

run terraform init
terraform plan
terraform apply
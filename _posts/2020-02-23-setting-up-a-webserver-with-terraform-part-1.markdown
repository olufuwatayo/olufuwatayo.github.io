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

[In the last post](http://www.olufuwatayo.com/blog/Introduction-to-IAC-with-terraform/), we did set up an AWS ec2 instance in terraform but it's just an instance with nothing running on it.

Today we would work on how to set up the ec2 instance with ssh keys, Security group, user data and install NGINX (Install a web server )

To achieve this we need the following components

* **[VPC](https://aws.amazon.com/vpc/)**,  Our instance needs to be in a VPC with a subnet and an Internet gateway so we can access it over the internet

* **[SSH key](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)**, We need this ssh key so we can ssh into our instance using terminal or putty

* **[Security Group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)**, This is important as by default all access to our instances are blocked by amazon virtual firewall unless we explicitly allow ports and ip address to access our instance

* **[User data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)** : When you launch an instance in Amazon EC2, you have the option of passing user data to the instance that can be used to perform common automated configuration tasks and even run scripts after the instance starts

**VPC: **

Amazon Virtual Private Cloud ([Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html#default-vpc-components)) lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including a selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.

A default VPC is suitable for getting started quickly, and for launching public instances such as a blog or simple website. You can modify the components of your default VPC as needed.

![Screenshot 2020-02-23 at 11.02.24.png](/uploads/Screenshot%202020-02-23%20at%2011.02.24.png)

When you deploy a new terraform instance without specifying the default VPC terraform deploys it in whatever your default VPC would be.

In this case, we would just deploy our instance in a default VPC and create an ssh key for it.

There are two ways to we can launch our instance in a VPC in aws using terraform. We can use the default VPC which comes with all accounts, or we can create our own vpc with all needed components I wrote another article about creating AWS networking components in terraforming if you want to learn how you can check it here

To do that we simply ignore the subnet field when creating the aws instance and terraform would deploy in the default subnet.
Create a new file named ec2.tf and add this block of code


{% highlight terraform %}
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-trusty-14.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["099720109477"] # Canonical
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  key_name      = "ty"
  security_groups   = ["${aws_security_group.allow_from_my_ip.name}"]
  user_data = data.template_file.myuserdata.template
  

  tags = {
    Name = "Nginx_web_server"
  }
}

{% endhighlight %}

**#SSH key **

There are multiple ways to create ssh keys for your ec2 instance in aws using terraform.

* You can import your pub key to aws manually from the console or using aws cli

* Generate a key pair and upload your pub key using terraform

* Make terraform create the public key pair and reference it as a variable

As a beginner, you should import an ssh key to AWS and reference it as a key for our AWS ec2 instance.

To generate your ssh key do this on your mac type this command `ssh-keygen` and enter the name of your key-pair, by default your key pairs would be saved to your `~/.ssh/id_rsa`.

![Screenshot 2020-02-21 at 14.05.24.png](/uploads/Screenshot%202020-02-21%20at%2014.05.24.png)

Once you have that you can login to aws, click on ec2, click on key pairs and import your key pair. Paste the content of your keyfile that ends with the .pub

![Screenshot 2020-02-22 at 19.47.07.png](/uploads/Screenshot%202020-02-22%20at%2019.47.07.png)

And now we have a key pair in aws that we can reference in terraform. Under the aws_instance terraform resource we can add the key here like this key_name      = "ty"
This line in the aws_instance specifies the public we want to attach to the instance.
{% highlight terraform %}
key_name      = "ty"
{% endhighlight %}

You can also let terraform import the public key to AWS for you like this and reference it using terraform.

To do this generate your ssh key with this command  `ssh-keygen` and copy the content of the public key to `“public_key = "copy_content_of_public_key_ssh_here”` section of your AWS key pair.

{% highlight terraform %}
resource "aws_key_pair" "name_of_pub_key" {
  key_name   = "ty"
  public_key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD3F6tyPEFEzV0LX3X8BsXdMsQz1x2cEikKDEY0aIj41qgxMCP/iteneqXSIFZBp5vizPvaoIR3Um9xK7PGoW8giupGn+EPuxIA4cDM4vzOqOkiMPhz5XK0whEjkVzTo4+S0puvDZuwIsdiW9mxhJc7tgBNL0cYlWSYVkz4G/fslNfRPW5mYAM49f4fhtxPb5ok4Q2Lg9dPKVHO/Bgeu5woMc7RY0p1ej6D4CKFE6lymSDJpW0YHX/wqE9+cfEauh7xZcG0q9t2ta6F6fmX0agvpFyZo8aFbXeUBr7osSCJNgvavWbM/06niWrOvYX2xwWdhXmXSrbX8ZbabVohBK41 email@example.com"
}
{% endhighlight %}



You can also use terraform generate the public and private key pair with the help of open ssh and use it as a public key pair but that is beyond the scope of this article. To do that you can use this block of code but this is not what we are planning to do today.

{% highlight terraform %}
variable "key_name" {}

resource "tls_private_key" "example" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "generated_key" {
  key_name   = "${var.key_name}"
  public_key = "${tls_private_key.example.public_key_openssh}"
}

data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-trusty-14.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["099720109477"] # Canonical
}

resource "aws_instance" "web" {
  ami           = "${data.aws_ami.ubuntu.id}"
  instance_type = "t2.micro"
  key_name      = "${aws_key_pair.generated_key.key_name}"

  tags {
    Name = "HelloWorld"
  }
}
{% endhighlight %}


[Part 2 here ](http://www.olufuwatayo.com/setting-up-a-webserver-with-terraform-part-2/)

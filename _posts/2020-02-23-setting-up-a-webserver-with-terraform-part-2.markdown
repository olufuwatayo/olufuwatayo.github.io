---
title: Setting up a webserver with terraform part 2
date: 2020-02-23 16:49:00 Z
---

In the [previous section](http://www.olufuwatayo.com/setting-up-a-webserver-with-terraform-part-1/), we created our [AWS keypair](http://www.olufuwatayo.com/setting-up-a-webserver-with-terraform-part-1/) now we would attach it to our instance.

Once we have a key pair the next thing is to create a security group.
According to aws definition of [security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) , A security group acts as a virtual firewall that controls the traffic for one or more instances.
When you launch an instance, you can specify one or more security groups; otherwise, we use the default security group.

![Screenshot 2020-02-23 at 11.09.38.png](/uploads/Screenshot%202020-02-23%20at%2011.09.38.png)

To do that we can add this block of codes to terraform, what this does is that it allows all tcp traffic from only our IP address and can allow all outgoing connection from the instance to any ip address. To get your public IP address just type this in the terminal `curl ipconfig.co`

Create a new file called **sg.tf**
{% highlight terraform %}

resource "aws_security_group" "allow_from_my_ip" {
name        = "allow_from_my_ip"
description = "Allow all inbound traffic from my ip "
#vpc_id      = "${aws_vpc.main.id}"

ingress {
from_port   = 0
to_port     = 0
protocol    = "-1"
cidr_blocks = ["100.34.34.34/32"]  #add your IP address here to get your IP address type curl ifconfig.co in your terminal
}

egress {
from_port       = 0
to_port         = 0
protocol        = "-1"
cidr_blocks     = ["0.0.0.0/0"] #we want to open the outgoing connections to the world
}
}

{% endhighlight %}

To set up the Nginx Webserver create a file called myuserdata.tpl and paste this shell script
This is just a simple shell script that does two things update your repo list and install nginx webserver
`#!/bin/bash
sudo apt-get update -y
sudo apt-get install nginx -y`

In the ec2.tf add this block of code below so we can fetch our user data template and let AWS run it at launch time.
{% highlight terraform %}
data "template_file" "myuserdata" {
template = "${file("${path.cwd}/myuserdata.tpl")}"
}
{% endhighlight %}
This is calling a data resource called template_file and we assigned the resource name myuserdata

The location of the file is "path.cwd" which is current working directory/filename with the extension tpl. Terraform would load this file up and parse it as an ec2 user data. You can also use cloud-init but for the sake of simplicity let us use user data.


After this, we can launch our ec2 instance with an Nginx web server running on it.

Lastly, we would need to get the public ip address of our instance using the output variable here is a definition from terraform about output variables

> When building potentially complex infrastructure, Terraform stores hundreds or thousands of attribute values for all your resources. But as a user of Terraform, you may only be interested in a few values of importance, such as a load balancer IP, VPN address, etc.

Outputs are a way to tell Terraform what data is important. This data is outputted when apply is called, and can be queried using the terraform output command.

create a file called outputs.tf and paste this here
{% highlight terraform %}
output "public_ip" {
description = "List of public IP addresses assigned to the instances, if applicable"
value       = aws_instance.web.*.public_ip
}
{% endhighlight %}

run 
`terraform init`
![terraform init.png](/uploads/terraform%20init.png)

`terraform plan`
![terraform plan.png](/uploads/terraform%20plan.png)

`terraform apply`
![Terraform apply.png](/uploads/Terraform%20apply.png)

We have our web server up and running here.
![Nginx server up and running.png](/uploads/Nginx%20server%20up%20and%20running.png)


Don't forget to do a terraform destroy so you can destroy the infrastructure you just created if you don't need them anymore.
`terraform destroy`
![Terraform destroy.png](/uploads/Terraform%20destroy.png)
You can get the whole source codes here 
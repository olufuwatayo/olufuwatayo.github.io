---
title: IAAC - Terraform with Ansible
date: 2020-02-29 22:52:00 Z
---

In our last post, we learned the basics of Ansible and how we set up a basic web server instance with Ansible. Today we would be exploring how we can automate the whole process of creating our infrastructure with terraform and managing it with ansible.

If you are new to terraform you can read the introduction here and also spin up a simple web server here, Today we would be using terraform to create 5 ec2 instances and have ansible installs Nginx on the instances all automated.

There are so many ways we can achieve this by using ansible

We can use **terraform local exec**:
The local-exec provisioner invokes a local executable after a resource is created. This invokes a process on the machine running Terraform, not on the resource. See the remote-exec provisioner to run commands on the resource.


To do that we would add this block to our terraform resource code and we would reference our Ansible playbook.
{% highlight terraform %}

  provisioner "local-exec" {
    command = "sleep 120; ansible-playbook -u ubuntu -i '${self.public_ip},' main.yml"
  }

{% endhighlight %}
What this does is it sleeps for 120 secs and wait for the instance to boot before running ansible locally. We need the ansible-playbook file and the folders to run ansible in the same folder as the terraform config files (although you can add it in another folder and still reference it). 
{% highlight terraform %}

aws_instance.web[0] (local-exec): Executing: ["/bin/sh" "-c" "sleep 120; ansible-playbook -u ubuntu -i '54.166.193.27,' main.yml"]
aws_instance.web[1]: Provisioning with 'local-exec'...
aws_instance.web[1] (local-exec): Executing: ["/bin/sh" "-c" "sleep 120; ansible-playbook -u ubuntu -i '54.80.114.58,' main.yml"]

{% endhighlight %}

You can get the full script here [https://github.com/olufuwatayo/terraform-ansible-local-exec](https://github.com/olufuwatayo/terraform-ansible-local-exec)
**Remote exec:**

The remote-exec provisioner invokes a script on a remote resource after it is created. We would use this to run the ansible playbook on the instance after it has been created. 


To do this we need to copy the files and folder to the instance and run ansible-playbook right inside the instance 

{% highlight terraform %}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  // count = 2
  key_name        = "ty"
  security_groups = ["${aws_security_group.allow_from_my_ip.name}"]


  provisioner "file" {
    source      = "." #the source is the current working directory
    destination = "~"

    connection {
      type = "ssh"
      user = "ubuntu"

      host = "${self.public_ip}"
    }
  }
  provisioner "remote-exec" {
    inline = ["sudo apt-get -y install python"] # We would install python because ansible needs python to run
    connection {
      type        = "ssh"
      user        = "ubuntu" # Username is ubuntu 
      host        = "${self.public_ip}"
      private_key = file("~/.ssh/id_rsa") #Terraform needs your private key so it can connect to your instance and of course run the commands
    }
  }


  provisioner "remote-exec" {
    inline = ["sudo apt-get -y install ansible"] # We would install ansible because ansible needs to run our playbooks

    connection {
      type        = "ssh"
      host        = "${self.public_ip}"
      user        = "ubuntu"              # Username is ubuntu 
      private_key = file("~/.ssh/id_rsa") #Terraform needs your private key so it can connect to your instance and of course run the commands
    }
  }


  provisioner "remote-exec" {
    inline = ["sudo ansible-playbook -i hosts.ini main.yml"] # run the playbook

    connection {
      type        = "ssh"
      host        = "${self.public_ip}"
      user        = "ubuntu"              # Username is ubuntu 
      private_key = file("~/.ssh/id_rsa") #Terraform needs your private key so it can connect to your instance and of course run the commands
    }
  }

  provisioner "local-exec" {
    command = <<EOT
      sleep 30;
	  echo "${aws_instance.web.public_ip}" | tee -a hosts.ini;
    EOT
  }

  tags = {
    Name = "Nginx_web_server"
  }
}

{% endhighlight %}

**User data** We can use Userdata  to install, python, ansible, clone the ansible repo and then make ansible run locally in the 

It's simple as adding this to your [user data file ](https://github.com/olufuwatayo/terraform-aws-ec2-nginx/blob/master/myuserdata.tpl)

{% highlight bash %}

#!/bin/bash
sudo apt-get update -y
sudo apt-get install ansible -y
sudo apt-get install ansible -y 
#git clone repo link here
#cd into repo link
#execute playbook 

{% endhighlight %}


Finally you can use [terraform-inventory](https://github.com/adammck/terraform-inventory) a GitHub module that actually fetches your terraform state file and uses the value IP addresses of the instances as ansible-playbook host inventory.   

#we can create a packer file that creates the ami and we can use the ami in terraform.
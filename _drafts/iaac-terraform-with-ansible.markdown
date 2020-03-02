---
title: IAAC - Terraform with Ansible
date: 2020-02-29 22:52:00 Z
---

In our last post, we learned the basics of Ansible and how we set up a basic instance with Ansible. Today we would be exploring how we can automate the whole process of creating our infrastructure with terraform and managing it with ansible.

If you are new to terraform you can read the introduction here and also spin up a simple web server here, Today we would be using terraform to create 5 ec2 instances and have ansible installs Nginx on the instances.

To do that first create your terra


#local Exec 
The local-exec provisioner invokes a local executable after a resource is created. This invokes a process on the machine running Terraform, not on the resource.

resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "sleep 120;  echo ${aws_instance.web.private_ip} >> private_ips.txt"
  }
}

#userdata to run ansible

#terraforming 
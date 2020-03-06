---
title: IAAC - Terraform with Ansible
date: 2020-02-29 22:52:00 Z
published: false
---

In our last post, we learned the basics of Ansible and how we set up a basic web server instance with Ansible. Today we would be exploring how we can automate the whole process of creating our infrastructure with terraform and managing it with ansible.

If you are new to terraform you can read the introduction here and also spin up a simple web server here, Today we would be using terraform to create 5 ec2 instances and have ansible installs Nginx on the instances all automated.

There are so many ways we can achieve this by using ansible

We can use **terraform local exec**:
The local-exec provisioner invokes a local executable after a resource is created. This invokes a process on the machine running Terraform, not on the resource. See the remote-exec provisioner to run commands on the resource.


To do that we would add this block to our terraform resource code and we would reference our Ansible playbook.

  provisioner "local-exec" {
    command = "sleep 120; ansible-playbook -u ubuntu -i '${self.public_ip},' main.yml"
  }

What this does is it sleeps for 120 secs and wait for the instance to boot before running ansible locally. We need the ansible-playbook file and the folders to run ansible in the same folder as the terraform config files (although you can add it in another folder and still reference it). 

aws_instance.web[0] (local-exec): Executing: ["/bin/sh" "-c" "sleep 120; ansible-playbook -u ubuntu -i '54.166.193.27,' main.yml"]
aws_instance.web[1]: Provisioning with 'local-exec'...
aws_instance.web[1] (local-exec): Executing: ["/bin/sh" "-c" "sleep 120; ansible-playbook -u ubuntu -i '54.80.114.58,' main.yml"]


You can get the full script here [https://github.com/olufuwatayo/terraform-ansible-local-exec](https://github.com/olufuwatayo/terraform-ansible-local-exec)
**Remote exec:**

The remote-exec provisioner invokes a script on a remote resource after it is created. This can be used to run a configuration management tool, bootstrap into a cluster, etc

provisioner "remote-exec" {
    inline = ["sudo dnf -y install python"]

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = "${file(var.ssh_key_private)}"
    }
  }

  provisioner "local-exec" {
    command = "ansible-playbook -u ubuntu -i '${self.public_ip},'  ${var.ssh_key_private} provision.yml" 
  }

this would translate to 
**User data** We can use Userdata  to install, python, ansible, clone the ansible repo and then make ansible run locally in the 

To do that let us create the infrastructure using terraform 


#local Exec 
The local-exec provisioner invokes a local executable after a resource is created. This invokes a process on the machine running Terraform, not on the resource.

resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "sleep 120;  echo ${aws_instance.web.private_ip} >> private_ips.txt"
  }
}


#terraforming there is a GitHub module that actually fetches your terraform state file and fetches the IP addresses of the instances and 

#we can create a packer file that creates the ami and we can use the ami in terraform.
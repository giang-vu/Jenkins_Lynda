# 1. Launch Jenkins Master
https://github.com/giang-vu/Jenkins_Lynda/tree/master/jenkins_nginx_aws#launch-jenkins-master

# 2. Launch Agent
Use bootstrap script to install Docker and Java 8 on Amazon AMI EC2.
```
#!/bin/bash
sudo yum update -y
sudo yum install -y docker
sudo yum install -y openjdk-8-jdk
sudo service docker start
sudo chkconfig docker on
```
Add new user and configure SSH
```
# sudo su -
# useradd jenkins
# usermod -a -G docker jenkins
# su jenkins -
# mkdir .ssh
# chmod 700 .ssh
# vim .ssh/authorized_keys

<public key>

# chmod 600 .ssh/authorized_keys
```
Add a new node:
- Name: docker-agent
- Remote root directory: /home/jenkins
- Labels: docker-agent
- Launch agent agents via SSH
- Host: <agent IP address>
- Add new SSH credential with private key, username: enkins
- Non verifying verification strategy

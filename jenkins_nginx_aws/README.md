# 1. Setup Jenkins
```
+------+         +---------+         +---------+
|      |         | NGINX   |         |         |
| User +---------> reverse +---------> Jenkins |
|      |         | proxy   |         |         |
+------+         +---------+         +---------+
```
# Launch Jenkins Master
Use bootstrap script to install Java 8, NGINX, Jenkins
```
#!/bin/bash
sudo wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update -y
sudo apt-get install -y openjdk-8-jdk
sudo apt-get install -y nginx
sudo apt-get install -y jenkins 
```
# Configure NGINX
NGINX will be the reverse proxy server in front of Jenkins web app
```
# sudo su -
# unlink /etc/nginx/sites-enabled/default
# vim /etc/nginx/conf.d/jenkins.conf

upstream jenkins {
	server 127.0.0.1:8080;
}

server {
	listen 80 default_server;
	listen [::]:80 default_server;
	location / {
		proxy_pass http://jenkins;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
	}
}

# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
# systemctl restart nginx
```
# Configure Simple Email Service (SES)
SES is supported only in 3 regions: US East (N. Virginia), EU (Ireland) and US West (Oregon)
# Create SMTP credentials for SES
Copy the SMTP credentials from AWS to 2 Jenkins sections: E-mail Notification (basic functionality for Jenkins to send email) and Extended E-mail Notification (plug-in that gives Jenkins jobs additional functionality for sending email)
- System Admin e-mail address
- SMTP server email-smtp.us-east-1.amazonaws.com
- Use SSL
- SMTP Port 465
# 2. Create build environment
```
+---------+         +------+         +---------+         +-----+         +---------------+
|         |         |      |         |         |         |     |         |               |
| Jenkins +---------> Key  +---------> Build   +---------> IAM +---------> Resources     |
|         |         | Pair |         | Servers |         |     |         | (S3, EC2, EB) |
|         |         |      |         |         |         |     |         |               |
+---------+         +------+         +---------+         +-----+         +---------------+
```
# Create Build Server's IAM Role, Security Group and Key Pair
IAM Role for EC2 with AWSElasticBeanstalkFullAccess policy. Security Group with only SSH from Jenkins Security Group
# Add Jenkins global credentials
- SSH username with private key
- ID build-servers
- Username ec2-user
- Private key from the Key Pair
# Launch Build Servers
Use Amazon AMI, Build Server's IAM Role, Security Group, Key Pair and bootstrap script
```
#!/bin/bash
yum update -y
yum install -y java-1.8.0-openjdk
yum install -y git
/user/bin/easy_install awsebcli
```
# Add new Jenkins node and connect to the build servers
We should use the private IP (or private DNS name) to keep traffic inside the AWS network. This will reduce costs associated with network ingress and egress and also keep the connection fast
- Remote root directory /home/ec2-user (the directory that Jenkins uses on the build servers to create workspaces)
- Launch slave agents via SSH
- Host (private IP or private DNS name of the build servers)
- Credentials ec2-user(build-servers)
- Non verifying verification strategy

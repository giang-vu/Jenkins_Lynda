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
Add new user and configure SSH.
```
# sudo su -
# useradd jenkins
# usermod -a -G docker jenkins
# su jenkins -
# mkdir .ssh
# chmod 700 .ssh
# vim .ssh/authorized_keys

[public key]

# chmod 600 .ssh/authorized_keys
```
Enable Docker Remote API by edit the ExecStart line in /lib/systemd/system/docker.service.
```
# vim /lib/systemd/system/docker.service

ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock

# systemctl daemon-reload
# service docker restart
# curl http://localhost:4243/version
[The output should be in JSON format]
```
For Ubuntu:
```
ExecStart=/usr/bin/docker daemon -H fd:// -H tcp://0.0.0.0:4243
```

# 3. Add new node and Docker plugin on Jenkins Master
Add new node:
- Name: docker-agent
- Remote root directory: /home/jenkins
- Labels: docker-agent
- Launch agent agents via SSH
- Host: [agent IP address]
- Add new SSH credential with private key, username: enkins
- Non verifying verification strategy

Add Docker plugin:
- Install Docker plugin.
- Add Jenkins system Cloud - Docker
- Docker Host URI: tcp://[agent IP address]:4243
- Test connection should return API version
- Check Enabled

# 4. Test by pipeline
Create a pipeline project.
```
pipeline {
    agent {
        docker {
            label 'docker-agent' //specify the agent by labels
            image 'node:9-alpine'
        }
    }
    stages {
        stage('Test'){
            steps {
                sh 'node --version'
            }
        }
    }
}
```
Check log to see the version output.
```

# 1. Bootstrap script
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update -y
sudo apt-get install -y openjdk-8-jdk
sudo apt-get install -y nginx
sudo apt-get install -y jenkins 
```

# 2. Configure NGINX
NGINX will be the reverse proxy server in front of Jenkins web app
```
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
```

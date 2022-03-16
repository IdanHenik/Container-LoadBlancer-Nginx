
# Nginx LB as Contnainer


![hi](https://i.postimg.cc/5NQbNCkD/Whats-App-Image-2022-03-16-at-1-33-47-PM-removebg-preview.png)

In this project i will show you how to create a contnainer based an Nginx image and configure it as Load Blancer.
In this example its funcation as load balancer between two apache servers.



# Project Architecture
![architecture](https://i.postimg.cc/kgRzKHTx/Ngnix-LB.png)

# Requirements 
- 2 Virtual Machines
- another host for Docker Contnainer

# Installing apache
our first step will be to install apache on both of the virtual machines, I precisely choose to use diffrent OS (Ubuntu & CentOS) beacause each of them has diffrent apache default page so we could see the load balancer at work 
. Okay we talked enough let's start !

- CentsOS:
First clean up yum:
```bash
 sudo yum clean all
```
then update yum:
```bash
  sudo yum update
```
now we ready to install Apache:
```bash
  sudo yum install httpd 
```
- Ubuntu:
First apt update :
```bash
  sudo apt update
```
then install Apache:
```bash
  sudo apt install apache2
```
so now we got Apache installed on our VM's and ready to the next step :)

# Firewall/UFW
in order to use the apache as a web server (http & https) we need to allow it in the firewall-cmd or ufw depends in the os you are using, lucky I will show them both.
- CentOS:
to allow http service:
```bash
  sudo firewall-cmd --permanent --add-port=80/tcp
```
to allow https service:
```bash
  sudo firewall-cmd --permanent --add-port=443/tcp
```
now we MUST to reload to firewall:
```bash
  sudo firewall-cmd --reload
```
-Ubuntu:
 all we need to do in Ubuntu is:
 ```bash
  sudo ufw allow 'Apache Full'
```
and it will open in the ufw port 80 and port 443
we can check ufw statu to be sure
```bash
  sudo ufw status
```
# Http Request
so now we handled the firewall/ufw if we send http to the vm's address we should see two diffrent html pages
in my case :
```bash
  http://192.168.1.135
  http://192.168.1.119
```
Result : ![resualt](https://i.postimg.cc/65VyjLJL/Slide2.png)
![result2](https://i.postimg.cc/5yBNr1GN/Slide3.png)

Great ! now we got our apache servers ready.
all we need to do is create the Nginx contnainer and configure it as Load Blancer :)

# Docker Container
for the next step we need docker installed in our enviroment so make sure you have it.
if you don't have it just enter the single command:
- For RHEL/CentOS:
```bash
  sudo yum install docker -y
```
- For Ubuntu:
```bash
  sudo apt install docker -y
```
 # Nginx 
so the first step is to make a dedicated directory :
```bash
  mkdir nginx-container
```
then we need to create a configuration file so it funcation as load balancer :
```bash 
cd nginx-container
vim nginx.conf
```
we are going to write it down from Scartch so pay attention:
```bash
http {
        upstream all {
            server 192.168.1.135:80;
            server 192.168.1.119:80;
}
server {
            listen 80;
            location / {
                         proxy_pass http://all/;
                         }
             }
}
events {}
```
and then `:wq`

the next step is to create our dockerfile :
```bash
  vim dockerfile
```

```bash
  FROM nginx
  COPY nginx.conf /etc/nginx/nginx.conf
```
- the dockerfile is going to pull Nginx offical image then overwrite the configuration with the new one we wrote !

And now we have our dockerfile ready.
# Ngnix-Container
well here we our at the final step all we left to do it to build the docker image and run the container.
so..first step build :
```bash
  sudo docker build -t Ngnixlb .
```
- you can change `Ngnixlb` if you like to it's the container name.
second and final step !
```bash
  sudo docker run -p 80:80 Ngnixlb .
```
- `80:80` stands for the host:container port you can change it but you MUST change it in the configuration also.

We done, apache servers is on and the nginx running as a containerzed load balancer

If it works you should enter the ip of the machine thats running the container and if you hit refresh few times you will be able to see the diffrent html pages wich means IT WORKS !

![final](https://i.postimg.cc/596D5vkP/Slide4.png)

![fial](https://i.postimg.cc/prffQsNh/Whats-App-Image-2022-03-16-at-17-23-25-removebg-preview.png)



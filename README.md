# Load-Balancer_Solution_With_Nginx


- By now we have learned what Load Balancing is used for and have configured an LB solution using Apache, but a DevOps engineer must be a versatile professional and know different alternative solutions for the same problem. That is why, in this project we will configure an Nginx Load Balancer solution.
 - It is also extremely important to ensure that connections to  Web solutions are secure and information is encrypted in transit - we will also cover connection over secured HTTP (HTTPS protocol), its purpose and what is required to implement it. 
- When data is moving between a client (browser) and a Web Server over the Internet - it passes through multiple network devices and, if the data is not encrypted, it can be relatively easy intercepted by someone who has access to the intermediate equipment. This kind of information security threat is called Man-In-The-Middle (MIMT) attack.
- This threat is real - users that share sensitive information (bank details, social media access credentials, etc.) via non-secured channels, risk their data to be compromised and used by cybercriminals.
- SSL and its newer version, TSL - is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS - even though SSL was replaced by TLS, the term is still being widely used.
- SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.
- There are different types of SSL/TLS certificates - we can learn more about them here. We can also watch a tutorial on how SSL works here or an additional resource hereIn this project you will register website with LetsEnrcypt Certificate Authority, to automate certificate issuance we will use a shell client recommended by LetsEncrypt - cetrbot.

## Task
This project consists of two parts:
1. Configure Nginx as a Load Balancer.
2. Register a new domain name and configure secured connection using SSL/TLS certificates.

Target architecture will look like this:

![image](https://github.com/user-attachments/assets/ea5312f0-30c0-4184-87e5-fea548778276)

## Part 1 - Configure Nginx As A Load Balancer:

1. Create an EC2 VM based on Ubuntu Server 24.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 - this port is used for secured HTTPS connections).

![image](https://github.com/user-attachments/assets/406d1ccb-a73b-4b0c-bf7a-01f8b464bbaf)

![image](https://github.com/user-attachments/assets/bcacd6d4-ad3b-4d42-84b3-c47e647f1967)

2. Update /etc/hosts file for local DNS with Web Servers' names (e.g. Web1 and Web2) and their local IP addresses

```
sudo vi /etc/hosts
```
![image](https://github.com/user-attachments/assets/3c260041-9380-4f85-ab08-5321586ebea3)


3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers.
   
Update the instance and Install Nginx Install Nginx
```
sudo apt update
```
![image](https://github.com/user-attachments/assets/2ea0b197-d2f6-438a-acd6-e465adc76a4c)

```
sudo apt install nginx -y
```
![image](https://github.com/user-attachments/assets/6e26d944-9a4c-4478-8091-68c9183c171e)

Open the default nginx configuration file

```
sudo vi /etc/nginx/nginx.conf
```
```
http {
    # Other configurations
    
    upstream myproject {
        server Web1 weight=5;
        server Web2 weight=5;
    }

    server {
        listen 80;
        server_name www.domain.com;

        location / {
            proxy_pass http://myproject;
        }

        # Comment out the following line
        # include /etc/nginx/sites-enabled/*;
    }
}

```
Replace your already purchased domain name in the section for domain name, save and exit

![image](https://github.com/user-attachments/assets/d6baa79d-557b-404d-bfe1-d2baab29aba0)


```
sudo systemctl restart nginx
sudo systemctl status nginx
```

## Part 2 - Register a new domain name and configure secured connection using SSL/TLS certificates
- Let us make necessary configurations to make connections to our Tooling Web Solution secured.
- !In order to get a valid SSL certificate - you need to register a new domain name, you can do it using any Domain name registrar - a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Domain.com, Bluehost.com.Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)

1. Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)

2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP.

![image](https://github.com/user-attachments/assets/f2676e63-c75f-43ad-930e-a0dc69009aa1)

![image](https://github.com/user-attachments/assets/d2cd4007-ff44-4779-911b-63f4e77828f2)

We might have noticed, that every time we restart or stop/start EC2 instance - We get a new public IP address. When we want to associate we domain name - it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem, learn how to allocate an Elastic IP and associate it with an EC2 server on this page.

3. Update A record in registrar to point to Nginx LB using Elastic IP address.


Adding records inside the Hosted Zone;

![image](https://github.com/user-attachments/assets/25e62c18-3bcc-448d-97ff-8f1fbdccd737)

4. Configure Nginx to recognize your new domain name.

```
sudo vi /etc/nginx/nginx.conf
```
![image](https://github.com/user-attachments/assets/29eaca5b-d29e-47a4-84f8-32abb56b52b3)

After do configuration insdie the nginx.conf need to check the syntax

```
sudo nginx -t
```
Reload the changes.

```
sudo systemctl reload nginx
```

Accessing the domain on web-browser;

![image](https://github.com/user-attachments/assets/a8104246-13fd-454e-8c45-36e5d9b6b623)

5. Install certbot and request for an SSL/TLS certificate.

- Ensure 'snapd' service is active and running

```
sudo systemctl status snapd
```

![image](https://github.com/user-attachments/assets/2f6af894-e055-4276-9dd3-8461f80aa917)

- Install certbot using snapd package manager.
```
 sudo snap install --classic certbot
```

- Create a symlink for certbot.

````
     sudo ln -s /snap/bin/certbot /usr/bin/certbot
````

- Follow the prompt to configure and request for ssl certificate.
```
sudo certbot --nginx
```
![image](https://github.com/user-attachments/assets/5cc7c397-e753-4de9-a350-4d8a435d2bbd)

Visit your website to confirm SSL has been successfully installed



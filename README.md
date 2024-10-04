---

# AWS-EC2-Hosting-Guide

This repository provides a step-by-step guide to deploying and hosting backend applications on AWS EC2 using the t2.micro Free Tier. It covers setting up an instance, configuring security groups, SSH access, and deploying an Express.js app, along with tips for managing and securing your EC2 instance.

## Step 01

Launch a new EC2 instance using the t2.micro Free Tier.
Choose the Ubuntu Server as your AMI (Amazon Machine Image).
Download and securely store the .pem file for SSH access.

- Open your instance -> Connect -> SSH Client
- Connect to your instance using its Public DNS. Copy the command (e.g., `ec2-0-000-000-00.ap-southeast-2.compute.amazonaws.com`).

## Step 02

Connect to your instance using the Public DNS through the command prompt.

- Open the directory where your `.pem` file is located through cmd. (e.g., `cd Downloads`)
- Use the SSH command provided by AWS to connect.

## Step 03

Update all dependencies and packages for your virtual machine.

```bash
sudo apt update
sudo apt upgrade -y
```

## Step 04

Install necessary dependencies: Node.js, npm, and Git.

```bash
sudo apt install nodejs npm git -y
```

Add the Node.js 18.x repository and install it:

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```

Check the versions of the installed dependencies to ensure everything is installed correctly:

```bash
node -v
npm -v
git -v
```

## Step 05

Clone your GitHub repository to the virtual machine, navigate to the correct directory, and install dependencies (Because `node_modules` is not pushed to GitHub).

```bash
git clone https://github.com/Sameen-K-A/Instant-Fix.git
cd Instant-Fix
npm install
```

Ensure that `node_modules` is installed successfully.

## Step 06

Set up your environment variables in the virtual machine.

```bash
nano .env
```

Enter or copy-paste all your environment credentials into the file, then save.

- Press **Ctrl + X**
- Press **Y** to confirm
- Press **Enter**

Ensure your `.env` file is saved successfully:

```bash
cat .env
```

## Step 07

Edit your **AWS Inbound Rules**:

- Go to the instance **Security** tab -> Click on **Security Groups** -> **Edit Inbound Rules**
- Add a rule: (Type: Custom TCP Rule, Port Range: your_portNumber, Source: 0.0.0.0/0) 
- Add another rule: (Type: Custom TCP Rule, Port Range: your_portNumber, Source: ::/0)
- Save the rules

## Step 08

Install PM2 to ensure your application keeps running even if there's a crash or your shell disconnects.

```bash
sudo npm install -g pm2
pm2 --version
```

Start your application:

```bash
pm2 start app.js
```

Other PM2 commands:

- Check app status: `pm2 status`
- Restart the app: `pm2 restart app`
- Stop the app: `pm2 stop app`
- View logs: `pm2 logs`

Check if your application is running by visiting:

```bash
http://Your_instance_publicIP:port_number
```

(e.g., `http://x.xxx.xxx.xx:3000`)

Now your application running successfully 

![](https://i.pinimg.com/originals/73/cc/a4/73cca45a93f91944b2c9fdd4b05c3c53.gif)

## Step 09

Purchase a free or paid domain name (GoDaddy, Hostinger, etc.).

Manage your domain DNS and add 2 **A Records**:

- `{Name: @, Target: Your public IP, TTL: 30 min}`
- `{Name: www, Target: Your public IP, TTL: 30 min}`

## Step 10

Install and configure NGINX:

```bash
sudo apt install nginx
```

Navigate back to the root directory and list the files:

```bash
cd ..
ls
```

Open and edit the NGINX configuration:

```bash
sudo nano /etc/nginx/sites-available/default
```

Set up your server name and location:

```nginx
server_name yourdomainname.com www.yourdomainname.com;

location / {
    proxy_pass http://localhost:your_portNumber;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
}
```

Test and restart NGINX:

```bash
sudo nginx -t
sudo nginx -s reload
```

Make sure your NGINX setup is successful. Open a new tab in your browser, and enter your instance's public IP.

## Step 11

Add an SSL certificate for HTTPS secure communication between the client and server. SSL is applied using Certbot.

```bash
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python3-certbot-nginx
```

Run Certbot to set up the SSL certificate:

```bash
sudo certbot --nginx -d domainname.com -d www.domainname.com
```

Certbot SSL certificates expire after 90 days. To ensure it auto-renews

```bash
sudo certbot renew --dry-run
```
---

# Conclusion

Thank you for following this guide on hosting a Node.js application on AWS EC2. I hope it has helped you deploy your application smoothly and understand the basic configurations required. If you encounter any issues or have any suggestions for improvement, feel free to raise an issue or contact me.

If you found this helpful, don't forget to give the repository a ⭐ on GitHub!

Happy coding!

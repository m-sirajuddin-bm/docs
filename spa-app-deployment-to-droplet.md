# SPA App Deployment to Digital Ocean Droplet

I will be explaining how to deploy Vue3 SPA app to digital ocean droplet.

## Pre-requisites

Before proceeding, make sure you have completed the following tutorials:
- [Ubuntu initial server setup guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04)
- [Initial Server Setup with Ubuntu](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04)
- [How To Install Nginx on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)
- [How To Secure Nginx with Let's Encrypt on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-18-04)
> NOTE: Throughout this guide, we will be using the non-root user created in the initial server setup, so `sudo` is prefixed to most commands.

## Step 1: Refresh your local package index

```bash
sudo apt update
```

### Install node
```bash
cd ~
curl -sL https://deb.nodesource.com/setup_18.x -o /tmp/nodesource_setup.sh
sudo bash /tmp/nodesource_setup.sh
sudo apt install nodejs
node -v
```
> NOTE: You can specify a different Node.js version by changing setup_18.x to your desired version, such as setup_16.x.

### Install NVM (Node Version Manager)
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

### Install npm
```bash
sudo apt install npm
```

### Install yarn if needed
```bash
sudo npm install -g yarn
```
>NOTE: Yarn is optional. It is used here for its perceived performance benefits over npm.


### Source bashrc
```bash
source ~/.bashrc
```

### Clone repo to /var/www
```bash
sudo git clone <repo-url> /var/www/<your-app-name>
```
>NOTE: If your repository is private, you'll need to use a personal access token to clone it.
 
```bash
sudo git clone https://<personal-token>@github.com/<repo-url>.git /var/www/<your-app-name>
```

[Managing your github personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

### Install dependencies
```bash
sudo yarn install
```

### Build the app
```bash
sudo yarn build
```

### Install http-server
```bash
npm install -g http-server
```

### Move node to /usr/local/bin/node
```bash
sudo ln -s ~/.nvm/versions/node/v18.20.0/lib/node_modules /usr/local/bin/node
```

> NOTE: Adjust the path to your Node.js binary if it differs.
> Please find where is your global node folder and copy the modules.
> If you do not want to copy the modules to /usr/local/bin, In the ExecStart below you can use "node_module_path/http-server/bin/http-server"

### Create a systemd service file
```bash
sudo nano /etc/systemd/system/<your_app_name>.service
```

Add the following content to the service file and replace the placeholders with the appropriate values.

```bash
[Unit]
Description=Test App

[Service]
ExecStart=/usr/local/bin/node/http-server/bin/http-server /var/www/<your-app-name>/dist -p 3000 -d false
Type=simple
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=<your-app-name>
User=<your-created-user>
# Group=<alternate group>
# Environment=NODE_ENV=production PORT=1337

[Install]
WantedBy=multi-user.target
```

### Enable the service
```bash
sudo systemctl enable <your_app_name>.service
```

### Start, stop, restart, and check the status of the service
```bash
sudo systemctl start <your_app_name>.service
sudo systemctl stop <your_app_name>.service
sudo systemctl restart <your_app_name>.service
sudo systemctl status <your_app_name>.service
```

### Set up server block for Nginx
```bash
sudo nano /etc/nginx/sites-available/<your_main_domain>
```

> NOTE: It's better to keep the server block file name as main domain name, so that you can add server blocks for each subdomain.

```bash
# Add the following content to the server block
# Replace the placeholders with the appropriate values
# The domain your_app_domain could be domain.com or subdomain.domain.com
# For example: If main domain, "google.com" or if you are creating for subdomain then, "mail.google.com"
server {
    server_name <your_app_domain> www.<your_app_domain>;

    # This line to is added to manage the router my SPA app
    error_page 404 /;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # The following will be added by pre-requisite step (How To Install Nginx on Ubuntu) above. If not, please add it manually.

    # Please make sure the following line is not added to multiple blocks.
    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/<your_app_domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<your_app_domain>/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}
server {
    if ($host = www.<your_app_domain>) {
        return 301 https://$host$request_uri;
    }

    if ($host = <your_app_domain>) {
        return 301 https://$host$request_uri;
    }

    listen 80;
    server_name <your_app_domain> www.<your_app_domain>;
    return 404;
}
```

### Test and restart Nginx
sudo nginx -t
sudo systemctl restart nginx

> NOTE: Make sure to replace <your-app-name>, <your-created-user>, <your_app_domain>, and other placeholders with your actual values. Additionally, review the configurations and commands to ensure they align with your specific setup and requirements.

//////////////////bash script

#!/bin/bash
set -e
#  Edit these 
DB_NAME="itelect104"
DB_USER="itelect104"
DB_PASS="StrongPass123!"
REPO_URL="https://github.com/mlgclschool/itelect104_rp.oreiro.git"
DOMAIN="redenp.me"
EMAIL="rp.oreiro@gmail.com
PHP="8.5"
# 
sudo apt update && sudo apt upgrade -y
sudo apt install nginx mysql-server git -y
sudo systemctl enable nginx mysql
sudo mysql -e "CREATE DATABASE IF NOT EXISTS ${DB_NAME};"
sudo mysql -e "CREATE USER IF NOT EXISTS '${DB_USER}'@'localhost' IDENTIFIED BY
'${DB_PASS}';"
sudo mysql -e "GRANT ALL PRIVILEGES ON ${DB_NAME}.* TO '${DB_USER}'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"
sudo add-apt-repository ppa:ondrej/php -y && sudo apt update
sudo apt install php${PHP}-fpm php${PHP}-mysql php${PHP}-xml \
php${PHP}-mbstring php${PHP}-curl php${PHP}-zip php${PHP}-bcmath -y
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
sudo git clone ${REPO_URL} /var/www/itelect104_rp.oreiro
sudo chown -R www-data:www-data /var/www/itelect104_rp.oreiro
cd /var/www/itelect104_rp.oreiro
sudo -u www-data composer install --no-dev --optimize-autoloader
sudo cp .env.example .env
sudo sed -i "s/DB_DATABASE=.*/DB_DATABASE=${DB_NAME}/" .env
sudo sed -i "s/DB_USERNAME=.*/DB_USERNAME=${DB_USER}/" .env
sudo sed -i "s/DB_PASSWORD=.*/DB_PASSWORD=${DB_PASS}/" .env
Laravel + AWS Ubuntu Exam Reference — Advanced Deployment Tasks
Page 7
sudo sed -i "s|APP_URL=.*|APP_URL=https://${DOMAIN}|" .env
sudo -u www-data php artisan key:generate
sudo -u www-data php artisan migrate --force
sudo -u www-data php artisan storage:link
sudo tee /etc/nginx/sites-available/default > /dev/null <<EOF
server {
listen 80;
server_name ${DOMAIN} www.${DOMAIN};
return 301 https://\$host\$request_uri;
}
server {
listen 443 ssl;
server_name ${DOMAIN} www.${DOMAIN};
root /var/www/itelect104_rp.oreiro/public;
index index.php;
ssl_certificate /etc/letsencrypt/live/${DOMAIN}/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/${DOMAIN}/privkey.pem;
location / { try_files \$uri \$uri/ /index.php?\$query_string; }
location ~ \.php$ {
include snippets/fastcgi-php.conf;
fastcgi_pass unix:/var/run/php/php${PHP}-fpm.sock;
}
}
EOF
sudo ln -sf /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d ${DOMAIN} -d www.${DOMAIN} \--non-interactive --agree-tos -m ${EMAIL}
sudo systemctl reload nginx


---------------------------------------------------------------


Laravel + MySQL + Nginx on AWS Ubuntu — Complete Setup Guide
A complete step-by-step guide to deploying a Laravel application on a fresh AWS EC2 Ubuntu instance, including common error fixes.

Table of Contents
Connect to EC2
Update the Systemw
Install Nginx
Install MySQL
Open AWS Security Group Ports
Install Git & Clone Repo
Install PHP & Composer
Move Project & Configure Laravel
Configure Nginx for Laravel
Set Up MySQL Database & User
Run Migrations
Error Fixes & Troubleshooting

Create an instance on AWS Console and make sure to download your keypair (.pem) file

Open git Bash and cd to downloads or where you downloaded your .pem file

1. Connect to EC2

    ssh -i your-key.pem ubuntu@your-ec2-public-ip


2. Update the System

    sudo apt update && sudo apt upgrade -y


3. Install Nginx

    sudo apt install nginx -y

    sudo systemctl start nginx

    sudo systemctl enable nginx

Verify:

    sudo systemctl status nginx

Visit http://your-ec2-public-ip — you should see the Nginx welcome page.

4. Install MySQL

        sudo apt install mysql-server -y
        sudo systemctl start mysql
        sudo systemctl enable mysql

Run security setup:
        sudo mysql_secure_installation

Follow the prompts:
Set a root password

    Remove anonymous users → Yes

    Disallow remote root login → Yes

    Remove test database → Yes

    Reload privilege tables → Yes

Verify:
    sudo mysql -u root -p


5. Open AWS Security Group Ports
In the AWS Console:
Go to EC2 → Security Groups
Select your instance's security group
Add Inbound Rules:
Type
Port
Source
HTTP
80
0.0.0.0/0
HTTPS
443
0.0.0.0/0
SSH
22
Your IP only
MySQL
3306
Your IP only


6. Install Git & Clone Re	po

    sudo apt install git -y

Note: git config --global user.name and user.email are only required when committing code. You can skip them if you are only cloning/pulling on the server.
Public repo:
git clone https://github.com/your-username/your-repo.git

For Private repos only (SSH):
# Generate SSH key
ssh-keygen -t ed25519 -C "you@example.com"

# Copy the public key
cat ~/.ssh/id_ed25519.pub

Then go to GitHub → Settings → SSH and GPG Keys → New SSH Key, paste the key, and save.
# Test connection
ssh -T git@github.com

# Clone

    git clone git@github.com:your-username/your-repo.git


7. Install PHP & Composer

    sudo apt install php php-fpm php-mysql php-mbstring php-xml php-bcmath php-curl php-zip unzip -y

Check PHP version:

    php -v

Install Composer:

    sudo apt install composer -y


8. Move Project & Configure Laravel

Check your cloned folder name:

    ls ~

Move to web root (replace your-repo with actual folder name):

    sudo mv your-repo /var/www/your-repo

Example used in this guide: sudo mv itelect104_pub /var/www/itelect104

Install Laravel dependencies:

    cd /var/www/itelect104

    sudo composer install

Set up environment file:

    sudo cp .env.example .env

    sudo nano .env

Update these values:

APP_NAME=itelect104
APP_ENV=production
APP_KEY=
APP_DEBUG=false
APP_URL=http://your-ec2-public-ip

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=itelect104
DB_USERNAME=itelect104user
DB_PASSWORD=yourpassword

Generate app key:

    sudo php artisan key:generate

Set permissions:

    sudo chown -R www-data:www-data /var/www/itelect104
    sudo chmod -R 755 /var/www/itelect104/storage
    sudo chmod -R 755 /var/www/itelect104/bootstrap/cache



9. Configure Nginx for Laravel

    sudo nano /etc/nginx/sites-available/itelect104

Paste this config:

    server {
        listen 80;
        server_name your-ec2-public-ip;

        root /var/www/itelect104/public;
        index index.php index.html;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ /\.ht {
            deny all;
        }
    }

Enable the site:

    sudo ln -s /etc/nginx/sites-available/itelect104 /etc/nginx/sites-enabled/

    sudo nginx -t

    sudo systemctl reload nginx


10. Set Up MySQL Database & User

    sudo mysql

    CREATE DATABASE itelect104;
    CREATE USER 'itelect104user'@'localhost' IDENTIFIED BY 'yourpassword';
    GRANT ALL PRIVILEGES ON itelect104.* TO 'itelect104user'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;

Verify the user was created:

    sudo mysql

    SELECT user, host FROM mysql.user;

    EXIT;

Test login:

    mysql -u itelect104user -p


11. Run Migrations

        cd /var/www/itelect104

        sudo php artisan config:clear
        sudo php artisan cache:clear
        sudo php artisan migrate



--------------------------------------------------------

Laravel + AWS Ubuntu Exam Reference — Advanced Deployment Tasks
Page 1
Laravel + MySQL + Nginx
on AWS Ubuntu
Advanced Deployment Tasks — Exam Reference
Prerequisite: base Laravel + MySQL + Nginx already running on EC2
Task Pts §
Proper security group 5 Base
MySQL running 5 Base
Nginx running 5 Base
Laravel running 5 Base
Access with domain 5 §1
HTTPS access with domain 5 §1
Restricted SSH 5 §2
Restricted to only by domain 10 §3
Cloudflare setup 10 §3
Bash script – one-time setup 10 §4
Docker setup 20 §5
CI/CD 20 §6
Laravel + AWS Ubuntu Exam Reference — Advanced Deployment Tasks
Page 2
§1 Domain Access + HTTPS + Certbot 5 + 5 pts
Point your domain to EC2, get a free SSL cert from Certbot, then force-redirect all HTTP to HTTPS.
Step 1 — DNS: point domain to EC2
1. Add A record: In your DNS provider, add an A record: yourdomain.com → EC2 public IP
2. Add www A record: www.yourdomain.com → same EC2 IP
3. Wait for propagation (instant on Cloudflare, up to a few minutes elsewhere)


Step 2 — Open ports in AWS Security Group
EC2 → Security Groups → Inbound rules → make sure these are open:
Port 80 (HTTP) — Source: 0.0.0.0/0 ← needed for Certbot verification
Port 443 (HTTPS) — Source: 0.0.0.0/0 ← (can restrict to Cloudflare later, see §3)


Step 3 — Install Certbot & get SSL certificate


    sudo apt install certbot python3-certbot-nginx -y


    sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com



# Certbot will ask for your email and automatically edit your Nginx config


Step 4 — Force HTTPS (required by instructor)
Edit Nginx config so HTTP always redirects to HTTPS:


    sudo nano /etc/nginx/sites-available/laravel


Make sure it has TWO server blocks like this:


# Block 1: redirect HTTP → HTTPS
    server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;
    }
# Block 2: serve the site over HTTPS
server {
listen 443 ssl;
server_name yourdomain.com www.yourdomain.com;
root /var/www/laravel/public;
index index.php;
ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
include /etc/letsencrypt/options-ssl-nginx.conf;
location / { try_files $uri $uri/ /index.php?$query_string; }
location ~ \.php$ {
Laravel + AWS Ubuntu
include snippets/fastcgi-php.conf;
fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
}
}
Exam Reference — Advanced Deployment Tasks
sudo nginx -t && sudo systemctl reload nginx
 Visiting http://yourdomain.com now auto-redirects to https://yourdomain.com — this earns both the domain
and HTTPS points.
Step 5 — Test auto-renewal
sudo certbot renew --dry-run # certificate auto-renews every 90 days
Page 3
Laravel + AWS Ubuntu
§2
Restricted SSH
Exam Reference — Advanced Deployment Tasks
5 pts
Lock down port 22 so only YOUR IP can SSH in — not the whole internet.
AWS Security Group — change the SSH rule
1. EC2 console: Instances → select instance → Security tab → click the Security Group
2. Edit inbound rules: find the SSH rule (port 22)
3. Change source: from 0.0.0.0/0 → My IP (AWS auto-fills your current IP)
4. Save rules
 If your IP changes (different WiFi, mobile data), you'll be locked out. Just update the rule again with your new
IP.
Extra hardening — sshd_config
sudo nano /etc/ssh/sshd_config
Add or update these lines:
PermitRootLogin no # block direct root SSH
PasswordAuthentication no # key-pair only, no passwords
PubkeyAuthentication yes
AllowUsers ubuntu
sudo systemctl restart sshd
 EC2 already uses key pairs — this just enforces it officially and blocks root login.
Page 4
Laravel + AWS Ubuntu Exam Reference — Advanced Deployment Tasks
Page 5
§3 Cloudflare Setup + Restrict to Domain Only 10 + 10 pts
Route all traffic through Cloudflare (proxy), then block direct EC2 IP access — visitors must use your domain.
Part 1 — Cloudflare DNS setup
1. Create account: cloudflare.com → add your domain
2. Add A record: yourdomain.com → EC2 public IP — set Proxy to ON (orange cloud)
3. Update nameservers: at your domain registrar, replace them with Cloudflare's two nameservers
4. SSL/TLS mode: Cloudflare dashboard → SSL/TLS → set to Full (strict)
 Full (strict) requires your server to already have a valid SSL cert (from Certbot in §1).
Part 2 — Restrict AWS Security Group to Cloudflare IPs only
Remove the 0.0.0.0/0 rules for ports 80 and 443. Replace with these Cloudflare IP ranges (one rule each):
173.245.48.0/20 103.21.244.0/22 103.22.200.0/22 103.31.4.0/22
141.101.64.0/18 108.162.192.0/18 190.93.240.0/20 188.114.96.0/20
197.234.240.0/22 198.41.128.0/17 162.158.0.0/15 104.16.0.0/13
104.24.0.0/14 172.64.0.0/13 131.0.72.0/22
 Direct access to your EC2 IP will time out. Traffic must go through your domain via Cloudflare. That's the
'restricted to only by domain' requirement.
Part 3 — Nginx real IP (paste inside the server { } block)
set_real_ip_from 173.245.48.0/20; set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22; set_real_ip_from 103.31.4.0/22;
set_real_ip_from 141.101.64.0/18; set_real_ip_from 108.162.192.0/18;
set_real_ip_from 190.93.240.0/20; set_real_ip_from 162.158.0.0/15;
set_real_ip_from 104.16.0.0/13; set_real_ip_from 104.24.0.0/14;
real_ip_header CF-Connecting-IP;
sudo nginx -t && sudo systemctl reload nginx
Laravel + AWS Ubuntu Exam Reference — Advanced Deployment Tasks
Page 6
§4 Bash Script — One-Time Setup 10 pts
One script that builds the entire stack from scratch on a fresh Ubuntu EC2. Edit the 7 config variables at the top, then run
it.
Create and run
nano setup.sh # paste the script below
chmod +x setup.sh
./setup.sh
setup.sh — clean, no clutter
#!/bin/bash
set -e
#  Edit these 
DB_NAME="laravel_db"
DB_USER="laravel_user"
DB_PASS="StrongPass123!"
REPO_URL="https://github.com/you/yourrepo.git"
DOMAIN="yourdomain.com"
EMAIL="you@yourdomain.com"
PHP="8.2"
# 
sudo apt update && sudo apt upgrade -y
sudo apt install nginx mysql-server git -y
sudo systemctl enable nginx mysql
sudo mysql -e "CREATE DATABASE IF NOT EXISTS ${DB_NAME};"
sudo mysql -e "CREATE USER IF NOT EXISTS '${DB_USER}'@'localhost' IDENTIFIED BY
'${DB_PASS}';"
sudo mysql -e "GRANT ALL PRIVILEGES ON ${DB_NAME}.* TO '${DB_USER}'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"
sudo add-apt-repository ppa:ondrej/php -y && sudo apt update
sudo apt install php${PHP}-fpm php${PHP}-mysql php${PHP}-xml \
php${PHP}-mbstring php${PHP}-curl php${PHP}-zip php${PHP}-bcmath -y
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
sudo git clone ${REPO_URL} /var/www/laravel
sudo chown -R www-data:www-data /var/www/laravel
cd /var/www/laravel
sudo -u www-data composer install --no-dev --optimize-autoloader
sudo cp .env.example .env
sudo sed -i "s/DB_DATABASE=.*/DB_DATABASE=${DB_NAME}/" .env
sudo sed -i "s/DB_USERNAME=.*/DB_USERNAME=${DB_USER}/" .env
sudo sed -i "s/DB_PASSWORD=.*/DB_PASSWORD=${DB_PASS}/" .env
Laravel + AWS Ubuntu Exam Reference — Advanced Deployment Tasks
Page 7
sudo sed -i "s|APP_URL=.*|APP_URL=https://${DOMAIN}|" .env
sudo -u www-data php artisan key:generate
sudo -u www-data php artisan migrate --force
sudo -u www-data php artisan storage:link
sudo tee /etc/nginx/sites-available/laravel > /dev/null <<EOF
server {
listen 80;
server_name ${DOMAIN} www.${DOMAIN};
return 301 https://\$host\$request_uri;
}
server {
listen 443 ssl;
server_name ${DOMAIN} www.${DOMAIN};
root /var/www/laravel/public;
index index.php;
ssl_certificate /etc/letsencrypt/live/${DOMAIN}/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/${DOMAIN}/privkey.pem;
location / { try_files \$uri \$uri/ /index.php?\$query_string; }
location ~ \.php$ {
include snippets/fastcgi-php.conf;
fastcgi_pass unix:/var/run/php/php${PHP}-fpm.sock;
}
}
EOF
sudo ln -sf /etc/nginx/sites-available/laravel /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d ${DOMAIN} -d www.${DOMAIN} \--non-interactive --agree-tos -m ${EMAIL}
sudo systemctl reload nginx
Laravel + AWS Ubuntu Exam Reference — Advanced Deployment Tasks
Page 8
§5 Docker Setup 20 pts
3 files to create, then 5 commands to run. That's it.
Step 1 — Install Docker
sudo apt install docker.io docker-compose -y
sudo systemctl enable --now docker
sudo usermod -aG docker ubuntu && newgrp docker
Step 2 — Create 3 files in your project
File 1: Dockerfile (in project root)
nano Dockerfile
FROM php:8.2-fpm
RUN apt-get update && apt-get install -y \
git curl zip unzip libpng-dev libonig-dev libxml2-dev libzip-dev && \
docker-php-ext-install pdo_mysql mbstring zip
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
WORKDIR /var/www
COPY . .
RUN composer install --no-dev --optimize-autoloader && \
chown -R www-data:www-data /var/www
EXPOSE 9000
CMD ["php-fpm"]
File 2: docker-compose.yml (in project root)
nano docker-compose.yml
version: '3.8'
services:
app:
build: .
restart: unless-stopped
volumes: ['.:/var/www']
networks: [app-network]
depends_on: [db]
nginx:
image: nginx:alpine
restart: unless-stopped
ports: ['80:80', '443:443']
volumes:- '.:/var/www'
Laravel + AWS Ubuntu Exam Reference — Advanced Deployment Tasks
Page 9- './docker/nginx/default.conf:/etc/nginx/conf.d/default.conf'- '/etc/letsencrypt:/etc/letsencrypt:ro'
networks: [app-network]
depends_on: [app]
db:
image: mysql:8.0
restart: unless-stopped
environment:
MYSQL_DATABASE: laravel_db
MYSQL_USER: laravel_user
MYSQL_PASSWORD: StrongPass123!
MYSQL_ROOT_PASSWORD: rootpassword
volumes: ['db-data:/var/lib/mysql']
networks: [app-network]
networks:
app-network:
volumes:
db-data:
File 3: docker/nginx/default.conf
mkdir -p docker/nginx && nano docker/nginx/default.conf
server {
listen 80;
server_name yourdomain.com;
return 301 https://$host$request_uri;
}
server {
listen 443 ssl;
server_name yourdomain.com;
root /var/www/public;
index index.php;
ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
location / { try_files $uri $uri/ /index.php?$query_string; }
location ~ \.php$ {
fastcgi_pass app:9000;
include fastcgi_params;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
}
Step 3 — Update .env then run
In your .env file, change the database host to db (the Docker service name):
DB_HOST=db
Laravel + AWS Ubuntu
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=StrongPass123!
Exam Reference — Advanced Deployment Tasks
docker-compose up -d --build
docker-compose exec app php artisan key:generate
docker-compose exec app php artisan migrate --force
docker-compose exec app php artisan storage:link
 Quick commands to remember: docker-compose ps (status) | docker-compose logs -f (logs) |
docker-compose down (stop)
Page 10
Laravel + AWS Ubuntu Exam Reference — Advanced Deployment Tasks
Page 11
§6 CI/CD with GitHub Actions 20 pts
Auto-deploy to EC2 on every git push to main. 1 file to create, 3 secrets to add.
Step 1 — Add 3 secrets to GitHub
Repo → Settings → Secrets and variables → Actions → New repository secret
Secret name Value
EC2_SSH_KEY Full content of your .pem private key file
EC2_HOST Your EC2 public IP or domain name
EC2_USER ubuntu
Step 2 — Create the workflow file
mkdir -p .github/workflows
nano .github/workflows/deploy.yml
name: Deploy to EC2
on:
push:
branches: [main]
jobs:
deploy:
runs-on: ubuntu-latest
steps:- uses: actions/checkout@v3- name: Deploy via SSH
uses: appleboy/ssh-action@v1.0.0
with:
host: ${{ secrets.EC2_HOST }}
username: ${{ secrets.EC2_USER }}
key: ${{ secrets.EC2_SSH_KEY }}
script: |
cd /var/www/laravel
git pull origin main
#  With Docker 
docker-compose down
docker-compose up -d --build
docker-compose exec -T app composer install --no-dev
docker-compose exec -T app php artisan migrate --force
docker-compose exec -T app php artisan config:cache
#  Without Docker 
# composer install --no-dev
Laravel + AWS Ubuntu Exam Reference — Advanced Deployment Tasks
Page 12
# php artisan migrate --force
# php artisan config:cache
# sudo systemctl reload nginx
Step 3 — Push to trigger it
git add .github/workflows/deploy.yml
git commit -m "add CI/CD"
git push origin main
# Go to GitHub → Actions tab to watch it run
 Green checkmark = deployed. Red X = check the logs. Every push to main auto-deploys from now on.
Quick Reference — File Locations
File Path Command to create
Bash setup script ~/setup.sh nano setup.sh
Dockerfile project/Dockerfile nano Dockerfile
Docker Compose project/docker-compose.yml nano docker-compose.yml
Nginx (Docker) project/docker/nginx/default.co
nf
mkdir -p docker/nginx && nano
docker/nginx/default.conf
GitHub CI/CD project/.github/workflows/deplo
y.yml
mkdir -p .github/workflows &&
nano
.github/workflows/deploy.yml
Nginx (no Docker) /etc/nginx/sites-available/lara
vel
sudo nano /etc/nginx/sites-avail
able/laravel
SSH config /etc/ssh/sshd_config sudo nano /etc/ssh/sshd_config
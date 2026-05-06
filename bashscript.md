```
#!/bin/bash
set -e

# ── Edit these ──────────────────────────────────────
REPO_URL="https://github.com/your-username/your-repo.git"
PROJECT="itelect104"
DB_NAME="itelect104"
DB_USER="itelect104user"
DB_PASS="yourpassword"
DOMAIN="yourdomain.com"
EMAIL="you@yourdomain.com"
# ────────────────────────────────────────────────────

sudo apt update && sudo apt upgrade -y
sudo apt install nginx mysql-server git php php-fpm php-mysql \
  php-mbstring php-xml php-bcmath php-curl php-zip unzip composer -y

sudo systemctl enable nginx mysql

sudo mysql -e "CREATE DATABASE ${DB_NAME};"
sudo mysql -e "CREATE USER '${DB_USER}'@'localhost' IDENTIFIED BY '${DB_PASS}';"
sudo mysql -e "GRANT ALL PRIVILEGES ON ${DB_NAME}.* TO '${DB_USER}'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"

git clone ${REPO_URL} ~/${PROJECT}
sudo mv ~/${PROJECT} /var/www/${PROJECT}

cd /var/www/${PROJECT}
sudo composer install
sudo cp .env.example .env
sudo sed -i "s/APP_ENV=.*/APP_ENV=production/" .env
sudo sed -i "s/APP_DEBUG=.*/APP_DEBUG=false/" .env
sudo sed -i "s|APP_URL=.*|APP_URL=https://${DOMAIN}|" .env
sudo sed -i "s/DB_DATABASE=.*/DB_DATABASE=${DB_NAME}/" .env
sudo sed -i "s/DB_USERNAME=.*/DB_USERNAME=${DB_USER}/" .env
sudo sed -i "s/DB_PASSWORD=.*/DB_PASSWORD=${DB_PASS}/" .env
sudo php artisan key:generate
sudo php artisan config:clear
sudo php artisan cache:clear
sudo php artisan migrate --force

sudo chown -R www-data:www-data /var/www/${PROJECT}
sudo chmod -R 755 /var/www/${PROJECT}/storage
sudo chmod -R 755 /var/www/${PROJECT}/bootstrap/cache

sudo tee /etc/nginx/sites-available/${PROJECT} > /dev/null <<EOF
server {
    listen 80;
    server_name ${DOMAIN} www.${DOMAIN};
    root /var/www/${PROJECT}/public;
    index index.php index.html;
    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php-fpm.sock;
    }
    location ~ /\.ht {
        deny all;
    }
}
EOF

sudo ln -s /etc/nginx/sites-available/${PROJECT} /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d ${DOMAIN} -d www.${DOMAIN} \
  --non-interactive --agree-tos -m ${EMAIL}
sudo systemctl reload nginx
```
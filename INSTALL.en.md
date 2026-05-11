Installation steps
==

### Installation steps for setting up a Wordpress+WooCommerce+Trondealer

>Requirements:
>1. GNU/Linux OS. A VPS[Hostinger/Linode/OVH/Vultr/DigitalOcean any other, works] with 2vCPUs/2GB RAM. Tested on Ubuntu/Debian.
>2. Docker/Docker-Compose
>3. Nginx as reverse proxy + Let's Encrypt
>4. Docker Stack
>5. Redis
>6. Nginx virtualhost and Let's Encrypt certificate creation
>7. Trondealer plugin + other store plugins

#### 1. Ubuntu/Debian OS

Upgrade the system:
```bash
sudo apt update && sudo apt dist-upgrade -y && sudo apt clean
```

Adjusting system for performance. Run in your console:

```bash
sudo tee -a /etc/sysctl.conf <<EOF
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 15
net.core.somaxconn = 4096
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_keepalive_intvl = 15
EOF
```

#### 2. Docker
Install it with:

**Debian**

```bash
# Add Docker's official GPG key:
sudo apt update && sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
 docker-buildx-plugin docker-compose-plugin
```

**Ubuntu**

```bash
# Add Docker's official GPG key:
sudo apt update && sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update && sudo apt install -y docker-ce docker-ce-cli \
 containerd.io docker-buildx-plugin docker-compose-plugin
```

Docker-Compose(works on both, Debian/Ubuntu):

```bash
sudo su

cat << EOF > /usr/local/bin/docker-compose
#!/bin/bash
exec docker compose "$@"
EOF

chmod +x /usr/local/bin/docker-compose
```

#### 3. Nginx and Let's Encrypt

**Nginx**

```bash
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring -y
curl -fsSL https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
  | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null

echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/mainline/ubuntu $(lsb_release -cs) nginx" \
| sudo tee /etc/apt/sources.list.d/nginx.list

sudo apt update && sudo apt install nginx -y

sudo systemctl enable nginx
```

**Let's Encrypt**

```bash
sudo apt install -y curl gnupg2 ca-certificates lsb-release \
 dirmngr software-properties-common apt-transport-https 
sudo apt install -y certbot python3-certbot-nginx
```

#### 4. Docker stack

In your server, just clone this repo:

```bash
git clone https://github.com/Koratsuki/wordpress-woocommerce-trondealer.git
cd wordpress-woocommerce-trondealer
```

Adjust variables according to your needs inside ./config directory and deploy stack. This is an automated wordpress install, so, modify and adjust:
```text
WP_URL=http://localhost:8000
WP_TITLE=TronDealer
WP_ADMIN_USER=admin
WP_ADMIN_PASSWORD=admin
WP_ADMIN_EMAIL=admin@trondealer.local
```

**Note:** If you're on a prod environment, avoid using admin/admin combination as user/pass. Make sure you use a hardened password. 

And run your stack:

```bash
docker-compose up -d
```

This will deploy wordpress+woocommerce+trondealer and some other plugins. This topic will me described later on. It take some time installing plugins depending on your internet speed. I recommend open another terminal and go to the dame folder and watch the process until finished.

#### 5. Redis
Add this in your wp-config.php. It should be placed inside `./data/wordpress`:

```php
/** REDIS-CACHE **/
define('WP_REDIS_HOST', 'redis');
define('WP_REDIS_PORT', 6379);
define('WP_REDIS_TIMEOUT', 1);
define('WP_REDIS_READ_TIMEOUT', 1);
define('WP_REDIS_DATABASE', 0);
```

Restart stack:

```bash
docker-compose stop
docker-compose up -d
```

Or just restart wordpress container:

```bash
docker restart wp-app
```

Go to wordpress Admin panel/Plugins. Enable this installing wordpress plugin: "Redis Object Cache". "Activate" it, and go to settings and "Enable Object Caché"

#### 6. Nginx virtualhost and Let's Encrypt certificate creation
Before doing that, we need to adjust some stuff.

1. copy `./nginx/nginx.conf` to `/etc/nginx` on your server
2. If you're using a `dev environment`, copy **wp-dev.conf**  to `./etc/nginx/conf.d`, and adjust server name drective. To use self-signed certificates in your dev environment, run in the server's console:

```bash
mkdir -p /etc/nginx/certs
openssl ecparam -name secp384r1 -out /etc/nginx/certs/ecparam.pem
openssl ecparam -in /etc/nginx/certs/ecparam.pem -genkey -noout -out /etc/nginx/certs/server.key
openssl req -new -key /etc/nginx/certs/server.key -out /etc/nginx/certs/server.csr -sha256
openssl req -x509 -days 3650 -key /etc/nginx/certs/server.key -in /etc/nginx/certs/server.csr -out /etc/nginx/certs/server.pem
openssl dhparam -out /etc/nginx/certs/dhparam.pem 2048
```

The path of the certificates are already defined in the config.

3. If you're using a `prod environment`, copy **wp-prod.conf**  to `./nginx/nginx/conf.d`, and adjust server name drective[store.trondealer.com in this example]. To enable Let's Encrypt certificate, run in the server's console:

```bash
certbot --nginx -d store.trondealer.com
```

Wait until finished and then go to the `WP_URL` variable value you have defined earlier. Nice, you will be able to see your shinning Wordpress up&running.

#### 7. Trondealer plugin + other store plugins 
The stack comes with these plugins by default:

- hide-my-wp. A security plugin.
- redis-cache. Redis cache object plugin.
- simple-cloudflare-turnstile. Security plugin
- stops-core-theme-and-plugin-updates. Updates plugin.
- updraftplus. Backu n' restore plugin.
- woocommerce. Ecommerce plugin for wordpress.
- wordfence. A security plugin.
- wordpress-seo. SEO plugin.
- wp-mail-smtp. Wordpress email plugin.
- wpforms-lite. Wordpress forms plugin.

All you have to do is go to the plugins menu in the admin dashboard and configure it to make it work.

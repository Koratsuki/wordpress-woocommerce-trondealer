Pasos de instalación
==

### Pasos de instalación para configurar Wordpress+WooCommerce+Trondealer

>Requisitos:
>1. Sistema operativo GNU/Linux. Un VPS [Hostinger/Linode/OVH/Vultr/DigitalOcean o cualquier otro, funciona] con 2 vCPUs/2 GB de RAM. Probado en Ubuntu/Debian.
>2. Docker / Docker-Compose
>3. Nginx como proxy inverso + Let's Encrypt
>4. Docker Stack
>5. Redis
>6. Host virtual de Nginx y creación de certificado Let's Encrypt
>7. Plugin Trondealer + otros plugins para la tienda

#### 1. Sistema operativo Ubuntu/Debian

Actualizar el sistema:

```bash
sudo apt update && sudo apt dist-upgrade -y && sudo apt clean
```

Ajuste del sistema para rendimiento. Ejecuta en tu consola:

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

Aplicalos los cambios con:

```bash
sysctl -p
```

2. Docker

Instálalo con:

**Debian**
```bash
# Agregar la clave GPG oficial de Docker:
sudo apt update && sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Agregar el repositorio a las fuentes de Apt:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
 docker-buildx-plugin docker-compose-plugin
```

Ubuntu

```bash
# Agregar la clave GPG oficial de Docker:
sudo apt update && sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Agregar el repositorio a las fuentes de Apt:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update && sudo apt install -y docker-ce docker-ce-cli \
 containerd.io docker-buildx-plugin docker-compose-plugin
```

Docker-Compose (funciona en Debian/Ubuntu):

```bash
sudo su

cat << EOF > /usr/local/bin/docker-compose
#!/bin/bash
exec docker compose "$@"
EOF

chmod +x /usr/local/bin/docker-compose
```

3. Nginx y Let's Encrypt

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

Let's Encrypt

```bash
sudo apt install -y curl gnupg2 ca-certificates lsb-release \
 dirmngr software-properties-common apt-transport-https 
sudo apt install -y certbot python3-certbot-nginx
```

4. Docker stack

En tu servidor, simplemente clona este repositorio:

```bash
git clone https://github.com/usuario/repo.git
cd repo
```

Ajusta las variables según tus necesidades dentro del directorio ./config y despliega el stack. Esta es una instalación automatizada de WordPress, así que modifica y ajusta:

```text
WP_URL=http://localhost:8000
WP_TITLE=TronDealer
WP_ADMIN_USER=admin
WP_ADMIN_PASSWORD=admin
WP_ADMIN_EMAIL=admin@trondealer.local
```

Nota: Si estás en un entorno de producción, evita usar la combinación `admin/admin` como usuario/contraseña. Asegúrate de usar una contraseña robusta.

Y ejecuta tu stack:

```bash
docker-compose up -d
```

Esto desplegará WordPress + WooCommerce + Trondealer y algunos otros plugins. Este tema se describirá más adelante. Tomará un tiempo instalar los plugins dependiendo de tu velocidad de internet. Recomiendo abrir otra terminal e ir a la misma carpeta y monitorear el proceso hasta que termine.

5. Redis

Agrega esto en tu wp-config.php. Debe ubicarse dentro de ./data/wordpress:

```php
/** REDIS-CACHE **/
define('WP_REDIS_HOST', 'redis');
define('WP_REDIS_PORT', 6379);
define('WP_REDIS_TIMEOUT', 1);
define('WP_REDIS_READ_TIMEOUT', 1);
define('WP_REDIS_DATABASE', 0);
```

Reinicia el stack:

```bash
docker-compose stop
docker-compose up -d
```

O simplemente reinicia el contenedor de WordPress:

```bash
docker restart wp-app
```

Ve al panel de administración de WordPress / Plugins. Habilita instalando el plugin de WordPress: "Redis Object Cache". "Actívalo" y ve a configuración y "Habilitar Object Cache".

6. Host virtual de Nginx y creación de certificado Let's Encrypt
Antes de hacer eso, necesitamos ajustar algunas cosas.

Copia `./nginx/nginx.conf` a `/etc/nginx` en tu servidor.

Si estás usando un entorno de desarrollo, copia `wp-dev.conf` a `/etc/nginx/conf.d` y ajusta la directiva server_name. Para usar certificados autofirmados en tu entorno de desarrollo, ejecuta en la consola del servidor:

```bash
mkdir -p /etc/nginx/certs
openssl ecparam -name secp384r1 -out /etc/nginx/certs/ecparam.pem
openssl ecparam -in /etc/nginx/certs/ecparam.pem -genkey -noout -out /etc/nginx/certs/server.key
openssl req -new -key /etc/nginx/certs/server.key -out /etc/nginx/certs/server.csr -sha256
openssl req -x509 -days 3650 -key /etc/nginx/certs/server.key -in /etc/nginx/certs/server.csr -out /etc/nginx/certs/server.pem
openssl dhparam -out /etc/nginx/certs/dhparam.pem 2048
```

La ruta de los certificados ya está definida en la configuración.

Si estás usando un entorno de producción, copia wp-prod.conf a ./nginx/nginx/conf.d y ajusta la directiva server_name [store.trondealer.com en este ejemplo]. Para habilitar el certificado Let's Encrypt, ejecuta en la consola del servidor:

```bash
certbot --nginx -d store.trondealer.com
```

Espera hasta que termine y luego ve al valor de la variable WP_URL que definiste anteriormente. ¡Bien, podrás ver tu flamante WordPress funcionando!

7. Plugin Trondealer + otros plugins para la tienda
El stack trae estos plugins por defecto:

hide-my-wp. Un plugin de seguridad.

redis-cache. Plugin de caché con Redis.

simple-cloudflare-turnstile. Plugin de seguridad.

stops-core-theme-and-plugin-updates. Plugin para actualizaciones.

updraftplus. Plugin de copias de seguridad y restauración.

woocommerce. Plugin de comercio electrónico para WordPress.

wordfence. Un plugin de seguridad.

wordpress-seo. Plugin de SEO.

wp-mail-smtp. Plugin de correo para WordPress.

wpforms-lite. Plugin de formularios para WordPress.

Todo lo que tienes que hacer es ir al menú de plugins en el escritorio de administración y configurarlos para que funcionen.

Proyecto WordPress + WooCommerce + Trondealer con Docker
==

Este proyecto proporciona un entorno completo, listo para producción o para desarrollo local, de una tienda online basada en WordPress con WooCommerce. Utiliza Docker Compose para orquestar todos los servicios necesarios, garantizando consistencia y reproducibilidad en diferentes máquinas.

### Descripción general del proyecto

Cuando despliegues esta configuración, obtendrás una tienda online totalmente funcional sin necesidad de configuración manual. Todos los componentes están contenedorizados y aislados, pero trabajan juntos sin problemas.

### Qué incluye

- **WordPress** – El CMS principal, ejecutando la última imagen oficial.
- **WooCommerce** – Preinstalado y activado, convierte WordPress en una potente plataforma de comercio electrónico.
- **MariaDB 12** – Contenedor de base de datos dedicado con almacenamiento persistente.
- **WP‑CLI** – Un servicio separado (o contenedor de inicialización) que automatiza la instalación y configuración de WooCommerce, plugins, productos de ejemplo y ajustes de la tienda.
- **phpMyAdmin** (opcional) – Una cómoda herramienta de gestión de bases de datos accesible desde el navegador.
- **Redis** – Plugin de caché para WordPress.

### Arquitectura y componentes

El proyecto se construye alrededor de estos servicios principales de Docker:

| Servicio      | Propósito                                           | Puertos expuestos (ejemplo) |
|---------------|-----------------------------------------------------|------------------------------|
| `wordpress`   | Sirve la tienda (frontend) y el panel de administración | `8000` → `80`               |
| `db`          | Base de datos MariaDB para WordPress                | `3306` → `3306`              |
| `cli‑setup`   | Ejecutor único de WP‑CLI para automatizar la configuración | – (solo interno)           |
| `phpmyadmin`  | Interfaz web para gestionar la base de datos        | `8080` → `80`               |
| `redis`       | Caché de objetos Redis                              | – (solo interno)           |

Todos los servicios comparten una red Docker dedicada (`woo_network`) y utilizan volúmenes con nombre para la persistencia de datos:
- `wordpress_data` – Almacena los archivos principales de WordPress, temas, plugins y subidas.
- `db_data` – Conserva el estado de la base de datos MariaDB.

### Automatización y configuración sin intervención («zero‑touch»)

El proyecto está diseñado para funcionar **recién salido de la caja**. Cuando inicias el entorno por primera vez:

1. El contenedor de base de datos se inicializa con un esquema nuevo.
2. WordPress se instala automáticamente usando variables de entorno (nombre de la base de datos, usuario, contraseña, etc.).
3. Un contenedor WP‑CLI dedicado ejecuta un script de configuración que:
   - Descarga y activa WooCommerce.
   - Instala plugins adicionales (por ejemplo, WooCommerce PayPal Payments, Elementor o cualquier otro que definas).
   - Configura opciones esenciales de WooCommerce: dirección de la tienda, moneda, país, compra como invitado, etc.
   - Crea las páginas necesarias de WooCommerce (Carrito, Finalizar compra, Mi cuenta, Tienda).
   - Importa un conjunto de productos de ejemplo (los datos de ejemplo por defecto de WooCommerce).
4. Después de ejecutarse correctamente, el contenedor WP‑CLI se detiene, dejando el contenedor de WordPress en ejecución, limpio y listo para usar.

### Puntos de personalización

Aunque la configuración está automatizada, puedes adaptarla fácilmente a tus necesidades:

- **Plugins** – Añade o elimina plugins editando el script de WP‑CLI (ej. `wp plugin install … --activate`).
- **Ajustes de WooCommerce** – Modifica la configuración por defecto de la tienda (moneda, pasarelas de pago, zonas de envío) directamente en el mismo script.
- **Datos de ejemplo** – Reemplaza los productos de ejemplo de WooCommerce con tu propia importación CSV/XML.
- **Puertos y dominios** – Cambia los puertos publicados en el archivo `docker-compose.yml` para evitar conflictos.

Debido a que toda la configuración está basada en código (Infraestructura como Código), puedes controlarla con un sistema de versiones, compartirla con tu equipo y replicarla en cualquier máquina que ejecute Docker.

### Configurar WordPress+WooCommerce+Trondealer

Lee las notas de [Instalación](./INSTALL.es.md).

### Qué obtienes después de ejecutar `docker-compose up -d`

- Un sitio web de comercio electrónico funcionando en `http://localhost:8000`
- El área de administración de WordPress en `http://localhost:8000/wp-admin` (las credenciales del usuario administrador se definen en el archivo `.env`)
- WooCommerce completamente activado, con productos de ejemplo y páginas por defecto
- Una base de datos MariaDB accesible mediante phpMyAdmin en `http://localhost:8080/phpmyadmin/` (servidor: `db`, credenciales del archivo `.env`)
- Volúmenes de datos persistentes – incluso si detienes los contenedores, los datos de tu tienda (productos, pedidos, ajustes) permanecen seguros

### Casos de uso

- **Desarrollo local** – Crea y prueba temas o plugins de WooCommerce sin afectar una tienda real.
- **Demostración / preproducción** – Levanta rápidamente una tienda representativa para presentaciones a clientes o aseguramiento de calidad.
- **Aprendizaje** – Experimenta con los entresijos de WooCommerce, pasarelas de pago o personalizaciones en un entorno desechable.

### Referencias

Youtube: [https://www.youtube.com/watch?v=fAfTWrj7YoE](https://www.youtube.com/watch?v=fAfTWrj7YoE)

Guía: [https://trondealer.com/en/guides/woocommerce-stablecoins-tutorial](https://trondealer.com/en/guides/woocommerce-stablecoins-tutorial)

Plugin: [https://trondealer.com/en/woocommerce](https://trondealer.com/en/woocommerce)

Github: [https://github.com/qvapay/trondealer-woocommerce](https://github.com/qvapay/trondealer-woocommerce)

Fees: [https://trondealer.com/en/pricing](https://trondealer.com/en/pricing)

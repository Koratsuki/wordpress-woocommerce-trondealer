Dockerized WordPress + WooCommerce + Trondealer Project
==

This project provides a complete, production‑ready or local development environment for a WordPress‑based e‑commerce store with WooCommerce. It uses Docker Compose to orchestrate all required services, ensuring consistency and reproducibility across different machines.

### Project Overview

When you deploy this setup, you get a fully functional online store without any manual configuration. All components are containerized and isolated, yet they work together seamlessly.

### What’s Included

- **WordPress** – The core CMS, running the latest official image.
- **WooCommerce** – Pre‑installed and activated, turning WordPress into a powerful e‑commerce platform.
- **MariaDB 12** – Dedicated database container with persistent storage.
- **WP‑CLI** – A separate service (or init container) that automates the installation and configuration of WooCommerce, plugins, sample products, and store settings.
- **phpMyAdmin** (optional) – A convenient database management tool, accessible via a web browser.
- **Redis** – Cache plugin for wordpress

### Architecture & Components

The project is built around this main Docker services:

| Service       | Purpose                                    | Exposed Ports (example) |
|---------------|--------------------------------------------|--------------------------|
| `wordpress`   | Serves the store frontend and admin panel  | `8000` → `80`           |
| `db`          | MariaDB database for WordPress               | `3306` → `3306`         |
| `cli‑setup`   | One‑time WP‑CLI runner to automate setup   | – (internal only)       |
| `phpmyadmin`  | Web interface for database management      | `8080` → `80`           |
| `redis`   | Redis object cache   | – (internal only)       |

All services share a dedicated Docker network (`woo_network`) and use named volumes for data persistence:
- `wordpress_data` – Stores WordPress core files, themes, plugins, and uploads.
- `db_data` – Retains the MariaDB database state.

### Automation & Zero‑Touch Setup

The project is designed to work **out of the box**. When you start the environment for the first time:

1. The database container initialises with a fresh schema.
2. WordPress installs itself automatically using environment variables (database name, user, password, etc.).
3. A dedicated WP‑CLI container runs a setup script that:
   - Downloads and activates WooCommerce.
   - Installs additional plugins (e.g., WooCommerce PayPal Payments, Elementor, or any you define).
   - Configures essential WooCommerce options: store address, currency, country, guest checkout, etc.
   - Creates the necessary WooCommerce pages (Cart, Checkout, My Account, Shop).
   - Imports a set of sample products (WooCommerce’s default dummy data).
4. After successful execution, the WP‑CLI container exits, leaving the running WordPress container clean and ready to use.

### Customization Points

Even though the setup is automated, you can easily adapt it to your needs:

- **Plugins** – Add or remove plugins by editing the WP‑CLI script (e.g., `wp plugin install … --activate`).
- **WooCommerce settings** – Modify the default store configuration (currency, payment gateways, shipping zones) directly in the same script.
- **Sample data** – Replace the built‑in WooCommerce sample products with your own CSV/XML import.
- **Ports & domains** – Change the published ports in the `docker-compose.yml` file to avoid conflicts.

Because the entire configuration is code‑driven (Infrastructure as Code), you can version control it, share it with your team, and replicate it on any machine that runs Docker.

### Setup Wordpress+WooCommerce+Trondealer

Read the [Installation](./INSTALL.en.md) notes.

### What You Get After Running `docker-compose up -d`

- A running e‑commerce website at `http://localhost:8000`
- WordPress admin area at `http://localhost:8000/wp-admin` (admin user credentials defined in the `.env` file)
- WooCommerce fully activated with sample products and default pages
- A MariaDB database accessible via phpMyAdmin at `http://localhost:8080/phpmyadmin/` (server: `db`, credentials from `.env`)
- Persistent data volumes – even if you stop the containers, your store data (products, orders, settings) remains safe

### Use Cases

- **Local development** – Build and test WooCommerce themes/plugins without affecting a live store.
- **Demo / staging** – Quickly spin up a representative store for client presentations or quality assurance.
- **Learning** – Experiment with WooCommerce internals, payment gateways, or customisations in a disposable environment.

### References

Youtube: [https://www.youtube.com/watch?v=fAfTWrj7YoE](https://www.youtube.com/watch?v=fAfTWrj7YoE)

Guide: [https://trondealer.com/en/guides/woocommerce-stablecoins-tutorial](https://trondealer.com/en/guides/woocommerce-stablecoins-tutorial)

Plugin: [https://trondealer.com/en/woocommerce](https://trondealer.com/en/woocommerce)

GIthub: [https://github.com/qvapay/trondealer-woocommerce](https://github.com/qvapay/trondealer-woocommerce)

Fees: [https://trondealer.com/en/pricing](https://trondealer.com/en/pricing)

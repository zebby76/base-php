
# 🐘 PHP Base Image – zebb76/base-php

This repository provides a **production-ready, multi-variant PHP base image** built on Alpine and PHP 8.4.7, ideal for extending with any PHP application or framework (Laravel, Symfony, WordPress, etc.).

✅ Hosted on Docker Hub: [zebb76/base-php](https://hub.docker.com/r/zebb76/base-php)

---

## ✨ Features

- ✅ PHP 8.4.7 only – consistent and up-to-date
- 📦 Multiple runtime variants: `cli`, `fpm`, `nginx`, `apache`
- ⚡ Lightweight Alpine Linux base
- 📂 Composer pre-installed
- 🚀 Multi-platform builds (`linux/amd64`, `linux/arm64`)
- 🔁 Advanced build system using `docker buildx bake`

---

## 📦 Available Variants

| Tag                          | Description              | Platform(s)           |
|-----------------------------|--------------------------|------------------------|
| `zebb76/base-php:8.4.7-cli`    | PHP CLI only              | `amd64`, `arm64`       |
| `zebb76/base-php:8.4.7-fpm`    | PHP-FPM engine            | `amd64`, `arm64`       |
| `zebb76/base-php:8.4.7-apache` | Apache + mod_php          | `amd64`, `arm64`       |
| `zebb76/base-php:8.4.7-nginx`  | Nginx + PHP-FPM setup     | `amd64`, `arm64`       |

> Variants are defined in the `docker-bake.hcl` using the `VARIANTS` list.

---

## 🛠️ How to Build

### 1. Setup Buildx

```bash
docker buildx create --use
```

### 2. Build a Specific Variant

```bash
docker buildx bake cli
```

### 3. Build All Variants

```bash
docker buildx bake default
```

---

## 🧪 Example Usage

Extend from this image in your own `Dockerfile`:

```dockerfile
FROM zebb76/base-php:8.4.7-fpm

COPY . /app
WORKDIR /app

RUN composer install

CMD ["php-fpm"]
```

---

## 🏷️ Tags Generated

These are generated dynamically based on Git metadata and bake target.

- `zebb76/base-php:8.4.7-cli`
- `zebb76/base-php:8.4.7-fpm`
- `zebb76/base-php:8.4.7-apache`
- `zebb76/base-php:8.4.7-nginx`
- Optionally with `-dev` suffix (e.g. `:8.4.7-cli-dev`)

---

## 📁 Project Structure

```text
.
├── Dockerfile             # Base build logic for PHP
├── docker-bake.hcl        # Multi-variant build config
└── README.md              # This documentation
```

---

## 📜 License

This project is licensed under the MIT License.

---

## 📬 Contact

Questions or improvements? [Open an issue](https://github.com/your-repo/issues) or reach out directly.


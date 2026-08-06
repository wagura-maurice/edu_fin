# EduFin Platform - Technical Architecture Review & Implementation Guide

## Dual-System Ecosystem: WordPress + Laravel Portal

**Document Type:** Technical Architecture Review  
**Date:** August 6, 2026  
**Version:** 1.0  
**Classification:** Technical — Internal Review  

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Architecture Overview](#2-architecture-overview)
3. [Compatibility Audit](#3-compatibility-audit)
4. [Infrastructure & Server Environment](#4-infrastructure--server-environment)
5. [Application Stack Review](#5-application-stack-review)
6. [Storage & Content Delivery Strategy](#6-storage--content-delivery-strategy)
7. [Cross-System Integration](#7-cross-system-integration)
8. [Performance Optimization](#8-performance-optimization)
9. [Security Review](#9-security-review)
10. [Scalability Assessment](#10-scalability-assessment)
11. [Implementation Guide](#11-implementation-guide)
12. [Risk Register & Mitigations](#12-risk-register--mitigations)
13. [Recommendations Summary](#13-recommendations-summary)

---

## 1. Executive Summary

### 1.1 Scope

This document provides a comprehensive technical architecture review and implementation guide for the EduFin dual-system ecosystem, comprising:

- **WordPress Site** — Company landing page (MySQL-backed)
- **Laravel Portal** — Client portal, admin panel, and REST API (PostgreSQL-backed)

The review evaluates compatibility, scalability, and security best practices against the following mandated specifications:

| Specification | Requirement |
|---------------|-------------|
| Operating System | Ubuntu 22.04 LTS |
| Web Server / Load Balancer | Nginx |
| SSL/TLS | Let's Encrypt |
| PHP Runtime | PHP 8.4 (Strict) |
| Dependency Management | Composer (PHP), NPM (JS) |
| WordPress Database | MySQL |
| Laravel Database | PostgreSQL |
| Laravel Frontend | Livewire, Alpine.js, Tailwind CSS |
| WordPress Storage | Local filesystem on VPS/VDS |
| Laravel Storage | Cloudflare R2 (S3-compatible) |
| CDN Strategy | R2 offloading for all high-traffic file/image requests |

### 1.2 Verdict Summary

| Dimension | Rating | Notes |
|-----------|--------|-------|
| **Compatibility** | ✅ Viable with caveats | PHP 8.4 is cutting-edge; requires Laravel 11.x+ and verified plugin support |
| **Security** | ✅ Strong | Dual-database isolation, R2 offloading, Cloudflare edge protection |
| **Performance** | ✅ Excellent | R2 CDN offloading dramatically reduces server I/O |
| **Scalability** | ✅ Adequate for MVP→Mid-scale | Single-server co-hosting supports the project's value tier; path to separation exists |
| **Cost Efficiency** | ✅ Optimal | R2 zero-egress + single VDS = minimal recurring cost |

### 1.3 Key Findings

1. **PHP 8.4 Strict Requirement** — PHP 8.4 is the latest stable release. Laravel 11.x supports it, but the team must verify all third-party Composer packages and WordPress plugins declare PHP 8.4 compatibility. A compatibility lockfile audit is mandatory before deployment.

2. **Dual-Database Architecture** — Running MySQL (WordPress) and PostgreSQL (Laravel) on a single server is fully supported. Resource allocation must be tuned to prevent memory contention, especially on the 8GB development VPS.

3. **R2 Offloading Strategy** — This is the architecture's strongest design decision. By routing all Laravel file/image traffic through Cloudflare R2 with a custom domain, the Ubuntu server handles only application logic. This effectively eliminates storage I/O as a scaling bottleneck.

4. **WordPress Local Storage** — Acceptable for the landing page's low-volume administrative uploads. However, media files served to public visitors should ideally be fronted by Cloudflare's CDN (proxy-enabled DNS records) to reduce origin requests.

5. **Single-Server Co-Hosting** — Viable for the project's current value tier (WordPress: KSH 70K, Laravel: KSH 180K). The Nginx reverse proxy cleanly separates the two applications by domain. The architecture includes a documented path to split into separate servers when traffic demands it.

---

## 2. Architecture Overview

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              EDUFIN PLATFORM ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│                              INTERNET / USERS                                   │
│                                  │                                              │
│                                  ▼                                              │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                              CLOUDFLARE                                  │  │
│  │                                                                          │  │
│  │  • DNS Management          • WAF (Web Application Firewall)             │  │
│  │  • SSL/TLS Termination     • DDoS Protection                            │  │
│  │  • CDN Caching             • Bot Protection                             │  │
│  │  • Rate Limiting           • Geo-blocking (optional)                    │  │
│  │                                                                          │  │
│  │  R2 Custom Domain: cdn.edufin.co.ke → Cloudflare R2 Bucket              │  │
│  │  (Serves all Laravel file/image assets with zero egress fees)           │  │
│  │                                                                          │  │
│  └──────────────────────────────────┬──────────────────────────────────────┘  │
│                                     │                                          │
│              ┌──────────────────────┼──────────────────────┐                  │
│              │                      │                      │                  │
│              ▼                      ▼                      ▼                  │
│     ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐         │
│     │ edufin.co.ke    │   │app.edufin.co.ke │   │cdn.edufin.co.ke │         │
│     │   WORDPRESS     │   │    LARAVEL      │   │  CLOUDFLARE R2  │         │
│     │  Landing Page   │   │  Portal + Admin │   │  Object Storage │         │
│     │  + /api/v1      │   │                 │   │                 │         │
│     └────────┬────────┘   └────────┬────────┘   └─────────────────┘         │
│              │                      │                      │                  │
│              ▼                      ▼                      │                  │
│  ┌─────────────────────────────────────────────────────────┐                 │
│  │                    UBUNTU 22.04 LTS                      │                 │
│  │                  (Contabo VPS / VDS)                    │                 │
│  │                                                          │                 │
│  │  ┌──────────────────────────────────────────────────┐  │                 │
│  │  │              NGINX (Reverse Proxy)                │  │                 │
│  │  │                                                  │  │                 │
│  │  │  • Routes by domain (SNI for TLS)                │  │                 │
│  │  │  • SSL via Let's Encrypt (certbot)               │  │                 │
│  │  │  • Gzip/Brotli compression                      │  │                 │
│  │  │  • Static file caching                          │  │                 │
│  │  └──────────────────────────────────────────────────┘  │                 │
│  │                    │                  │                 │                 │
│  │         ┌──────────┘                  └─────────┐      │                 │
│  │         ▼                                        ▼      │                 │
│  │  ┌──────────────────┐              ┌──────────────────┐│                 │
│  │  │   PHP-FPM 8.4    │              │   PHP-FPM 8.4    ││                 │
│  │  │   (WordPress)    │              │   (Laravel)      ││                 │
│  │  │   Pool: www-wp   │              │   Pool: www-lar  ││                 │
│  │  │   User: wpuser   │              │   User: laravel  ││                 │
│  │  └────────┬─────────┘              └────────┬─────────┘│                 │
│  │           │                                  │          │                 │
│  │           ▼                                  ▼          │                 │
│  │  ┌──────────────────┐              ┌──────────────────┐│                 │
│  │  │      MySQL       │              │   PostgreSQL     ││                 │
│  │  │   (WordPress)    │              │    (Laravel)     ││                 │
│  │  │  Content only    │              │  All business    ││                 │
│  │  │  No PII          │              │  data & PII      ││                 │
│  │  └──────────────────┘              └────────┬─────────┘│                 │
│  │                                              │          │                 │
│  │                                              ▼          │                 │
│  │                                    ┌──────────────────┐ │                 │
│  │                                    │      Redis       │ │                 │
│  │                                    │ Cache & Sessions │ │                 │
│  │                                    └──────────────────┘ │                 │
│  │                                                          │                 │
│  │  ┌──────────────────┐                                   │                 │
│  │  │  Local Storage   │                                   │                 │
│  │  │  (WordPress)     │                                   │                 │
│  │  │  /var/www/wp/    │                                   │                 │
│  │  │  wp-content/     │                                   │                 │
│  │  └──────────────────┘                                   │                 │
│  │                                                          │                 │
│  └──────────────────────────────────────────────────────────┘                 │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Data Flow Summary

| Flow | Source | Destination | Protocol | Purpose |
|------|--------|-------------|----------|---------|
| Public browsing | Browser | WordPress (via Nginx) | HTTPS | Marketing content |
| Portal access | Browser | Laravel (via Nginx) | HTTPS | Client portal |
| File/image serving | Browser/App | Cloudflare R2 | HTTPS | CDN-delivered assets |
| File upload | Laravel | Cloudflare R2 | HTTPS (S3 API) | Store documents |
| WordPress → Laravel | WordPress | Laravel | Standard HTML links | Links to login/register |
| Laravel → CBS | Laravel | Core Banking | HTTPS + mTLS | Financial operations |
| Mobile App → API | Flutter | Laravel API | HTTPS + JWT | Client functionality |

---

## 3. Compatibility Audit

### 3.1 PHP 8.4 Compatibility

**Status: ⚠️ Viable with mandatory verification**

PHP 8.4 is the latest stable PHP release. While it offers performance improvements and new features, its recency means not all ecosystem packages have declared explicit compatibility.

#### Laravel Stack Compatibility

| Component | PHP 8.4 Support | Action Required |
|-----------|-----------------|-----------------|
| Laravel 11.x | ✅ Supported | Use Laravel 11.x minimum |
| Livewire 3.x | ✅ Supported | Verify on install |
| Filament 3.x | ✅ Supported | Verify on install |
| Alpine.js | N/A (JS runtime) | No PHP dependency |
| Tailwind CSS | N/A (Build tool) | No PHP dependency |
| Sanctum | ✅ Supported | Bundled with Laravel |
| Horizon | ✅ Supported | Verify on install |
| Eloquent/PGSQL driver | ✅ Supported | `pdo_pgsql` extension |

#### WordPress Compatibility

| Component | PHP 8.4 Support | Action Required |
|-----------|-----------------|-----------------|
| WordPress 6.x Core | ✅ Supported (6.5+) | Use WordPress 6.5+ |
| Yoast SEO | ✅ Supported | Verify on install |
| WP Rocket | ✅ Supported | Verify on install |
| Wordfence | ✅ Supported | Verify on install |
| Custom EduFin plugins | ⚠️ Must verify | Test against PHP 8.4 |

#### Mandatory Pre-Deployment Checks

```bash
# 1. Check Composer platform requirements
composer check-platform-reqs

# 2. Audit all packages for PHP 8.4 compatibility
composer audit

# 3. Verify Laravel version supports PHP 8.4
php artisan --version
# Must be Laravel 11.x or higher

# 4. Run Laravel's compatibility checker
php artisan about

# 5. For WordPress, verify plugin compatibility
wp plugin list --fields=name,version,status
# Manually verify each plugin's "Tested up to" version
```

#### Risk: Deprecated Features in PHP 8.4

PHP 8.4 deprecates several features that may affect older code:

| Deprecated Feature | Impact | Mitigation |
|--------------------|--------|------------|
| Implicitly nullable parameter types | Warnings on `?` omission | Use explicit `?Type` or `Type|null` |
| `E_STRICT` merged into `E_WARNING` | Changed error reporting | Update error handling |
| Various date/time function changes | Minor | Test date-dependent code |

**Recommendation:** Set `error_reporting = E_ALL & ~E_DEPRECATED` in development to catch issues early, but allow deprecation warnings to surface during the audit phase.

### 3.2 Dual-Database Compatibility

**Status: ✅ Fully Supported**

Running MySQL and PostgreSQL on a single Ubuntu server is a well-established pattern. Both databases coexist without conflict.

#### Resource Considerations

| Database | Default Memory | Recommended Tuning (8GB VPS) | Recommended Tuning (24GB VDS) |
|----------|---------------|-----------------------------|-------------------------------|
| MySQL (WordPress) | ~400MB | `innodb_buffer_pool_size=512M` | `innodb_buffer_pool_size=1G` |
| PostgreSQL (Laravel) | ~256MB | `shared_buffers=512M` | `shared_buffers=4G` |
| Redis (Cache) | ~128MB | `maxmemory=256mb` | `maxmemory=1gb` |
| PHP-FPM (WordPress) | ~256MB | 4 workers × 64MB | 8 workers × 128MB |
| PHP-FPM (Laravel) | ~512MB | 8 workers × 64MB | 16 workers × 128MB |
| **Total Allocated** | — | **~2.1GB** | **~8.3GB** |
| **Remaining for OS** | — | **~5.9GB** ✅ | **~15.7GB** ✅ |

#### PHP Extensions Required

```bash
# Install all required PHP 8.4 extensions for both applications
sudo apt-get install -y \
    php8.4-fpm \
    php8.4-mysql \
    php8.4-pgsql \
    php8.4-redis \
    php8.4-xml \
    php8.4-curl \
    php8.4-gd \
    php8.4-mbstring \
    php8.4-zip \
    php8.4-bcmath \
    php8.4-intl \
    php8.4-opcache \
    php8.4-readline \
    php8.4-imagick \
    php8.4-ldap
```

### 3.3 Nginx + PHP-FPM Multi-Pool Configuration

**Status: ✅ Standard Pattern**

Nginx supports multiple PHP-FPM pools with separate configurations, users, and resource limits. This is the recommended approach for co-hosting.

```
/etc/php/8.4/fpm/pool.d/
├── www.conf          # Default (disabled or repurposed)
├── wordpress.conf    # WordPress pool
└── laravel.conf      # Laravel pool
```

Each pool runs as a separate Linux user, providing process-level isolation.

---

## 4. Infrastructure & Server Environment

### 4.1 Ubuntu 22.04 LTS Hardening

#### Initial Server Setup

```bash
#!/bin/bash
# Server hardening script - run as root

# 1. Update system
apt update && apt upgrade -y

# 2. Create dedicated users for each application
useradd -r -m -d /var/www/wordpress -s /bin/bash wpuser
useradd -r -m -d /var/www/laravel -s /bin/bash laravel

# 3. Install fail2ban for SSH protection
apt install -y fail2ban

# 4. Configure UFW firewall
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp        # SSH (consider changing port)
ufw allow 80/tcp        # HTTP
ufw allow 443/tcp       # HTTPS
ufw enable

# 5. Disable root SSH login
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
systemctl restart sshd

# 6. Install automatic security updates
apt install -y unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades

# 7. Install and configure AppArmor
apt install -y apparmor apparmor-utils
systemctl enable apparmor
systemctl start apparmor
```

### 4.2 Nginx Configuration

#### Main Nginx Configuration

```nginx
# /etc/nginx/nginx.conf

user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 2048;
    multi_accept on;
}

http {
    # Basic Settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;
    client_max_body_size 50M;  # For file uploads

    # Security Headers
    server {
        # Applied globally via include
    }

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # SSL Optimization
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml application/json
               application/javascript application/xml+rss
               application/atom+xml image/svg+xml;

    # Rate Limiting Zones
    limit_req_zone $binary_remote_addr zone=wordpress:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=laravel:10m rate=20r/s;
    limit_req_zone $binary_remote_addr zone=api:10m rate=60r/m;

    # Upstream Definitions (PHP-FPM pools)
    upstream wordpress-php {
        server unix:/run/php/php8.4-fpm-wordpress.sock;
    }

    upstream laravel-php {
        server unix:/run/php/php8.4-fpm-laravel.sock;
    }

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

#### WordPress Site Configuration

```nginx
# /etc/nginx/sites-available/edufin-wordpress.conf

server {
    listen 80;
    server_name edufin.co.ke www.edufin.co.ke;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name edufin.co.ke www.edufin.co.ke;

    # SSL certificates (Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/edufin.co.ke/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/edufin.co.ke/privkey.pem;

    # Root directory
    root /var/www/wordpress;
    index index.php index.html;

    # Rate limiting
    limit_req zone=wordpress burst=20 nodelay;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # WordPress permalinks
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    # Block sensitive files
    location ~* /(wp-config\.php|readme\.html|license\.txt) {
        deny all;
    }

    # Block PHP execution in uploads
    location ~* /wp-content/uploads/.*\.php$ {
        deny all;
    }

    # PHP processing
    location ~ \.php$ {
        fastcgi_pass wordpress-php;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_read_timeout 60;
    }

    # Static file caching
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # Let's Encrypt challenge
    location ^~ /.well-known/acme-challenge/ {
        root /var/www/letsencrypt;
    }
}
```

#### Laravel Site Configuration

```nginx
# /etc/nginx/sites-available/edufin-laravel.conf

server {
    listen 80;
    server_name app.edufin.co.ke;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name app.edufin.co.ke;

    # SSL certificates (Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/edufin.co.ke/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/edufin.co.ke/privkey.pem;

    # Root directory
    root /var/www/laravel/public;
    index index.php index.html;

    # Rate limiting
    limit_req zone=laravel burst=30 nodelay;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https://cdn.edufin.co.ke; font-src 'self' data:; connect-src 'self';" always;

    # Laravel routing
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # PHP processing
    location ~ \.php$ {
        fastcgi_pass laravel-php;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_read_timeout 120;
    }

    # Static asset caching (compiled assets)
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # Block access to sensitive files
    location ~ /\.(?!well-known).* {
        deny all;
    }

    # Block access to storage directory (files served via R2)
    location ^~ /storage/ {
        deny all;
        return 404;
    }

    # Let's Encrypt challenge
    location ^~ /.well-known/acme-challenge/ {
        root /var/www/letsencrypt;
    }
}
```

### 4.3 Let's Encrypt SSL Configuration

```bash
#!/bin/bash
# SSL certificate setup with Let's Encrypt

# 1. Install certbot
apt install -y certbot python3-certbot-nginx

# 2. Obtain certificates for all domains
certbot --nginx -d edufin.co.ke -d www.edufin.co.ke \
    --non-interactive --agree-tos --email admin@edufin.co.ke \
    --redirect

certbot --nginx -d app.edufin.co.ke \
    --non-interactive --agree-tos --email admin@edufin.co.ke \
    --redirect

# 3. Set up auto-renewal
echo "0 3 * * * certbot renew --quiet --post-hook 'systemctl reload nginx'" | \
    crontab -

# 4. Test renewal
certbot renew --dry-run
```

> **Note:** If Cloudflare is in "Full (Strict)" SSL mode, Let's Encrypt certificates on the origin are still required. Cloudflare terminates SSL at the edge and re-encrypts to the origin. Alternatively, Cloudflare Origin Certificates (15-year validity) can be used instead of Let's Encrypt to eliminate renewal management.

### 4.4 PHP-FPM Pool Configuration

#### WordPress Pool

```ini
# /etc/php/8.4/fpm/pool.d/wordpress.conf

[wordpress]
user = wpuser
group = wpuser
listen = /run/php/php8.4-fpm-wordpress.sock
listen.owner = www-data
listen.group = www-data

; Process management
pm = dynamic
pm.max_children = 8
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
pm.max_requests = 500

; Resource limits
php_admin_value[memory_limit] = 128M
php_admin_value[upload_max_filesize] = 50M
php_admin_value[post_max_size] = 50M
php_admin_value[max_execution_time] = 60

; Security
php_admin_flag[expose_php] = off
php_admin_value[open_basedir] = /var/www/wordpress:/tmp
```

#### Laravel Pool

```ini
# /etc/php/8.4/fpm/pool.d/laravel.conf

[laravel]
user = laravel
group = laravel
listen = /run/php/php8.4-fpm-laravel.sock
listen.owner = www-data
listen.group = www-data

; Process management (higher limits for Laravel)
pm = dynamic
pm.max_children = 15
pm.start_servers = 4
pm.min_spare_servers = 2
pm.max_spare_servers = 6
pm.max_requests = 1000

; Resource limits
php_admin_value[memory_limit] = 256M
php_admin_value[upload_max_filesize] = 50M
php_admin_value[post_max_size] = 50M
php_admin_value[max_execution_time] = 120

; Security
php_admin_flag[expose_php] = off
php_admin_value[open_basedir] = /var/www/laravel:/tmp

; OPcache (critical for performance)
php_admin_value[opcache.enable] = 1
php_admin_value[opcache.memory_consumption] = 128
php_admin_value[opcache.interned_strings_buffer] = 16
php_admin_value[opcache.max_accelerated_files] = 10000
php_admin_value[opcache.revalidate_freq] = 0
php_admin_value[opcache.validate_timestamps] = 0  ; Set to 0 in production
```

---

## 5. Application Stack Review

### 5.1 WordPress Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| WordPress | 6.5+ | CMS platform |
| PHP | 8.4 | Runtime (per requirement) |
| MySQL | 8.0 | Content database |
| Nginx | 1.24+ | Web server |
| Redis | 7.x | Object cache (optional) |

**Assessment:** WordPress 6.5+ runs on PHP 8.4. The stack is compatible. The only consideration is ensuring all third-party plugins declare PHP 8.4 compatibility.

### 5.2 Laravel Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| Laravel | 11.x | Application framework |
| PHP | 8.4 | Runtime (per requirement) |
| PostgreSQL | 16 | Primary database |
| Livewire | 3.x | Reactive UI components |
| Alpine.js | 3.x | Lightweight JS reactivity |
| Tailwind CSS | 3.x | Utility-first CSS |
| Filament | 3.x | Admin panel |
| Sanctum | 4.x | API authentication |
| Horizon | 5.x | Queue management |

**Assessment:** Laravel 11.x fully supports PHP 8.4. The Livewire + Alpine.js + Tailwind CSS stack (the "TALL stack") is the recommended modern Laravel frontend. Filament 3.x is built on this same stack, ensuring consistency.

### 5.3 Frontend Build Pipeline

```json
// package.json (Laravel)
{
  "scripts": {
    "dev": "vite",
    "build": "vite build"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "laravel-vite-plugin": "^1.0.0",
    "tailwindcss": "^3.4.0",
    "alpinejs": "^3.14.0",
    "@tailwindcss/forms": "^0.5.7"
  }
}
```

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
            ],
            refresh: true,
        }),
    ],
});
```

---

## 6. Storage & Content Delivery Strategy

### 6.1 Strategy Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        STORAGE & CDN STRATEGY                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  WORDPRESS STORAGE                                                              │
│  ═══════════════════                                                            │
│  • Local filesystem: /var/www/wordpress/wp-content/uploads/                    │
│  • Served via Nginx (with Cloudflare CDN proxy in front)                       │
│  • Low volume: admin uploads, marketing images                                 │
│  • Backed up via rsync to external storage                                     │
│                                                                                 │
│  LARAVEL STORAGE                                                                │
│  ═════════════════                                                              │
│  • Cloudflare R2: All file/image storage                                       │
│  • Served via R2 custom domain: cdn.edufin.co.ke                              │
│  • High volume: KYC documents, statements, profile images                      │
│  • Zero egress fees, global CDN delivery                                       │
│  • Server NEVER serves these files directly                                    │
│                                                                                 │
│  ARCHITECTURE GOAL ACHIEVED:                                                    │
│  • Server handles ONLY application logic                                        │
│  • All file/image I/O offloaded to Cloudflare R2                               │
│  • Minimal disk I/O on the Ubuntu server                                       │
│  • Mobile app fetches files directly from R2 (not the server)                  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Cloudflare R2 Configuration

#### R2 Bucket Setup

```bash
# Using Cloudflare dashboard or API:

# 1. Create R2 bucket
# Bucket name: edufin-assets
# Location: Automatic (or closest to users)

# 2. Enable public access via custom domain
# Custom domain: cdn.edufin.co.ke
# This creates a Cloudflare DNS record pointing to R2

# 3. Create API token for Laravel
# Token name: laravel-app
# Permissions: Object Read & Write
# Bucket: edufin-assets
# Save: Access Key ID, Secret Access Key, Account ID
```

#### Laravel R2 Configuration

```php
// config/filesystems.php

'disks' => [
    // Default disk set to R2
    'default' => env('FILESYSTEM_DISK', 'r2'),

    'r2' => [
        'driver' => 's3',
        'key' => env('R2_ACCESS_KEY_ID'),
        'secret' => env('R2_SECRET_ACCESS_KEY'),
        'region' => 'auto',
        'bucket' => env('R2_BUCKET', 'edufin-assets'),
        'endpoint' => env('R2_ENDPOINT'),
        'url' => env('R2_URL', 'https://cdn.edufin.co.ke'),
        'visibility' => 'private',  // Files are private by default
        'throw' => true,
    ],

    // For publicly accessible files (profile images, etc.)
    'r2-public' => [
        'driver' => 's3',
        'key' => env('R2_ACCESS_KEY_ID'),
        'secret' => env('R2_SECRET_ACCESS_KEY'),
        'region' => 'auto',
        'bucket' => env('R2_BUCKET', 'edufin-assets'),
        'endpoint' => env('R2_ENDPOINT'),
        'url' => env('R2_URL', 'https://cdn.edufin.co.ke'),
        'visibility' => 'public',
        'throw' => true,
    ],
],
```

```bash
# .env

R2_ACCESS_KEY_ID=your_access_key
R2_SECRET_ACCESS_KEY=your_secret_key
R2_BUCKET=edufin-assets
R2_ENDPOINT=https://your-account-id.r2.cloudflarestorage.com
R2_URL=https://cdn.edufin.co.ke
FILESYSTEM_DISK=r2
```

#### Document Upload Service

```php
// app/Services/Document/DocumentStorageService.php

<?php

namespace App\Services\Document;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Str;

class DocumentStorageService
{
    /**
     * Upload a file to R2 (private by default).
     * Returns the storage path for database reference.
     */
    public function uploadPrivate(UploadedFile $file, string $directory): array
    {
        $fileName = Str::uuid() . '.' . $file->getClientOriginalExtension();
        $path = $file->storeAs($directory, $fileName, 'r2');

        return [
            'path' => $path,
            'disk' => 'r2',
            'mime_type' => $file->getMimeType(),
            'size' => $file->getSize(),
            'original_name' => $file->getClientOriginalName(),
        ];
    }

    /**
     * Upload a public file (e.g., profile image) to R2.
     * Returns the full CDN URL for direct access.
     */
    public function uploadPublic(UploadedFile $file, string $directory): array
    {
        $fileName = Str::uuid() . '.' . $file->getClientOriginalExtension();
        $path = $file->storeAs($directory, $fileName, 'r2-public');

        return [
            'path' => $path,
            'url' => Storage::disk('r2-public')->url($path),
            'mime_type' => $file->getMimeType(),
            'size' => $file->getSize(),
        ];
    }

    /**
     * Generate a temporary signed URL for private file access.
     * URL expires after specified minutes.
     */
    public function temporaryUrl(string $path, int $minutes = 15): string
    {
        return Storage::disk('r2')->temporaryUrl(
            $path,
            now()->addMinutes($minutes)
        );
    }

    /**
     * Stream a file response (for inline viewing).
     */
    public function streamFile(string $path)
    {
        return Storage::disk('r2')->response($path);
    }

    /**
     * Delete a file from R2.
     */
    public function delete(string $path): bool
    {
        return Storage::disk('r2')->delete($path);
    }
}
```

### 6.3 R2 CDN Delivery for Mobile App

The mobile app (Flutter) fetches files directly from R2 via the CDN domain, **never** from the Laravel server:

```
Mobile App File Flow:
─────────────────────

1. App requests file metadata from Laravel API:
   GET /api/v1/documents/{id}
   → Returns: { "path": "kyc/uuid.pdf", "type": "private" }

2. App requests download URL from Laravel API:
   POST /api/v1/documents/{id}/url
   → Returns: { "url": "https://cdn.edufin.co.ke/kyc/uuid.pdf?X-Amz-Signature=..." }

3. App downloads file DIRECTLY from R2 CDN:
   GET https://cdn.edufin.co.ke/kyc/uuid.pdf?X-Amz-Signature=...
   → File served from Cloudflare's global edge network
   → Laravel server NOT involved in file transfer
```

### 6.4 WordPress Storage Strategy

WordPress uses local storage for marketing content (low volume). Cloudflare's CDN proxy caches these files at the edge:

```nginx
# WordPress static file caching (in Nginx config)
location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

**Cloudflare DNS for WordPress:**
- DNS record: `edufin.co.ke` → A record → Server IP
- Cloudflare proxy: **Enabled** (orange cloud)
- This caches WordPress static assets at Cloudflare's edge

**Backup Strategy for WordPress uploads:**
```bash
# Daily backup of WordPress uploads to Backblaze B2 or R2
0 2 * * * rsync -avz /var/www/wordpress/wp-content/uploads/ \
    backup@storage::wordpress-uploads/$(date +\%Y-\%m-\%d)/
```

---

## 7. Cross-System Integration

### 7.1 File Upload Boundaries

| System | Storage Location | Volume | Content Type |
|--------|-----------------|--------|--------------|
| WordPress | Local filesystem | Low | Marketing images, blog media |
| Laravel | Cloudflare R2 | High | KYC documents, statements, profile images |

**Rule:** WordPress NEVER stores user documents or PII. All client-facing file uploads go through Laravel and are stored in R2.

### 7.2 Data Consistency Between Systems

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      DATA CONSISTENCY MODEL                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PRINCIPLE: Laravel is the source of truth for ALL business data.              │
│  WordPress is the source of truth for content only.                            │
│                                                                                 │
│  ┌─────────────────┐                    ┌─────────────────────────────────┐    │
│  │   WORDPRESS     │                    │   LARAVEL                       │    │
│  │   MySQL         │                    │   PostgreSQL                    │    │
│  │                 │                    │                                 │    │
│  │  • Posts        │   NO SHARED        │  • Users                        │    │
│  │  • Pages        │   DATABASE         │  • Account Holders              │    │
│  │  • Media        │   TABLES           │  • Loans                        │    │
│  │  • Settings     │                    │  • Documents                    │    │
│  │  • Menus        │                    │  • Payments                     │    │
│  │                 │                    │  • Audit Logs                   │    │
│  └────────┬────────┘                    └────────┬────────────────────────┘    │
│           │                                       │                            │
│           │       No API Communication           │                            │
│           │       (Independent Systems)           │                            │
│           │                                       │                            │
│           └───────────────┬───────────────────────┘                            │
│                           │                                                    │
│                           ▼                                                    │
│                   ┌──────────────────┐                                        │
│                   │  NO INTEGRATION  │                                        │
│                   │  (Independent    │                                        │
│                   │   Systems)       │                                        │
│                   │                  │                                        │
│                   │  WordPress links │                                        │
│                   │  to app.edufin   │                                        │
│                   │  .co.ke/login    │                                        │
│                   └──────────────────┘                                        │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 7.3 Integration Points

The two systems (WordPress and Laravel) are **independent**. There is no SSO or API integration between them. WordPress links to the Laravel application via standard HTML links:

- **Login:** `app.edufin.co.ke/login`
- **Register:** `app.edufin.co.ke/register`

WordPress does NOT consume Laravel APIs. All authentication and business logic reside entirely within Laravel.

### 7.4 File Upload Best Practices

#### Laravel (R2) Upload Flow

```
1. Client uploads file via portal or API
2. Laravel validates file (type, size, content)
3. Laravel uploads to R2 via S3-compatible API
4. Laravel stores metadata in PostgreSQL (path, type, size)
5. Laravel returns CDN URL or signed URL to client
6. Client accesses file via R2 CDN (not the server)
```

#### WordPress (Local) Upload Flow

```
1. Admin uploads image via WordPress media library
2. WordPress stores file in /var/www/wordpress/wp-content/uploads/
3. Nginx serves the file (with Cloudflare CDN caching)
4. No client-facing uploads allowed through WordPress
```

---

## 8. Performance Optimization

### 8.1 Nginx Performance Tuning

```nginx
# Key performance settings in nginx.conf

http {
    # Enable Brotli compression (if module installed)
    brotli on;
    brotli_comp_level 6;
    brotli_types text/plain text/css application/json application/javascript
                 application/xml image/svg+xml;

    # Connection optimization
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    keepalive_requests 1000;

    # Buffer optimization
    client_body_buffer_size 16k;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 8k;

    # Open file cache
    open_file_cache max=1000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
}
```

### 8.2 PHP OPcache Configuration

```ini
; /etc/php/8.4/fpm/conf.d/10-opcache.ini

opcache.enable=1
opcache.enable_cli=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=32
opcache.max_accelerated_files=20000
opcache.revalidate_freq=0
opcache.validate_timestamps=0  ; Production: 0 (deploy clears cache)
opcache.fast_shutdown=1
opcache.jit=tracing
opcache.jit_buffer_size=128M
```

### 8.3 R2 Offloading Impact

```
WITHOUT R2 OFFLOADING:
─────────────────────
Server handles: Application logic + File serving + Image delivery
Server I/O: HIGH (every file request hits disk)
Bandwidth: HIGH (all file transfers through server)
Mobile app: Downloads files from server (slow, server-bound)

WITH R2 OFFLOADING:
───────────────────
Server handles: Application logic ONLY
Server I/O: MINIMAL (no file serving)
Bandwidth: LOW (only API responses)
Mobile app: Downloads files from R2 CDN (fast, global edge)
R2 Egress: $0 (zero egress fees)
```

### 8.4 Database Performance

#### PostgreSQL Tuning (24GB VDS)

```ini
# /etc/postgresql/16/main/postgresql.conf

# Memory
shared_buffers = 4GB
effective_cache_size = 12GB
work_mem = 64MB
maintenance_work_mem = 512MB

# WAL
wal_buffers = 16MB
max_wal_size = 2GB
min_wal_size = 256MB

# Query Planning
random_page_cost = 1.1  # NVMe has near-random access
effective_io_concurrency = 200

# Connections
max_connections = 100
```

#### MySQL Tuning (WordPress)

```ini
# /etc/mysql/mysql.conf.d/mysqld.cnf

innodb_buffer_pool_size = 1G
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT
max_connections = 50
query_cache_size = 0  # Disabled in MySQL 8.0+
```

### 8.5 Redis Caching

```ini
# /etc/redis/redis.conf

maxmemory 1gb
maxmemory-policy allkeys-lru
save ""  # Disable persistence (cache only)
```

### 8.6 Cloudflare CDN Configuration

| Setting | Value | Purpose |
|---------|-------|---------|
| SSL Mode | Full (Strict) | Encrypt to origin |
| Always Use HTTPS | On | Redirect HTTP to HTTPS |
| Auto Minify | CSS, JS, HTML | Reduce file sizes |
| Brotli | On | Compress responses |
| Caching Level | Standard | Cache static resources |
| Browser Cache TTL | 1 year | Long cache for static assets |
| Upload Max Size | 100MB | Allow large file uploads |

---

## 9. Security Review

### 9.1 Security Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           SECURITY LAYERS                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  LAYER 1: CLOUDFLARE EDGE                                                       │
│  • WAF (OWASP Core Rule Set)                                                   │
│  • DDoS Protection (L3/L4/L7)                                                  │
│  • Bot Management                                                               │
│  • Rate Limiting                                                                │
│  • SSL/TLS Termination                                                          │
│  • IP Reputation Filtering                                                      │
│                                                                                 │
│  LAYER 2: UBUNTU SERVER                                                         │
│  • UFW Firewall (22, 80, 443 only)                                             │
│  • Fail2ban (SSH brute-force protection)                                       │
│  • Automatic Security Updates                                                   │
│  • AppArmor (application confinement)                                          │
│  • SSH key-only authentication                                                  │
│  • Root login disabled                                                          │
│                                                                                 │
│  LAYER 3: NGINX                                                                 │
│  • Rate limiting per zone                                                       │
│  • Security headers (HSTS, X-Frame-Options, etc.)                              │
│  • Sensitive file blocking                                                      │
│  • PHP execution blocked in upload directories                                  │
│  • Server tokens hidden                                                         │
│                                                                                 │
│  LAYER 4: PHP-FPM                                                               │
│  • Separate pools (process isolation)                                          │
│  • Separate Linux users (filesystem isolation)                                 │
│  • open_basedir restrictions                                                    │
│  • Resource limits (memory, execution time)                                    │
│  • expose_php disabled                                                          │
│                                                                                 │
│  LAYER 5: DATABASE                                                              │
│  • PostgreSQL: pg_hba.conf (local connections only)                            │
│  • MySQL: bind-address 127.0.0.1                                               │
│  • Separate database users with minimal privileges                              │
│  • Encrypted connections (TLS)                                                 │
│  • Regular backups with encryption                                              │
│                                                                                 │
│  LAYER 6: APPLICATION                                                           │
│  • Laravel: CSRF, XSS, SQLi prevention (built-in)                              │
│  • Laravel: RBAC, Policies, Gates                                               │
│  • Laravel: PII encryption at application level                                 │
│  • Laravel: Audit logging                                                       │
│  • WordPress: Minimal plugins, regular updates                                  │
│  • WordPress: File editing disabled                                             │
│                                                                                 │
│  LAYER 7: STORAGE                                                               │
│  • R2: Private buckets by default                                               │
│  • R2: Signed URLs for private file access                                      │
│  • R2: API tokens with minimal scope                                            │
│  • WordPress: Local files backed up externally                                  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 9.2 Database Security

#### PostgreSQL Security

```ini
# /etc/postgresql/16/main/postgresql.conf

listen_addresses = 'localhost'  # Only local connections
ssl = on
password_encryption = scram-sha-256
```

```ini
# /etc/postgresql/16/main/pg_hba.conf

# TYPE  DATABASE     USER      ADDRESS       METHOD
local   all          postgres                peer
local   all          laravel                 md5
host    all          all       127.0.0.1/32  scram-sha-256
host    all          all       ::1/128       scram-sha-256
# Reject all external connections
host    all          all       0.0.0.0/0     reject
```

#### MySQL Security

```ini
# /etc/mysql/mysql.conf.d/mysqld.cnf

bind-address = 127.0.0.1
mysqlx-bind-address = 127.0.0.1
```

### 9.3 R2 Security

| Setting | Value | Purpose |
|---------|-------|---------|
| Bucket visibility | Private | Prevent public access to sensitive files |
| API token scope | Object Read & Write only | Minimal permissions |
| Signed URLs | 15-minute expiry | Time-limited access to private files |
| CORS | Restricted to edufin.co.ke domains | Prevent cross-origin abuse |

### 9.4 Laravel Application Security

```php
// app/Http/Middleware/SecurityHeaders.php

<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class SecurityHeaders
{
    public function handle(Request $request, Closure $next)
    {
        $response = $next($request);

        $response->headers->set('X-Content-Type-Options', 'nosniff');
        $response->headers->set('X-Frame-Options', 'SAMEORIGIN');
        $response->headers->set('X-XSS-Protection', '1; mode=block');
        $response->headers->set('Referrer-Policy', 'strict-origin-when-cross-origin');
        $response->headers->set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
        $response->headers->set('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');

        return $response;
    }
}
```

### 9.5 WordPress Security

```php
// wp-config.php security settings

define('DISALLOW_FILE_EDIT', true);    // No theme/plugin editor
define('DISALLOW_FILE_MODS', true);    // No plugin installation via admin
define('FORCE_SSL_ADMIN', true);       // HTTPS for admin
define('WP_AUTO_UPDATE_CORE', 'minor'); // Auto-update core
$table_prefix = 'edf_';                // Non-default prefix

// Disable XML-RPC (add to .htaccess or Nginx)
// Block author enumeration
// Limit login attempts (via plugin or Nginx)
```

```nginx
# Block XML-RPC in Nginx
location = /xmlrpc.php {
    deny all;
    access_log off;
    log_not_found off;
}
```

---

## 10. Scalability Assessment

### 10.1 Current Architecture Capacity

| Metric | Development (VPS) | Production (VDS) |
|--------|-------------------|------------------|
| **Concurrent Users** | ~50 | ~500+ |
| **Requests/sec** | ~100 | ~1,000 |
| **Database Connections** | 50 (MySQL) + 50 (PG) | 50 (MySQL) + 100 (PG) |
| **File Storage** | 100GB SSD | 180GB NVMe + R2 (unlimited) |
| **RAM Available** | 8GB | 24GB |

### 10.2 Scalability Analysis

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          SCALABILITY PATH                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  STAGE 1: CURRENT (Single VDS)                                                  │
│  ────────────────────────────────────                                           │
│  • WordPress + Laravel + MySQL + PostgreSQL + Redis on one server              │
│  • Capacity: ~500 concurrent users                                              │
│  • R2 handles all file I/O (no scaling needed for storage)                     │
│  • Cost: ~$40/month                                                             │
│                                                                                 │
│  STAGE 2: SEPARATED DATABASES (When DB becomes bottleneck)                      │
│  ──────────────────────────────────────────────────────────────                 │
│  • Move PostgreSQL to managed database service                                  │
│  • Keep MySQL on the VDS (low load)                                             │
│  • Capacity: ~1,000 concurrent users                                            │
│  • Additional cost: ~$60/month (managed PG)                                     │
│                                                                                 │
│  STAGE 3: SEPARATED APPLICATIONS (When CPU becomes bottleneck)                  │
│  ────────────────────────────────────────────────────────────────────           │
│  • WordPress on one VPS                                                         │
│  • Laravel on one VDS                                                           │
│  • Managed databases                                                            │
│  • Capacity: ~2,000 concurrent users                                            │
│  • Additional cost: ~$100/month total                                           │
│                                                                                 │
│  STAGE 4: LOAD BALANCED (When single Laravel server is insufficient)            │
│  ──────────────────────────────────────────────────────────────────────         │
│  • Multiple Laravel servers behind load balancer                                │
│  • WordPress on dedicated VPS                                                   │
│  • Managed databases with read replicas                                         │
│  • Capacity: ~10,000+ concurrent users                                          │
│  • Additional cost: ~$300+/month                                                │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 10.3 R2's Role in Scalability

R2 is the **single most important scalability decision** in this architecture:

| Benefit | Impact |
|---------|--------|
| **Zero egress fees** | No cost penalty for high file transfer volume |
| **Global CDN** | Files served from edge locations worldwide |
| **Unlimited storage** | No storage capacity planning needed |
| **Server I/O elimination** | Server CPU/RAM freed for application logic |
| **Mobile app offloading** | App downloads files from R2, not the server |

**Without R2:** The server would handle all file requests, requiring significant bandwidth and I/O capacity. At 500+ users downloading documents, the server would become I/O-bound.

**With R2:** The server handles only API requests (small JSON payloads). File transfers are completely decoupled from server capacity.

### 10.4 Project Value Assessment

| Component | Value | Architecture Adequacy |
|-----------|-------|----------------------|
| WordPress | KSH 70,000 | ✅ Single VPS/VDS is more than sufficient |
| Laravel | KSH 180,000 | ✅ VDS + R2 + managed DB path supports growth |
| **Total** | **KSH 250,000** | ✅ Architecture matches investment level |

**Assessment:** The architecture is appropriately scaled for the project's value tier. The R2 offloading strategy provides enterprise-grade file delivery at zero marginal cost, which is exceptional value for a KSH 250K project. The scalability path allows growth without architectural rewrites.

---

## 11. Implementation Guide

### 11.1 Server Setup Sequence

```
PHASE 1: SERVER PROVISIONING
─────────────────────────────
1. Provision Contabo VPS/VDS with Ubuntu 22.04 LTS
2. SSH into server with root credentials
3. Run server hardening script (Section 4.1)
4. Create application users (wpuser, laravel)

PHASE 2: SOFTWARE INSTALLATION
───────────────────────────────
5. Add PHP 8.4 repository (ondrej/php PPA)
6. Install PHP 8.4 and all extensions (Section 3.2)
7. Install Nginx
8. Install MySQL 8.0
9. Install PostgreSQL 16
10. Install Redis 7.x
11. Install Composer
12. Install Node.js 20 LTS and NPM

PHASE 3: DATABASE SETUP
────────────────────────
13. Secure MySQL installation (mysql_secure_installation)
14. Create WordPress database and user
15. Secure PostgreSQL installation
16. Create Laravel database and user
17. Configure PostgreSQL pg_hba.conf (Section 9.2)

PHASE 4: APPLICATION DEPLOYMENT
───────────────────────────────
18. Clone WordPress to /var/www/wordpress
19. Configure wp-config.php (Section 9.5)
20. Clone Laravel to /var/www/laravel
21. Run composer install
22. Run npm install && npm run build
23. Configure .env (database, R2, Redis)
24. Run php artisan migrate
25. Run php artisan key:generate
26. Run php artisan storage:link (if needed)

PHASE 5: PHP-FPM CONFIGURATION
───────────────────────────────
27. Configure WordPress pool (Section 4.4)
28. Configure Laravel pool (Section 4.4)
29. Configure OPcache (Section 8.2)
30. Restart PHP-FPM

PHASE 6: NGINX CONFIGURATION
─────────────────────────────
31. Configure main nginx.conf (Section 4.2)
32. Configure WordPress site (Section 4.2)
33. Configure Laravel site (Section 4.2)
34. Enable sites (symlink to sites-enabled)
35. Test: nginx -t
36. Restart Nginx

PHASE 7: SSL/TLS SETUP
──────────────────────
37. Install certbot (Section 4.3)
38. Obtain Let's Encrypt certificates
39. Set up auto-renewal cron
40. Configure Cloudflare SSL mode (Full Strict)

PHASE 8: CLOUDFLARE R2 SETUP
────────────────────────────
41. Create R2 bucket (edufin-assets)
42. Configure R2 custom domain (cdn.edufin.co.ke)
43. Create R2 API token
44. Configure Laravel filesystems.php (Section 6.2)
45. Test file upload to R2
46. Test file retrieval from CDN URL

PHASE 9: CLOUDFLARE DNS
───────────────────────
47. Configure DNS records (Section 2.1)
48. Enable Cloudflare proxy (orange cloud) for all records
49. Configure WAF rules
50. Configure rate limiting rules
51. Configure page rules for caching

PHASE 10: FINAL VERIFICATION
────────────────────────────
52. Test WordPress site (https://edufin.co.ke)
53. Test Laravel portal (https://app.edufin.co.ke) and admin panel (https://app.edufin.co.ke/admin)
54. Test API (https://edufin.co.ke/api/v1/packages)
55. Test R2 CDN (https://cdn.edufin.co.ke/test-file.txt)
56. Test SSL Labs grade (A+ expected)
57. Verify backups are configured
```

### 11.2 PHP 8.4 Installation on Ubuntu 22.04

```bash
#!/bin/bash
# PHP 8.4 is not in Ubuntu 22.04's default repositories.
# Use the ondrej/php PPA which provides PHP 8.4 packages.

# 1. Add PPA
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

# 2. Install PHP 8.4 and FPM
sudo apt install -y \
    php8.4-fpm \
    php8.4-cli \
    php8.4-common \
    php8.4-mysql \
    php8.4-pgsql \
    php8.4-redis \
    php8.4-xml \
    php8.4-curl \
    php8.4-gd \
    php8.4-mbstring \
    php8.4-zip \
    php8.4-bcmath \
    php8.4-intl \
    php8.4-opcache \
    php8.4-readline \
    php8.4-imagick

# 3. Verify installation
php -v
# Should show PHP 8.4.x

# 4. Install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# 5. Install Node.js 20 LTS
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# 6. Verify
node -v  # v20.x.x
npm -v   # 10.x.x
composer --version
```

### 11.3 Deployment Script (Laravel)

```bash
#!/bin/bash
# /var/www/laravel/deploy.sh

set -e

APP_DIR="/var/www/laravel"
cd $APP_DIR

echo "==> Pulling latest code..."
git pull origin main

echo "==> Installing dependencies..."
composer install --no-dev --optimize-autoloader

echo "==> Building frontend assets..."
npm ci
npm run build

echo "==> Running migrations..."
php artisan migrate --force

echo "==> Clearing and caching..."
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache

echo "==> Restarting services..."
php artisan horizon:terminate
sudo systemctl reload php8.4-fpm
sudo systemctl reload nginx

echo "==> Deployment complete!"
```

---

## 12. Risk Register & Mitigations

| # | Risk | Probability | Impact | Mitigation |
|---|------|-------------|--------|------------|
| 1 | PHP 8.4 incompatibility with a package | Medium | High | Run `composer audit` before deployment; pin versions |
| 2 | WordPress plugin incompatible with PHP 8.4 | Low | Medium | Verify each plugin's "Tested up to" version |
| 3 | Memory contention on 8GB VPS | Medium | Medium | Tune database memory settings (Section 3.2) |
| 4 | R2 API token compromise | Low | High | Use minimal-scope tokens; rotate quarterly |
| 5 | Let's Encrypt certificate expiry | Low | High | Set up auto-renewal cron; monitor |
| 6 | Single server failure | Medium | High | Daily backups; documented recovery procedure |
| 7 | Cloudflare outage | Very Low | High | DNS failover plan; R2 has independent SLA |
| 8 | Database corruption | Low | Critical | Daily automated backups; point-in-time recovery |

---

## 13. Recommendations Summary

### 13.1 Architecture Strengths

1. **R2 Offloading** — Eliminates storage I/O as a bottleneck; zero egress cost
2. **Dual-Database Isolation** — Clean separation of content and business data
3. **PHP-FPM Pool Separation** — Process and filesystem isolation between apps
4. **Cloudflare Edge Protection** — Enterprise-grade security at no additional cost
5. **API-First Laravel** — Mobile app integration without additional backend work

### 13.2 Critical Recommendations

| Priority | Recommendation |
|----------|----------------|
| **P0** | Run PHP 8.4 compatibility audit before deployment |
| **P0** | Configure R2 with private buckets and signed URLs |
| **P0** | Implement database memory tuning for the VPS (8GB) |
| **P1** | Set up automated backups for both databases |
| **P1** | Configure Cloudflare WAF with OWASP Core Rule Set |
| **P1** | Implement monitoring (Uptime Robot + Sentry) |
| **P2** | Consider Cloudflare Origin Certificates instead of Let's Encrypt |
| **P2** | Implement CI/CD pipeline for automated deployments |
| **P2** | Set up log aggregation for security monitoring |

### 13.3 Final Verdict

| Dimension | Score (1-10) | Notes |
|-----------|:------------:|-------|
| Compatibility | 8/10 | PHP 8.4 is cutting-edge but supported |
| Security | 9/10 | Multi-layered, defense-in-depth |
| Performance | 9/10 | R2 offloading is a game-changer |
| Scalability | 8/10 | Clear path from single-server to distributed |
| Cost Efficiency | 10/10 | R2 zero-egress + single VDS = minimal cost |
| **Overall** | **8.8/10** | **Production-ready with documented mitigations** |

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-08-06 | EduFin Team | Initial technical architecture review |

---

*This document provides the comprehensive technical architecture review and implementation guide for the EduFin dual-system ecosystem. It should be used as the authoritative reference for all infrastructure, security, and deployment decisions.*

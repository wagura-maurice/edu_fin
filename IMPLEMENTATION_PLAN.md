# EduFin Platform - Dual-Platform Implementation Plan

## WordPress (edufin.co.ke) + Laravel (app.edufin.co.ke) — Two Independent Systems with Path-Based API

**Date:** August 6, 2026  
**Version:** 1.0  
**Status:** Technical Specification

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Infrastructure & Deployment](#2-infrastructure--deployment)
3. [Authentication Strategy](#3-authentication-strategy)
4. [Data Integration Layer](#4-data-integration-layer)
5. [User Experience Consistency](#5-user-experience-consistency)
6. [Security Implementation](#6-security-implementation)
7. [Development Workflow](#7-development-workflow)
8. [Monitoring & Operations](#8-monitoring--operations)

---

## 1. Architecture Overview

### 1.1 High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              EDUFIN DUAL-PLATFORM ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│                                    CLOUDFLARE                                       │
│                          (DNS, CDN, WAF, SSL Termination)                          │
│                                        │                                            │
│              ┌─────────────────────────┼─────────────────────────┐                 │
│              │                         │                         │                  │
│              ▼                         ▼                         ▼                  │
│     ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐         │
│     │  edufin.co.ke   │      │ app.edufin.co.ke│      │ cdn.edufin.co.ke│         │
│     │  (WordPress +   │      │    (Laravel)    │      │  (Cloudflare R2)│         │
│     │   /api/v1 path) │      │  Portal + Admin │      │  Static Assets  │         │
│     └────────┬────────┘      └────────┬────────┘      └─────────────────┘         │
│              │                        │                                              │
│              │                        │                                              │
│              ▼                        ▼                                              │
│     ┌─────────────────┐     ┌─────────────────────────────────┐                    │
│     │   WordPress     │     │        LARAVEL SERVER           │                    │
│     │    Server       │     │  ┌───────────┐ ┌───────────┐   │                    │
│     │                 │     │  │  Portal   │ │    API    │   │                    │
│     │ • Marketing     │     │  │  (Web)    │ │  (Mobile) │   │                    │
│     │ • Blog/News     │     │  └─────┬─────┘ └─────┬─────┘   │                    │
│     │ • SEO Content   │     │        └──────┬──────┘         │                    │
│     │ • Landing Pages │     │               ▼                │                    │
│     └────────┬────────┘     │  ┌─────────────────────────┐  │                    │
│              │              │  │    Business Services    │  │                    │
│              │              │  │  • KYC  • Loans  • Docs │  │                    │
│              │              │  └───────────┬─────────────┘  │                    │
│              │              └──────────────┼────────────────┘                    │
│              │                             │                                       │
│              ▼                             ▼                                       │
│     ┌─────────────────┐         ┌─────────────────────────────────┐              │
│     │   MySQL/MariaDB │         │          PostgreSQL             │              │
│     │   (WP Content)  │         │    (Application Data)           │              │
│     └─────────────────┘         └─────────────────────────────────┘              │
│                                                                                     │
│  NOTE: edufin.co.ke serves WordPress at root; /api/v1/* is reverse-proxied         │
│        to the Laravel server (path-based API routing on the main domain).          │
│                                                                                     │
│                              SHARED SERVICES                                      │
│     ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐               │
│     │      Redis      │  │   Cloudflare R2 │  │   Email/SMS     │               │
│     │  (Sessions,     │  │   (File Storage)│  │   (Notifications│               │
│     │   Cache)        │  │                 │  │    Services)    │               │
│     └─────────────────┘  └─────────────────┘  └─────────────────┘               │
│                                                                                   │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Domain Structure

| Domain | Platform | Purpose |
|--------|----------|---------|
| `edufin.co.ke` | WordPress + Laravel API (path) | Marketing, SEO, Blog, Public Content; `/api/v1/*` reverse-proxied to Laravel |
| `app.edufin.co.ke` | Laravel | Client Portal, Dashboard, KYC, Loans; Filament admin at `/admin` |
| `cdn.edufin.co.ke` | Cloudflare R2 | Static Assets, Media Files |

### 1.3 Technology Stack

| Component | WordPress Site | Laravel Application |
|-----------|---------------|---------------------|
| **Runtime** | PHP 8.2 | PHP 8.3 |
| **Framework** | WordPress 6.x | Laravel 11.x |
| **Database** | MySQL 8.0 | PostgreSQL 16 |
| **Cache** | Redis (Object Cache) | Redis (Cache + Sessions) |
| **Web Server** | Nginx | Nginx |
| **Process Manager** | PHP-FPM | PHP-FPM + Supervisor |
| **Queue** | - | Laravel Horizon (Redis) |

---

## 2. Infrastructure & Deployment

### 2.1 DNS Configuration (Cloudflare)

```
# DNS Records for edufin.co.ke

# Root domain - WordPress (with /api/v1 path reverse-proxied to Laravel)
edufin.co.ke          A       <WP_SERVER_IP>        Proxied (Orange Cloud)
www.edufin.co.ke      CNAME   edufin.co.ke          Proxied

# Application subdomain - Laravel (portal + admin panel)
app.edufin.co.ke      A       <LARAVEL_SERVER_IP>   Proxied

# CDN subdomain - R2 bucket
cdn.edufin.co.ke      CNAME   <R2_BUCKET>.r2.dev    Proxied

# Email records
edufin.co.ke          MX      10 mail.edufin.co.ke
edufin.co.ke          TXT     "v=spf1 include:_spf.google.com ~all"
```

### 2.2 SSL/TLS Configuration

```
# Cloudflare SSL Settings
SSL/TLS Mode: Full (Strict)
Minimum TLS Version: TLS 1.2
Automatic HTTPS Rewrites: ON
Always Use HTTPS: ON

# Origin Certificates
- Generate via Cloudflare Dashboard > SSL/TLS > Origin Server
- Valid for 15 years, only works with Cloudflare proxy
- Install on both WordPress and Laravel servers
```

### 2.3 Nginx Configuration - WordPress

```nginx
# /etc/nginx/sites-available/wordpress.conf

server {
    listen 443 ssl http2;
    server_name edufin.co.ke www.edufin.co.ke;
    
    root /var/www/wordpress;
    index index.php index.html;
    
    # Cloudflare Origin Certificate
    ssl_certificate /etc/ssl/cloudflare/edufin.co.ke.pem;
    ssl_certificate_key /etc/ssl/cloudflare/edufin.co.ke.key;
    
    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript;
    
    # Static File Caching
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|pdf|woff|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    
    # WordPress Permalinks
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    
    # PHP Processing
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_read_timeout 300;
    }
    
    # Block sensitive files
    location ~* ^/(wp-config\.php|readme\.html|license\.txt|xmlrpc\.php) {
        deny all;
    }
    
    # Block PHP in uploads
    location ~* /wp-content/uploads/.*\.php$ {
        deny all;
    }
}

server {
    listen 80;
    server_name edufin.co.ke www.edufin.co.ke;
    return 301 https://$server_name$request_uri;
}
```

### 2.4 Nginx Configuration - Laravel

```nginx
# /etc/nginx/sites-available/laravel.conf

server {
    listen 443 ssl http2;
    server_name app.edufin.co.ke;
    
    root /var/www/laravel/public;
    index index.php;
    
    # Cloudflare Origin Certificate
    ssl_certificate /etc/ssl/cloudflare/edufin.co.ke.pem;
    ssl_certificate_key /etc/ssl/cloudflare/edufin.co.ke.key;
    
    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Gzip
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript;
    
    # Laravel Routing
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    # PHP Processing
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_read_timeout 300;
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
    }
    
    # Block dotfiles
    location ~ /\. {
        deny all;
    }
}

server {
    listen 80;
    server_name app.edufin.co.ke;
    return 301 https://$server_name$request_uri;
}
```

### 2.5 Docker Compose (Development/Staging)

```yaml
# docker-compose.yml
version: '3.8'

services:
  # WordPress Stack
  wordpress:
    image: wordpress:php8.2-fpm
    container_name: edufin_wordpress
    restart: unless-stopped
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: ${WP_DB_USER}
      WORDPRESS_DB_PASSWORD: ${WP_DB_PASSWORD}
      WORDPRESS_DB_NAME: ${WP_DB_NAME}
    volumes:
      - wordpress_data:/var/www/html
      - ./wordpress/themes/edufin:/var/www/html/wp-content/themes/edufin
    networks:
      - edufin_network

  mysql:
    image: mysql:8.0
    container_name: edufin_mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${WP_DB_NAME}
      MYSQL_USER: ${WP_DB_USER}
      MYSQL_PASSWORD: ${WP_DB_PASSWORD}
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - edufin_network

  # Laravel Stack
  laravel:
    build:
      context: ./laravel
      dockerfile: Dockerfile
    container_name: edufin_laravel
    restart: unless-stopped
    environment:
      APP_ENV: production
      DB_CONNECTION: pgsql
      DB_HOST: postgres
      REDIS_HOST: redis
      SESSION_DRIVER: redis
      QUEUE_CONNECTION: redis
    volumes:
      - ./laravel:/var/www/html
    networks:
      - edufin_network

  postgres:
    image: postgres:16-alpine
    container_name: edufin_postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${LARAVEL_DB_NAME}
      POSTGRES_USER: ${LARAVEL_DB_USER}
      POSTGRES_PASSWORD: ${LARAVEL_DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - edufin_network

  redis:
    image: redis:7-alpine
    container_name: edufin_redis
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    networks:
      - edufin_network

  nginx:
    image: nginx:alpine
    container_name: edufin_nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/ssl:/etc/nginx/ssl
    networks:
      - edufin_network

volumes:
  wordpress_data:
  mysql_data:
  postgres_data:
  redis_data:

networks:
  edufin_network:
    driver: bridge
```

---

## 3. Authentication Strategy

WordPress and Laravel are now **independent systems**. There is no Single Sign-On
(SSO) between them. WordPress handles only anonymous marketing/content visitors and
has no user accounts tied to the portal. All authentication — for both clients and
staff — happens entirely on the Laravel application at `app.edufin.co.ke`.

### 3.1 Authentication Overview

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          AUTHENTICATION ARCHITECTURE                                │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  WORDPRESS (edufin.co.ke)                LARAVEL (app.edufin.co.ke)                │
│  ───────────────────────────             ─────────────────────────────             │
│  • No user accounts                      • Single source of identity                │
│  • No login / sessions                   • All clients + staff authenticate here   │
│  • Public marketing content only         • Session-based auth (web)                │
│  • "Login" / "Get Started" buttons       • JWT via Sanctum (mobile API)            │
│    link out to app.edufin.co.ke          • Role-based redirect after login         │
│                                          • Filament admin panel at /admin          │
│                                                                                     │
│  LOGIN & REGISTRATION FLOW                                                          │
│  ─────────────────────────────                                                      │
│                                                                                     │
│  ┌──────────┐    ┌───────────────────┐    ┌──────────────────────────────────┐   │
│  │   User   │───►│ app.edufin.co.ke  │───►│ Role-based redirect              │   │
│  │          │    │ /login            │    │  • Client → /dashboard           │   │
│  │          │    │ /register         │    │  • Staff   → /admin              │   │
│  └──────────┘    └───────────────────┘    └──────────────────────────────────┘   │
│                                                                                     │
│  1. User clicks "Login" or "Get Started" on WordPress (or visits app directly)     │
│  2. User is taken to app.edufin.co.ke/login (or /register for new users)           │
│  3. Laravel authenticates the user (session cookie for web, Sanctum token mobile)  │
│  4. After login, Laravel redirects by role:                                         │
│       - Client  → app.edufin.co.ke/dashboard                                        │
│       - Staff   → app.edufin.co.ke/admin  (Filament)                                │
│  5. Logout clears the Laravel session only (no cross-domain coordination needed)   │
│                                                                                     │
│  MOBILE API AUTHENTICATION                                                          │
│  ─────────────────────────────                                                      │
│  • Mobile clients authenticate via POST /api/v1/auth/login (on edufin.co.ke)       │
│  • Laravel Sanctum issues a bearer token (JWT-style)                                │
│  • Token sent in Authorization: Bearer <token> header on subsequent requests        │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Login & Registration Routes (Laravel)

```php
// routes/web.php

use App\Http\Controllers\Auth\LoginController;
use App\Http\Controllers\Auth\RegisterController;

// Authentication routes (session-based for the web portal)
Route::get('/login', [LoginController::class, 'showLoginForm'])->name('login');
Route::post('/login', [LoginController::class, 'login']);
Route::post('/logout', [LoginController::class, 'logout'])->name('logout');

// Registration / onboarding
Route::get('/register', [RegisterController::class, 'showRegistrationForm'])->name('register');
Route::post('/register', [RegisterController::class, 'register']);

// Protected portal routes
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
    // ... KYC, loans, documents, etc.
});

// Filament admin panel (staff only) — served at /admin
// Configured in app/Providers/Filament/AdminPanelProvider.php
// Access is restricted to users with the 'staff' / 'admin' role.
```

### 3.3 Role-Based Redirect After Login

```php
// app/Http/Controllers/Auth/LoginController.php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    public function login(Request $request)
    {
        $credentials = $request->validate([
            'email'    => ['required', 'email'],
            'password' => ['required'],
        ]);

        if (!Auth::attempt($credentials, $request->boolean('remember'))) {
            return back()->withErrors([
                'email' => 'The provided credentials do not match our records.',
            ])->onlyInput('email');
        }

        $request->session()->regenerate();

        return $this->authenticatedRedirect();
    }

    /**
     * Redirect users based on their role after successful login.
     *  - Clients  → /dashboard
     *  - Staff    → /admin (Filament)
     */
    protected function authenticatedRedirect()
    {
        $user = Auth::user();

        if ($user->hasRole(['admin', 'staff', 'loan_officer', 'compliance'])) {
            return redirect()->intended(route('filament.admin.pages.dashboard'));
        }

        return redirect()->intended(route('dashboard'));
    }

    public function logout(Request $request)
    {
        Auth::logout();
        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return redirect()->route('login');
    }
}
```

### 3.4 Mobile API Authentication (Sanctum)

```php
// routes/api.php  (served at edufin.co.ke/api/v1)

use App\Http\Controllers\Api\AuthController;

Route::prefix('auth')->group(function () {
    Route::post('/login', [AuthController::class, 'login']);
    Route::post('/register', [AuthController::class, 'register']);
    Route::middleware('auth:sanctum')->group(function () {
        Route::post('/logout', [AuthController::class, 'logout']);
        Route::get('/me', [AuthController::class, 'me']);
    });
});
```

```php
// app/Http/Controllers/Api/AuthController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function login(Request $request)
    {
        $request->validate([
            'email'    => ['required', 'email'],
            'password' => ['required'],
            'device'   => ['required', 'string'],
        ]);

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            return response()->json(['message' => 'Invalid credentials'], 401);
        }

        $token = $user->createToken($request->device)->plainTextToken;

        return response()->json([
            'token' => $token,
            'user'  => [
                'id'            => $user->uuid,
                'email'         => $user->email,
                'display_name'  => $user->display_name,
                'roles'         => $user->getRoleNames(),
            ],
        ]);
    }

    public function register(Request $request)
    {
        $request->validate([
            'first_name' => ['required', 'string', 'max:255'],
            'last_name'  => ['required', 'string', 'max:255'],
            'email'      => ['required', 'email', 'unique:users,email'],
            'phone'      => ['required', 'string', 'max:20'],
            'password'   => ['required', 'confirmed', 'min:8'],
            'device'     => ['required', 'string'],
        ]);

        $user = User::create([
            'first_name' => $request->first_name,
            'last_name'  => $request->last_name,
            'email'      => $request->email,
            'phone'      => $request->phone,
            'password'   => Hash::make($request->password),
        ])->assignRole('client');

        $token = $user->createToken($request->device)->plainTextToken;

        return response()->json([
            'token' => $token,
            'user'  => [
                'id'           => $user->uuid,
                'email'        => $user->email,
                'display_name' => $user->display_name,
                'roles'        => $user->getRoleNames(),
            ],
        ], 201);
    }

    public function me(Request $request)
    {
        return response()->json([
            'user' => [
                'id'           => $request->user()->uuid,
                'email'        => $request->user()->email,
                'display_name' => $request->user()->display_name,
                'roles'        => $request->user()->getRoleNames(),
            ],
        ]);
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json(['message' => 'Logged out']);
    }
}
```

### 3.5 Authentication Summary

| Concern | Web Portal | Mobile API |
|---------|-----------|------------|
| **Login URL** | `app.edufin.co.ke/login` | `edufin.co.ke/api/v1/auth/login` |
| **Register URL** | `app.edufin.co.ke/register` | `edufin.co.ke/api/v1/auth/register` |
| **Credential store** | Laravel `users` table (PostgreSQL) | Same |
| **Session mechanism** | Laravel session (Redis driver) | Laravel Sanctum bearer token |
| **Client landing** | `app.edufin.co.ke/dashboard` | Mobile app screens |
| **Staff landing** | `app.edufin.co.ke/admin` (Filament) | N/A |
| **WordPress involvement** | None — links out to Laravel only | None |

> **Note:** Because the two systems are independent, there are no shared cookies,
> no SSO tokens, and no cross-domain session tables. WordPress simply links to
> `app.edufin.co.ke/login` and `app.edufin.co.ke/register`.

---

## 4. Data Integration Layer

### 4.1 Data Ownership Model

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              DATA OWNERSHIP & FLOW                                   │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  WORDPRESS (Content Only)              LARAVEL (All Business Data)                 │
│  ─────────────────────────             ───────────────────────────                 │
│                                                                                     │
│  ┌─────────────────────┐              ┌─────────────────────────────┐             │
│  │ • Blog Posts        │              │ • User Accounts             │             │
│  │ • Pages             │              │ • Authentication            │             │
│  │ • Media Library     │              │ • KYC Data                  │             │
│  │ • SEO Metadata      │              │ • Loan Facilities           │             │
│  │ • Menu Structure    │              │ • Education Beneficiaries   │             │
│  │ • Theme Settings    │              │ • Collateral                │             │
│  │ • Contact Forms     │              │ • Transactions              │             │
│  │   (native WP,       │              │ • Documents                 │             │
│  │    no Laravel call) │              │ • Notifications             │             │
│  │                     │              │ • Audit Logs                │             │
│  └─────────────────────┘              └─────────────────────────────┘             │
│                                                                                     │
│  RULE: WordPress NEVER stores financial or PII data                                │
│  RULE: WordPress does NOT call Laravel APIs — the two systems are independent      │
│  RULE: All user/business data lives in Laravel (PostgreSQL)                        │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Data Ownership Model (Independent Systems)

WordPress and Laravel are **fully independent**. WordPress does **not** call any
Laravel API. Marketing content (packages, rates, calculator figures shown on the
public site) is maintained directly inside WordPress as static content/CPT entries.
There is no `EduFin_API` client plugin and no server-to-server dependency from
WordPress to Laravel.

| Data | Owner | Notes |
|------|-------|-------|
| Blog posts, pages, SEO, media | WordPress | Managed in wp-admin |
| Marketing copy for packages/rates | WordPress | Static content; not fetched from Laravel |
| Contact / inquiry forms | WordPress | Handled natively (e.g. Contact Form 7 / WPForms); submissions emailed, not sent to Laravel |
| User accounts, KYC, loans, transactions | Laravel | PostgreSQL; the only system of record for business data |

> **Key change:** Previously WordPress fetched packages/rates from Laravel and
> submitted inquiries to a Laravel endpoint. That coupling has been removed.
> WordPress is now a standalone marketing site.

### 4.3 Laravel Public API Endpoints (Mobile App & Future Integrations)

> These endpoints are exposed at `edufin.co.ke/api/v1/*` and are intended for the
> **mobile app and future third-party integrations** — **not** for WordPress.
> WordPress does not consume these endpoints.

```php
// app/Http/Controllers/Api/PublicController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\FinancingPackage;
use App\Models\Inquiry;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Cache;

class PublicController extends Controller
{
    /**
     * Get financing packages (public, cached)
     */
    public function packages(Request $request)
    {
        $category = $request->query('category');
        $cacheKey = 'public_packages_' . ($category ?? 'all');
        
        $packages = Cache::remember($cacheKey, 3600, function () use ($category) {
            $query = FinancingPackage::where('is_active', true)
                ->orderBy('education_level');
            
            if ($category) {
                $query->where('category', $category);
            }
            
            return $query->get()->map(fn($p) => [
                'id' => $p->slug,
                'name' => $p->name,
                'education_level' => $p->education_level,
                'min_amount' => $p->min_amount,
                'max_amount' => $p->max_amount,
                'interest_rate_from' => $p->interest_rate_min,
                'interest_rate_to' => $p->interest_rate_max,
                'description' => $p->short_description,
            ]);
        });
        
        return response()->json(['data' => $packages]);
    }
    
    /**
     * Submit inquiry (mobile app / direct integrations)
     */
    public function submitInquiry(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|max:255',
            'phone' => 'required|string|max:20',
            'message' => 'required|string|max:2000',
            'source' => 'required|string|in:mobile,direct',
        ]);
        
        $inquiry = Inquiry::create([
            ...$validated,
            'ip_address' => $request->ip(),
        ]);
        
        // Dispatch notification job
        dispatch(new \App\Jobs\ProcessInquiry($inquiry));
        
        return response()->json([
            'success' => true,
            'message' => 'Thank you for your inquiry.',
            'reference' => $inquiry->reference_number,
        ]);
    }
    
    /**
     * Get calculator rates
     */
    public function calculatorRates()
    {
        $rates = Cache::remember('calculator_rates', 86400, function () {
            return [
                'education_levels' => [
                    ['key' => 'primary', 'label' => 'Primary School', 'max' => 200000],
                    ['key' => 'secondary', 'label' => 'Secondary School', 'max' => 400000],
                    ['key' => 'undergraduate', 'label' => 'Undergraduate', 'max' => 800000],
                    ['key' => 'masters', 'label' => "Master's Degree", 'max' => 700000],
                    ['key' => 'phd', 'label' => 'PhD', 'max' => 1000000],
                ],
                'interest_rates' => ['min' => 12.0, 'max' => 18.0, 'default' => 14.0],
                'durations' => [
                    ['months' => 12, 'label' => '1 Year'],
                    ['months' => 24, 'label' => '2 Years'],
                    ['months' => 36, 'label' => '3 Years'],
                    ['months' => 48, 'label' => '4 Years'],
                ],
                'min_deposit_percentage' => 10,
            ];
        });
        
        return response()->json(['data' => $rates]);
    }
}
```

---

## 5. User Experience Consistency

### 5.1 Shared Design System

```
SHARED ASSETS (cdn.edufin.co.ke)
├── /css/
│   ├── variables.css      # CSS custom properties
│   ├── components.css     # Shared component styles
│   └── utilities.css      # Utility classes
├── /js/
│   ├── components.js      # Shared JS components
│   └── analytics.js       # Unified analytics
├── /fonts/
│   └── inter/             # Shared typography
└── /images/
    ├── logo.svg
    ├── logo-white.svg
    └── icons/
```

### 5.2 Shared CSS Variables

```css
/* cdn.edufin.co.ke/css/variables.css */

:root {
  /* Brand Colors */
  --edufin-primary: #1e40af;
  --edufin-primary-dark: #1e3a8a;
  --edufin-primary-light: #3b82f6;
  --edufin-secondary: #059669;
  --edufin-accent: #f59e0b;
  
  /* Neutral Colors */
  --edufin-gray-50: #f9fafb;
  --edufin-gray-100: #f3f4f6;
  --edufin-gray-500: #6b7280;
  --edufin-gray-900: #111827;
  
  /* Semantic Colors */
  --edufin-success: #10b981;
  --edufin-warning: #f59e0b;
  --edufin-error: #ef4444;
  
  /* Typography */
  --edufin-font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  
  /* Spacing */
  --edufin-spacing-4: 1rem;
  --edufin-spacing-6: 1.5rem;
  --edufin-spacing-8: 2rem;
  
  /* Border Radius */
  --edufin-radius-md: 0.375rem;
  --edufin-radius-lg: 0.5rem;
  
  /* Shadows */
  --edufin-shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
}
```

### 5.3 Shared Header (WordPress)

```php
<!-- WordPress: header.php -->
<?php
// WordPress no longer has SSO sessions or user accounts tied to the portal.
// The header always shows Login and Get Started buttons that link to Laravel.
$portal_url = 'https://app.edufin.co.ke';
?>

<header class="edufin-header">
  <div class="edufin-header__container">
    <a href="<?php echo home_url(); ?>" class="edufin-header__logo">
      <img src="https://cdn.edufin.co.ke/images/logo.svg" alt="EduFin Kenya" />
    </a>
    
    <nav class="edufin-header__nav">
      <a href="<?php echo home_url('/products'); ?>">Products</a>
      <a href="<?php echo home_url('/how-it-works'); ?>">How It Works</a>
      <a href="<?php echo home_url('/calculator'); ?>">Calculator</a>
      <a href="<?php echo home_url('/blog'); ?>">Blog</a>
      <a href="<?php echo home_url('/contact'); ?>">Contact</a>
    </nav>
    
    <div class="edufin-header__auth">
      <a href="<?php echo $portal_url; ?>/login" class="edufin-btn edufin-btn--outline">Login</a>
      <a href="<?php echo $portal_url; ?>/register" class="edufin-btn edufin-btn--primary">
        Get Started
      </a>
      <a href="<?php echo $portal_url; ?>/dashboard" class="edufin-btn edufin-btn--ghost">
        My Portal
      </a>
    </div>
  </div>
</header>
```

### 5.4 Shared Header (Laravel Blade)

```blade
<!-- Laravel: resources/views/components/header.blade.php -->
<!-- Marketing links point back to WordPress (config('app.wordpress_url')).
     Login/register are served on app.edufin.co.ke directly. -->

<header class="edufin-header">
  <div class="edufin-header__container">
    <a href="{{ config('app.wordpress_url') }}" class="edufin-header__logo">
      <img src="https://cdn.edufin.co.ke/images/logo.svg" alt="EduFin Kenya" />
    </a>
    
    <nav class="edufin-header__nav">
      <a href="{{ config('app.wordpress_url') }}/products">Products</a>
      <a href="{{ config('app.wordpress_url') }}/how-it-works">How It Works</a>
      <a href="{{ config('app.wordpress_url') }}/calculator">Calculator</a>
      <a href="{{ config('app.wordpress_url') }}/blog">Blog</a>
    </nav>
    
    <div class="edufin-header__auth">
      @auth
        <span>Hi, {{ auth()->user()->display_name }}</span>
        <a href="{{ route('dashboard') }}" class="edufin-btn edufin-btn--primary">Dashboard</a>
      @else
        <a href="https://app.edufin.co.ke/login" class="edufin-btn edufin-btn--outline">Login</a>
        <a href="https://app.edufin.co.ke/register" class="edufin-btn edufin-btn--primary">Get Started</a>
      @endauth
    </div>
  </div>
</header>
```

---

## 6. Security Implementation

### 6.1 Security Layers

```
LAYER 1: CLOUDFLARE (Edge Security)
• WAF Rules (OWASP Core Rule Set)
• DDoS Protection
• Bot Management
• Rate Limiting
• SSL/TLS Termination

LAYER 2: APPLICATION SECURITY

WordPress:                          Laravel:
• Disable XML-RPC                   • CSRF Protection
• Disable File Editing              • XSS Prevention (Blade)
• Strong Admin Passwords            • SQL Injection Prevention
• Limited Login Attempts            • Rate Limiting
• Security Headers                  • Input Validation
                                    • MFA for Sensitive Ops

LAYER 3: DATA SECURITY
• Encryption at Rest
• Encryption in Transit (TLS 1.2+)
• PII Encryption (Application-level)
• Secure File Storage

LAYER 4: NETWORK ISOLATION
• WordPress cannot access Laravel database
• Internal services on private network
• Database not publicly accessible
```

### 6.2 WordPress Security Configuration

```php
// wp-config.php

// Disable file editing
define('DISALLOW_FILE_EDIT', true);
define('DISALLOW_FILE_MODS', true);

// Force SSL
define('FORCE_SSL_ADMIN', true);

// Security keys (use unique values from https://api.wordpress.org/secret-key/1.1/salt/)
define('AUTH_KEY',         'unique-phrase');
define('SECURE_AUTH_KEY',  'unique-phrase');
define('LOGGED_IN_KEY',    'unique-phrase');
define('NONCE_KEY',        'unique-phrase');

// Custom table prefix
$table_prefix = 'edf_';

// Limit revisions
define('WP_POST_REVISIONS', 5);
```

### 6.3 API Security Middleware (Laravel)

```php
// app/Http/Middleware/VerifyApiKey.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class VerifyApiKey
{
    public function handle(Request $request, Closure $next, string $service = 'default')
    {
        $apiKey = $request->header('X-API-Key');
        
        if (!$apiKey) {
            return response()->json(['error' => 'API key required'], 401);
        }
        
        $validKey = match($service) {
            'wordpress' => config('services.wordpress.api_key'),
            'mobile' => config('services.mobile.api_key'),
            default => config('services.default.api_key'),
        };
        
        if (!hash_equals($validKey, $apiKey)) {
            \Log::warning('Invalid API key attempt', [
                'ip' => $request->ip(),
                'service' => $service,
            ]);
            
            return response()->json(['error' => 'Invalid API key'], 401);
        }
        
        return $next($request);
    }
}
```

---

## 7. Development Workflow

### 7.1 Repository Structure

```
edufin/
├── .github/
│   └── workflows/
│       ├── wordpress-deploy.yml
│       └── laravel-deploy.yml
├── wordpress/
│   ├── themes/edufin/
│   └── plugins/
│       └── edufin-integration/
├── laravel/
│   ├── app/
│   ├── config/
│   ├── database/
│   ├── resources/
│   └── routes/
├── shared/
│   ├── css/
│   ├── js/
│   └── images/
├── infrastructure/
│   ├── docker-compose.yml
│   └── nginx/
└── docs/
```

### 7.2 CI/CD Pipeline (Laravel)

```yaml
# .github/workflows/laravel-deploy.yml

name: Deploy Laravel

on:
  push:
    branches: [main]
    paths: ['laravel/**']

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          
      - name: Install Dependencies
        run: composer install --no-dev --optimize-autoloader
        working-directory: ./laravel
        
      - name: Run Tests
        run: php artisan test
        working-directory: ./laravel

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/laravel
            git pull origin main
            composer install --no-dev --optimize-autoloader
            php artisan migrate --force
            php artisan config:cache
            php artisan route:cache
            php artisan queue:restart
```

---

## 8. Monitoring & Operations

### 8.1 Monitoring Stack

| Tool | Purpose | Platform |
|------|---------|----------|
| Cloudflare Analytics | Traffic, threats, performance | Both |
| Laravel Telescope | Requests, queries, jobs | Laravel |
| Query Monitor | Slow queries, errors | WordPress |
| Uptime Robot | Availability monitoring | Both |
| Sentry | Error tracking | Both |

### 8.2 Health Check Endpoints

```php
// Laravel: routes/api.php
Route::get('/health', function () {
    return response()->json([
        'status' => 'healthy',
        'timestamp' => now()->toIso8601String(),
        'services' => [
            'database' => DB::connection()->getPdo() ? 'up' : 'down',
            'redis' => Redis::ping() ? 'up' : 'down',
            'queue' => Queue::size('default') >= 0 ? 'up' : 'down',
        ],
    ]);
});
```

### 8.3 Backup Strategy

| Data | Frequency | Retention | Location |
|------|-----------|-----------|----------|
| PostgreSQL (Laravel) | Daily | 30 days | Backblaze B2 |
| MySQL (WordPress) | Daily | 30 days | Backblaze B2 |
| Media Files | Weekly | 90 days | Cloudflare R2 |
| Application Code | On deploy | Git history | GitHub |

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-08-06 | EduFin Team | Initial implementation plan |

---

*This document provides the technical implementation guide for the EduFin dual-platform architecture. Follow these specifications to ensure consistent, secure, and maintainable deployment.*

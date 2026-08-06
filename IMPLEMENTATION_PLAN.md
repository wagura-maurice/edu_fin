# EduFin Platform - Dual-Platform Implementation Plan

## WordPress (edufin.co.ke) + Laravel (app.edufin.co.ke)

**Date:** August 6, 2026  
**Version:** 1.0  
**Status:** Technical Specification

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Infrastructure & Deployment](#2-infrastructure--deployment)
3. [Authentication & SSO Strategy](#3-authentication--sso-strategy)
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
│     │  edufin.co.ke   │      │ app.edufin.co.ke│      │ api.edufin.co.ke│         │
│     │   (WordPress)   │      │    (Laravel)    │      │  (Laravel API)  │         │
│     └────────┬────────┘      └────────┬────────┘      └────────┬────────┘         │
│              │                        │                        │                   │
│              │                        └────────────┬───────────┘                   │
│              │                                     │                               │
│              ▼                                     ▼                               │
│     ┌─────────────────┐              ┌─────────────────────────────────┐          │
│     │   WordPress     │              │        LARAVEL SERVER           │          │
│     │    Server       │              │  ┌───────────┐ ┌───────────┐   │          │
│     │                 │              │  │  Portal   │ │    API    │   │          │
│     │ • Marketing     │◄────────────►│  │  (Web)    │ │  (Mobile) │   │          │
│     │ • Blog/News     │   Auth API   │  └─────┬─────┘ └─────┬─────┘   │          │
│     │ • SEO Content   │              │        └──────┬──────┘         │          │
│     │ • Landing Pages │              │               ▼                │          │
│     └────────┬────────┘              │  ┌─────────────────────────┐  │          │
│              │                       │  │    Business Services    │  │          │
│              │                       │  │  • KYC  • Loans  • Docs │  │          │
│              │                       │  └───────────┬─────────────┘  │          │
│              │                       └──────────────┼────────────────┘          │
│              │                                      │                            │
│              ▼                                      ▼                            │
│     ┌─────────────────┐              ┌─────────────────────────────────┐         │
│     │   MySQL/MariaDB │              │          PostgreSQL             │         │
│     │   (WP Content)  │              │    (Application Data)           │         │
│     └─────────────────┘              └─────────────────────────────────┘         │
│                                                                                   │
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
| `edufin.co.ke` | WordPress | Marketing, SEO, Blog, Public Content |
| `app.edufin.co.ke` | Laravel | Client Portal, Dashboard, KYC, Loans |
| `api.edufin.co.ke` | Laravel | Mobile API, Webhooks, Integrations |
| `cdn.edufin.co.ke` | Cloudflare R2 | Static Assets, Media Files |
| `admin.edufin.co.ke` | Laravel (Filament) | Internal Admin Panel |

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

# Root domain - WordPress
edufin.co.ke          A       <WP_SERVER_IP>        Proxied (Orange Cloud)
www.edufin.co.ke      CNAME   edufin.co.ke          Proxied

# Application subdomain - Laravel
app.edufin.co.ke      A       <LARAVEL_SERVER_IP>   Proxied
api.edufin.co.ke      A       <LARAVEL_SERVER_IP>   Proxied
admin.edufin.co.ke    A       <LARAVEL_SERVER_IP>   Proxied

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
    server_name app.edufin.co.ke api.edufin.co.ke admin.edufin.co.ke;
    
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
    server_name app.edufin.co.ke api.edufin.co.ke admin.edufin.co.ke;
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
      - ./wordpress/plugins/edufin-sso:/var/www/html/wp-content/plugins/edufin-sso
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

## 3. Authentication & SSO Strategy

### 3.1 SSO Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              SSO AUTHENTICATION FLOW                                 │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  LARAVEL = IDENTITY PROVIDER (IdP)                                                 │
│  WORDPRESS = SERVICE PROVIDER (SP)                                                 │
│                                                                                     │
│  SCENARIO 1: User visits WordPress, wants to access portal                         │
│  ─────────────────────────────────────────────────────────                         │
│                                                                                     │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐        │
│  │   User   │───►│  WordPress   │───►│   Laravel    │───►│   Laravel    │        │
│  │          │    │  "Login"     │    │  /sso/login  │    │  Dashboard   │        │
│  │          │    │   Button     │    │              │    │              │        │
│  └──────────┘    └──────────────┘    └──────────────┘    └──────────────┘        │
│                                                                                     │
│  1. User clicks "Login" on WordPress                                               │
│  2. WordPress redirects to: app.edufin.co.ke/sso/login?redirect=edufin.co.ke      │
│  3. User authenticates on Laravel (or is already logged in)                        │
│  4. Laravel creates session + issues SSO token                                     │
│  5. Laravel redirects back to WordPress with token                                 │
│  6. WordPress validates token via API, creates local session                       │
│                                                                                     │
│  SCENARIO 2: User logs in on Laravel, visits WordPress                             │
│  ─────────────────────────────────────────────────────                             │
│                                                                                     │
│  1. User logs into Laravel portal                                                  │
│  2. Laravel sets SSO cookie (shared domain: .edufin.co.ke)                        │
│  3. User visits WordPress site                                                     │
│  4. WordPress detects SSO cookie, validates via Laravel API                        │
│  5. WordPress creates local session, user appears logged in                        │
│                                                                                     │
│  SCENARIO 3: User logs out                                                         │
│  ─────────────────────────────                                                     │
│                                                                                     │
│  1. User clicks logout on either platform                                          │
│  2. Platform calls Laravel /sso/logout endpoint                                    │
│  3. Laravel invalidates session + SSO token                                        │
│  4. Both platforms clear local sessions                                            │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Database Migrations (Laravel)

```php
// database/migrations/2026_08_06_create_sso_tables.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        // One-time tokens for cross-domain auth
        Schema::create('sso_tokens', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->string('token', 64)->unique();
            $table->string('service')->default('wordpress');
            $table->string('ip_address', 45)->nullable();
            $table->timestamp('expires_at');
            $table->timestamp('used_at')->nullable();
            $table->timestamps();
            
            $table->index(['token', 'expires_at']);
        });
        
        // Persistent sessions for cross-domain
        Schema::create('sso_sessions', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->string('session_id', 64)->unique();
            $table->string('service');
            $table->string('ip_address', 45)->nullable();
            $table->timestamp('last_activity');
            $table->timestamp('expires_at');
            $table->timestamps();
            
            $table->index(['session_id', 'expires_at']);
        });
    }
    
    public function down(): void
    {
        Schema::dropIfExists('sso_sessions');
        Schema::dropIfExists('sso_tokens');
    }
};
```

### 3.3 Laravel SSO Service

```php
// app/Services/SsoService.php

namespace App\Services;

use App\Models\User;
use App\Models\SsoToken;
use App\Models\SsoSession;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Cookie;

class SsoService
{
    private const TOKEN_EXPIRY_MINUTES = 5;
    private const SESSION_EXPIRY_HOURS = 24;
    private const SSO_COOKIE_NAME = 'edufin_sso';
    private const SSO_COOKIE_DOMAIN = '.edufin.co.ke';
    
    /**
     * Generate a one-time SSO token for cross-domain authentication
     */
    public function generateToken(User $user, string $service = 'wordpress'): string
    {
        // Clean up expired tokens
        SsoToken::where('user_id', $user->id)
            ->where('expires_at', '<', now())
            ->delete();
        
        $token = Str::random(64);
        
        SsoToken::create([
            'user_id' => $user->id,
            'token' => hash('sha256', $token),
            'service' => $service,
            'ip_address' => request()->ip(),
            'expires_at' => now()->addMinutes(self::TOKEN_EXPIRY_MINUTES),
        ]);
        
        return $token;
    }
    
    /**
     * Validate and consume a one-time SSO token
     */
    public function validateToken(string $token, string $service = 'wordpress'): ?User
    {
        $hashedToken = hash('sha256', $token);
        
        $ssoToken = SsoToken::where('token', $hashedToken)
            ->where('service', $service)
            ->where('expires_at', '>', now())
            ->whereNull('used_at')
            ->first();
        
        if (!$ssoToken) {
            return null;
        }
        
        // Mark token as used (one-time use)
        $ssoToken->update(['used_at' => now()]);
        
        return $ssoToken->user;
    }
    
    /**
     * Create a persistent SSO session
     */
    public function createSession(User $user, string $service = 'wordpress'): string
    {
        $sessionId = Str::random(64);
        
        SsoSession::create([
            'user_id' => $user->id,
            'session_id' => hash('sha256', $sessionId),
            'service' => $service,
            'ip_address' => request()->ip(),
            'last_activity' => now(),
            'expires_at' => now()->addHours(self::SESSION_EXPIRY_HOURS),
        ]);
        
        // Cache for fast lookups
        Cache::put(
            "sso_session:{$sessionId}",
            ['user_id' => $user->id, 'service' => $service],
            now()->addHours(self::SESSION_EXPIRY_HOURS)
        );
        
        return $sessionId;
    }
    
    /**
     * Validate an SSO session
     */
    public function validateSession(string $sessionId): ?User
    {
        // Check cache first
        $cached = Cache::get("sso_session:{$sessionId}");
        
        if ($cached) {
            return User::find($cached['user_id']);
        }
        
        // Fallback to database
        $hashedSession = hash('sha256', $sessionId);
        
        $session = SsoSession::where('session_id', $hashedSession)
            ->where('expires_at', '>', now())
            ->first();
        
        if (!$session) {
            return null;
        }
        
        $session->update(['last_activity' => now()]);
        
        return $session->user;
    }
    
    /**
     * Set SSO cookie for cross-domain authentication
     */
    public function setSsoCookie(string $sessionId): \Symfony\Component\HttpFoundation\Cookie
    {
        return Cookie::make(
            self::SSO_COOKIE_NAME,
            $sessionId,
            self::SESSION_EXPIRY_HOURS * 60,
            '/',
            self::SSO_COOKIE_DOMAIN,
            true,  // secure
            true,  // httpOnly
            false, // raw
            'Lax'  // sameSite
        );
    }
    
    /**
     * Invalidate all SSO sessions for a user
     */
    public function invalidateAllSessions(User $user): void
    {
        $sessions = SsoSession::where('user_id', $user->id)->get();
        
        foreach ($sessions as $session) {
            Cache::forget("sso_session:{$session->session_id}");
        }
        
        SsoSession::where('user_id', $user->id)->delete();
        SsoToken::where('user_id', $user->id)->delete();
    }
}
```

### 3.4 Laravel SSO Controller

```php
// app/Http/Controllers/SsoController.php

namespace App\Http\Controllers;

use App\Services\SsoService;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class SsoController extends Controller
{
    public function __construct(
        private SsoService $ssoService
    ) {}
    
    /**
     * SSO Login - Redirect from WordPress
     * GET /sso/login?redirect=https://edufin.co.ke/my-account
     */
    public function login(Request $request)
    {
        $redirect = $request->query('redirect', config('app.wordpress_url'));
        
        // Validate redirect URL is within our domain
        if (!$this->isValidRedirect($redirect)) {
            abort(400, 'Invalid redirect URL');
        }
        
        // If user is already authenticated
        if (Auth::check()) {
            return $this->handleAuthenticatedRedirect($redirect);
        }
        
        // Store intended redirect and show login form
        session(['sso_redirect' => $redirect]);
        
        return redirect()->route('login');
    }
    
    /**
     * Validate SSO token - Called by WordPress
     * POST /api/sso/validate
     */
    public function validateToken(Request $request)
    {
        $request->validate([
            'token' => 'required|string|size:64',
            'service' => 'required|string|in:wordpress',
        ]);
        
        // Verify API key
        if ($request->header('X-API-Key') !== config('services.wordpress.api_key')) {
            return response()->json(['valid' => false, 'error' => 'Unauthorized'], 401);
        }
        
        $user = $this->ssoService->validateToken(
            $request->input('token'),
            $request->input('service')
        );
        
        if (!$user) {
            return response()->json(['valid' => false, 'error' => 'Invalid token'], 401);
        }
        
        // Create persistent session for WordPress
        $sessionId = $this->ssoService->createSession($user, 'wordpress');
        
        return response()->json([
            'valid' => true,
            'session_id' => $sessionId,
            'user' => [
                'id' => $user->uuid,
                'email' => $user->email,
                'display_name' => $user->display_name,
                'first_name' => $user->first_name,
                'last_name' => $user->last_name,
                'is_verified' => $user->kyc_verified,
            ],
        ]);
    }
    
    /**
     * Validate SSO session - Called by WordPress on each request
     * POST /api/sso/session
     */
    public function validateSession(Request $request)
    {
        $request->validate(['session_id' => 'required|string|size:64']);
        
        if ($request->header('X-API-Key') !== config('services.wordpress.api_key')) {
            return response()->json(['valid' => false], 401);
        }
        
        $user = $this->ssoService->validateSession($request->input('session_id'));
        
        if (!$user) {
            return response()->json(['valid' => false], 401);
        }
        
        return response()->json([
            'valid' => true,
            'user' => [
                'id' => $user->uuid,
                'display_name' => $user->display_name,
                'is_verified' => $user->kyc_verified,
            ],
        ]);
    }
    
    /**
     * SSO Logout
     */
    public function logout(Request $request)
    {
        $user = Auth::user();
        
        if ($user) {
            $this->ssoService->invalidateAllSessions($user);
        }
        
        Auth::logout();
        $request->session()->invalidate();
        
        $redirect = $request->query('redirect', config('app.wordpress_url'));
        
        return redirect($redirect);
    }
    
    private function handleAuthenticatedRedirect(string $redirect)
    {
        $user = Auth::user();
        
        // Generate one-time token
        $token = $this->ssoService->generateToken($user, 'wordpress');
        
        // Create SSO session
        $sessionId = $this->ssoService->createSession($user, 'wordpress');
        
        // Build redirect URL with token
        $redirectUrl = $redirect . (str_contains($redirect, '?') ? '&' : '?') 
            . 'sso_token=' . $token;
        
        return redirect($redirectUrl)
            ->withCookie($this->ssoService->setSsoCookie($sessionId));
    }
    
    private function isValidRedirect(string $url): bool
    {
        $parsed = parse_url($url);
        $host = $parsed['host'] ?? '';
        
        $allowedDomains = ['edufin.co.ke', 'www.edufin.co.ke', 'app.edufin.co.ke'];
        
        return in_array($host, $allowedDomains);
    }
}
```

### 3.5 Laravel Routes

```php
// routes/web.php
use App\Http\Controllers\SsoController;

Route::prefix('sso')->group(function () {
    Route::get('/login', [SsoController::class, 'login'])->name('sso.login');
    Route::post('/logout', [SsoController::class, 'logout'])->name('sso.logout');
});

// routes/api.php
Route::prefix('sso')->group(function () {
    Route::post('/validate', [SsoController::class, 'validateToken']);
    Route::post('/session', [SsoController::class, 'validateSession']);
});
```

### 3.6 WordPress SSO Plugin

```php
<?php
/**
 * Plugin Name: EduFin SSO
 * Description: Single Sign-On integration with EduFin Laravel Portal
 * Version: 1.0.0
 */

if (!defined('ABSPATH')) exit;

class EduFin_SSO {
    
    private const LARAVEL_URL = 'https://app.edufin.co.ke';
    private const API_URL = 'https://api.edufin.co.ke';
    private const SSO_COOKIE = 'edufin_sso';
    
    private $api_key;
    
    public function __construct() {
        $this->api_key = defined('EDUFIN_API_KEY') ? EDUFIN_API_KEY : '';
        
        add_action('init', [$this, 'check_sso_session']);
        add_action('template_redirect', [$this, 'handle_sso_callback']);
        add_action('wp_logout', [$this, 'handle_logout']);
        add_filter('login_url', [$this, 'custom_login_url'], 10, 3);
        add_shortcode('edufin_login_button', [$this, 'login_button_shortcode']);
    }
    
    /**
     * Check SSO cookie and validate session on every request
     */
    public function check_sso_session() {
        if (is_user_logged_in()) return;
        
        $session_id = $_COOKIE[self::SSO_COOKIE] ?? null;
        if (!$session_id) return;
        
        // Check cache first
        $cached = wp_cache_get($session_id, 'edufin_sso');
        if ($cached !== false) {
            $this->login_wp_user($cached);
            return;
        }
        
        // Validate with Laravel API
        $response = $this->api_request('/sso/session', [
            'session_id' => $session_id,
        ]);
        
        if ($response && $response['valid']) {
            wp_cache_set($session_id, $response['user'], 'edufin_sso', 300);
            $this->login_wp_user($response['user']);
        }
    }
    
    /**
     * Handle SSO callback from Laravel
     */
    public function handle_sso_callback() {
        if (!isset($_GET['sso_token'])) return;
        
        $token = sanitize_text_field($_GET['sso_token']);
        
        // Validate token with Laravel
        $response = $this->api_request('/sso/validate', [
            'token' => $token,
            'service' => 'wordpress',
        ]);
        
        if (!$response || !$response['valid']) {
            wp_die('SSO authentication failed. Please try again.');
        }
        
        // Cache user data
        wp_cache_set($response['session_id'], $response['user'], 'edufin_sso', 300);
        
        // Login WordPress user
        $this->login_wp_user($response['user']);
        
        // Redirect to clean URL
        wp_safe_redirect(remove_query_arg('sso_token'));
        exit;
    }
    
    /**
     * Login or create WordPress user
     */
    private function login_wp_user($user_data) {
        $email = sanitize_email($user_data['email'] ?? '');
        $edufin_id = sanitize_text_field($user_data['id'] ?? '');
        
        if (empty($email) || empty($edufin_id)) return;
        
        // Find existing user
        $wp_user = get_user_by('email', $email);
        
        if (!$wp_user) {
            // Create new WordPress user
            $username = sanitize_user(explode('@', $email)[0]);
            $counter = 1;
            while (username_exists($username)) {
                $username = sanitize_user(explode('@', $email)[0]) . $counter++;
            }
            
            $user_id = wp_insert_user([
                'user_login' => $username,
                'user_email' => $email,
                'user_pass' => wp_generate_password(32),
                'display_name' => $user_data['display_name'] ?? '',
                'first_name' => $user_data['first_name'] ?? '',
                'last_name' => $user_data['last_name'] ?? '',
                'role' => 'subscriber',
            ]);
            
            if (is_wp_error($user_id)) return;
            
            $wp_user = get_user_by('ID', $user_id);
        }
        
        // Update meta
        update_user_meta($wp_user->ID, 'edufin_user_id', $edufin_id);
        update_user_meta($wp_user->ID, 'edufin_verified', $user_data['is_verified'] ?? false);
        
        // Login
        if (!is_user_logged_in()) {
            wp_set_current_user($wp_user->ID);
            wp_set_auth_cookie($wp_user->ID, true);
        }
    }
    
    /**
     * Custom login URL - Redirect to Laravel
     */
    public function custom_login_url($login_url, $redirect, $force_reauth) {
        $sso_url = self::LARAVEL_URL . '/sso/login';
        
        if ($redirect) {
            $sso_url = add_query_arg('redirect', urlencode($redirect), $sso_url);
        } else {
            $sso_url = add_query_arg('redirect', urlencode(home_url()), $sso_url);
        }
        
        return $sso_url;
    }
    
    /**
     * Handle logout
     */
    public function handle_logout() {
        setcookie(self::SSO_COOKIE, '', time() - 3600, '/', '.edufin.co.ke', true, true);
    }
    
    /**
     * Login button shortcode
     */
    public function login_button_shortcode($atts) {
        $atts = shortcode_atts(['text' => 'Login to Portal', 'class' => 'edufin-btn'], $atts);
        
        if (is_user_logged_in()) {
            return sprintf(
                '<a href="%s/dashboard" class="%s">Go to Portal</a>',
                self::LARAVEL_URL,
                esc_attr($atts['class'])
            );
        }
        
        $login_url = self::LARAVEL_URL . '/sso/login?redirect=' . urlencode(get_permalink());
        
        return sprintf(
            '<a href="%s" class="%s">%s</a>',
            esc_url($login_url),
            esc_attr($atts['class']),
            esc_html($atts['text'])
        );
    }
    
    /**
     * Make API request to Laravel
     */
    private function api_request($endpoint, $data = []) {
        $response = wp_remote_post(self::API_URL . '/api' . $endpoint, [
            'timeout' => 10,
            'headers' => [
                'Content-Type' => 'application/json',
                'Accept' => 'application/json',
                'X-API-Key' => $this->api_key,
            ],
            'body' => json_encode($data),
        ]);
        
        if (is_wp_error($response)) {
            error_log('EduFin SSO API Error: ' . $response->get_error_message());
            return null;
        }
        
        $status = wp_remote_retrieve_response_code($response);
        if ($status !== 200) return null;
        
        return json_decode(wp_remote_retrieve_body($response), true);
    }
}

new EduFin_SSO();
```

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
│  │ • Contact Forms*    │              │ • Transactions              │             │
│  │                     │              │ • Documents                 │             │
│  │ * Forms submit to   │              │ • Notifications             │             │
│  │   Laravel API       │              │ • Audit Logs                │             │
│  └─────────────────────┘              └─────────────────────────────┘             │
│                                                                                     │
│  RULE: WordPress NEVER stores financial or PII data                                │
│  RULE: All user data flows through Laravel APIs                                    │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 WordPress API Client

```php
// wp-content/plugins/edufin-integration/class-edufin-api.php

class EduFin_API {
    
    private static $instance = null;
    private $base_url = 'https://api.edufin.co.ke/api/v1';
    private $api_key;
    
    public static function instance() {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
    
    private function __construct() {
        $this->api_key = defined('EDUFIN_API_KEY') ? EDUFIN_API_KEY : '';
    }
    
    /**
     * Get financing packages for display
     */
    public function get_packages($category = null) {
        $cache_key = 'packages_' . ($category ?? 'all');
        $cached = wp_cache_get($cache_key, 'edufin_api');
        
        if ($cached !== false) return $cached;
        
        $endpoint = '/packages';
        if ($category) {
            $endpoint .= '?category=' . urlencode($category);
        }
        
        $response = $this->request('GET', $endpoint);
        
        if ($response) {
            wp_cache_set($cache_key, $response, 'edufin_api', 3600);
        }
        
        return $response;
    }
    
    /**
     * Submit contact/inquiry form
     */
    public function submit_inquiry($data) {
        return $this->request('POST', '/inquiries', [
            'name' => sanitize_text_field($data['name']),
            'email' => sanitize_email($data['email']),
            'phone' => sanitize_text_field($data['phone']),
            'message' => sanitize_textarea_field($data['message']),
            'source' => 'wordpress',
        ]);
    }
    
    /**
     * Get loan calculator rates
     */
    public function get_calculator_rates() {
        $cached = wp_cache_get('calculator_rates', 'edufin_api');
        
        if ($cached !== false) return $cached;
        
        $response = $this->request('GET', '/calculator/rates');
        
        if ($response) {
            wp_cache_set('calculator_rates', $response, 'edufin_api', 86400);
        }
        
        return $response;
    }
    
    /**
     * Make API request
     */
    private function request($method, $endpoint, $data = null) {
        $args = [
            'method' => $method,
            'timeout' => 15,
            'headers' => [
                'Content-Type' => 'application/json',
                'Accept' => 'application/json',
                'X-API-Key' => $this->api_key,
            ],
        ];
        
        if ($data && $method !== 'GET') {
            $args['body'] = json_encode($data);
        }
        
        $response = wp_remote_request($this->base_url . $endpoint, $args);
        
        if (is_wp_error($response)) {
            error_log('EduFin API Error: ' . $response->get_error_message());
            return null;
        }
        
        $status = wp_remote_retrieve_response_code($response);
        
        if ($status >= 400) {
            error_log("EduFin API Error: HTTP {$status}");
            return null;
        }
        
        return json_decode(wp_remote_retrieve_body($response), true);
    }
}

function edufin_api() {
    return EduFin_API::instance();
}
```

### 4.3 Laravel Public API Endpoints

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
     * Submit inquiry from WordPress
     */
    public function submitInquiry(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|max:255',
            'phone' => 'required|string|max:20',
            'message' => 'required|string|max:2000',
            'source' => 'required|string|in:wordpress,mobile,direct',
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
$is_logged_in = is_user_logged_in();
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
      <?php if ($is_logged_in): ?>
        <a href="<?php echo $portal_url; ?>/dashboard" class="edufin-btn edufin-btn--primary">
          My Portal
        </a>
      <?php else: ?>
        <a href="<?php echo $portal_url; ?>/sso/login?redirect=<?php echo urlencode(get_permalink()); ?>" 
           class="edufin-btn edufin-btn--outline">Login</a>
        <a href="<?php echo $portal_url; ?>/register" class="edufin-btn edufin-btn--primary">
          Get Started
        </a>
      <?php endif; ?>
    </div>
  </div>
</header>
```

### 5.4 Shared Header (Laravel Blade)

```blade
<!-- Laravel: resources/views/components/header.blade.php -->

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
        <a href="{{ route('login') }}" class="edufin-btn edufin-btn--outline">Login</a>
        <a href="{{ route('register') }}" class="edufin-btn edufin-btn--primary">Get Started</a>
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
│       ├── edufin-sso/
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

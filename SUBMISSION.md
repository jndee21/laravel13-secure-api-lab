# Role-Based Access Control (RBAC) API - Submission Document

**Student Name:** Jean Cabiling  
**Date:** November 10, 2026  
**Repository:** `laravel13-secure-api-lab`  

---

## 1. Project Overview
This project implements Role-Based Access Control (RBAC) in a Laravel 13 REST API using Laravel Sanctum for token authentication and custom HTTP middleware for role authorization. A 401 Unauthorized response is given when a user tries to access private API data without a valid token. Once authenticated, the server grants access based on user roles (`admin` or `student`).

---

## 2. Key Middleware & Route Configuration

### A. Custom Middleware
* **`App\Http\Middleware\CheckRole`**: Restricts route access to users possessing any of the specified roles.
* **`App\Http\Middleware\AdminMiddleware`**: Restricts route access strictly to users with the `admin` role.

### B. Middleware Aliases (`bootstrap/app.php`)
```php
$middleware->alias([
    'role' => \App\Http\Middleware\CheckRole::class,
    'admin' => \App\Http\Middleware\AdminMiddleware::class,
]);
# Role-Based Access Control (RBAC) API - Submission Document

**Student Name:** Jean Cabiling  
**Date:** November 10, 2026  
**Repository:** `laravel13-secure-api-lab`  

---

## 1. Project Overview
This project implements Role-Based Access Control (RBAC) in a Laravel 13 REST API using Laravel Sanctum for token authentication and custom HTTP middleware for role authorization.

---

## 2. Key Middleware & Route Configuration

### A. Custom Middleware
* `App\Http\Middleware\CheckRole`: Restricts route access to users possessing any of the specified roles.
* `App\Http\Middleware\AdminMiddleware`: Restricts route access strictly to users with the `admin` role.

### B. Middleware Aliases (`bootstrap/app.php`)
```php
$middleware->alias([
    'role' => \App\Http\Middleware\CheckRole::class,
    'admin' => \App\Http\Middleware\AdminMiddleware::class,
]);

### C. API Routes (`routes/api.php`)
```php
<?php

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Route;

// Public Login Route
Route::post('/login', function (Request $request) {$request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    $user = User::where('email',$request->email)->first();

    if (! $user || ! Hash::check($request->password,$user->password)) {
        return response()->json(['message' => 'Invalid credentials'], 401);
    }

    $token =$user->createToken('auth_token')->plainTextToken;

    return response()->json([
        'access_token' => $token,
        'token_type' => 'Bearer',
        'user' => $user,
    ]);
});

// Protected Routes (Sanctum)
Route::middleware('auth:sanctum')->group(function () {

    Route::get('/profile', function (Request $request) {
        return response()->json($request->user());
    });

    Route::middleware('role:admin,student')->get('/students', function () {
        return response()->json([
            ['id' => 1, 'name' => 'John Doe', 'course' => 'BSIT'],
            ['id' => 2, 'name' => 'Jane Smith', 'course' => 'BSCS'],
        ]);
    });

    Route::middleware('admin')->get('/admin/dashboard', function () {
        return response()->json(['message' => 'Welcome to the Admin Dashboard!']);
    });

    Route::middleware('role:student')->get('/student/dashboard', function () {
        return response()->json(['message' => 'Welcome to the Student Dashboard!']);
    });
});

## 3. Reflection Questions

### Question 1: What is the difference between Authentication and Authorization?
- **Authentication** verifies *who you are*. It confirms the identity of a user attempting to access a system (e.g., submitting valid email/password credentials to receive an API Bearer token).
- **Authorization** determines *what you are allowed to do*. Once identity is verified via authentication, authorization checks whether the user has the permissions or roles (e.g., `admin` or `student`) required to perform specific actions or access protected resources.

### Question 2: Why return a 401 Unauthorized status vs. a 403 Forbidden status?
- **401 Unauthorized** indicates missing, invalid, or expired authentication credentials. It signals to the client that identity has not been established and they must authenticate (log in) first.
- **403 Forbidden** indicates that the server understands the user's identity (authentication succeeded), but the authenticated user lacks the necessary roles or privileges to access the requested resource.

---

## 4. Postman Test Evidence Log

| # | Screenshot File | Endpoint | Method | Token Used | Expected Status | Result |
|---|---|---|---|---|---|---|
| 1 | `screenshots/01-admin-login.png` | `/api/login` | POST | None (Admin credentials) | 200 OK | Passed |
| 2 | `screenshots/02-admin-allowed.png` | `/api/admin/dashboard` | GET | Admin Token | 200 OK | Passed |
| 3 | `screenshots/03-admin-denied.png` | `/api/student/dashboard` | GET | Admin Token | 403 Forbidden | Passed |
| 4 | `screenshots/04-student-login.png` | `/api/login` | POST | None (Student credentials) | 200 OK | Passed |
| 5 | `screenshots/05-student-allowed.png` | `/api/student/dashboard` | GET | Student Token | 200 OK | Passed |
| 6 | `screenshots/06-student-denied.png` | `/api/admin/dashboard` | GET | Student Token | 403 Forbidden | Passed |
| 7 | `screenshots/07-shared-route.png` | `/api/students` | GET | Student/Admin Token | 200 OK | Passed |
| 8 | `screenshots/08-no-token.png` | `/api/admin/dashboard` | GET | None | 401 Unauthorized | Passed |

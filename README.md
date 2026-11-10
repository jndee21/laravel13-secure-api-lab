# Laravel 13 RBAC API Lab

This repository contains a completed Role-Based Access Control (RBAC) API project built using **Laravel 13** and **Laravel Sanctum**.

## 1. Project Overview
The API secures routes based on user authentication and assigned roles (`admin` vs `student`).

- **Public Routes:** `/api/login`
- **Protected User Routes:** `/api/profile`
- **Role-Restricted Routes:**
  - `/api/admin/dashboard` (Admin only)
  - `/api/student/dashboard` (Student only)
  - `/api/students` (Admin & Student shared access)

---

## 2. API Routes (`routes/api.php`)

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

# Laravel Route Best Practices (PSR-12 Compliant)

## 🧭 1. Follow PSR-12 Formatting Standards
PSR-12 enforces clean and consistent PHP code, including:
- 4 spaces indentation
- One space after commas
- Blank lines between groups
- Use `::class` instead of string class names

---

## ✅ 2. Use Controller Classes Instead of Closures
```php
use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;

Route::get('/users', [UserController::class, 'index'])->name('users.index');
Route::get('/users/{user}', [UserController::class, 'show'])->name('users.show');
Route::post('/users', [UserController::class, 'store'])->name('users.store');
```
**Why:** Easier to test, maintain, and keeps concerns separated.

---

## 🧩 3. Use Route Grouping for Common Prefixes and Middleware
```php
use App\Http\Controllers\Admin\DashboardController;

Route::prefix('admin')
    ->middleware(['auth', 'role:admin'])
    ->as('admin.')
    ->group(function () {
        Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
        Route::resource('users', App\Http\Controllers\Admin\UserController::class);
    });
```

**Benefits:** DRY code, organization, clarity.

---

## ⚙️ 4. Use Resource Routes When Appropriate
```php
use App\Http\Controllers\PostController;

Route::resource('posts', PostController::class);
Route::resource('posts', PostController::class)->only(['index', 'store', 'destroy']);
```

---

## 🧱 5. Keep Route Files Organized
### Example 1:
```
routes/
├── web.php
├── api.php
├── admin.php
└── auth.php
```
In `RouteServiceProvider`:
```php
public function boot(): void
{
    $this->routes(function () {
        Route::middleware('web')
            ->group(base_path('routes/web.php'));

        Route::middleware(['web', 'auth', 'role:admin'])
            ->prefix('admin')
            ->group(base_path('routes/admin.php'));

        Route::middleware('api')
            ->prefix('api')
            ->group(base_path('routes/api.php'));
    });
}
```
### Example 2:
```
routes/
├── web/
│   ├── frontend.php
│   ├── admin.php
│   ├── user.php
│   └── vendor.php
│
├── api/
│   ├── v1.php
│   ├── v2.php
│   └── internal.php
│
├── system/
│   ├── channels.php
│   ├── console.php
│   └── breadcrumbs.php
```

In `RouteServiceProvider`:
```php
    public function boot(): void
    {
        $this->routes(function () {
            // Web routes
            Route::middleware('web')
                ->group(base_path('routes/web/frontend.php'));

            Route::middleware(['web', 'auth', 'role:admin'])
                ->prefix('admin')
                ->group(base_path('routes/web/admin.php'));

            Route::middleware(['web', 'auth'])
                ->prefix('user')
                ->group(base_path('routes/web/user.php'));

            // API routes (versioned)
            Route::prefix('api/v1')
                ->middleware('api')
                ->group(base_path('routes/api/v1.php'));

            Route::prefix('api/v2')
                ->middleware('api')
                ->group(base_path('routes/api/v2.php'));

            // System routes (not exposed publicly)
            Route::middleware('console')
                ->group(base_path('routes/system/console.php'));

            Route::middleware('channels')
                ->group(base_path('routes/system/channels.php'));
        });
    }
}
``` 
---

## ✨ 6. Follow Naming Conventions
✅ Good:
```php
Route::get('/profile/edit', [ProfileController::class, 'edit'])->name('profile.edit');
```
🚫 Bad:
```php
Route::get('/profileEdit', [ProfileController::class, 'edit']);
```

---

## 🔒 7. Explicitly Define Middleware and Guards
```php
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
});
```

---

## 🧰 8. Use Route Model Binding
```php
Route::get('/users/{user}', [UserController::class, 'show']);
```
Controller:
```php
public function show(User $user)
{
    return view('users.show', compact('user'));
}
```

---

## 🧾 9. Follow PSR-12 Example Formatting
```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ProductController;

Route::middleware(['auth'])
    ->prefix('products')
    ->as('products.')
    ->group(function () {
        Route::get('/', [ProductController::class, 'index'])->name('index');
        Route::get('/{product}', [ProductController::class, 'show'])->name('show');
        Route::post('/', [ProductController::class, 'store'])->name('store');
    });
```

---

## 🧠 10. Use Route Caching for Production
```bash
php artisan route:cache
```

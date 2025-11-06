# Laravel Middleware Guide (PSR-12 Standard)

## ✅ What is Middleware?

Middleware acts as a filter between a client request and the application response. Laravel middleware allows you to inspect, modify, or reject requests before they reach the controller.

---

## ✅ Why Use Middleware?

| Purpose           | Examples                                      |
| ----------------- | --------------------------------------------- |
| Security          | Authentication, Authorization                 |
| Request handling  | CORS Handling, Request Logging                |
| Response updating | Add headers, Modify response before returning |

---

## ✅ Middleware Directory Structure

```
app/
└── Http/
    ├── Kernel.php
    └── Middleware/
        ├── Authenticate.php
        ├── EnsureUserIsActive.php
        └── LogRequestActivity.php
```

> For large applications, group middleware based on concern:

```
app/Http/Middleware/Web/
app/Http/Middleware/Api/
app/Http/Middleware/Auth/
```

---

## ✅ PSR-12 Coding Standard Middleware Example

```php
<?php

declare(strict_types=1);

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsActive
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        // ✅ BEFORE controller execution
        if ($request->user() && !$request->user()->is_active) {
            return response()->json([
                'message' => 'Your account is inactive.',
            ], Response::HTTP_FORBIDDEN);
        }

        /** @var Response $response */
        $response = $next($request);

        // ✅ AFTER controller execution
        $response->headers->set('X-App-Version', '1.0.0');

        return $response;
    }
}
```

✅ Strict types
✅ Typed method signature
✅ Clear responsibility

---

## ✅ Registering Middleware

### Route-specific Middleware

```php
// app/Http/Kernel.php
protected $routeMiddleware = [
    'active' => \App\Http\Middleware\EnsureUserIsActive::class,
];
```

### Middleware Groups

```php
protected $middlewareGroups = [
    'web' => [
        // web middleware stack
    ],
    'api' => [
        // api middleware stack
    ],
];
```

### Global Middleware

```php
protected $middleware = [
    \App\Http\Middleware\TrustProxies::class,
];
```

---

## ✅ Using Middleware

### Apply to a Route

```php
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware('active');
```

### Apply to a Route Group

```php
Route::middleware(['auth', 'active'])->group(function () {
    Route::get('/profile', ProfileController::class);
});
```

### Apply inside Controller constructor

```php
public function __construct()
{
    $this->middleware('active');
}
```

---

## ✅ Middleware Flow Diagram

```
[ Request ] → [ Middleware (Before) ] → [ Controller ] → [ Middleware (After) ] → [ Response ]
```

---

## ✅ Best Practices

| Recommendation     | ✅ Do This                           | ❌ Avoid This                 |
| ------------------ | ----------------------------------- | ---------------------------- |
| Naming conventions | EnsureUserIsActive                  | UserCheckMiddleware          |
| Responsibility     | Single responsibility               | Too much logic in middleware |
| Business logic     | Keep light (use services if needed) | Heavy DB operations          |
| Typed return       | Response type-hint                  | Mixed return types           |
| Response style     | Consistent JSON structure           | Plain string responses       |

---

## ✅ Standard Error Response Format

```php
return response()->json([
    'status'  => false,
    'message' => 'Your account is inactive.',
], Response::HTTP_FORBIDDEN);
```

---

## 🧠 Useful Tips

* Avoid writing business logic directly inside middleware
* If middleware becomes complex, move logic to Service/Repository layer
* Reuse middleware using groups (*web*, *api*)

---

## ✅ Summary

Laravel middleware is a powerful tool to enforce validation and authorization rules globally or per route. Following PSR-12 ensures clean, maintainable code.

---

# Laravel Exception Handling — PSR-12 Compliant Guide

## ✅ Overview

This document demonstrates how to write clean and maintainable **Exception Handling** in Laravel following **PSR-12 coding standards**.

---

## ✅ Custom Exception Class

Location: `app/Exceptions/InvalidOrderException.php`

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\JsonResponse;

class InvalidOrderException extends Exception
{
    /**
     * Report the exception.
     */
    public function report(): void
    {
        logger()->error('Invalid order action attempted.', [
            'message' => $this->getMessage(),
        ]);
    }

    /**
     * Render the exception into an HTTP response.
     */
    public function render(): JsonResponse
    {
        return response()->json([
            'error' => $this->getMessage(),
        ], 422);
    }
}
```

---

## ✅ Usage Inside Controller

Location: `app/Http/Controllers/OrderController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Exceptions\InvalidOrderException;
use App\Models\Order;
use Illuminate\Http\JsonResponse;

class OrderController extends Controller
{
    public function cancel(string $id): JsonResponse
    {
        $order = Order::findOrFail($id);

        if ($order->status !== 'pending') {
            throw new InvalidOrderException('Only pending orders can be canceled.');
        }

        $order->update(['status' => 'canceled']);

        return response()->json([
            'message' => 'Order canceled successfully.',
        ]);
    }
}
```

---

## ✅ Customize Global Exception Handling

Modify: `app/Exceptions/Handler.php`

```php
<?php

namespace App\Exceptions;

use Throwable;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Illuminate\Database\Eloquent\ModelNotFoundException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Illuminate\Http\JsonResponse;

class Handler extends ExceptionHandler
{
    protected $dontReport = [];

    public function register(): void
    {
        $this->reportable(function (Throwable $e): void {
            // Sentry/Bugsnag logging can go here
        });
    }

    public function render($request, Throwable $exception): JsonResponse
    {
        if ($exception instanceof ModelNotFoundException) {
            return response()->json([
                'error' => 'Resource not found.',
            ], 404);
        }

        if ($exception instanceof NotFoundHttpException) {
            return response()->json([
                'error' => 'Route not found.',
            ], 404);
        }

        return parent::render($request, $exception);
    }
}
```

---

## ✅ JSON Response for Validation Errors

Add in `Handler.php`:

```php
public function invalidJson($request, ValidationException $exception): JsonResponse
{
    return response()->json([
        'message' => $exception->getMessage(),
        'errors' => $exception->errors(),
    ], $exception->status);
}
```

---

## 🔥 Best Practices

| Best Practice                                  | Benefit                               |
| ---------------------------------------------- | ------------------------------------- |
| Use custom exceptions                          | Better readability and reusability    |
| Use `report()` for logging                     | Keeps separation of concerns          |
| Return JSON for APIs                           | Easier for frontend and API consumers |
| Avoid try/catch in controllers unless required | Leverage Laravel Handler              |
| Follow PSR-12 formatting                       | Cleaner, standardized codebase        |

---

# Laravel Controller Best Practices (PSR-12 Standard)

## ✅ Controller Structure and Best Practices

### 1. Namespace Declaration

Each controller must have the proper namespace at the top of the file.

```php
<?php

namespace App\Http\Controllers;
```

---

### 2. Imports / Use Statements

* Must be placed after namespace.
* Import request classes, services, or model dependencies.
* Avoid unused imports.

```php
use App\Services\UserService;
use App\Http\Requests\UserStoreRequest;
use App\Http\Requests\UserUpdateRequest;
use Illuminate\Http\JsonResponse;
```

---

### 3. PSR-12 Class Format

* Class name must be `PascalCase`.
* Controller should only handle HTTP flow, not business logic.

```php
class UserController extends Controller
{
    public function __construct(
        private readonly UserService $userService
    ) {}
```

---

### 4. Method Naming Convention & Order (RESTful)

| Method      | Responsibility         |
| ----------- | ---------------------- |
| `index()`   | Fetch list of records  |
| `store()`   | Create new record      |
| `show()`    | Fetch a single record  |
| `update()`  | Update existing record |
| `destroy()` | Delete record          |

---

### 5. Validation Best Practice

* Use **Form Requests** instead of writing validation in controllers.

```php
public function store(UserStoreRequest $request): JsonResponse
{
    $user = $this->userService->create($request->validated());

    return response()->json($user, 201);
}
```

---

### 6. Response Standard

Always return a `JsonResponse` or `Response` with proper HTTP status code.

```php
public function show(int $id): JsonResponse
{
    $user = $this->userService->find($id);

    return response()->json($user);
}
```

---

### 7. Avoid Fat Controllers (Thin Controller Principle)

Business logic should not be written in controllers.
Controllers should:

* Receive request →
* Call Service Layer →
* Return Response

---

### ✅ Example: Full Controller (PSR-12 Compliant)

```php
<?php

namespace App\Http\Controllers;

use App\Services\UserService;
use App\Http\Requests\UserStoreRequest;
use App\Http\Requests\UserUpdateRequest;
use Illuminate\Http\JsonResponse;

class UserController extends Controller
{
    public function __construct(
        private readonly UserService $userService
    ) {}

    public function index(): JsonResponse
    {
        $users = $this->userService->getAll();

        return response()->json($users);
    }

    public function store(UserStoreRequest $request): JsonResponse
    {
        $user = $this->userService->create($request->validated());

        return response()->json($user, 201);
    }

    public function show(int $id): JsonResponse
    {
        $user = $this->userService->find($id);

        return response()->json($user);
    }

    public function update(UserUpdateRequest $request, int $id): JsonResponse
    {
        $updatedUser = $this->userService->update($id, $request->validated());

        return response()->json($updatedUser);
    }

    public function destroy(int $id): JsonResponse
    {
        $this->userService->delete($id);

        return response()->json(null, 204);
    }
}
```

---

### 🎯 Summary Checklist

# Laravel Service & Repository Architecture (PSR-12 Standard)

## ✅ Goal

Keep controllers thin by moving business logic to **Service Layer** and database interaction to **Repository Layer**.

---

## 🏛 Folder Structure

```
app/
├── Http/
│   └── Controllers/
├── Services/
│   └── UserService.php
└── Repositories/
    └── UserRepository.php
```

---

## ✅ Repository Layer

Repository acts as the **data access layer**: handles DB queries only.

📄 **app/Repositories/UserRepository.php**

```php
<?php

namespace App\Repositories;

use App\Models\User;

class UserRepository
{
    public function getAll(): array
    {
        return User::all()->toArray();
    }

    public function find(int $id): ?User
    {
        return User::find($id);
    }

    public function create(array $data): User
    {
        return User::create($data);
    }

    public function update(User $user, array $data): User
    {
        $user->update($data);
        return $user;
    }

    public function delete(User $user): bool
    {
        return $user->delete();
    }
}
```

---

## ✅ Service Layer

Service contains **business logic** and interacts with repository.

📄 **app/Services/UserService.php**

```php
<?php

namespace App\Services;

use App\Repositories\UserRepository;
use App\Models\User;
use Exception;

class UserService
{
    public function __construct(
        private readonly UserRepository $repository
    ) {}

    public function getAll(): array
    {
        return $this->repository->getAll();
    }

    public function find(int $id): ?User
    {
        return $this->repository->find($id);
    }

    public function create(array $data): User
    {
        return $this->repository->create($data);
    }

    public function update(int $id, array $data): User
    {
        $user = $this->find($id);

        if (! $user) {
            throw new Exception('User not found');
        }

        return $this->repository->update($user, $data);
    }

    public function delete(int $id): bool
    {
        $user = $this->find($id);

        if (! $user) {
            throw new Exception('User not found');
        }

        return $this->repository->delete($user);
    }
}
```

---

## 💡 Dependency Injection

Controller constructor injects **UserService**, and UserService injects **UserRepository**, achieving **single responsibility**.

✅ Controller → ➡️ Service → ➡️ Repository → ➡️ Model

---

## 🔥 Benefits

| Benefit                | Why it matters                                                 |
| ---------------------- | -------------------------------------------------------------- |
| Separation of concerns | Keeps controllers clean and readable                           |
| Reusable logic         | Service methods can be reused in other controllers or commands |
| Replaceable DB layer   | Easily switch from Eloquent to API/Third-party data source     |
| Easy to test           | Unit test service without DB, mock repository                  |

---

## 🎯 Summary

* Controller = only request/response handling
* Service = business logic
* Repository = DB layer
* PSR-12 formatting enforced

---

> **Thin Controllers. Fat Services. Clean Architecture.**


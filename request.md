# Laravel Request Validation (PSR-12 Compliant)

## ✅ Overview

This document provides the best standard approach for handling **Laravel Request Validation** according to:

* Laravel Official Documentation
* PSR-12 Coding Standards
* Clean Code & SRP (Single Responsibility Principle)

---

## 📌 Why Use Form Requests Instead of Inline Validation?

| Inline Validation         | Form Request Validation             |
| ------------------------- | ----------------------------------- |
| Makes controller messy    | Controller stays clean and readable |
| No authorization handling | Includes `authorize()` logic        |
| Not reusable              | Fully reusable across controllers   |
| Hard to test              | Easy to unit test                   |

---

## 📁 Folder Structure

```
app/
 └── Http/
      └── Requests/
           ├── StoreUserRequest.php
           └── UpdateUserRequest.php
```

---

## ✅ StoreUserRequest (POST - Create)

**File:** `app/Http/Requests/StoreUserRequest.php`

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     */
    public function authorize(): bool
    {
        return true;
    }

    /**
     * Prepare the data for validation.
     */
    protected function prepareForValidation(): void
    {
        $this->merge([
            'email' => strtolower((string) $this->email),
        ]);
    }

    /**
     * Get validation rules.
     */
    public function rules(): array
    {
        return [
            'name'      => ['required', 'string', 'max:255'],
            'email'     => ['required', 'email', 'unique:users,email'],
            'password'  => ['required', 'string', 'min:8'],
            'age'       => ['nullable', 'integer', 'min:18'],
        ];
    }

    /**
     * Custom error messages.
     */
    public function messages(): array
    {
        return [
            'name.required'     => 'Name is required.',
            'email.unique'      => 'Email already exists.',
            'password.min'      => 'Password must be at least 8 characters.',
        ];
    }

    /**
     * Custom attribute names.
     */
    public function attributes(): array
    {
        return [
            'email'     => 'email address',
            'password'  => 'account password',
        ];
    }
}
```

---

## ✅ UpdateUserRequest (PUT/PATCH - Update)

**File:** `app/Http/Requests/UpdateUserRequest.php`

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class UpdateUserRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     */
    public function authorize(): bool
    {
        return true;
    }

    /**
     * Prepare the data for validation.
     */
    protected function prepareForValidation(): void
    {
        $this->merge([
            'email' => strtolower((string) $this->email),
        ]);
    }

    /**
     * Get validation rules.
     */
    public function rules(): array
    {
        $userId = $this->route('user');

        return [
            'name'      => ['sometimes', 'string', 'max:255'],
            'email'     => [
                'sometimes',
                'email',
                Rule::unique('users', 'email')->ignore($userId),
            ],
            'password'  => ['sometimes', 'string', 'min:8'],
            'age'       => ['nullable', 'integer', 'min:18'],
        ];
    }

    /**
     * Custom error messages.
     */
    public function messages(): array
    {
        return [
            'email.unique'      => 'Email already exists for another user.',
            'password.min'      => 'Password must be at least 8 characters.',
        ];
    }

    /**
     * Custom attribute names.
     */
    public function attributes(): array
    {
        return [
            'email' => 'email address',
        ];
    }
}
```

---

## ✅ Controller Usage (Clean & PSR-12 Compliant)

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StoreUserRequest;
use App\Http\Requests\UpdateUserRequest;
use App\Models\User;

class UserController extends Controller
{
    public function store(StoreUserRequest $request)
    {
        User::create($request->validated());

        return response()->json([
            'message' => 'User created successfully.',
        ]);
    }

    public function update(UpdateUserRequest $request, User $user)
    {
        $user->update($request->validated());

        return response()->json([
            'message' => 'User updated successfully.',
        ]);
    }
}
```

---

## 🔥 Key Differences

| POST (Create)           | PUT/PATCH (Update)                                 |
| ----------------------- | -------------------------------------------------- |
| Uses `StoreUserRequest` | Uses `UpdateUserRequest`                           |
| All fields are required | Fields are validated only if present (`sometimes`) |

---

## 🧠 PSR-12 Rules Applied

* Braces on new lines
* Typed return values (`: array`, `: bool`)
* Organized methods in order (authorize → prepareForValidation → rules → messages → attributes)

---

## ✅ Conclusion

Using **Form Request Validation** keeps controllers clean, organized, and follows Laravel best practices while adhering to **PSR-12 standards**.

> Always use Request classes in place of `$request->validate()` for better maintainability.

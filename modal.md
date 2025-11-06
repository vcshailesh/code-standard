# Laravel Eloquent Model — PSR‑12 Standard Guide

## ✅ Folder Structure (Recommended)

```
app/
└── Models/
    ├── User.php
    ├── Order.php
    └── Traits/
        └── HasUuid.php
```

---

## ✅ PSR‑12 Coding Standards Applied

* Import statements alphabetically
* One class per file
* Clear order of sections

```
1. Properties ($fillable, $casts, etc.)
2. Boot methods (boot / booted)
3. Relationships
4. Accessors & Mutators (Attribute)
5. Scopes (Local + Global)
```

---

## ✅ `User` Model (Relationships + Accessor + Mutator + Scopes)

**`app/Models/User.php`**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Support\Str;

class User extends Authenticatable
{
    use HasFactory;

    /**
     * Mass assignable attributes.
     *
     * @var array<int, string>
     */
    protected array $fillable = [
        'uuid',
        'name',
        'email',
        'status',
        'password',
    ];

    /**
     * Hidden attributes.
     *
     * @var array<int, string>
     */
    protected array $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * Attribute casting.
     *
     * @var array<string, string>
     */
    protected array $casts = [
        'email_verified_at' => 'datetime',
    ];

    /**
     * Model event booting (auto-generate UUID).
     */
    protected static function booted(): void
    {
        static::creating(function (Model $model): void {
            if (empty($model->uuid)) {
                $model->uuid = Str::uuid()->toString();
            }
        });
    }

    /**
     * Relationship: User has many Orders.
     */
    public function orders(): HasMany
    {
        return $this->hasMany(Order::class);
    }

    /**
     * Accessor & Mutator (normalize name).
     */
    protected function name(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
            set: fn (string $value) => strtolower($value)
        );
    }

    /**
     * Local Scope — filter active users.
     */
    public function scopeActive(Builder $query): Builder
    {
        return $query->where('status', '=', 'active');
    }

    /**
     * Global Scope — apply automatically for all queries.
     */
    protected static function boot(): void
    {
        parent::boot();

        static::addGlobalScope('notDeleted', function (Builder $builder) {
            $builder->whereNull('deleted_at');
        });
    }
}
```

---

## ✅ `Order` Model (Relationship Reference)

**`app/Models/Order.php`**

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Order extends Model
{
    use HasFactory;

    protected array $fillable = [
        'order_no',
        'amount',
        'status',
        'user_id',
    ];

    /**
     * Relationship: Order belongs to user.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

---

## ✅ Summary

| Feature                  | Covered | In Code               |
| ------------------------ | ------- | --------------------- |
| PSR‑12 coding compliance | ✅       | Entire file           |
| Relationships            | ✅       | `orders()` / `user()` |
| Accessor + Mutator       | ✅       | `name()` attribute    |
| `booted()` Model Events  | ✅       | UUID auto-create      |
| Local Scope              | ✅       | `scopeActive()`       |
| Global Scope             | ✅       | `addGlobalScope()`    |

---

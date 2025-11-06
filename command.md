# Laravel Console Commands PSR-12 Guide

## ✅ Overview

Console (Artisan) commands help to automate application-level tasks. Following **PSR-12 coding standards** ensures clean, readable, and maintainable code.

---

## 📌 Command Creation

Use Artisan to create a new command:

```bash
php artisan make:command SendReportCommand
```

Command will be generated at:

```
app/Console/Commands/SendReportCommand.php
```

---

## ✅ PSR-12 Compliant Structure

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Services\ReportService;

class SendReportCommand extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'report:send {--type=daily : Report type (daily|weekly)}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send reports to admin via email.';

    /** @var ReportService */
    private ReportService $reportService;

    /**
     * Inject dependencies (Constructor Injection).
     */
    public function __construct(ReportService $reportService)
    {
        parent::__construct();

        $this->reportService = $reportService;
    }

    /**
     * Execute the console command.
     */
    public function handle(): int
    {
        $reportType = $this->option('type');

        $this->info("Sending {$reportType} report...");

        try {
            $this->reportService->send($reportType);
            $this->info('✅ Report sent successfully!');
        } catch (\Throwable $exception) {
            $this->error('❌ Error: ' . $exception->getMessage());

            return Command::FAILURE;
        }

        return Command::SUCCESS;
    }
}
```

---

## 🧠 Registering Command (Kernel)

Add the command in `app/Console/Kernel.php`:

```php
protected $commands = [
    \App\Console\Commands\SendReportCommand::class,
];
```

### Optional: Schedule Command

```php
protected function schedule(Schedule $schedule): void
{
    $schedule->command('report:send --type=daily')->daily();
}
```

---

## 🔥 Best Practices

| Key Area             | Best Practice                                                        |
| -------------------- | -------------------------------------------------------------------- |
| Naming               | Use action-based names (`SendReportCommand`)                         |
| Responsibility       | Command only manages CLI flow; business logic stays in Service Layer |
| Dependency Injection | Inject services into constructor                                     |
| Messaging            | Use `$this->info()`, `$this->error()`, `$this->warn()`               |
| Exit Codes           | Always return `Command::SUCCESS` or `Command::FAILURE`               |

---

## ✨ Argument & Option Usage

```php
protected $signature = 'user:notify {userId} {--queue}';
```

### Example: Default option values

```php
protected $signature = 'report:send {--type=daily}';
```

---

## 🔐 User Interaction

```php
if ($this->confirm('Do you want to continue?')) {
    // logic
}
```

---

## 📊 Progress Bar Example

```php
$bar = $this->output->createProgressBar($users->count());

foreach ($users as $user) {
    // process
    $bar->advance();
}

$bar->finish();
```

---

### ✅ Result: Your Commands Are Now

✔ PSR-12 Compliant
✔ Clean and Maintainable
✔ Testable & Scalable

---

> For extended architecture (Service + Repository Layer), ask: **"Generate console command architecture with service + repository"**

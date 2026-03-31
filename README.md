Laravel Task Management API - Assessment

Overview
--------
This project is a Laravel-style implementation for the Task Management API assessment. It implements core task operations, business rules, feature tests, seeders, and a minimal development server for manual API testing.

Requirements
------------
- PHP >= 8.0
- Composer
- MySQL

Quick Local Setup
-----------------
1. Copy the environment example and configure MySQL credentials:

```bash
cp .env.example .env
# edit .env to set DB_HOST, DB_DATABASE, DB_USERNAME, DB_PASSWORD
```

2. Install dependencies:

```bash
composer install
```

3. Generate the application key, migrate and seed:

```bash
php artisan key:generate
php artisan migrate
php artisan db:seed --class=TaskSeeder
```

4. Run the feature tests:

```bash
php artisan test
```

5. Start the local server and test the API:

Use the Laravel built-in server if you have a full environment:

```bash
php artisan serve --host=127.0.0.1 --port=8000
```

Or use the provided minimal `server.php` for quick manual testing (SQLite-backed):

```bash
php -S 127.0.0.1:8001 server.php
curl http://127.0.0.1:8001/api/tasks
```

Files of interest
-----------------
- `database/migrations/2026_03_30_000000_create_tasks_table.php` — creates `tasks` table and unique title+due_date index
- `app/Models/Task.php` — Eloquent model and helper methods
- `app/Http/Controllers/TaskController.php` — API methods
- `app/Http/Requests/StoreTaskRequest.php` and `UpdateTaskRequest.php` — validation rules
- `app/Http/Resources/TaskResource.php` — JSON representation
- `database/seeders/TaskSeeder.php` and `database/factories/TaskFactory.php` — seed test data

API Endpoints
-------------

1) Create Task
- POST /api/tasks
- Validation: `title` (required, unique per `due_date`), `priority` in `low|medium|high`, `due_date` today or later
- Returns: 201 with created task

2) List Tasks
- GET /api/tasks
- Behavior: sorted by priority (`high` -> `medium` -> `low`) then `due_date` ascending. Optional `status` query parameter to filter.
- Returns: JSON list or an empty `data` array with a message if no tasks exist

3) Update Task Status
- PATCH /api/tasks/{id}/status
- Validation: `status` must be one of `pending|in_progress|done`
- Business rule: status can only progress (cannot revert). Invalid transitions return 422.

4) Delete Task
- DELETE /api/tasks/{id}
- Business rule: only tasks with status `done` may be deleted. Other attempts return 403.

5) Daily Report (bonus)
- GET /api/tasks/report?date=YYYY-MM-DD
- Returns counts per priority and status for the specified date.

Example Requests
----------------

Create task
```bash
curl -X POST http://localhost:8000/api/tasks \\
	-H "Content-Type: application/json" \\
	-d '{"title":"Fix login bug","due_date":"2026-04-01","priority":"high"}'
```

List tasks
```bash
curl http://localhost:8000/api/tasks
```

Update status
```bash
curl -X PATCH http://localhost:8000/api/tasks/1/status \\
	-H "Content-Type: application/json" \\
	-d '{"status":"in_progress"}'
```

Delete task
```bash
curl -X DELETE http://localhost:8000/api/tasks/1
```

Daily report
```bash
curl http://localhost:8000/api/tasks/report?date=2026-03-30
```

Deployment Notes (short)
------------------------
Recommended hosts: Railway, Render, or any provider supporting PHP + MySQL.

Basic steps:
1. Push repo to GitHub and connect to the provider.
2. Provision a MySQL database and add DB credentials to environment variables.
3. Set `APP_ENV=production` and `APP_DEBUG=false`.
4. Run migrations on the host: `php artisan migrate --force`.


# Hunt ST — Technical Assessment

Code for the two-part Laravel assessment:

1. **N+1 query review**: fixing an N+1 problem on a page that lists 50 posts and their authors.
2. **Laravel 11 → 13 upgrade**: upgrading this app one major version at a time.

The screen recordings explain the reasoning for both parts:

- [Part I — N+1 query review](https://drive.google.com/file/d/1FJgNc6TWTmXf4MR_kEqgaGN1FZPjlhp9/view?usp=sharing)
- [Part II — Laravel 11 → 13 upgrade](https://drive.google.com/file/d/1FvYiCFgV94uAwkXiutw-3IZfSW0t8Cj-/view?usp=sharing)

This README covers how to run the project and what changed.

**Stack:** Laravel 13 · PHP 8.3+ · Inertia + React/TypeScript (Breeze) · Pest · Telescope · SQLite

## Running locally

```bash
composer install
npm install
cp .env.example .env
php artisan key:generate
touch database/database.sqlite
php artisan migrate --seed   # 1 test user + 100 posts
composer dev                 # server, queue, logs and Vite
```

- `/posts` is the page from Part 1.
- `/telescope` shows each request's queries.
- Run the tests with `./vendor/bin/pest`.

## Part 1 — N+1 fix

[Video walkthrough](https://drive.google.com/file/d/1FJgNc6TWTmXf4MR_kEqgaGN1FZPjlhp9/view?usp=sharing) · Commit `4bd6303`:

- **[routes/web.php](routes/web.php):** the `/posts` route eager-loads the author with `Post::with('author:id,name')` and selects only the columns the page uses (`id`, `title`, `author_id`, `created_at`).
- **[AppServiceProvider](app/Providers/AppServiceProvider.php):** `Model::preventLazyLoading()` is enabled outside production, so any new lazy load throws an exception.

| 100 seeded posts | Queries | Time (local SQLite) |
| ---------------- | ------: | ------------------: |
| Before           |      51 |              5.4 ms |
| After            |       2 |              1.2 ms |

## Part 2 — Upgrade

[Video walkthrough](https://drive.google.com/file/d/1FvYiCFgV94uAwkXiutw-3IZfSW0t8Cj-/view?usp=sharing)

| Commit    | Change                                                                                                     |
| --------- | ---------------------------------------------------------------------------------------------------------- |
| `b60e751` | Laravel 11 → 12.70. No application code changes needed.                                                    |
| `8a7d7c4` | Laravel 12 → 13.36, `laravel/tinker` ^3, PHP constraint raised to `^8.3`.                                  |
| `de672b2` | Laravel 13 skeleton hardening: cache `serializable_classes => false`, session `serialization => json`.     |

After each step, all 25 tests passed and `composer audit` was clean.

The frontend packages (Vite, `laravel-vite-plugin`, Tailwind, React) have not been upgraded to the 13.x skeleton's versions yet.

## AI usage disclosure

I used **Claude Code (Claude Opus)** to:

- check the codebase against the official Laravel 12 and 13 upgrade guides
- run the Composer upgrades and tests
- draft this README

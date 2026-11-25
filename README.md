# Laravel Production Checklist

A practical checklist I use before launching (or taking over) a Laravel application. It's opinionated, short, and based on real client projects — not a complete security audit.

Pull requests and suggestions are welcome.

---

## 1. Environment & config

- [ ] `APP_ENV=production` and `APP_DEBUG=false`
- [ ] `APP_KEY` set and backed up somewhere safe
- [ ] No `env()` calls outside `config/` files (they return `null` after `config:cache`)
- [ ] Separate credentials for production, staging and local
- [ ] `.env` is not web-accessible and not in git

## 2. Deploy script

```bash
php artisan down --render="errors::503"
git pull origin main
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache
php artisan queue:restart
php artisan up
```

- [ ] Migrations are new, dated files — never edit a migration that already ran in production
- [ ] Rollback plan written down before the deploy

## 3. Queues

- [ ] Redis (or SQS) driver in production, not `sync` or `database` for heavy loads
- [ ] Workers managed by Supervisor or Horizon
- [ ] `retry_after` is greater than the longest job's `timeout`
- [ ] Jobs are idempotent (safe to run twice)
- [ ] Failed jobs table exists and someone gets alerted

## 4. Caching

- [ ] Cache only measured slow reads (aggregates, settings, third-party API responses)
- [ ] Never cache per-user permissions under a shared key
- [ ] Invalidation lives in model observers/events, not scattered in controllers
- [ ] Locks or `Cache::flexible()` on very hot keys to avoid stampedes

## 5. Security

- [ ] Authorization in Policies/Gates, not only hidden buttons
- [ ] Rate limiting on login, OTP, password reset and public APIs
- [ ] File uploads validated by MIME type and size, stored outside `public/` when private
- [ ] `composer audit` run, and dependencies updated
- [ ] HTTPS enforced, secure + HttpOnly cookies, HSTS
- [ ] Webhooks verify signatures

## 6. Database

- [ ] Indexes on foreign keys and columns used in `where` / `orderBy`
- [ ] N+1 queries checked on the heaviest pages (Debugbar / Telescope / `preventLazyLoading()` in non-production)
- [ ] Automated daily backups, **and a tested restore**

## 7. Monitoring

- [ ] Error tracking (Sentry, Flare or similar)
- [ ] Uptime check on the homepage and a `/health` route
- [ ] Log rotation configured; logs don't fill the disk
- [ ] Scheduler (`schedule:run`) cron verified on the server

## 8. SEO basics (for public sites)

- [ ] Unique titles and meta descriptions
- [ ] XML sitemap generated from models
- [ ] Canonical URLs and consistent trailing slashes
- [ ] Google Search Console connected

## 9. Ownership & handover

- [ ] Repository, hosting, domain and third-party accounts owned by the client
- [ ] README with setup steps, environment variables and deploy process
- [ ] Laravel and PHP versions documented, with an upgrade plan

---

## Further reading

- [Laravel caching in production: what I cache, what I never cache](https://dev.to/arun_tyagi_developer/laravel-caching-in-production-what-i-cache-what-i-never-cache-and-why-3a6k)
- [Versioning a Laravel REST API without making a mess](https://dev.to/arun_tyagi_developer/versioning-a-laravel-rest-api-without-making-a-mess-3kdc)
- [Multi-tenant Laravel: single database or database-per-tenant?](https://aruntyagi-laravel.hashnode.dev/multi-tenant-laravel-single-database-or-database-per-tenant-how-i-decide)
- [How to fix the N+1 query problem in Laravel](https://aruntyagi.com/blog/how-to-fix-the-n1-query-problem-in-laravel)
- [Laravel 11 vs Laravel 12](https://aruntyagi.com/blog/laravel-11-vs-laravel-12)

## Author

Maintained by **Arun Tyagi**, freelance Laravel developer in Noida, India — [aruntyagi.com](https://aruntyagi.com).
Need a production review or a Laravel upgrade? See [Laravel development services](https://aruntyagi.com/laravel-development).

## License

MIT

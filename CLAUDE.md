# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

Rails 8.1 (`PawbiesNet::Application`) on Ruby 4.0.1, SQLite for all four databases (primary + Solid Cache, Solid Queue, Solid Cable), Propshaft asset pipeline, Hotwire (Turbo + Stimulus) over importmap-rails, Tailwind via `tailwindcss-rails`, Puma fronted by Thruster, deployed with Kamal.

Because Solid Queue/Cache/Cable are database-backed (see `config/database.yml` for production's four-DB setup and `db/{cache,queue,cable}_schema.rb`), there is no Redis/Sidekiq dependency. In production `SOLID_QUEUE_IN_PUMA=true` runs the job supervisor inside the web Puma — splitting it out requires uncommenting the `job` host block in `config/deploy.yml`.

## Commands

- `bin/setup` — install gems, prepare DB, clear logs/tmp, then exec `bin/dev` (pass `--skip-server` to stop before the server, `--reset` to wipe the DB).
- `bin/dev` — runs `Procfile.dev` via foreman: `bin/rails server` + `bin/rails tailwindcss:watch`. Requires both to render styled pages in dev.
- `bin/rails test` — Minitest suite. Run a single file with `bin/rails test test/controllers/homepage_controller_test.rb`, a single test with `bin/rails test test/.../foo_test.rb:42` (line number) or `-n /pattern/`.
- `bin/rails test:system` — Capybara + Selenium system tests (not part of `bin/ci` by default; uncomment in `config/ci.rb` to include).
- `bin/rubocop` — lint; config inherits `rubocop-rails-omakase`.
- `bin/brakeman --no-pager` — static security scan. `bin/bundler-audit` for gem CVEs, `bin/importmap audit` for JS deps.
- `bin/ci` — runs the full pre-merge pipeline locally (setup, rubocop, all three security scans, `bin/rails test`, `db:seed:replant`). Mirrors `.github/workflows/ci.yml`.
- `bin/kamal deploy` — production deploy. Aliases: `bin/kamal console`, `bin/kamal shell`, `bin/kamal logs`, `bin/kamal dbc`.

## Conventions worth knowing

- Style is Omakase Rails — overrides go in `.rubocop.yml`. Don't fight the defaults.
- `config/application.rb` calls `config.autoload_lib(ignore: %w[assets tasks])`, so any new top-level `lib/` subdirectory you don't want autoloaded must be added there.
- `config/routes.rb` has `root "homepage#show"`; the `HomepageController` is currently a placeholder action with an empty body.
- The `/up` route is the Rails health check for load balancers — keep it returning 200.

Title: feat(skills): add Ruby on Rails framework testing skill

## Summary

Adds `strix/skills/frameworks/rails.md`, a 233-line knowledge package for testing Ruby on Rails apps (6.1 through 8). The loader picks it up automatically from `strix/skills/frameworks/`. The frameworks list in `strix/skills/README.md` and the table in `docs/advanced/skills.mdx` now include `rails`.

## What the skill covers

- Attack surface: `routes.rb` / `resources`, `before_action` chains, strong parameters, ActiveRecord, ERB/HAML, ActionCable, ActiveStorage, ActionMailer previews, engines and admin gems, credentials / `secret_key_base`, Sidekiq/ActiveJob dashboards
- Mass assignment: `permit!`, nested attributes, `accepts_nested_attributes_for`
- SQL injection: interpolated `where` / `order` / `pluck` / `find_by_sql` / `calculate` and `Arel.sql` wrapping (including the 6.1+ `UnknownAttributeReference` on raw `order` strings)
- Unsafe `render inline:` / `render file:`, `constantize` / `send` / `public_send`
- Open redirects: `redirect_to params[...]`, `allow_other_host`, `raise_on_open_redirects` / `action_on_open_redirect` defaults by `load_defaults`
- CSRF gaps: `skip_forgery_protection`, `null_session` on cookie-auth, GET mutators
- IDOR via `Model.find(params[:id])` without `current_user` scoping
- Unsafe deserialization: `YAML.unsafe_load`, Marshal cookies vs JSON (`load_defaults` 7.0+)
- ActiveStorage direct upload and blob proxy/redirect
- ActionCable channel authorization
- Exposed `/rails/info`, `/rails/mailers`, web-console, Sidekiq Web, letter_opener
- `config.hosts` / DNS rebinding
- Source-aware `brakeman` and `bundler-audit`

Each pattern has a black-box probe, a white-box grep/scanner hint, a validation rule, and a false-positive note.

## How it was validated

- `uv run python` against `get_all_skill_names()`, `validate_requested_skills()`, `load_skills(["rails"])`, and `render_system_prompt(skills=["rails"])`. The body is injected under `<rails>` the same way other requested skills are (after the cache point). An unknown name (`nope`) is rejected. With `rails.md` hidden, `rails` is not a valid skill name.
- Seeded Rails 7.2.2 app (`/tmp/vulnshop`): `POST /users` with `user[admin]=true&user[role]=admin` created an admin; interpolated `order(Arel.sql(...))` flipped row order under the skill's `CASE WHEN (SELECT 1)=1` vs `=0` probes; `GET /login?next=https://evil.example/` returned `302 Location: https://evil.example/`.
- `brakeman` on that app: Mass Assignment (2), SQL Injection (1), Redirect (1).
- Commands (exit 0 unless noted):
  - `uv run ruff format --check .` — 255 files already formatted
  - `uv run ruff check .` — All checks passed
  - `uv run mypy strix/` — Success: no issues found in 137 source files
  - `uv run bandit -r strix/ -c pyproject.toml` — No issues identified
  - `uv run pytest -q` on skill-touching tests: 65 passed, 1 failed (`test_every_vulnerability_skill_declares_its_phrasings` wants a phrasing ledger entry for `query_language_injection`; that skill is already on `main`)
  - Full pytest on this tree: 2532 passed, 9 failed, 3 xfailed. The 9 failures are budget-pause, e2e budget lifecycle, pricing, and the phrasing ledger above. None load or mention this skill.

`make check-all` stops in `pyright` (945 errors on this tree, including files this change does not touch). CI runs ruff, mypy, and bandit, not pyright.

## Not covered

- Live ActionCable, ActiveStorage, Sidekiq Web, or mailer-preview exploitation on the seeded app (those routes were not mounted there)
- `bundler-audit` against a production `Gemfile.lock`
- Cookie forgery with a leaked `secret_key_base`
- A full Strix agent run against a remote Rails target
- pyright (`make check-all`) on this environment

## Checklist

- [x] `uv run ruff format --check .`
- [x] `uv run ruff check .`
- [x] `uv run mypy strix/`
- [x] `uv run bandit -r strix/ -c pyproject.toml`
- [ ] `make check-all` (pyright)
- [ ] All tests pass
- [ ] Linked GitHub issue
- [ ] Tested in real scenarios
- [ ] A human reviewed this change
- [ ] Screenshots attached by hand

## Demo

![demo](https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1115b/rails-framework-skill/preview.gif)

<video src="https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1115b/rails-framework-skill/out.mp4" controls width="720"></video>

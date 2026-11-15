# Demo: Rails framework skill

Recorded from the working tree of `feat/rails-framework-skill` (`83e37386071188c50a6cad612c1bbd08702bae81`) plus a seeded Rails 7.2.2 app in `/tmp/vulnshop`.

`out.mp4` / `preview.gif` are an animated replay of `commands.log`. The banner on the film reads: Animated replay of real captured command output.

## What was captured

1. **Before.** `rails.md` was moved aside. `ls strix/skills/frameworks/` showed `django.md`, `fastapi.md`, `nestjs.md`, `nextjs.md`. `get_all_skill_names()` did not include `rails`. `validate_requested_skills(["rails"])` returned an invalid-name error.
2. **After.** The file was restored. `ls` listed `rails.md`. The loader returned the Rails body (`permit!` present). `render_system_prompt(skills=["rails"])` inlined it under `<rails>` after the cache point and listed `frameworks/rails` in `<available_skills>`.
3. **Refusal.** `validate_requested_skills(["nope"])` returned `Invalid skill name(s): ['nope']`.
4. **Repo checks.** `ruff format --check`, `ruff check`, `mypy strix/`, and `bandit -r strix/` each exited 0.
5. **Tests.** Skill-touching pytest: 65 passed, 1 failed (`test_every_vulnerability_skill_declares_its_phrasings` for `query_language_injection`, already on `main`). Broader pytest summaries are in `logs/`.
6. **Seeded app.** Rails 7.2.2 on 127.0.0.1:3000 with `permit!`, `User.order(Arel.sql("#{params[:sort]} ..."))`, and `redirect_to params[:next], allow_other_host: true`.
   - `POST /users` with `user[email]=a@x&user[admin]=true&user[role]=admin` → 201, `admin: true`, `role: admin`.
   - `GET /users?sort=CASE+WHEN+(SELECT+1)=1+THEN+email+ELSE+id+END` vs `=0` flipped row order (email vs id).
   - `GET /login?next=https://evil.example/` → 302, `Location: https://evil.example/`.
7. **Brakeman.** High mass-assignment (2), high SQL injection (1), weak redirect (1). Exit 3 (warnings present).

## Media

See `logs/ffprobe.txt`. MP4 is 1920×1080, 30 fps, 720 frames, 24.0 s, H.264 yuv420p. GIF is 720×405, 12 fps, 288 frames, 24.0 s, 5.4 MiB.

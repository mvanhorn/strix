# Demo: `strix diff`

Method: **fixture runs written with strix.report** (`strix.report.writer.write_run_record`, `write_vulnerabilities`, `strix.report.sarif.write_sarif`). No live scan.

Working directory for the captured commands: `/tmp/strix-diff-demo` with `./strix_runs/before-fix` and `./strix_runs/after-fix`.

## Fixture

- `before-fix` (run A): four findings — SQL injection at `app.py:4`, XSS at `search.py:18`, CSRF (high) at `settings.py:22`, IDOR (medium) at `users.py:9`.
- `after-fix` (run B): XSS and IDOR gone; CSRF still present at medium; SQL injection moved to `db/query.py:12`; new high command injection at `upload.py:40`.

## Captured commands

Exact stdin/stdout/stderr and exit codes are in `commands.log`:

1. `strix diff before-fix after-fix` — strict match: 2 new, 3 fixed, 1 still present (`high -> medium`).
2. `strix diff before-fix after-fix --match class` — class match treats the moved SQL injection as still present: 1 new, 2 fixed, 2 still present.
3. `strix diff before-fix after-fix --json | jq .` — stable JSON (summary, ids, locations, severities).
4. `strix diff before-fix after-fix --fail-on-new high; echo $?` — prints the report, then `1`.
5. `strix diff before-fix missing-run` — missing-run error and the available run names; exit 1.

## Media

- `out.mp4`: HyperFrames composition of the exact `commands.log` lines. Banner: `Animated replay of real captured command output`. 1920×1080, 30 fps, 24.0 s, 720 frames, H.264 yuv420p.
- `preview.gif`: ffmpeg `palettegen` / `paletteuse` from that MP4. 720×405, 12 fps (288 frames / 24.0 s), last ~2 s held on the fail-on-new + missing-run output, 7.1 MB.
- `screenshots/`: stills from the same composition at 0.5 s, 6 s, 18 s, and 23 s.
- `logs/ffprobe.txt`: ffprobe for both files.

## Checks recorded under `logs/`

See `logs/` for full command output of format/lint/mypy/bandit, pyright, pytest, and the upstream `main` controls for tests that already fail or hang there.

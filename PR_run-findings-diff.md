Title: feat(cli): compare findings between two runs with strix diff

## Summary

`strix diff RUN-A RUN-B` compares findings from two completed runs and prints what is new, what was fixed, and what is still present (including severity changes such as `high -> medium`). Each argument is a run name under `./strix_runs` or a path to a run directory. It does not start a scan, talk to Docker, or need a model.

This is a post-hoc report command, not an in-scan suppress list. Open issue #904 (`--baseline-run`) is a different feature: it would hide known findings while a scan is running.

Matching reuses the fingerprints already written into `findings.sarif`. `--fail-on-new` uses the same severity order and unknown-label handling as `--fail-on`, but exits `1` (the scan gate still exits `2`).

## Usage

```bash
strix diff before-fix after-fix
strix diff before-fix after-fix --match class
strix diff before-fix after-fix --fail-on-new high
```

`--json` prints a stable document (sorted keys) with `summary`, `new` / `fixed` / `still_present`, per-finding ids, locations, and severities. If the two runs used different targets or scan modes, the command warns and still compares. A missing run, a missing `vulnerabilities.json`, or unreadable JSON exits `1` with a clear error (missing names also list the runs under `./strix_runs`).

Shell completion and `strix --help` include `diff`. CLI docs, the CI page, and the README show a re-test-after-fix example.

## How matching works

- `--match strict` (default): SARIF primary fingerprint (`partialFingerprints.primaryLocationLineHash`) — location plus rule.
- `--match class`: file-independent class fingerprint (`properties.strix_vuln_class_hash`), so a finding whose path moved still pairs.

Identity comes from the existing helpers in `strix/report/sarif.py` (`_primary_fingerprint`, `_class_fingerprint`), exposed as `fingerprints_for_finding()` so this command tracks the on-disk SARIF. In-flight fingerprint fixes land automatically: #1590 (stabilize synthetic fingerprints across targets and aliases), #1618 (qualify CWE rule ids with the finding class), and #1575 (ruleId collapsing distinct findings).

Within a fingerprint group, findings pair greedily by sorted id.

## Testing

Commands actually run on this branch:

- [x] `make format-check` — 259 files already formatted
- [x] `make lint-check` (`uv run ruff check .`) — All checks passed
- [x] `uv run mypy strix/` — Success: no issues found in 139 source files
- [x] `uv run bandit -r strix/ -c pyproject.toml` — No issues identified
- [x] `uv run pyright strix/report/diff.py strix/interface/diff_cli.py` — 0 errors, 0 warnings
- [x] `uv run pytest -q tests/test_cli_diff.py tests/test_report_diff.py` — 27 passed
- [x] `uv run pytest -q tests/test_cli_diff.py tests/test_report_diff.py tests/test_completions.py tests/test_cli_fail_on.py tests/test_sarif.py` — 93 passed
- [x] Isolated `uv run pytest -q` excluding three files that hang or fail on clean `main` (see below), plus two `--deselect`s — **2189 passed, 2 deselected** in 112.16s

`make check-all` / `uv run pyright strix/` reports **945 errors** on this tree. The same pyright run is already red on unmodified files (`strix/report/writer.py` is 22 errors with no edits). CI does not run pyright (ruff, mypy, bandit, pytest only).

The following fail or hang on **clean upstream `main` as well** (`4719329`):

- `tests/test_pricing.py::test_resolves_common_bare_model_names` — `grok` is `None`
- `tests/test_report_coverage.py::test_every_vulnerability_skill_declares_its_phrasings` — missing `query_language_injection`
- `tests/test_budget_pause_policy.py` — `_start_child_runner() missing 1 required keyword-only argument: 'skills'`
- `tests/test_tool_call_ids_providers.py` — GNU `timeout 40` exits **124** on both `main` and this branch (hangs in a cost-callback auth poll)

Full command logs are on `demo/run-findings-diff` under `logs/`.

## Not covered

- No live scan in the demo (no model credentials). Fixture runs were written with `strix.report` so the on-disk shape matches a real completed run (`run.json`, `vulnerabilities.json`, `findings.sarif`).
- `make check-all` is not green because of pre-existing pyright errors on `strix/`.
- Unfiltered `uv run pytest -q` was not left running to completion: `tests/test_tool_call_ids_providers.py` hangs on `main` too.

CONTRIBUTING asks for an issue before a code change. None is linked from this body.

- [ ] Issue opened first and linked here
- [ ] Tested in a real (non-fixture) scan
- [ ] A human reviewed the diff

## Demo

![demo](https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1115b/run-findings-diff/preview.gif)

<video src="https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1115b/run-findings-diff/out.mp4" controls></video>

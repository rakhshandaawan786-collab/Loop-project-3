# The Morning Brief with a Memory

A scheduled loop whose second run clearly builds on its first — proven by
running it twice and checking it never repeats what it already recorded.

## What this demonstrates

- **Unattended schedule** — `morning_brief.py` is a single self-contained
  run, meant to be triggered daily by cron / Task Scheduler (no
  interaction needed once scheduled).
- **The spine** — `.brief_state.json` is the persistent memory. Every run
  reads it, diffs the current repo state against it, and overwrites it
  with the new picture. This is what lets run #2 "remember" run #1.

## Repository contents

| File | Purpose |
|---|---|
| `morning_brief.py` | The script that runs once per schedule tick |
| `sample_repo/` | A stand-in for "your repo" — contains TODO comments to scan |
| `.brief_state.json` | The spine — last-seen TODOs, used to compute the diff |
| `progress.md` | The running log — one dated entry appended per run |

## How it works

Each run:
1. Reads `.brief_state.json` (empty dict if it's the very first run ever)
2. Scans `sample_repo/` for `# TODO: ...` comments
3. Computes what's **new** and what's been **resolved** since last time
4. Appends a dated entry to `progress.md` — only the diff, never a full
   repeat of everything found
5. Overwrites `.brief_state.json` with the current full picture

## Run it yourself

```bash
python3 morning_brief.py
```

Run it once — everything found is "new" (no memory yet). Edit a file in
`sample_repo/` (add or resolve a TODO), run it again — the second entry
in `progress.md` will only mention what changed.

## Scheduling it for real (unattended)

**Linux/Mac (cron)** — run daily at 7am:
```
0 7 * * * cd /path/to/morning-brief && python3 morning_brief.py
```

**Windows (Task Scheduler)**:
1. Open Task Scheduler → Create Basic Task
2. Trigger: Daily, pick a time
3. Action: Start a program → `python.exe`, arguments:
   `morning_brief.py`, "Start in": your project folder

## Adapting to a real repo

Point `REPO_DIR` in `morning_brief.py` at your actual project, and/or
swap `scan_todos()` for a `git log --since=yesterday` call if you'd
rather track commits instead of TODOs — the spine logic (state file +
diff) stays exactly the same either way.

## Why this matters (the actual lesson)

Without `.brief_state.json`, every run would look like the first run —
full of "new" findings that were actually reported yesterday too. The
spine is what turns a stateless script into something with continuity.

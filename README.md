# GitHub Automation — README Counter

A scheduled GitHub Actions workflow that automatically increments a counter in this repository's README and logs each run.

This is built to make your github profile look more active then it actually is.
## What this does

This repo runs a workflow (`.github/workflows/increment-readme.yml`) on a schedule via GitHub Actions. On each trigger, it:

1. Decides whether to make any commits today, and how many, based on a weekday/weekend probability pattern
2. Updates the `Count: N` line in `README.md`
3. Logs the run (timestamp, number of commits, run ID) to `activity-log.md`
4. Commits and pushes the changes

## How it works

### Schedule

The workflow runs every 6 hours (`0 */6 * * *`, UTC) via `on.schedule`. It can also be triggered manually from the **Actions** tab using `workflow_dispatch`.

### Activity logic

| Day type | Chance of activity | Commits if active |
|---|---|---|
| Weekday (Mon–Fri) | 85% | 1–6 |
| Weekend (Sat–Sun) | 30% | 1–2 |
| Any day | 10% chance of a "burst" | +3–7 extra commits |

If a run decides on `0` commits, nothing is committed or logged — the workflow exits quietly.

### Files touched

- **`README.md`** — must contain a line formatted exactly as `Count: <number>`. The workflow finds this line, increments it, and writes it back.
- **`activity-log.md`** — auto-created on first run. Appends one table row per active run:

  | Timestamp (UTC) | Commits Made | Run ID |
  |---|---|---|

### Commit messages

Counter-update commits use a randomly selected message from a fixed pool (e.g. `chore: update counter`, `docs: update README stats`) rather than a single repeated string.

## Setup

1. **Add the counter line to `README.md`:**
   ```
   Count: 48
   ```

2. **Enable write permissions for Actions:**
   Go to `Settings → Actions → General → Workflow permissions` and select **Read and write permissions**. Without this, the workflow's `git push` step will fail.

3. **(Optional) Test manually:**
   Go to the **Actions** tab → select the workflow → click **Run workflow** to trigger it immediately instead of waiting for the schedule.

## Local development notes

If you're also committing to `main` manually while the workflow is active, your local branch can diverge from the remote (the workflow may push in between your commits). If `git push` is rejected as non-fast-forward:

```bash
git pull --rebase
git push
```

or set a default reconciliation strategy once:

```bash
git config pull.rebase true   # or: git config pull.rebase false
```

## Known limitations

- **Synthetic activity, not real work.** This workflow generates commits that don't correspond to actual code changes. It's useful for testing scheduling/automation patterns, but it does not represent genuine development activity — anyone inspecting the actual diffs will see repeated one-line README changes.
- **Log growth is unbounded.** `activity-log.md` grows by one row per active run indefinitely; consider rotating or truncating it periodically if this repo runs long-term.
- **Scheduled workflows pause after 60 days of repo inactivity.** GitHub disables `schedule` triggers automatically if there's no other repo activity; you'll need to manually re-enable it if that happens.

## License

Add your preferred license here (e.g. MIT).

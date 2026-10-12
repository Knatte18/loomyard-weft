# Board writers keep unknown fields; lyx cleanup follow-ups

`board-field-drop` · Task · bug, board, batten · running · Merge-Bouncer

Two small bug items, bundled (operator, 2026-10-12).

## 1. An old lyx process drops board fields it does not know (from `board-field-drop`)

### Seen (2026-10-12)

`board-entry-age` landed and deployed new entry fields (`created`, `revised`), and a backfill wrote them for every entry.
Within minutes a status write from a run dropped every date: board commit `5e5402f5` in the board repo rewrote `board.json` without the new fields, and the same happened again after a second backfill.
The writers were long-lived `lyx batten run` and `lyx shed step --until-stop` processes started before the deploy: the old binary decodes an entry into its own struct, so a field it does not know is lost when it writes the board back.

### Why it matters

Any deploy that adds a board field loses that field's data while a run started before the deploy is still writing status, and runs live for hours.
`run_settings` is such a field now: a dropped value silently starts a task without its settings.
The same holds for any other record file a long-lived process rewrites whole.

### Idea

- A board writer keeps fields it does not know: it decodes entries so unknown keys survive a rewrite, or a status write patches only `status` instead of rewriting the entry.
- Or a writer refuses to write a board whose schema is newer than it knows, naming the restart it needs.

## 2. lyx cleanup follow-ups (from `lyx-cleanup-followups`)

### Seen (2026-10-12, PR #574 of `test-tmp-cleanup`)

The PR's own "To check" list left these open after the task landed:

- `TestSweep/two_concurrent_sweeps_both_succeed` in `internal/lyxtemp` failed once and passed on rerun: review the concurrent-sweep path for a race, then fix the code or the test.
  `merge-review` (landed after) added retries when removing a run directory in `internal/lyxtemp`, which may already fix it; check first.
- `lyx cleanup` refuses outside the prime, but no test pins the refusal.
- `lyx batten cleanup` skips and reports a deleted run directory without a `status.json` instead of committing it, but no test pins that.
- The `testtiming` doc lacks the resign arrow.

### Idea

Fix the race if one remains, and add the missing tests and the doc arrow in the same change.

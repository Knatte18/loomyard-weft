# Drift-Verify: no guard halt on decision-record edits, shorter halt notes

Fixes issues #146 and #147, both Drift-Verify halts seen in task `dop-pumps-off-negative-spp`.

## The judge guard and the decision record (#146)

The judge guard compares both checkouts with the snapshot taken before the judge runs.
It already exempted a committed `lym loom decision add`, but two cases still halted a run for an edit the parent made:

- the guard could read the record between the verb's write and its commit, when it shows as an uncommitted change;
- a `lym loom decision replace` during the session was always named as a change.

Changes, all in `internal/driftengine` and wired in `internal/loomcli/driftwiring.go`:

- `TakeJudgeSnapshot` and `JudgeSnapshot.Changes` hold the run's decision lock, which every `lym loom decision` verb holds through its commit.
  The guard therefore never reads a verb's write before it lands.
  It reaches the lock through the new seam `Deps.LockDecisionRecord`, wired to loomcli's `holdDecisionLock`.
- The decision record is also exempt when it differs only by added entries from the record as the newest `decision replace` commit wrote it.
  That covers a replace during the session, and an add after one.
  The new seam `Deps.ReplacedDecisionRecord` reads that commit's copy.
  The wiring finds the newest `Lym-Record-Replaced` trailer that names the record, through loomcli's own trailer parser.
- `Changes` returns `GuardChange` values: the text as before, plus the changed file's path.
  The halt dates each change by its file's modification time against the judge session's window:
  inside the window, before the session opened, or after the guard halted the returned session, which names another process.
- The session mark records the guard's halt of a returned session (`guard_halt`: its time and the changes named).
  A resume re-checks the guard against the snapshot first.
  It then says "the resume re-checked the judge guard …", names what still differs and what was restored since the last check, and no longer repeats the first halt's text.
  A check that finds nothing clears the halt.

## The halt note over judge rulings (#147)

A Stuck over rulings of `does not match`, stored or fresh, under either diff mode, now leads with the count of findings and the way forward.
The way forward is `lym drift amend <slug> --entries <file>` then `lym loom resume`, or the route that fixes code.
Cells that share a case, key, kind and reason become one finding.
Each such finding names its row range, its row count and the first and last rows' old and new values.
The findings the reason carries are capped at 1500 characters, and the reason says how many it left out.
Every per-cell finding goes to `_lym/drift/logs/verify-findings.md`, which the reason names by its worktree-relative path.
The logs directory is already exempt from the judge guard.
The halt note `lym loom` writes therefore stays short, whatever the number of changed rows.

## Design decisions

- The decision record stays guarded, not excluded.
  Only committed edits shaped like `decision add` or `decision replace` are exempt, so a judge's edit of the goal or decisions still halts.
  Bound: a commit the judge session makes with a replace trailer naming the record goes unseen, as a committed added entry already did.
- The guard tells the actor from modification times, as the issue suggests.
  A time inside the window names "the judge session or a process working beside it", since a time alone cannot tell the two apart.
- A guard check that runs before the judge has returned records no halt, so a judge chair that may still be live never makes its own later edit read as another process's.
- Both issues share `verify.go`'s halt paths, so they land in one commit.

## Tests

- `TestJudgeSnapshot` gains rows for:
  a committed replace;
  an add after a replace;
  an edit after a replace, which is named;
  an uncommitted replace, which is named.
  Every row now also asserts that each checkout read happens under the decision lock and that the lock is released.
- `TestDriftVerify` gains two cases.
  The first covers the halt over many ruled cells:
  a repeated key over 150 rows becomes one finding, 100 cases are capped,
  the way forward comes first, and the findings file holds every finding.
  The second covers a resume that names what still differs, what was restored and when each file changed.
- The `replacedDecisionRecord` wiring in loomcli runs real git through the fabric and has no test of its own.
  The exemption it feeds is pinned in `TestJudgeSnapshot`.

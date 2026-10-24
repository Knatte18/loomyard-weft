# Card 7 — green-record

**Create:**
- `internal/gatecli#greenRecord`
- `internal/gatecli#greenEntry`
- `internal/gatecli#tierFingerprint`
- `internal/gatecli#readGreen`
- `internal/gatecli#recordGreen`
- `internal/gatecli#greenRecord.covered`
- `internal/gatecli#TestGreenRecord`

**Edit:**
- `internal/gatecli#runTest`
- `internal/gatecli#execution`
- `internal/gatecli#execution.runListed`
- `internal/gatecli#execution.report`
- `internal/gatecli#tierRun`
- `internal/gateslot#RunRecord`
- `internal/gateslot#TierSummaryLine`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli#scenarioConfig`
- `internal/gateslot#TestWriteRunRecord`
- `internal/gateslot#testFailedTier`
- `PATTERN.md`
- `internal/gatecli/doc.go`

**Intent:**
`lyx gate test` carries a green tier forward over an unchanged tree (D4), at every site and manual run alike.

The green record is one JSON file per worktree, `green.json` in the worktree's never-tracked `.lyx` gate directory, `gateslot.WaitDir(anchor)`, so its path is spelled through `internal/lyxdirs`; it is read tolerantly (`PATTERN-tolerant-records`), an unreadable or malformed file reading as empty.
A `greenEntry` holds the tier fingerprint, HEAD's tree and commit, the module directory, the args after `--`, and either `whole` or the green package set, plus its write time.
`tierFingerprint` hashes the tier's declaration without `parallel`, `timeout_min` and `slots`, so a changed command, env, targets or counter never carries.

Before the first tier, `runTest` checks the worktree with `verifytree.DirtyPaths`: a dirty worktree neither reads nor writes the record.
On a clean one it reads HEAD's tree and commit (read-only `git rev-parse` through `gitexec.Run`), and before acquiring slots each listed tier subtracts what `greenRecord.covered` holds green for the current tree: a whole entry covers every target, and a package entry covers those packages under the `go` scoper.
An entry with empty args covers a request with empty args and the guard step's own `-run` filter; any other args need an exact match, so `-- -count=1` reruns.
A tier with nothing left is carried: it takes no slot, runs no command, writes a run record with the new `Carried` field set, and prints its summary line in the existing shape, `TierSummaryLine` marking it with `carried` after the scope form, with exit 0, so `gateslot.FailedTier` and every other summary reader still parse it.
A tier with a remainder runs the remainder; under the `none` scoper only a whole entry carries.

`recordGreen` writes an entry only after the tier passed and only when HEAD's tree is unchanged and the worktree still clean at that point, so a tree that changes during a run records nothing.
A tier whose carried part and passed remainder together cover every package its `all_targets` expand to, listed once with `go list` in the tier's directory, writes a whole entry for the tree; otherwise it writes the packages it ran.
The guard step writes its entries under its own `-run` args, so such an entry never covers an empty-args request and an exact repeat of the guard group carries.
Writers serialize through a lock file beside the record (`internal/lock`), and each write prunes entries of other trees but keeps each fingerprint's 20 newest whole-green commits, so card 8 finds an older ancestor.
The carry sits under `verifytree`'s command-and-tree skip, which every site keeps.
The record only saves time, so every failure around it decides nothing: an unreadable record, a failed lock, a failed `DirtyPaths` call or a failed HEAD read is logged at Warn and the call runs every tier as if no record existed, and a failed write is logged and leaves the tier's own outcome and exit code unchanged.

`TestGreenRecord` pins the rows `TestGateTest_Scenario` does not reach: the fingerprint's ignored `parallel` field, which still carries, an empty-args entry covering the guard step's `-run` filter, an entry under `-- -count=1` carrying an exact repeat of those args, the `none` scoper carrying only a whole entry, the 20-commit pruning and the malformed-file read; the changed-command and `-- -count=1` carry breaks are left to the scenario.
`testFailedTier` gains a row feeding a carried `TierSummaryLine` beside a failing one, asserting `FailedTier` parses the carried line as exit 0 and still names the failing tier.
`TestGateTest_Scenario` gains rows: a second `--gate` run over the same clean tree carries every tier, writes carried records and returns while the row holds every slot through the scenario's `acquire` helper, so a call that took a slot would block; a dirty worktree carries nothing and writes nothing; an `all` run after a green `diff` run over the same tree runs only the packages the `diff` run did not cover, and a third `all` run carries whole, since the second wrote a whole entry; a changed tier command, a different tree and `-- -count=1` each break the carry; and a tier whose command, set through `scenarioConfig`, edits a tracked file and commits it before passing records nothing.

The gatecli doc states the record and its bound beside the slot variable's: a carried tier trusts a pass over the same tree, tier declaration and args on this worktree, so a flaky test that passed once is not rerun; a Go toolchain change, inherited environment variables outside the tier's `env` and ignored files such as `.dev-bin` are outside the fingerprint and the clean check; the record never leaves the machine, so an adopted run tests afresh; and a forged or hand-edited whole entry carries every later gate on that tree, Merge-In and Finalize included, which is accepted because a session that can write the worktree can change the code and tests under review, so the record guards only honest repeats and no further deny is added, since `cp` and `tee` would still reach it.

`PATTERN.md` gains this line under Shed and loom, after `PATTERN-gate-slot`:

```
- `PATTERN-gate-carry` — Skipping a test tier as already green: only `lyx gate test`'s green record, over a clean tree.
```

`PATTERN.md` already sits above its headroom line, so this card's `lint-pattern` step passes only when the overview is no longer than at HEAD.
In the same edit, these four entries take the shortened wording the parent's `changes` decisions of 2026-10-24 record, verbatim, which saves at least the new line's length; no rule changes:

```
- `PATTERN-template-overlay` — Judging config document against its template: only `configengine`'s effective template.
- `PATTERN-agent-filed-issues` — Filing GitHub issue: only `lyx selfreport create` in production. (test)
- `PATTERN-never-force-add` — Keeping transients unindexed: `.git/info/exclude`; never `git add -f`.
- `PATTERN-pane-binary-resolution` — Starting `lyx` in a reed pane or window: resolve it via `panebin.go` only. (test) — [background](pattern/PATTERN-pane-binary-resolution.md)
```

**ImpactSummary:** Adds the per-worktree green record and the carried tier to `lyx gate test`, one run-record field and the carried mark on the summary line; every gate site inherits the carry.

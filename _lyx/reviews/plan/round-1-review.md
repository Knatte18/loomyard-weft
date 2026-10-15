---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/01-cli-and-output.md:37
    summary: Card 1's --work contains --out refusal is unreachable, because every --work that holds --out equals or is an ancestor of --out's directory and the earlier overlap check already refuses it, so the row at line 62 expecting the contains reason fails.
    fix: In card 1's Intent (line 37), drop the separate `--out` lies inside it (`--work contains --out`) step from the ordered --work checks and state that the overlap check with `--out`'s directory is what refuses a --work holding --out; in the TestForkauditRefusals --work rows (line 62), change the row a marked, empty `--work` with `--out` inside a subdirectory of it to expect the reason `--work overlaps` naming `--out`'s directory instead of `contains --out`.
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/05-pack-check.md:45
    summary: Card 5's refusal row for --pack-check with a missing --code passes no --records, so card 4's runForkaudit appends a missing --records too, and card 5 leaves the order of its two checks open, so the row can be refused by the --records check before the --code check is reached.
    fix: In card 5's TestForkauditRefusals rows (line 45), have the `--code` row pass `--records` naming an existing empty `t.TempDir()` so only `--code` is missing; and in card 5's Intent (line 26), state the order, `--code` checked before `--records`.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/01-cli-and-output.md:36
    summary: With the default --out, the --out directory-missing refusal fires on a worktree with no .scratch/ (this worktree has none), because .scratch/ is only created later by --work's mkdir -p, so the default invocation D1 prescribes exits 1 naming --out.
    fix: In card 1's Intent (line 36), state that `--out`'s directory-missing refusal applies only to an explicit `--out`, and that when `--out` is defaulted its `.scratch/` directory is created with `mkdir -p` after every check has passed, beside `--work`'s own creation, and that the script's header comment says so.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/01-cli-and-output.md:29
    summary: Card 1 forbids any other line of any file under tools/forkaudit/ from spelling the run-record literals, but card 2's fork-file fixture reads a plan card under @WORKTREE@/_lyx/plan/ and cards 4 to 6's tests write trace files under .lyx/logs and commit state.json and plan cards at their _lyx paths, so the plan contradicts itself.
    fix: In card 1's Intent (line 29), narrow the rule to the script and its `.jq` filters, the decision record's constraint scope - No other line of `forkaudit.sh` or of a `.jq` file under `tools/forkaudit/` spells these literals; the Go tests build the paths from `internal/lyxdirs`'s `LyxDirName` and `DotLyxDirName`.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/04-trace-cross-check.md:27
    summary: D5's report header lists the discovery paths, and D2 counts --records and --code among them, but card 1's header names only the projects, hub and work paths and cards 4 and 5 add the records path only when it is missing and the code path never.
    fix: In card 4's Intent (line 27), add that the report header names the resolved `--records` path on every run; in card 5's Intent, add that under `--pack-check` the header names the resolved `--code` path.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/02-fork-records.md:42
    summary: The decision record's constraint that a task adding a pack layout adds its prompt marker row to tools/forkaudit in the same task is carried by no card; card 2 says only that a new layout adds one row, not that the layout's task must.
    fix: In card 2's Intent (line 42), add that the comment above the marker table, and the script's header comment, state the rule - a task that adds a Webster pack layout adds its marker row here in the same task.
---

### [BLOCKING] `--work contains --out` is shadowed by the overlap check

**Location:** `_lyx/plan/01-cli-and-output.md:37`, with the test row at line 62.

**Issue:**
The ordered `--work` checks run the overlap check (equals or is an ancestor of `--out`'s directory) before the `--out` lies inside it check.
`--out` lies inside `--work` exactly when `--out`'s directory is `--work` or a descendant of it, so every input that would reach `--work contains --out` has already been refused with `--work overlaps`.
The named input for that reason, a marked, empty `--work` with `--out` inside a subdirectory of it, is rejected upstream by that overlap check, so the row as written asserts a reason the script can never print.
D1 is still met with the step dropped: a `--work` holding `--out` exits 1 naming `--work`, deleting nothing.

### [BLOCKING] Card 5's `--code` refusal row can hit the `--records` refusal first

**Location:** `_lyx/plan/05-pack-check.md:45`, with the checks at line 26 and card 4's `runForkaudit` default at `_lyx/plan/04-trace-cross-check.md:45`.

**Issue:**
Card 4 makes `runForkaudit` append `--records` naming a path that does not exist whenever the arguments carry none.
Card 5's `--code` row gives `--pack-check` and a missing `--code` but no `--records`, so under `--pack-check` both directories are missing.
Card 5 says both checks run before the work directory is touched but not which runs first, so the `--records` check, which exits 1 naming `--records`, may reject the input before the `--code` refusal is reached.
The `--records` row passes `--records` explicitly and keeps the default `--code`, the repo root, so it is reachable.

### [MEDIUM] The default `--out` fails on a worktree without `.scratch/`

**Location:** `_lyx/plan/01-cli-and-output.md:36`.

**Issue:**
D1 makes `.scratch/fork-audit.md` under the repo root the default report path.
Card 1 refuses `--out` when its directory is missing, a bullet placed before the `--work` checks, and creates `.scratch/` only through `--work`'s `mkdir -p` afterwards.
`.scratch/` is gitignored and absent from a fresh worktree (absent in this one), so `forkaudit.sh --since <date>` with default paths exits 1 naming `--out`.
Moving the `--out` check after `--work`'s creation is no fix either: then an `--out` refusal would follow `--work`'s creation and clearing, against card 1's own rows asserting `--work` is unchanged.
No test catches it, since every test passes explicit `t.TempDir()` paths (line 58).

### [MEDIUM] The single-spelling rule covers files the plan itself makes spell the literals

**Location:** `_lyx/plan/01-cli-and-output.md:29`.

**Issue:**
Card 1 says no other line of any file under `tools/forkaudit/` spells `_lyx/webster/state.json`, `_lyx/plan/`, `.lyx/logs` or the records suffix.
Card 2's `fork-file.jsonl` fixture holds a Read of a plan card under `@WORKTREE@/_lyx/plan/` (`02-fork-records.md:55`), and card 5 asserts that read is skipped from the missed-file list, so the literal must be in the fixture.
Card 4's `TestForkauditTraceCrossCheck` writes trace files under each worktree's logs directory and `recordsRepo` commits the state file at its path, and card 6 commits plan cards under the plan directory; a Go test cannot read the bash variable block.
The decision record's constraint binds the script ("the literals sit once, in a variable block at the script's head"), and `PATTERN-lyxdirs-single-declarer` binds production files, so the over-broad rule is the plan's own, and its later cards break it.

### [LOW] The report header omits the `--records` and `--code` paths

**Location:** `_lyx/plan/04-trace-cross-check.md:27`.

**Issue:**
D5 puts the discovery paths in the header, and D2 lists `--records` and `--code` among the flags that override discovery paths.
Card 1's header names the projects, hub and work paths (line 45 of card 1); card 4 adds the records path only in the no-cross-check line, and card 5 adds nothing, so a default run's report does not say which records or code repo it read.

### [LOW] The marker-row obligation for future layouts is carried by no card

**Location:** `_lyx/plan/02-fork-records.md:42`.

**Issue:**
The decision record's Constraints say a task that adds a pack layout adds its prompt marker row to `tools/forkaudit` in the same task.
Webster is out of scope, so the only place this task can record the obligation is the tool itself; card 2 describes the mechanism ("A new layout adds one row") but no card writes the obligation where the next layout's task, `kickstart-per-card`, would read it.

## Rejected

- Fidelity walk: D1 (cards 1 and 2), D2 (cards 1 and 2), D3 (card 2's marker table, card 4's cross-check and `trace:<value>` rule), D4 (card 2), D5 (cards 1 to 3, card 4's samples additions), D6 (card 6), D7 (cards 4 and 5), D8 and the Constraints on read-only git, no `--show-toplevel`, no system temp, test speed, isolation and no Python (card 1 and every test card), and the "Added after Discussion" PATTERN line (card 1, net -2 bytes against the current 15557-byte `PATTERN.md`) are carried. Every acceptance criterion maps to a test row or assertion in cards 1 to 6, and the docs-unchanged criterion to the `## verify:` `git diff --quiet` line. Card 6's tie-break by lower session id and card 2's unreadable-candidate refusal serve D5's determinism and the failed-run acceptance criterion, so neither is unlicensed scope.
- Test economy: `TestForkauditTraceCrossCheck`, `TestForkauditPackCheck` and `TestForkauditPlanSection` each pin a behavior no row of `TestForkauditReport` or `TestForkauditRefusals` covers (the cross-check join, the pack-correctness section, the plan section), each with its own records or code repo per row, so no existing test can be named as covering them; not a defect. The budget rows (2, 3, 4, 5) match the new top-level tests per card, since `cmd/lyx/testbudget_test.go` excludes `TestMain`.
- Card order: cards 5 and 6 use `placeForks`/`forkFixture` (card 2) and `recordsRepo`/`recordsRun` (card 4) only from earlier cards. Card 6's edits to `forkFixture` (a new field), `placeForks` and `recordsRepo` (behavior only, the plan-card field already declared on `recordsRun` by card 4) change no parameter or result, so no re-sign arrow is due.
- Card 4's run scoping: `git log` with `--glob` patterns that match no ref lists nothing rather than falling back to `HEAD` (checked on git 2.53), and `spawnedAt` is written as `time.Now().UTC().Format(time.RFC3339)` in `internal/websterengine/beginbatch.go`, so the `-S<UTC second>` pickaxe can match it.
- Verify coverage: the `## verify:` section runs `go build`, the three `go vet` tiers, `go test ./...` and `go test -tags integration ./...` in the template's order, covering `tools/forkaudit` and `cmd/lyx`; `internal/gitkit` is only used, never targeted.
- Attack surface: card 1's Intent bounds the `--work` deletion (path comparisons, marker rule, literal names only); cards 4 to 6 name the read-only git verbs they run (`log`, `show`, `cat-file`, `ls-tree`) and write only into the work directory.

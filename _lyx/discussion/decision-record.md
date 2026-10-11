# Decision record — gate-test-hygiene

## Goal

Stop two shared caps, the `PATTERN.md` overview size and the per-package test budget, from blocking runs at Publish after main is merged in, or make the fix mechanical where blocking cannot be prevented.
Add a gofmt check to the test suite and bring the tree to gofmt-clean.
Fold the last test-side `/proc` probe copies onto `internal/testkit/tmuxkit`.

## Scope

In:

- PATTERN overview headroom: a headroom line below the hard cap, enforced on growth by a diff-scoped lint beside the comment lint, plus a one-time trim of the overview below that line.
- The overview's hard-cap finding names how many bytes to cut.
- The test-budget guard's failure names the row value to set.
- A Tier 1 gofmt guard over the module's Go files, and the files it flags fixed in the same change.
- Every test file outside `internal/testkit/tmuxkit` reads `/proc` only through the kit's probes: `loomcli`'s `smoke_helpers_test.go`, `orchcli`'s `smoke_live_test.go`, `reedcli`'s `smoke_proctree_test.go`, `pairteardown`'s `teardown_integration_test.go`, `shedverbs`'s `loop_integration_test.go` and `gatecli`'s `test_integration_test.go`.
- Docs for each of the above in the same commit.

Out:

- Any change to Publish's routing: a Publish verify failure still stops Stuck with the existing `lyx loom goto --to Webster-Burler` way forward and the Publish failure record.
- Any change to the test-budget file's format.
- A guard that bans `/proc` reads outside the kit.
- Production `/proc` readers (`internal/proc`, `internal/reedengine`, `cmd/testtiming`).

## Decisions

### D1 — The overview gets a headroom line at 95% of the hard cap

Decision: `internal/pattern` declares a headroom line beside `MaxOverviewBytes`, derived as 95% of it (integer arithmetic, 15564 bytes today), never a second literal.
The hard cap stays 16 KiB and stays enforced by `TestOverview_RealFilePassesCheck` through `pattern.Check` on whatever tree the test runs in, merged trees included.

Rationale: a lower cap enforced by the same `go test` runs on the merged tree too, so it moves the collision instead of preventing it.
Headroom works only when the stricter line binds a branch's own change and the merged tree answers to the looser hard cap.
With every branch's growth ending at or under the line, which D3's round gate checks from the branch's merge base with its parent, two branches that both grow the overview breach the hard cap only when their base sat more than a full headroom below the line and each grew it by more than the headroom, which no ordinary entry addition does.

### D2 — A diff-scoped headroom lint binds growth only

Decision: a pure function in `internal/pattern` takes the overview's content at a base and at the head and returns a finding when the head is larger than the base and larger than the headroom line.
The finding names the head size, the base size, the line, and the bytes to cut to reach the larger of the base size and the line, the smallest cut that passes.
With the base above the line, that is the branch's own growth, so a fixer is never told to cut wording the base brought in.
An absent overview at the head passes; an absent overview at the base counts as zero bytes.
A change that leaves the overview's size unchanged, or shrinks it, passes at any size, so a branch that does not grow the overview is never blocked, even while main sits above the line after a merge.
`internal/pattern` stays a leaf: the function reads no git and no disk, and its callers hand it both contents.

Bound: it can block only a change that grows `PATTERN.md`, and only while the grown file is above the line.
It lets through any growth that ends at or under the line, any size change on main made outside a gate, a merged tree above the line at Publish, which runs no lint, and the bypasses D3 names.
The hard cap in `TestOverview_RealFilePassesCheck`, which every full verify runs, Publish's included, bounds everything the lint lets through.

Rationale: binding growth, not size, is what makes the headroom stick without blocking unrelated work, and it is the mechanical form of the target that failed when it lived only in prose.

### D3 — The headroom lint runs wherever the comment lint runs, with a parity verb

Decision: a new verb `lyx loom lint-pattern`, with the same base semantics as `lyx loom lint-comments`: without `--base` it compares the working tree's overview with HEAD's; with `--base <commit>` it compares that commit's with HEAD's.
It reports one JSON envelope, findings under the failure envelope's `findings` key, exits 0 when clean and 1 on a finding, takes no arguments, and has audience operator like `lint-comments`.
The webster card gate appends `lyx loom lint-pattern` after `lyx loom lint-comments`.
The Webster-Burler round gate (`loomshed.NewVerifyGate`) runs the same pattern function in-process from the task branch's merge base with its recorded parent branch to HEAD, before any test.
That base is not the comment lint's: in a loom run the comment lint diffs from the plan verify's last recorded pass, which sees only the round's own changes and, after a Publish merge-in, counts main's merged-in growth as the branch's.
The gate is told a merge-base reader for the headroom lint in every pair run, built as darn's wiring builds its merge-base reader; telling it does not switch a loom run's gate to the whole-diff form, and in a darn run the one reader serves both lints.
A merge-base read error is a returned gate error, as it is for darn.
In a standalone run, which records no parent branch, the gate is told no reader and skips the headroom lint.
The gate's headroom finding names the way forward: cut the named bytes from `PATTERN.md`, moving detail into the entries' background files, then re-step the row.
The verb's finding names the same way forward.
The verb and both gate sites call the same `internal/pattern` function, per `PATTERN-gate-self-check-parity`, and the pair is added to that entry's background file in the same commit.
The verb joins the module-config-free verb set in `loomcli` beside `lint-comments`, since it reads only git and files.

Bound: the verb reads git and files and writes nothing; a finding fails the gate whose command runs it, never the run.
The card gate is run by the fork, not by Go, so a fork that skips it lets its card's growth through until the next Webster-Burler round gate, which checks the whole branch's growth.
A standalone run's growth is checked by no lint.
Both are bounded by the hard cap, as D2's Bound states.

Rationale: the card gate reuses the comment lint's working-tree-to-HEAD scope for a card's uncommitted change.
The round gate needs the branch's change since its merge base with the parent, which only darn's gate reads today, so every pair run's gate is told that base for the headroom lint.
Diffing from the merge base after a Publish merge-in holds a branch to its own growth on top of main's content, so a branch that never grew the overview passes.
A separate verb keeps each lint's name true to what it checks.

### D4 — This task trims the overview to at most 15000 bytes

Decision: the overview is shortened in this task to at most 15000 bytes.
Each trimmed entry keeps its trigger, its requirement and its `(test)` mark; detail moves into the entry's background file, created when the entry has none.
An entry whose background file is created gains its `— [background](pattern/<name>.md)` link, and the link's bytes count toward the 15000.
No entry is dropped or merged away, and no rule changes.

Rationale: the overview is at 16217 bytes, above the new line, so without a trim the next task that adds an entry would have to cut over 650 bytes of unrelated wording.
Trimming below the line, not to it, leaves room for about two entries before any task must trim.

### D5 — The hard-cap finding names the bytes to cut

Decision: the oversize-overview finding of `pattern.Check` states the overview's size, the cap and the number of bytes over it.

Rationale: after a merge-only breach, Publish's failure record names how far the merged overview is over the cap.
In the Webster-Burler round the existing Publish way forward reaches, the merge base is the merged-in parent, whose overview passed the hard cap, and the branch's growth on top of it ends above the cap.
The round gate's headroom finding then names the cut back to the larger of the parent's size and the line, which is at least the cut the cap needs, and the fixer cuts that much.
The cap's byte count serves every verify that runs no headroom lint, such as Publish's and webster's plan-level verify gate.

### D6 — The test-budget failure names the row value to set

Decision: the over-budget message of `TestTestBudget_PackagesStayWithinBudget` names the package, its count and its budget, and tells the reader to set its row in `cmd/lyx/testdata/test-budget.yaml` to the count.
The no-row message tells the reader to add a row with the count.
The guard's logic, the file format and the rule that a raise lands in the same commit as its tests are unchanged.

Rationale: two branches that each raise a row by one make identical edits, so git merges them cleanly and the merged package overflows by the colliding raises.
Each raise was already a reviewed decision, so the post-merge fix is bookkeeping: a message naming the value makes it a one-line edit for the Webster-Burler fixer and spares the PR-Gate reviewer a hand count.

### D7 — A Tier 1 gofmt guard over the module's Go files

Decision: a new untagged test in `cmd/lyx`, beside the other repository guards, walks the module's `.go` files through `scankit` and compares each with `go/format.Source` of itself, the library `gofmt` formats with.
It skips `testdata` directories, which the Go tool ignores and which may hold deliberately unformatted fixtures, and so also skips the nested module under `internal/planglyph/testdata`.
It parses every file whatever its build tags.
A file that fails to parse is a failure naming the file.
Each failure names the file and the first line where the formatted text differs, showing the current and the formatted line, so the fix is an Edit; the message never tells the reader to run `gofmt -w` or any other rewriting tool, since agents change files only with Edit or Write.
It carries `//lyx:guard`, since it reads files outside its package, and requires a scan floor so a broken walk cannot pass vacuously, as the test-budget guard does.
It spawns nothing, so it is Tier 1 under `PATTERN-test-speed`.
The files it flags at implementation time are fixed in the same change.
The `cmd/lyx` row of `cmd/lyx/testdata/test-budget.yaml` rises by one in the same commit.
No PATTERN entry is added for it.

Rationale: a guard in the test suite runs in every gate that runs guards and in the plan's full verify without any per-plan verify edit, which a `gofmt -l` verify step would need.
`go/format` keeps the guard in-process, so it stays in Tier 1 and needs no `gofmt` binary on `PATH`.
gofmt-clean code is the Go toolchain's own convention, not a loomyard structural invariant, and an entry would spend overview bytes D4 just freed.

### D8 — Test-side `/proc` reads go through `tmuxkit` probes

Decision: `internal/testkit/tmuxkit` gains three read-only probes, the first two reading `/proc/<pid>/stat` and sharing their comm-safe stat parsing with the kit's own session probe: a parent-pid probe, a process-state probe that returns the stat state letter, and a comm probe that reads `/proc/<pid>/comm`.
Each follows the existing probes' shape: nothing on a missing or unreadable process, nothing where `/proc` does not exist.
`loomcli`'s `findWatchdogPIDs`, `findDriverPIDs` and `parentPID`, `orchcli`'s `killTreeProcs`, `reedcli`'s `linuxProcPPID`, `linuxProcComm` and the closure walk that uses them, and the zombie-counts-as-ended checks `pairteardown`'s and `gatecli`'s `processGone` and `shedverbs`'s `processEnded` read `/proc` only through the kit's probes (`Pids`, `ProcArgv`, `ProcCwd`, `ProcExe`, `IsWatchdog` and the three new ones).
Each package keeps its own matcher logic (`loom run` plus cwd with the parent-drop, `reed watchdog` plus `--hub-path`, the dir-reference kill, the descendant closure, an ended check's fallback where the state probe reports nothing) in its own test files, since each is used by one package; a copy whose logic is exactly a kit export is replaced by it.
`loomcli`'s `findWatchdogPIDs` uses `tmuxkit.IsWatchdog` for the adjacent `reed watchdog` match.
In `cmd/lyx/tierpurity_test.go`'s `allowedTmuxkitOutsideTmuxTier`, the row for `internal/loomcli/smoke_helpers_test.go` is reworded to name every kit probe that file now uses, and `internal/shedverbs/loop_integration_test.go` and `internal/gatecli/test_integration_test.go`, both `integration` tier, each gain a row naming the state probe and that the file starts no server.
No row is needed for `orchcli` (`llm` tier), `reedcli` (`tmux || llm`) or `pairteardown` (`tmux`).

Rationale: `PATTERN-testkit` puts support shared by two or more packages in one kit; the parent-pid parser exists in both `loomcli` and `reedcli`, the zombie check from the stat state in `pairteardown`, `shedverbs` and `gatecli`, and the raw `/proc` reads in six packages beside the kit's own.
The comm probe moves too, so after this task no test file outside the kit reads `/proc` itself, which is a property review can check with one grep.

## Constraints

- `PATTERN-leaf-packages`: `internal/pattern` keeps its closed import set; D2's function is pure and its callers read git.
- `PATTERN-gate-self-check-parity`: D3 adds the verb and both gate sites in this task, calling one package function, and the background file lists the pair.
- `PATTERN-cli-cobra` and `PATTERN-cliwire-sole-wiring`: `lint-pattern` is a cobra verb under `lyx loom` with `Short`, audience and JSON errors, wired like `lint-comments`.
- `PATTERN-gitrepo-client-boundary`, `PATTERN-gitexec-checked-call`: the verb's base read goes through the same git read path `commentlint` uses; no raw git.
- `PATTERN-test-speed`: the gofmt guard is untagged and spawns nothing, carries `//lyx:guard`, and calls `t.Parallel`; the gate-per-tier table in its background file names the headroom lint in the card-gate and round-gate rows.
- `PATTERN-testkit`: new probes live in `tmuxkit`; single-package matchers stay local; the kit keeps its `os/exec` exemption bounds, since the new probes read files only.
- `PATTERN-test-economy`: the headroom function's cases are rows of one table-driven test; the verb's tests extend whatever test covers `lint-comments`'s envelope where one exists.
- `PATTERN-test-isolation`: unchanged; every touched package already runs through `tmuxkit.Main` where it must.
- `PATTERN-refusal-way-forward`: the "Webster-Burler verify gate exhausted" row of `contracts/specs/refusal-spec.md` gains the headroom finding as a cause, with D3's way forward, in the same commit, and a test reaches the round gate's headroom finding and asserts its way forward.
- `PATTERN-comment-line-breaks`: new and changed comments use semantic line breaks.
- `PATTERN-sole-parsers`, `PATTERN-stencil-ownership`: the webster implementer stencil that names `lint-comments` names the new step through its stencil file, never embedded text.
- CLAUDE.md test budget: every package whose top-level test count rises gets its row raised, and the card lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.
- CLAUDE.md docs: `internal/pattern/doc.go` (the format section's caps and the headroom rule), `internal/websterengine/doc.go` (card gate composition), `internal/loomshed` docs (round gate), `internal/testkit/tmuxkit` package doc (the probe list), the `pattern/` background files above, and `docs/overview.md` wherever it names the card gate's steps, all in the same commit as the code.

## Auto-mode assumptions

- The headroom line is 95% of the cap, as the board entry suggests; nothing in the repo argues for another ratio.
- The trim target is 15000 bytes, as D4 sets it.
- The overview hard cap stays 16 KiB; nobody asked to raise it.
- Publish routing stays manual; a recipe change is outside a hygiene task.
- The gofmt guard lives in `cmd/lyx` with the other repository guards rather than in a new package.
- `reedcli`'s comm and ppid readers and the `pairteardown`, `shedverbs` and `gatecli` zombie checks are in scope though the board names only `loomcli` and `orchcli`, because they are the same copy class the board item targets.

## Open risks

- `small-fixes-batch` runs in parallel and may raise the same `cmd/lyx` budget row; a collision at Publish is then fixed by D6's message through the existing Webster-Burler route.
- Another branch in flight that grows `PATTERN.md` and lands first may push the merged overview above 15000 bytes; D4's bound is checked on this branch's tree, and the hard cap still guards the merged one.
- A change in Go's formatter between toolchains could flag files formatted by an older `gofmt`; the guard then fails with the exact line, and the fix is an Edit.
- The trim in D4 rewrites authoritative text; a reviewer must check each shortened entry against its prior wording for a dropped requirement.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - headroom-byte-count-overstates-cut (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - card6-old-reader-meaning-stated-in-untargeted-files (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- The headroom function, as table rows: grew from under the line to above it fails and names the bytes to cut to the line; grew from a base above the line fails and names the bytes to cut back to the base size; grew to exactly the line passes; shrank while above the line passes; unchanged above the line passes; absent at the base and present above the line fails; absent at the head passes.
- `lyx loom lint-pattern` without `--base` reports a finding for an uncommitted growth past the line and passes a clean tree; with `--base` it compares that commit with HEAD; its JSON envelopes match `lint-comments`'s shape.
- The webster card gate's steps end with `lyx loom lint-comments` then `lyx loom lint-pattern`.
- The Webster-Burler round gate fails on a branch whose overview grew past the line since its merge base with the parent, naming the bytes to cut and the way forward, and passes one that did not grow it.
- After a merge-in of a parent whose overview sits above the line, the round gate passes a branch that never grew the overview, and fails one that grew it on top of the parent's content, naming the bytes of its own growth.
- A loom run's round gate keeps its round-form comment lint base and impacted-set command when told the merge-base reader; a standalone run's gate skips the headroom lint.
- `pattern.Check`'s oversize finding names the bytes over the cap.
- The test-budget guard's over-budget and no-row messages name the value to set, pinned by `TestBudgetViolations`'s rows or an equivalent assertion.
- The gofmt guard passes on the tree after this task, fails naming file, line, current and formatted text for an unformatted source, fails naming the file for a source that does not parse, skips `testdata`, and fails on a scan below its floor.
- `gofmt -l` over the module prints nothing after this task.
- `PATTERN.md` is at most 15000 bytes, every prior entry name is still present, and `TestOverview_RealFilePassesCheck` passes.
- No `_test.go` file outside `internal/testkit/tmuxkit` names the `"/proc"` path literal.
- `loomcli`'s smoke tests that call `findWatchdogPIDs` and `findDriverPIDs`, `orchcli`'s live smoke cleanup, `reedcli`'s process-tree helpers and the `pairteardown`, `shedverbs` and `gatecli` ended checks behave as before; the tier-purity guard passes with the reworded `loomcli` row and the new `shedverbs` and `gatecli` rows.
- The test budget, tier-purity, testkit-enforcement and markdown-link guards pass.

## Notes for the plan writer

- The comment lint's two base modes live in `internal/loomcli/lintcomments.go` and `internal/commentlint`; the card gate steps in `internal/websterengine/cardgate.go`; the round gate in `internal/loomshed/verifygate.go`; darn's merge-base reader in `internal/loomcli/darnwiring.go`.
- `contracts/stencils/webster/webster-body-implementer.md`, `contracts/stencils/loom/loom-rubric-webster-review.md`, `contracts/specs/refusal-spec.md` and `docs/overview.md` mention `lint-comments`; each needs a look for whether it should name the new lint.
- The CLI command index the orch loads (`ly:cli`) may be generated from the cobra tree or pinned by a test; check how `lint-comments` appears there.
- `tmuxkit`'s session probe already parses `/proc/<pid>/stat` from the last `)`; the parent-pid probe shares that parse.

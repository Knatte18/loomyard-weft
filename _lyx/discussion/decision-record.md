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
- Every test file outside `internal/testkit/tmuxkit` reads `/proc` only through the kit's probes: `loomcli`'s `smoke_helpers_test.go`, `orchcli`'s `smoke_live_test.go` and `reedcli`'s `smoke_proctree_test.go`.
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
With every branch's growth ending at or under the line, two branches that both grow the overview breach the hard cap only when their base sat more than a full headroom below the line and each grew it by more than the headroom, which no ordinary entry addition does.

### D2 — A diff-scoped headroom lint binds growth only

Decision: a pure function in `internal/pattern` takes the overview's content at a base and at the head and returns a finding when the head is larger than the base and larger than the headroom line.
The finding names the head size, the base size, the line, and the bytes to cut to reach the line.
An absent overview at the head passes; an absent overview at the base counts as zero bytes.
A change that leaves the overview's size unchanged, or shrinks it, passes at any size, so a branch that does not grow the overview is never blocked, even while main sits above the line after a merge.
`internal/pattern` stays a leaf: the function reads no git and no disk, and its callers hand it both contents.

Bound: it can block only a change that grows `PATTERN.md`, and only while the grown file is above the line; it lets through any growth that ends at or under the line, and any size change on main made outside a gate.

Rationale: binding growth, not size, is what makes the headroom stick without blocking unrelated work, and it is the mechanical form of the target that failed when it lived only in prose.

### D3 — The headroom lint runs wherever the comment lint runs, with a parity verb

Decision: a new verb `lyx loom lint-pattern`, with the same base semantics as `lyx loom lint-comments`: without `--base` it compares the working tree's overview with HEAD's; with `--base <commit>` it compares that commit's with HEAD's.
It reports one JSON envelope, findings under the failure envelope's `findings` key, exits 0 when clean and 1 on a finding, takes no arguments, and has audience operator like `lint-comments`.
The webster card gate appends `lyx loom lint-pattern` after `lyx loom lint-comments`.
The Webster-Burler round gate (`loomshed.NewVerifyGate`) runs the same pattern function in-process from the merge base it already reads for the comment lint to HEAD, and, like the comment lint, passes it when it has no usable base.
The verb and both gate sites call the same `internal/pattern` function, per `PATTERN-gate-self-check-parity`, and the pair is added to that entry's background file in the same commit.
The verb joins the module-config-free verb set in `loomcli` beside `lint-comments`, since it reads only git and files.

Bound: the verb reads git and files and writes nothing; a finding fails the gate whose command runs it, never the run.

Rationale: the comment lint already defines the two diff scopes the gates need (a card's uncommitted change, a branch's change since its merge base), so the headroom lint reuses them instead of inventing a base.
A separate verb keeps each lint's name true to what it checks.

### D4 — This task trims the overview to at most 15000 bytes

Decision: the overview is shortened in this task to at most 15000 bytes.
Each trimmed entry keeps its trigger, its requirement and its `(test)` mark; detail moves into the entry's background file, created when the entry has none.
No entry is dropped or merged away, and no rule changes.

Rationale: the overview is at 16217 bytes, above the new line, so without a trim the next task that adds an entry would have to cut over 650 bytes of unrelated wording.
Trimming below the line, not to it, leaves room for a few entries before any task must trim.

### D5 — The hard-cap finding names the bytes to cut

Decision: the oversize-overview finding of `pattern.Check` states the overview's size, the cap and the number of bytes over it.

Rationale: after a merge-only breach the Webster-Burler fixer, reached through the existing Publish way forward, then has the exact amount to cut without counting.

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
The files it flags when this task is implemented are fixed in the same change; today `gofmt -l .` lists `internal/boardcli/cli.go`, `internal/fabricengine/pull.go`, `internal/pattern/doc.go`, `internal/reedengine/lifecycle_test.go`, `internal/shedverbs/loopwait_test.go` and `internal/shell/posix.go`, and the list is recomputed at implementation time.
The `cmd/lyx` row of `cmd/lyx/testdata/test-budget.yaml` rises by one in the same commit.
No PATTERN entry is added for it.

Rationale: a guard in the test suite runs in every gate that runs guards and in the plan's full verify without any per-plan verify edit, which a `gofmt -l` verify step would need.
`go/format` keeps the guard in-process, so it stays in Tier 1 and needs no `gofmt` binary on `PATH`.
gofmt-clean code is the Go toolchain's own convention, not a loomyard structural invariant, and an entry would spend overview bytes D4 just freed.

### D8 — Test-side `/proc` reads go through `tmuxkit` probes

Decision: `internal/testkit/tmuxkit` gains two read-only probes: a parent-pid probe that reads `/proc/<pid>/stat` and shares its comm-safe stat parsing with the kit's own session probe, and a comm probe that reads `/proc/<pid>/comm`.
Each follows the existing probes' shape: nothing on a missing or unreadable process, nothing where `/proc` does not exist.
`loomcli`'s `findWatchdogPIDs`, `findDriverPIDs` and `parentPID`, `orchcli`'s `killTreeProcs`, and `reedcli`'s `linuxProcPPID`, `linuxProcComm` and the closure walk that uses them read `/proc` only through the kit's probes (`Pids`, `ProcArgv`, `ProcCwd`, `ProcExe`, `IsWatchdog` and the two new ones).
Each package keeps its own matcher logic (`loom run` plus cwd with the parent-drop, `reed watchdog` plus `--hub-path`, the dir-reference kill, the descendant closure) in its own test files, since each is used by one package; a copy whose logic is exactly a kit export is replaced by it.
`loomcli`'s `findWatchdogPIDs` uses `tmuxkit.IsWatchdog` for the adjacent `reed watchdog` match.
The allowlist row for `internal/loomcli/smoke_helpers_test.go` in `cmd/lyx/tierpurity_test.go` is reworded to name every kit probe that file now uses; no new allowlist row is needed for `orchcli` (`llm` tier) or `reedcli` (`tmux || llm`).

Rationale: `PATTERN-testkit` puts support shared by two or more packages in one kit; the parent-pid parser exists in both `loomcli` and `reedcli`, and the raw `/proc` reads exist in three packages beside the kit's own.
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
- `PATTERN-comment-line-breaks`: new and changed comments use semantic line breaks.
- `PATTERN-sole-parsers`, `PATTERN-stencil-ownership`: the webster implementer stencil that names `lint-comments` names the new step through its stencil file, never embedded text.
- CLAUDE.md test budget: every package whose top-level test count rises gets its row raised, and the card lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.
- CLAUDE.md docs: `internal/pattern/doc.go` (the format section's caps and the headroom rule), `internal/websterengine/doc.go` (card gate composition), `internal/loomshed` docs (round gate), `internal/testkit/tmuxkit` package doc (the probe list), the `pattern/` background files above, and `docs/overview.md` wherever it names the card gate's steps, all in the same commit as the code.

## Auto-mode assumptions

- The headroom line is 95% of the cap, as the board entry suggests; nothing in the repo argues for another ratio.
- The trim target is 15000 bytes, chosen to leave room for about two entries below the line.
- The overview hard cap stays 16 KiB; nobody asked to raise it.
- Publish routing stays manual; a recipe change is outside a hygiene task.
- The gofmt guard lives in `cmd/lyx` with the other repository guards rather than in a new package.
- `reedcli`'s comm and ppid readers are in scope though the board names only `loomcli` and `orchcli`, because they are the same copy class the board item targets.

## Open risks

- `small-fixes-batch` runs in parallel and may raise the same `cmd/lyx` budget row; a collision at Publish is then fixed by D6's message through the existing Webster-Burler route.
- Another branch in flight that grows `PATTERN.md` and lands first may push the merged overview above 15000 bytes; D4's bound is checked on this branch's tree, and the hard cap still guards the merged one.
- A change in Go's formatter between toolchains could flag files formatted by an older `gofmt`; the guard then fails with the exact line, and the fix is an Edit.
- The trim in D4 rewrites authoritative text; a reviewer must check each shortened entry against its prior wording for a dropped requirement.

## Acceptance criteria

- The headroom function, as table rows: grew to above the line fails and names the bytes to cut; grew to exactly the line passes; shrank while above the line passes; unchanged above the line passes; absent at the base and present above the line fails; absent at the head passes.
- `lyx loom lint-pattern` without `--base` reports a finding for an uncommitted growth past the line and passes a clean tree; with `--base` it compares that commit with HEAD; its JSON envelopes match `lint-comments`'s shape.
- The webster card gate's steps end with `lyx loom lint-comments` then `lyx loom lint-pattern`.
- The Webster-Burler round gate fails on a branch whose overview grew past the line since the merge base and passes one that did not grow it; with no usable base the headroom lint passes.
- `pattern.Check`'s oversize finding names the bytes over the cap.
- The test-budget guard's over-budget and no-row messages name the value to set, pinned by `TestBudgetViolations`'s rows or an equivalent assertion.
- The gofmt guard passes on the tree after this task, fails naming file, line, current and formatted text for an unformatted source, skips `testdata`, and fails on a scan below its floor.
- `gofmt -l` over the module prints nothing after this task.
- `PATTERN.md` is at most 15000 bytes, every prior entry name is still present, and `TestOverview_RealFilePassesCheck` passes.
- No `_test.go` file outside `internal/testkit/tmuxkit` names the `"/proc"` path literal.
- `loomcli`'s smoke tests that call `findWatchdogPIDs` and `findDriverPIDs`, `orchcli`'s live smoke cleanup and `reedcli`'s process-tree helpers behave as before; the tier-purity guard passes with the reworded `loomcli` allowlist row.
- The test budget, tier-purity, testkit-enforcement and markdown-link guards pass.

## Notes for the plan writer

- The comment lint's two base modes live in `internal/loomcli/lintcomments.go` and `internal/commentlint`; the card gate steps in `internal/websterengine/cardgate.go`; the round gate in `internal/loomshed/verifygate.go`.
- `contracts/stencils/webster/webster-body-implementer.md`, `contracts/stencils/loom/loom-rubric-webster-review.md`, `contracts/specs/refusal-spec.md` and `docs/overview.md` mention `lint-comments`; each needs a look for whether it should name the new lint.
- The CLI command index the orch loads (`ly:cli`) may be generated from the cobra tree or pinned by a test; check how `lint-comments` appears there.
- `tmuxkit`'s session probe already parses `/proc/<pid>/stat` from the last `)`; the parent-pid probe shares that parse.

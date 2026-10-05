# Decision record — test-suite-measure

## Goal

Measure the Go test suite per test, make the integration tier faster through test seams and parallelism, stop test runs leaking tmux socket files, split the tag tiers so no tier below a dedicated LLM tier spawns an LLM, and consolidate PATTERN's Testing section into fewer entries.
`test-suite-prune-core` and `test-suite-prune-rest` consume this task's tool, report and rules to delete tests; this task deletes none.

## Scope

In:

- A per-test coverage and redundancy mode in the existing `cmd/testtiming` tool, plus per-package test count, wall time and serial time.
- The committed before-state in `docs/benchmarks/test-suite-timing.md`, with a slowest-tests table, and the committed per-package redundancy report.
- Speed fixes: test seams for the reed watchdog discovery interval, `reapExitTimeout` and verifyrun's `waitDelay`; `true` instead of `sleep 5` in `verifytree`'s integration test; `t.Parallel` on fabricengine's serial integration files where tests are independent; the cheap prefilter in `bareVocabularyTokenIn`; the hermetic git env in `internal/gitrepo/internal/gitoracle`.
- The tmux socket leak, in tests and in production reed teardown.
- Four test tiers with named build tags, the retag of every tagged test file to its tier, and a guard that keeps LLM spawns in the LLM tier.
- PATTERN's Testing section consolidated, with its background files, enforcement tests and every reference to a retired entry ID.
- Docs: `docs/benchmarks/running-tests.md`, `docs/benchmarks/test-suite-timing.md`, the verify-model section of `contracts/specs/loom-plan-spec.md`, and the verify text in `contracts/stencils/loom/loom-template-plan.md` and `loom-rubric-plan-review.md`.

Out:

- Deleting, folding or re-tiering tests for redundancy: `test-suite-prune-core` and `test-suite-prune-rest`.
- A per-package test budget guard: `test-suite-prune-rest`.
- Which gate runs which tier, including Webster's Go-derived card gate (`cardGateCommand`) and verify-gate selection: note `test-tiers`.
- Rewriting an LLM-spawning test onto a fake provider; such tests are retagged, not rewritten.
- Editing `scribe:testing` or `scribe:golang-testing` (separate repo).

## Decisions

### D1 — Measurement lives in `cmd/testtiming`

Decision: `cmd/testtiming` gains a redundancy mode (flag named at plan time) instead of a new `cmd/testprune`.
The mode reports, per package: top-level test count, wall time, serial time (sum of top-level test elapsed), and the redundant candidates.
Rationale: the tool already runs `go test -json` and reports per-package timing and the slowest tests, and the docs already point at it; a second command would duplicate its event parsing.

### D2 — How redundancy is computed

Decision:

- Unit: a top-level test function; subtests are not split out.
- Per package, the tool compiles one coverage-instrumented test binary (`go test -c -cover` with `-coverpkg` over the module's packages) and runs it once per top-level test (`-test.run '^Name$'`), collecting each test's covered block set.
- A test is a candidate when its covered block set is a subset of the union of the other tests' sets in the same package.
- For each candidate the report names the covering tests, ordered by overlap, so a prune finding names the covering test as `PATTERN-test-economy` requires.
- Because two tests can each be a candidate only through the other, the report also marks a removable set: a greedy pass that drops candidates one at a time, slowest first, and re-checks the rest against what remains.
- A test that covers no block at all (a pure static scan, a skipped test) is listed apart as "no coverage", never as redundant, since coverage cannot judge it.
- A test whose run spawns the built `lyx` binary or another subprocess is listed there too, never as redundant: the in-process coverage binary does not see the child's blocks, and the tool does not collect child coverage.
  The tool recognises such a test by a static scan of the package's test sources: the test, or a helper it calls in the same package, references `os/exec`, `lyxbin`, `tmuxkit` or another testkit spawn helper.
  The scan over-approximates, and a test it cannot classify counts as spawning, so an error lands in "no coverage", never in "redundant".
  Unclassifiable means a call the scan cannot resolve to source, such as a function value or an interface method; calls into production packages are not followed, since their blocks are covered in-process, and testkit and `hubforge` helpers are classified by the same scan.
- Tags default to every tier except `llm`; a test that skips at run time is reported as skipped.

Rationale: coverage blocks are what `go test -cover` gives without new dependencies; module-wide `-coverpkg` lets a CLI test's coverage of an engine count; the removable set keeps the prune tasks from deleting both halves of a mutually redundant pair.

### D3 — Where the reports are committed

Decision: the before-state goes into `docs/benchmarks/test-suite-timing.md` as a new dated section (machine, revision, load, test count, Tier 1 and integration wall time, slowest-tests table), and the redundancy report into `docs/benchmarks/test-redundancy.md`, one H2 per package, generated by the tool and committed as written.
Both are measurement reports, exempt from the perishable-specific rule; links in both obey `PATTERN-markdown-link-integrity`.
Rationale: `docs/benchmarks/` already holds the suite's measurements, and the prune tasks need one stable path to read.

### D4 — Speed fixes measured on the pre-retag test set

Decision: the plan orders the work so the before-state and the after-speed measurement both run `go test -tags integration ./...` on the same test set, before any tier retag (D6) moves a file in or out of that tag.
The PR states both wall times, then states the post-retag numbers separately.
Rationale: retagging moves tests between tiers and would change the integration tier's wall time without any speed fix, which would falsify the "dropped from the speed fixes alone" criterion.

### D5 — Speed-fix shapes

Decision:

- Each named production wait (watchdog discovery interval, `reapExitTimeout`, verifyrun `waitDelay`) becomes an injected field or option the test shortens, per instance, so the test can still call `t.Parallel`; never a package-level `var`, which a test would set as process-global state; production values are unchanged.
- fabricengine integration tests gain `t.Parallel` unless they touch process-global state; a test that stays serial gets the comment `PATTERN-test-parallel` (folded into `PATTERN-test-speed`, D8) requires, naming that state.
- `bareVocabularyTokenIn` returns no token before running its regex when the lowercased text contains neither `weft` nor `warp`; its findings are unchanged.
- `gitoracle`'s tests run under `gitkit.HermeticGitEnv()` through a `TestMain`, or the package joins the hermetic allowlist with its reason, whichever its shape allows.
- Any further test the slowest-tests table shows blocking on a production duration gets the same seam treatment.

Bound: a seam changes only the duration a test waits; production code reads the same values it reads today.
Rationale: each fix keeps the behavior under test and removes only real-time waiting.

### D6 — Four tiers and their tags

Decision:

| Tier | Build tag | Needs | Costs |
|---|---|---|---|
| 1 unit | none | nothing outside the process | seconds |
| 2 integration | `integration` | real git, processes, filesystem links, the built `lyx` binary; no tmux server | time |
| 3 tmux | `tmux` | a live tmux server, directly or through reed, often with the built binary | time |
| 4 llm | `llm` | a real LLM session | tokens |

- The `smoke` tag is retired: every `smoke` file moves to `tmux` or `llm`.
- An `integration` file that starts a tmux server moves to `tmux`.
- A file that spawns, or drives `lyx` into spawning, a real LLM session moves to `llm`, whichever tag it carried.
- Tags do not nest: `-tags integration,tmux` runs both, and `llm` runs only when named.
- A test that needs two substrates takes the higher tier.
- Each tag compiles on its own: a helper shared by files of different tiers lives in an untagged `_test.go` file or a testkit kit, never in a file of one tier that another tier's file uses.

Bound: the retag takes tmux-starting files out of `cardGateCommand`'s `-tags integration` run and compiles nothing for a retired `smoke` tag in in-flight plan verifies.
`cardGateCommand` is out of scope and stays as it is, so until note `test-tiers` decides gate selection Webster's card gates stop running the `tmux` tier.
This task's own plan verify, the final gate, runs `-tags tmux` wherever those tests ran under `integration` or `smoke` before.
Another task's verify runs `tmux` only if its plan writer follows the D10 text, so until `test-tiers` decides, the `tmux` tier outside this task's verify is run by hand or by an explicit verify.
Files moved to `llm` from `smoke` or from `integration` leave every automatic run, including `cardGateCommand`'s `-tags integration` run and any plan verify that ran `-tags smoke` or `-tags integration` over them; this is intended, since the tier costs tokens.
They are run by hand, and `go vet -tags llm ./...` compiles them so they do not rot.
No test file may end up under a tag that neither a run nor a compile check covers: `integration` is run by the standing gates, `tmux` by this task's plan verify and by hand until `test-tiers` decides, and `llm` is compile-checked only; `go vet -tags tmux ./...` compiles `tmux` files as well.

Rationale: the operator decided the four tiers on 2026-10-03 and the `llm` name on 2026-10-05; `tmux` names what tier 3 needs, where `smoke` named nothing; keeping `integration` for tier 2 avoids churning the tag on every git test, `cardGateCommand`, every plan verify and the verified-tree records.

### D7 — The LLM spawn guard

Decision: a new kit `internal/testkit/llmkit` holds the one helper that locates the LLM binary and skips the test when it is absent, replacing the per-package copies (`smokeClaudeBin` and the like).
A guard test in `cmd/lyx`, beside `tierpurity_test.go`, fails when a test file without the `llm` tag references `llmkit`, or when any test file outside `llmkit` calls `exec.LookPath` on an LLM binary name.
`llmkit` imports `os/exec` for `LookPath` only; that is a third, bounded exemption in `PATTERN-testkit`, recorded in its background.
The guard test also fails when a file under `internal/testkit/llmkit` uses an `exec` identifier other than `LookPath`, because the kit's import check cannot see which `os/exec` identifiers a kit uses.
Bound: the guard sees the static shape only; a tier-3 test that drives `lyx` into spawning an LLM without calling the helper is caught by review, which the PATTERN entry states.
Rationale: one helper gives the guard a single static shape to scan, and `PATTERN-testkit` already requires shared test support to live in one kit.

### D8 — PATTERN Testing consolidated

Decision: the Testing section ends with five entries:

- `PATTERN-test-economy` — unchanged rule; its background points at `scribe:testing` and `scribe:golang-testing` for the general rules and keeps only what is project-specific or stricter (a redundancy finding names the covering test; the removable-set reading of the D2 report).
- `PATTERN-test-speed` — absorbs `PATTERN-test-tier-purity`, `PATTERN-test-time-seam` and `PATTERN-test-parallel`, and carries the D6 tier table and the D7 LLM rule.
- `PATTERN-test-isolation` — absorbs `PATTERN-hermetic-git-tests` and `PATTERN-tmux-test-isolation`, and adds the socket rule from D9.
- `PATTERN-testkit` — gains the `llmkit` exemption.
- `PATTERN-hubforge-fixtures` — unchanged.

The two new entries each get a background file; the retired IDs' background files are deleted (`PATTERN-test-time-seam` and `PATTERN-test-parallel` had none).
Enforcement tests keyed to a retired entry (`tierpurity_test.go`, `tmuxisolation_test.go` and the hermetic-git check) keep their checks and name the new entry; every other reference to a retired ID in code comments, stencils, specs and docs is rewritten to the new one.
Rationale: the operator asked for the section revised, not grown; the merges group rules by what they protect (speed, isolation) rather than by mechanism.

### D9 — The tmux socket leak

Decision:

- The plan first runs the full suite under every tier but `llm` and lists the socket files it leaves in `/tmp/tmux-$UID/`, by prefix, to find which sources still leak after `tmuxkit` landed on 2026-10-03.
- Tests: every test that starts a tmux server takes its socket from `tmuxkit` (its `Socket` key or one new shared helper there), and that helper removes the socket file after `kill-server` in `t.Cleanup`; `tmuxkit.Main`'s sweep removes the files it kills; a test that builds a subprocess environment from scratch passes `TMUX_TMPDIR` through, so sockets land under `tmuxkit.Main`'s directory.
  Ad hoc key schemes (the reedengine contract keys, the reedcli attach and dotfill harnesses) move onto the helper.
- Production: if reed's teardown (`kill-server` in standalone geometry, warp-bare or any other geometry) leaves its socket file, reed removes it after confirming the server is gone.
  Bound: reed removes only the socket file for the `-L` key it killed, only inside the teardown that just killed that key's server, only when the path is a socket and no server answers on it; it never globs or sweeps the socket directory.
  A connection that succeeds, or a socket whose identity changed between the check and the removal, leaves the file alone; a server that restarts on the same key in that window is the residual race, accepted because the removal is limited to a key reed has just torn down.
- Socket files already in `/tmp/tmux-$UID/` from earlier runs are not cleaned by this task.

Rationale: `TMUX_TMPDIR` isolation already exists, so remaining leaks are escapes from it or files `kill-server` leaves behind; a shared helper fixes both in one place, and a bounded production removal keeps reed from touching servers it does not own.

### D10 — Docs and stencils

Decision: `docs/benchmarks/running-tests.md` describes the four tiers and their commands; the verify-model section of `contracts/specs/loom-plan-spec.md` names the `llm` tag for its never-automatic tier; `loom-template-plan.md` and `loom-rubric-plan-review.md` say the `llm` tag is compiled, never run (`go vet -tags llm <packages>`), and that `tmux` is a token-free tier a plan verify may run.
Which gate runs `tmux` is left to note `test-tiers`.
Rationale: the board asks for the tier rules in Plan-Write's verify text; gate selection is the note's.

### Added after Discussion (parent, 2026-10-05): Cross-tier test helpers carry a disjunctive build constraint

Decision: A test helper file shared by files of two tiers, whose helpers spawn a process or tmux server, carries the disjunction of those tiers as its constraint (for example `//go:build tmux || llm`), so it compiles under each tag alone; D6's untagged-file or testkit-kit placement stays the rule for a helper that spawns nothing.

Rationale: D6 requires each tag to compile on its own, but tier purity bars spawn tokens from untagged test files and PATTERN-testkit bars os/exec from kits, so the reedcli and shuttlecli smoke harnesses shared by tmux and llm files have no other home that compiles under both tags.

## Constraints

- `PATTERN-test-economy` and `PATTERN-test-speed` (which absorbs the time-seam, parallel and tier-purity entries, D8): the speed fixes follow them.
- `PATTERN-testkit`: `llmkit` is a kit under `internal/testkit/`, imported only from tests; its `os/exec` exemption is bounded and recorded (D7).
- `PATTERN-test-isolation` (absorbs `PATTERN-tmux-test-isolation` and `PATTERN-hermetic-git-tests`, D8): their checks survive the merge; the "has a tagged test file" rule keys on `integration`, `tmux` or `llm` instead of `integration` or `smoke`; the `!integration` and `integration` `TestMain` file pairs stay paired and both call `tmuxkit.Main`; the `!integration` files keep their constraint, so they are the `TestMain` under `tmux`-only and `llm`-only builds.
  The check's tag sets cover untagged, each of `integration`, `tmux` and `llm` alone, and `integration,tmux,llm` combined, and exactly one `TestMain` compiles under each set on each platform.
- `PATTERN-spawn-observability`: the production socket removal in reed is not a process spawn; a new `go test` spawn in `cmd/testtiming` is a dev tool outside `lyx` commands, so the rule does not apply to it.
- `PATTERN-lyxdirs-single-declarer`: the tool and docs write under `docs/`, not `_lyx` or `.lyx`.
- `PATTERN-markdown-link-integrity`: every link in the edited and new `docs/` files resolves.
- `PATTERN-comment-line-breaks` and semantic line breaks in every `.md` file.
- `PATTERN-quarry-cgo`: the coverage build runs with cgo on.
- A new cross-cutting invariant lands in `PATTERN.md` in the same commit as its code (CLAUDE.md).
- No production behavior changes beyond the D9 socket removal; seams keep production values.

## Auto-mode assumptions

- Tier-3 tag name `tmux` and keeping `integration` for tier 2 (D6) were picked without the operator.
- `cmd/testtiming` is extended rather than a new `cmd/testprune` (D1).
- Redundancy uses coverage blocks, module-wide `-coverpkg`, top-level tests and a greedy removable set (D2).
- The LLM guard goes through a new `llmkit` kit (D7).
- The consolidation is the board's proposal as written (D8).
- The measurement machine is whichever machine runs the task; the before-state names it, its revision and its load.

## Open risks

- Measurements taken while other runs load the machine are noisy; the before and after must run under comparable load, and the doc section records the load average.
- Per-test coverage runs every top-level test in its own process; on the full suite this can take a long time, and tmux-tier tests may behave differently run one at a time.
- Coverage blocks do not see assertions: a candidate may pin an error message or output shape no other test pins; the prune tasks check each before deleting.
- Socket leaks may come from a source the D9 survey does not reproduce on one run, such as a test killed by timeout.

## Acceptance criteria

- `go run ./cmd/testtiming` in redundancy mode runs on the module and writes the per-package report: test count, wall time, serial time, candidates with their covering tests, the removable set, and "no coverage" tests.
- `docs/benchmarks/test-suite-timing.md` holds the dated before-state with its slowest-tests table, and `docs/benchmarks/test-redundancy.md` holds the report; both are committed.
- On the pre-retag test set, `go test -tags integration ./...` wall time after the speed fixes is lower than the before-state, and the PR states both numbers and the post-retag numbers.
- Each seamed wait (watchdog interval, `reapExitTimeout`, `waitDelay`) is exercised by a test that shortens it, and no test waits out its production value.
- A full run of every tier but `llm` leaves no new file in `/tmp/tmux-$UID/`.
- A reed teardown in production geometry leaves no socket file for the server it killed, and never removes a socket whose server still answers.
- Every test retagged out of `integration` or `smoke` into `integration` or `tmux` still runs under this task's plan verify (`-tags tmux`; other gates follow `test-tiers`), every test retagged into `llm` from either tag is compiled and never run by a gate, every `llm` file compiles under `go vet -tags llm ./...`, and no tagged test file sits under a tag that neither a run (`integration` by the standing gates, `tmux` by this task's verify) nor a compile check covers.
- A test whose run spawns a subprocess appears in the redundancy report's "no coverage" list, never as a candidate.
- No `smoke` build tag remains; every tagged test file carries `integration`, `tmux` or `llm`.
- The LLM guard fails on a non-`llm` file referencing `llmkit` or looking up an LLM binary, and passes on the retagged tree.
- `PATTERN.md`'s Testing section has five entries; no code comment, stencil, spec or doc references a retired entry ID (the discussion artifacts are exempt); the enforcement tests still catch what they caught before under the new IDs.
- `go build ./...`, `go vet -tags integration,tmux,llm ./...`, and `go vet -tags integration ./...`, `go vet -tags tmux ./...` and `go vet -tags llm ./...` each on its own, `go test ./...`, `go test -tags integration ./...` and `go test -tags tmux ./...` pass.

## Notes for the plan writer

- Socket files in `/tmp/tmux-1000/` on 2026-10-05 were mostly dated 2026-10-03, before `tmuxkit` landed (`5805c58fb`); runs on 2026-10-05 still left `lyx-001-*`, `lyx-contract-*` and `lyx-Test*` files, so those sources escape `TMUX_TMPDIR`.
- `cmd/lyx/tierpurity_test.go`, `cmd/lyx/tmuxisolation_test.go`, `internal/shuttlecli/modelpin_test.go` and many `testmain_test.go` comments name the `smoke` tag or a retired PATTERN ID.
- Tests that look up `claude` on `PATH` today: grep `LookPath("claude")` under `internal/`.

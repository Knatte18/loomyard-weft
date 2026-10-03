# Decision record — test-suite-derived-guards

## Goal

Tests derive their expectations from the code they guard instead of keeping hand-written copies of lists owned elsewhere, so adding a module, command, config section, path constructor, stencil or recipe row edits no test file.
Every invariant scan runs on one shared harness instead of re-implementing root lookup, walk, allowlist and floor.
Integration and smoke tests leave no tmux socket or server behind.
This is part 2 of the test-suite cleanup: it carries the full scope of GitHub issues #326 and #327 and the board's socket-leak item, and builds on part 1's `internal/testkit` (`22e46970e`).
#326 and #327 hold the file:line evidence, recorded against `fafb660c3`; every location is re-verified against HEAD.

## Scope

In scope, all of it:

- #326 §1: one source-scan harness, every invariant scan converted to it.
- #326 §2, approved by the operator (relayed by the orchestrator, 2026-10-03): CLI-tree, config-registry and path-constructor tests derived from the real registries; per-CLI `NoArgs`/`UnknownSubcommand`/`Short` copies deleted; allowlists one entry per line.
- #327 §1–§7: recipe row list pinned once, the plan-fixture writer (`plankit`, which part 1 handed to this task, including the glyph-plan and file-tree fixture copies in `loomshed`, `loomcli` and `planglyph`), stencil seeding through the registry, one claims table per stencil, registry-wide shedrecipe conformance tests, wrong-layer test deletions plus the shared commit-status seam, and wide positional functions.
- The socket leak: one shared tmux test isolation, and a scan that makes every package with live-tier tests use it.
- The board's operator directive: an audit of the `CONSTRAINTS.md` invariants bearing on these tests, with the changes in D20.

Out of scope:

- Production behaviour, except D15's commit-status seam extraction, which moves code without changing what it does.
- Sweeping the dead sockets already in the operator's `/tmp/tmux-<uid>/`; that is a one-off `rm` on the operator's machine, not code.
- Sockets and sessions left by production `fabric remove`: `pair-lifecycle`.
- Card-level verify scope and the webster verify gate: `verify-publish-hardening`.
- Moving `CONSTRAINTS.md` to `_lyx/PATTERN.md` and extending the Fabric Vocabulary scan to test files: `pattern-init`.
- Mutation testing (evaluated and not adopted in #323).

## Decisions

### D1 — One scan harness: `internal/testkit/scankit`

Decision: a new kit, `internal/testkit/scankit`, imports the standard library only.
It provides:

- the module root, found from the kit's own source location via `runtime.Caller` and the enclosing `go.mod`, with no process spawned;
- a walk over named roots with the shared skip-dir set, a production/test file filter, an extension filter and slash-normalized relative paths, yielding each file's path, bytes and a lazily parsed `*ast.File`;
- an allowlist type keyed by entry that reports both violations not on the list and stale entries that matched nothing;
- a vacuous-scan floor that fails when fewer files than the stated minimum were scanned;
- an import-allowlist assertion for one package directory (stdlib plus named import paths).

The exact API is the plan's call.

### D2 — Every invariant scan converts to the harness

Decision: every test that enforces a `CONSTRAINTS.md` invariant by walking source or resolving the repo root uses `scankit` for root, walk, allowlist and floor: the `cmd/lyx` guards, every `seam_enforcement_test.go`, `leaf_enforcement_test.go` and other `*_enforcement_test.go`, the Testkit scan, `internal/lyxcwd/enforcement_test.go`, `contracts/stencils/citation_enforcement_test.go`, the docs-link scan and the sandbox coverage guard's root lookup.
Each scan keeps its own matching logic, allowlist entries, floor and token set verbatim.
Two tightenings apply to every converted scan: stale-entry detection on its allowlist, and a floor (at least one scanned file where the scan had none).
A stale entry found by the tightening is removed in the same change and itemised in the PR description.
Each converted scan is proved by planting one violation, watching it fail and removing the plant.

Bound: the harness is shared, so a harness bug weakens every scan at once.
The planted-violation proof per scan and the harness's own unit tests (walk filter, stale detection, floor) bound it.

Rationale: a new scan then costs only its matching logic.

### D3 — Import-allowlist guards stay in their own packages

Decision: each leaf and seam import guard stays in its own package as a short caller of the D1 import-allowlist assertion, with the package's allowlist beside it.
Guards with extra checks (for example `shedverbs`' resolver-token ban) keep those checks, on the harness walk.

Rationale: `go test ./internal/<pkg>` still catches that package's import violation, which a card's package-scoped verify relies on, and the `CONSTRAINTS.md` sections that name a package's enforcing test stay true.

### D4 — No `go env GOMOD` spawn in any scan

Decision: every scan resolves the module root through `scankit`; the `exec.LookPath("go")` skips and the `go env GOMOD` execs are removed.
The Test Tier Purity and Hermetic Env allowlist entries whose only reason was that exec become stale and are removed in the same change.

### D5 — One CLI-tree walk replaces the hand-kept help tree

Decision: one test in `cmd/lyx` walks `newRoot()` and asserts, for every command: non-empty `Short`; for every group, a bare invocation exits 0 and lists every non-hidden child, and `<group> bogus` exits 1 with a JSON `ok:false` envelope; `root.Long` names every mounted module.
It replaces `helptree_test.go`'s module and subcommand lists, `unknown_subcommand_test.go`'s hard-coded groups, `longlist_test.go` and `drift_test.go`.
`registration_test.go` stays as the independent oracle that every `Command()` package is mounted.
A test that pins one specific flag or behaviour rather than a list (for example `orch start --help` naming `--adopt`) stays.
Deleted: the per-CLI `NoArgs`, `UnknownSubcommand`, `EveryVerbHasNonEmptyShort` and `EveryCommandHasShort` copies, and the exact-verb-set tests (`webstercli`'s all-subcommands test, `loomcli`'s `RegisteredVerbs_ExactSet`).

Bound: a derived listing no longer notices a verb being deleted; that verb's own behaviour tests must catch it.
`go test` of one CLI package no longer checks `Short`; `go test ./cmd/lyx` does, and the plan-level verify runs it.
A deleted test whose comment cites a finding or crucible round is not deleted: its assertion moves into the walk or stays (D21).

### D6 — Config registry asserted by property

Decision: `internal/configreg`'s name-list test asserts properties instead of a literal list: names sorted, unique, and each with a template that resolves.
Tests that assert a specific module's attribute (`SeedOnly` for `models` and `burler`, the `fabric` template) stay.
A scan on `scankit` asserts that every production package declaring `ConfigTemplate` is registered in `Modules()`, as the independent oracle for membership.

Bound: the property test no longer notices a module dropped from `Modules()`; the registration scan does, and a new module's `ConfigTemplate` is caught there with no test edit.

### D7 — Path guards: one table of constructors, expectations from the rule

Decision: `constructoranchoring_test.go` and `notransients_test.go` merge into one table in `cmd/lyx`.
Each row names a constructor, a closure calling it on a `*lyxcwd.Location`, and its class (durable `_lyx`, transient `.lyx`, or hub-anchored).
The test runs every row against an unanchored and a subpath-anchored location and asserts the rule: a durable path sits under `AnchorPath()/_lyx`, never ends in `.lock`, has no `pause` base and no `prompts` segment; a transient path sits under `AnchorPath()/.lyx` and never under `_lyx`; a hub-anchored path sits under the hub.
The mirrored-pairs check (a transient path mirrors its durable sibling's subpath) is kept, over the same table.
No row carries an expected path literal.

Bound: a constructor nobody adds to the table is unguarded, as it is today; listing it is one row with no expected value.

Rationale: a path change edited three files, asserting each path twice; production has no path registry to derive from, so the table holds the constructors and the rule computes the expectation.

### D8 — Allowlists one entry per line

Decision: every allowlist a converted scan carries is written one entry per line without aligned values (a `[]string`, or a map whose values are not column-aligned by gofmt, such as a slice of `{entry, reason}` pairs).
`gitrepoboundary_test.go`'s pinned-method set is the first.

### D9 — Recipe row list pinned once

Decision: per #327 §1.
`TestNew_ProducerTable` and `TestNew_ProducerTableOrderUnchangedByWiring` are deleted as strict subsets of `TestNew_ShapeMatchesRecipe`.
`wantProducerTable`'s routing columns are replaced by a perch-shape property: every Bouncer's `OnStuck` names a Burler whose `OnStuck` names it back, with equal `Segment` and `MaxBounces`, and no orphan rows; only the concrete-type column stays pinned.
`sequence_test.go` and `coverage_guard_test.go` read that one table instead of their own copies; `coverage_guard_test.go` is restructured, never weakened.
Proof: swapping two `OnDone` targets in the recipe YAML still fails a test.

Bound: the property pins `MaxBounces` and `Segment` only as equal within a Bouncer/Burler pair, so a pair's `max_bounces` value or a consistently renamed segment label changes unnoticed, except Discussion-Bouncer's budget, which `resume_test.go` drives.
`OnDone` order through Publish stays guarded by `sequence_test.go`, and the PR-Gate and PR-Rework edges by `TestRecipe_PublishRoutesThroughPRGate`.

### D10 — `internal/testkit/plankit` writes plan fixtures

Decision: one kit renders a valid plan directory from an overview (approved flag, framing) and cards, using `planparser`'s current format constant and grammar.
It also writes the glyph-plan and file-tree fixtures that glyph resolution needs.
Every hand-written plan helper outside `internal/planparser` converts to it (the websterengine, webstercli, loomcli, loomrecipe, loomshed and planglyph helpers #327 §2 and part 1 list); each converted fixture still produces an equivalent valid plan.
Tests of the parser's rejection keep writing raw bytes, and `planparser`'s own tests keep their fixtures.
`TestValidate_GoldenFixture_ZeroFindings` stays as the parser-side anchor.

The Planparser Sole-Parser Invariant is restated: `planparser` is the sole production parser and writer; in tests, `plankit` is the one writer of valid plans, and a test may write raw plan bytes only to test rejection of a malformed plan.

Bound: this adds a second, test-only writer of the plan format.
It is bounded to `_test.go` callers by the Testkit importer rule, renders through `planparser`'s exported format constant so a format bump fails at one site, and the golden-fixture test stays the parser's anchor.

Rationale: each hand-written helper is a de facto second writer already, and one is better than many.

### D11 — `internal/testkit/stencilkit` seeds stencils through the registry

Decision: `stencilkit` seeds a temp directory from `stencils.Registry()` through `stencilstore`'s reconcile path, and removes named stencils for deliberate-absence tests.
The hand-listed seeders (#327 §3) convert; deliberate-absence fixtures become "seed all, then remove X".

Rationale: adding or renaming a stencil then edits no test file, and seeding goes through `stencilstore`, the owner the Stencil Ownership Invariant names.

### D12 — One claims table for stencil wording

Decision: one `contracts/stencils/claims_test.go` holds a table keyed by stencil file, each claim a `must` or `mustNot` phrase with a `why`, checked by one table-driven test over the raw stencil text.
Raw-stencil wording checks in `websterengine`, `loomengine`, `burlerengine`, `shedadapters` and `contracts/stencils` move into it.
A claim's `why` keeps its provenance (crucible round, finding, sabotage proof); a claim that cites one is protected (D21).
Claims that pin generic wording rather than a behaviour are dropped, each weighed against its `why` and listed in the PR description.
Left in place: assertions on rendered output (`Fill`, `RenderForkPrompt_*`, `RenderProgress_*`, ordering), marker-set and round-trip tests, Go-contract co-versioning checks, the rubric marker allowlist and `citation_enforcement_test.go`.

Bound: dropping generic-wording claims lets those phrases change unnoticed.
Only claims with no cited provenance may be dropped, and each drop is itemised.

### D13 — shedrecipe conformance over the whole registry

Decision: per #327 §5.
One `TestEveryEntry_RejectsUnknownConfigKey` ranges `Names()` with a per-name minimal valid config, replacing the per-entry copies.
`seedChildEntry` folds into `lifecycleEntryCases()`.
Each entry keeps exactly one `run_subdir` wiring row ("escaping fails") plus "CreatedBeforeAnyCall"; the rest of the `resolveUnderRoot` rows stay with `TestResolveUnderRoot`.
The `strings.Contains(err.Error(), field)` checks stay.
Proof: removing one entry's `configRejectUnknown` call fails the registry-wide test.

Bound: a new entry gets unknown-key coverage only once it has a minimal-config row; a missing row fails the test, so it cannot be skipped silently.

### D14 — Wrong-layer and duplicate tests

Decision: per #327 §6.
`loomcli`'s tests of `shedverbs.StepEnvelope`, `RenderStatusLine` and `PrintStatusLinesOnChange` that duplicate `shedverbs`' own are deleted, after diffing the two 16-key lists once; `TestStepEnvelope_FieldMapping` moves to `shedverbs` if `shedverbs` lacks it; the loom-specific tests #327 names stay.
`validate-plan`'s tests in `loomcli` and `webstercli` stay: the two verbs render findings into different envelope shapes, so there is no shared mapper to test once.

Bound: a deleted `loomcli` test is removed only case by case.
A case with no counterpart in `shedverbs`' own tests moves there first, for example `loomcli`'s `PrintStatusLinesOnChange` cases for the deduped unavailable line, a reprint after returning to an earlier line and the always-printed first line.
`loomcli`'s own wiring of `shedverbs` stays asserted by the loom-specific tests that stay.

### D15 — One commit-status seam

Decision: the commit-status seam duplicated in `internal/loomcli` and `internal/battencli` (deps struct, constructor, the three dispositions: commit hard-errors, push warns, skip while mid-merge) moves into one production package below both CLIs and is tested once there.
`battencli` keeps and tests only its marker-skip layer; the six duplicated tests collapse to one set.
`wiring_commitstatus_integration_test.go` stays (D21).
Behaviour and log text are unchanged.

Rationale: `battencli`'s copy says "carried over unchanged"; a fix in one copy would miss the other.

### D16 — Wide positional functions

Decision: no production signature changes for test convenience.
Where a wide function has many test call sites (`websterengine.RenderIntegrationPrompt`, `shedrun.StatusRel`), each test file routes its calls through one local helper, so a signature change edits one line per file.

Rationale: `verify-publish-hardening` is expected to retire Merriam's integration fork and with it `RenderIntegrationPrompt`; a local helper costs nothing to delete with it.

### D17 — `internal/testkit/tmuxkit`: isolated tmux per test package

Decision: `tmuxkit` has two parts.

- A `TestMain` entry point: creates a short per-process directory with `os.MkdirTemp`, sets `TMUX_TMPDIR` to it, unsets `TMUX` and `TMUX_PANE`, runs the tests, then runs `tmux -S <socket> kill-server` for every socket in its own directory and removes the directory.
  It never touches a socket outside its own directory, since packages run as concurrent processes.
  It fails setup when the directory plus `tmux-<uid>/` plus reed's longest key (61 bytes) would exceed the 107-byte socket path limit, naming `TMPDIR` as the remedy.
  When `tmux` is not on `PATH` it sets the environment and skips the sweep.
- A per-test socket fixture for tests that start tmux directly: a unique `-L` key, with `kill-server` in `t.Cleanup`.
  Tests that today hand-roll a `lyx-Test…`/`lyx-contract-…` key and their own `kill-server` cleanup convert to it.

Every tmux server a test starts, directly or through reed, the `lyx` binary or a reed pane, inherits `TMUX_TMPDIR`, because production reed (`internal/reedengine`) always addresses tmux by `-L <key>` and `OwnsTmuxEnv` compares only the socket basename.
On Windows (psmux) the entry point runs the tests unchanged.

Bound: the sweep kills only servers whose socket is under the kit's own directory.
A test binary killed by a panic or timeout skips the sweep and leaves one `lyx`-prefixed temp directory with its servers; the per-test fixture's `t.Cleanup` still covers ordinary failures.
Unsetting `TMUX` means no test can reach the caller's own tmux server; a test that exercises `$TMUX` handling sets it explicitly.

Rationale: sockets survive their servers today, and some packages have no cleanup at all; isolating the socket directory removes them all, whatever each test does.

### D18 — Tmux Test Isolation Invariant

Decision: a new `CONSTRAINTS.md` section: every test package with an `integration`- or `smoke`-tagged test file has a `TestMain` that runs its tests through `tmuxkit`'s entry point, under every tag set that compiles that package's tests.
Enforced by a scan in `cmd/lyx` on `scankit`, modelled on the Hermetic Git Test Environment scan, with an allowlist (entry plus reason) and stale-entry detection.
The allowlist starts empty.
An entry is admitted only for a package whose `TestMain` cannot call the entry point, and its reason names that obstacle; "does not start tmux" is not a reason.
Test Tier Purity gains one rule, parallel to its `gitkit` rule: every `tmuxkit` export except the `TestMain` entry point counts as a spawn and is barred from untagged files.
In an untagged run, the entry point spawns nothing, because no server exists to sweep.

Bound: the rule keys on "has a tagged test file", not on "starts tmux", so a tagged package that never starts tmux still carries the one call.
That over-approximation is deliberate: "starts tmux" has no static shape (the `lyx` binary and reed start servers indirectly), and the call costs one temp directory.

### D19 — Testkit Invariant restated

Decision: the section's scope changes from "a seam faked in two or more packages" to shared test support (fakes, builders, fixtures and the scan harness) used by two or more packages.
Its rules gain:

- `scankit` imports the standard library only, so every package's tests can import it.
- `tmuxkit` is exempt from the `os/exec` ban, bounded to running the `tmux` binary against sockets under its own directory or its own fixture keys.
- No other kit imports `tmuxkit`, as for `lyxbin`, so the exemption cannot reach another kit transitively and Tier Purity's `tmuxkit` rule sees every spawning call.
- `scankit` asserts scan results; `plankit` and `stencilkit` assert nothing beyond `t.Fatalf` on their own setup.

The enforcing scan is rebuilt on `scankit` and covers the new kits.

Bound: the `tmuxkit` exemption lets a second kit import `os/exec`.
Review keeps it to `tmux`, the rebuilt Testkit scan bars other kits from importing it, and D18's Tier Purity rule keeps its spawning exports out of untagged files.

### D20 — Invariant audit under the operator directive

Decision: of the invariants bearing on this task's tests, only these change:

- CLI / Cobra Invariant: gains an enforcement line stating D5 (one walk of `newRoot()` in `cmd/lyx`, `registration_test.go` as the mount oracle); per-CLI copies are no longer an obligation.
  Its intent is weakened only as D5's bound states.
- Testkit Invariant: restated per D19.
- Test Tier Purity Invariant: gains D18's `tmuxkit` rule; its `go env` allowlist entries retire (D4).
- Hermetic Git Test Environment Invariant: rule unchanged; its scan moves to `scankit` and its `go env` entry retires.
- Planparser Sole-Parser Invariant: restated per D10.
- New: Tmux Test Isolation Invariant (D18).

Unchanged in text, with their enforcement moved onto `scankit` where they have a scan: every Leaf and Seam invariant (D3), Cwd Resolution, Durable-vs-Ephemeral, Shed Recipe Registry, Agent Name, Stencil Ownership, Sandbox Suite Coverage, Markdown Link Integrity and Shed Verb-Set.

### D21 — Protected tests and verification

Decision: #323's rules and part 1's D9 hold.
Protected, restructured at most and never deleted: every `CONSTRAINTS.md` invariant scan, every regression test whose comment cites a finding, crucible round, sabotage proof or live observation, every `smoke`/`integration` live test, the gitrepo oracle, the fabricengine `livestate_*` harness and its selftests, and `wiring_commitstatus_integration_test.go`.
A protected assertion leaves its test only when the same behaviour stays asserted where this record names its new owner (D5's walk, D12's claims table, D14's `shedverbs` tests, D15's seam package), itemised in the test-count accounting.
Verification is `go test ./...` and `go test -tags integration ./...`, plus `go test -tags smoke` for every package whose smoke files change, green before and after.
Leaf test counts (top-level tests without subtests plus every subtest, from `go test -json`) are recorded before and after, and every drop is itemised against a deletion this record names.

### D22 — Test comments

Decision: part 1's D10 continues: a touched test drops doc comments that restate its name, and no comment counts rows or lists seams, modules or verbs by name.

### D23 — Docs

Decision: `internal/testkit/doc.go` and each new kit's package doc describe the kit; `docs/overview.md`'s test-infrastructure passage gains `scankit`, `tmuxkit`, `plankit` and `stencilkit`, and names D15's seam package if its module table lists packages at that layer.
`CONSTRAINTS.md` changes land in the commit that changes their enforcement.
No `manifest/designs/` doc: no CLI module is added or changes observable behaviour.

### D24 — Order of work

Decision: `scankit` and the Testkit restatement land first, then the scan conversions package by package, then the derived registry tests (D5–D8), then #327's items, then `tmuxkit` with its invariant and scan.
Each step leaves the suite green.

Rationale: the later scans (D18, the rebuilt Testkit scan) are written on the harness, and a red suite mid-task hides which conversion broke it.

## Constraints

- `CONSTRAINTS.md` is authoritative; each change in D20 lands with the code and scan that enforce it.
- Testkit Invariant (as restated in D19): kits are imported only from `_test.go` files, import no `internal/*cli` package, and only `lyxbin` and `tmuxkit` may import `os/exec`; no kit imports `lyxbin` or `tmuxkit`.
- Test Tier Purity: untagged test files spawn nothing; `scankit` resolves the root without a spawn; `tmuxkit`'s spawning exports stay in tagged files.
- Hermetic Git Test Environment: a package that gains a `TestMain` for D18 keeps or adds `gitkit.HermeticGitEnv()` where its tests spawn git.
- Fabric Vocabulary Invariant: kit production files are outside the owner set and must not use warp/weft.
- Planparser Sole-Parser Invariant (as restated in D10).
- A scan whose scaffolding changes keeps its allowlist, floor and token set verbatim apart from D2's two tightenings.
- Production behaviour and operator-facing text are unchanged; D15 moves code only.
- Semantic line breaks in every `.md` file and multi-line comment.
- Build needs cgo; the integration and smoke tiers need git and tmux.

## Auto-mode assumptions

- No operator answered questions in this session; every pick in `## Decisions` was made autonomously, guided by #323, #326, #327, part 1's decision record and the board body.
- #326 §2's operator approval was relayed by the orchestrator (`ly:orch`) mid-session and is taken as given; it also confirmed that changing the CLI/Cobra Invariant's enforcement is in scope.
- The survey's duplication still exists at HEAD: spot checks found the 11 leaf files, 14 `go env GOMOD` scans, the help-tree lists, the path-constructor tables, `configreg`'s literal list, the two commit-status seam copies and production reed addressing tmux only by `-L`.
- `TMUX_TMPDIR` redirects every `-L` socket, including those of servers started by child processes and reed panes, since they inherit the test process's environment.
- Kit names `scankit`, `tmuxkit`, `plankit` and `stencilkit` follow part 1's `<thing>kit` pattern.

## Open risks

- Size: several thousand changed lines across most packages; webster batches must stay package-local so a failure localizes.
- D2's tightenings may surface stale allowlist entries or floorless scans that scanned nothing; each is fixed and itemised, never waved through.
- A test that builds a child process's environment from scratch would drop `TMUX_TMPDIR` and create its socket in the default directory; the plan finds such spawns and passes the variable through.
- Unsetting `TMUX` may change a test that silently relied on running inside the caller's tmux; such a test is fixed to set the variable it needs.
- The smoke tier needs tmux and may be flaky on the host; a flaky result is re-run, not waved through.
- Deleting the per-CLI copies and exact-verb-set tests moves `Short` and verb-presence coverage to `cmd/lyx` (D5's bound); a card verifying one CLI package alone no longer sees it.

## Acceptance criteria

- `internal/testkit/scankit` exists, imports the standard library only, and has unit tests for its walk filter, stale-entry detection and floor.
- Every invariant scan named in D2 uses `scankit` for root, walk, allowlist and floor; no test file runs `go env GOMOD` or skips on a missing `go` toolchain to find the root; the Tier Purity and Hermetic Env allowlist entries for that exec are gone.
- Each converted scan was proved by one planted violation that failed it, with the plant removed; the PR description lists the proofs and every stale entry removed.
- Each leaf and seam package keeps its own import guard as a short `scankit` caller (D3).
- `cmd/lyx` has one `newRoot()` walk asserting `Short`, bare-group listing, `bogus` refusal with an `ok:false` envelope, and `root.Long` naming every module; `helptree_test.go`'s lists, `unknown_subcommand_test.go`'s hard-coded groups, `longlist_test.go` and `drift_test.go` are gone or folded in; no `*cli` package keeps a `NoArgs`, `UnknownSubcommand` or `Short` copy or an exact-verb-set test; `registration_test.go` is unchanged in what it asserts.
- Adding a module, a verb, a config section, a path constructor, a stencil or a recipe row edits no test file (shown by inspection of the derived tests; a path constructor is guarded once it has a table row).
- `configreg`'s name test asserts sorted, unique and template-backed names, with no literal list; removing one entry from `Modules()` fails the registration scan (planted, then removed).
- One path-constructor table in `cmd/lyx` replaces `constructoranchoring_test.go` and `notransients_test.go`, with no expected-path literal per row; moving a transient constructor under `_lyx` fails it.
- Touched allowlists are one entry per line without aligned values.
- D9's deletions are made; swapping two `OnDone` targets in the recipe YAML fails a test; `sequence_test.go` and `coverage_guard_test.go` carry no list of their own.
- `plankit` writes every valid plan fixture outside `planparser`; `format: 5` appears in no test file outside `planparser` and `plankit`; the Planparser invariant reads as D10 states.
- `stencilkit` seeds every stencil fixture; no test hand-lists stencil byte slices to seed a directory.
- `contracts/stencils/claims_test.go` holds the raw-stencil wording claims, each with a `why`; every dropped claim is listed with its reason.
- shedrecipe has one registry-wide unknown-key test; removing one entry's `configRejectUnknown` call fails it.
- D14's `loomcli` duplicates are deleted, every deleted case is asserted in `shedverbs`' own tests, and D15's seam lives in one production package with one test set; `wiring_commitstatus_integration_test.go` still passes.
- `tmuxkit` exists; every package with an `integration`/`smoke`-tagged test file runs its tests through its entry point, and the D18 scan fails when one such package's `TestMain` lacks the call (planted, then removed); an untagged `tmuxkit` fixture call fails Tier Purity (planted, then removed); a kit importing `tmuxkit` fails the Testkit scan (planted, then removed).
- After `go test -tags integration ./...` and the smoke tier, the default socket directory (`/tmp/tmux-<uid>/`) has no new socket from a test, and no tmux server started by a test is still running.
  A new socket is excluded only when its key equals `reedengine.ServerName` of one of the operator's own hub paths; any other new socket fails the criterion, whether or not its server answers.
- `CONSTRAINTS.md` carries every D20 change; `docs/overview.md` and the kit package docs are updated.
- `go test ./...`, `go test -tags integration ./...` and the smoke tier of every package whose smoke files changed are green before and after; leaf test counts before and after are recorded, and every drop is itemised against a deletion this record names.

## Notes for the plan writer

- Evidence: `gh issue view 323`, `326`, `327`; part 1's decision record is in the weft archive tag `archive/test-suite-change-amplification/*` under `_lyx/discussion/`.
- `internal/orchcli` already has a tag-split `TestMain` pair (`testmain_test.go` with `!integration` and an integration-tagged sibling); D18 must cover both.
- `cmd/lyx/hermeticenv_test.go` is the model for D18's scan; `cmd/lyx/gitkitspawn_test.go` holds the shared gitkit spawn definition that D18's `tmuxkit` rule parallels.
- `internal/lyxcwd/enforcement_test.go` already has a shared-walker shape inside one package.

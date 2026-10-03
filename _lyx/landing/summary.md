# Derive test expectations from code; isolate tmux in tests

Tests now take their expectations from the code they guard instead of keeping hand-written copies of lists owned elsewhere.
Adding a module, command, config section, stencil or recipe row edits no test file; a path constructor is guarded once it has a row in the path table.
Every invariant scan runs on one shared harness, and integration and smoke tests no longer leave tmux sockets or servers behind.
This is part 2 of the test-suite cleanup and builds on `internal/testkit` from part 1.
No production behaviour changes; the one production move is the commit-status seam below.

Closes #326
Closes #327

## What changes

- **Scan harness.**
  `internal/testkit/scankit` (stdlib only) provides root lookup via `runtime.Caller`, the walk, an allowlist with stale-entry detection, and a vacuous-scan floor.
  Every `CONSTRAINTS.md` invariant scan, including each leaf and seam import guard, runs on it.
  No scan spawns `go env GOMOD` any more, so the Tier Purity and Hermetic Env allowlist entries for that exec are gone.
  Allowlists are one entry per line.
- **CLI tree.**
  `cmd/lyx/clitree_test.go` walks `newRoot()` and asserts `Short`, bare-group listing, `bogus` refusal with an `ok:false` envelope, and that `root.Long` names every module.
  It replaces the hand-kept help tree, `longlist_test.go`, `drift_test.go` and the per-CLI `NoArgs`/`UnknownSubcommand`/`Short`/exact-verb-set copies.
  `registration_test.go` stays as the independent mount oracle.
- **Config registry and paths.**
  `configreg` is asserted by property (sorted, unique, template-backed) plus a scan that every `ConfigTemplate` package is registered.
  `constructoranchoring_test.go` and `notransients_test.go` merge into one constructor table whose expectations come from the durable/transient/hub rule, with no path literals.
- **Recipe rows.**
  Rows are pinned once; routing columns became a perch-shape property.
  `sequence_test.go` and `coverage_guard_test.go` read that table.
- **Fixtures.**
  `plankit` writes valid plans and `stencilkit` seeds stencils through `stencils.Registry()` and `stencilstore`.
  `planparser.RecognizedFormat` is now exported so a format bump fails at one site.
  Raw stencil wording checks moved into one claims table, `contracts/stencils/claims_test.go`.
- **Single owners.**
  `shedrecipe` unknown-key rejection is one registry-wide test.
  `loomcli`'s duplicates of `shedverbs` tests are deleted, with unique cases moved to `shedverbs` first.
  The commit-status seam duplicated in `loomcli` and `battencli` now lives in `internal/statuscommit` and is tested once.
  Wide positional calls (`RenderIntegrationPrompt`, `StatusRel`) go through one local helper per test file.
- **tmux isolation.**
  `tmuxkit.Main` gives each test process a private `TMUX_TMPDIR`, unsets `TMUX` and `TMUX_PANE`, and kills only servers under its own directory at exit.
  `tmuxkit.Socket` is the per-test fixture for tests that start tmux directly.
  Every package with an `integration`- or `smoke`-tagged test file calls it from `TestMain`.

## Invariants

`CONSTRAINTS.md` changes land with their enforcement:

- New Tmux Test Isolation Invariant, enforced by `cmd/lyx/tmuxisolation_test.go` (allowlist starts empty, with stale detection).
- Testkit Invariant restated for shared test support, with `tmuxkit` exempt from the `os/exec` ban and no kit importing it.
- Test Tier Purity: every `tmuxkit` export except `Main` is barred from untagged files.
- Planparser Sole-Parser Invariant: `plankit` is the one test writer of valid plans; raw bytes only to test rejection.
- CLI / Cobra Invariant gains the enforcement line for the single walk.
- `docs/overview.md` and the kit package docs describe the new kits.

## Decisions to review

- **Weaker coverage by design.**
  A derived listing no longer notices a deleted verb or a module dropped from `Modules()`; behaviour tests and the registration scans cover those.
  `go test` of one CLI package no longer checks `Short`; `go test ./cmd/lyx` does.
  The perch-shape property pins `MaxBounces` and `Segment` only as equal within a Bouncer/Burler pair.
- **Reasoned exceptions.**
  Four group invocations differ from the CLI invariant today and sit as entries in `cliTreeFindings`: `board notes` bare, `board notes bogus`, `config` bare and `selfreport bogus`.
  Check that each reason holds.
- **Dropped claims.**
  Stencil claims with no cited provenance and generic wording were dropped; each is itemised in the commit bodies.
  Claims citing a finding, crucible round or sabotage proof were kept.
- **Stamped seeding.**
  Seeding through `stencilstore` stamps stencils, so one `websterengine` test now compares the stamp-stripped body.
- **tmuxkit over-approximation.**
  The isolation rule keys on "has a tagged test file", not "starts tmux", because the `lyx` binary and reed start servers indirectly.
- **Process note.**
  A test binary killed by a panic or timeout skips the sweep and leaves one `lyx`-prefixed temp directory.
- **Shared harness.**
  A harness bug would weaken every scan, so each converted scan was proved by a planted violation, and `scankit` has unit tests for the walk filter, stale detection and floor.
  Stale allowlist entries the tightening found were removed in the same change.

## To check by hand

- Seven `loomcli` smoke tests fail under tmux on this host.
  They failed identically on the tree before the change, but were not fixed here; confirm they are pre-existing.
- The final criterion was not checked by hand: after `go test -tags integration ./...` and the smoke tier, `/tmp/tmux-<uid>/` should have no new socket and no test-started tmux server should still run.
- Two batches touched scans outside their plan: `internal/loomrecipe/shape_test.go` (called a deleted seeder), and `cmd/lyx/hermeticenv_test.go`, `cmd/lyx/spawnobservability_test.go` and `internal/testkit/doc.go` (so `tmuxkit`'s `exec.Command` use passes those scans).
- Leaf test counts before and after are recorded in the run; confirm each drop matches a deletion named above.

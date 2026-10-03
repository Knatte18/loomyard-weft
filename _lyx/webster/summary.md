# Test-suite derived guards: all 34 batches done

Every batch reported `status: done`, ending at head `a780bb78b`.
The plan-level verify fork reported `status: OK` with no deviations.

## What was built

- **scankit (1–10):** the stdlib-only scan harness `internal/testkit/scankit`.
  The Testkit scan, the `cmd/lyx` guards, Tier Purity, Hermetic Env, the `lyxcwd` scans, every leaf and seam import guard and the package-local scans now run on it.
  Each converted scan was proved by a planted violation.
- **Derived registry tests (11–15):** one `newRoot` walk in `cmd/lyx/clitree_test.go` replaces the hand-kept help tree and the per-CLI NoArgs, UnknownSubcommand, Short and verb-set copies.
  The config registry is asserted by property plus a registration scan.
  The path guards are one rule table, and the loom recipe rows are pinned once with a perch-shape property.
  Four group invocations differ from the CLI invariant today and sit as reasoned entries in `cliTreeFindings`: `board notes` bare, `board notes bogus`, `config` bare and `selfreport bogus`.
- **plankit and stencilkit (16–22):** `planparser.RecognizedFormat` is exported.
  `plankit` and `stencilkit` are added, and the webster, loom, burler, treadle, orch, shedadapters, friction and pattern fixtures converted to them.
  Stamped seeding flipped one webster test, which now compares the stamp-stripped body.
- **Claims table (23–25):** raw stencil wording checks live in `contracts/stencils/claims_test.go`.
  Claims without cited provenance were dropped and itemised in the commit bodies.
- **Single-owner tests (26–29):** shedrecipe unknown-key rejection is tested once over the registry.
  `loomcli`'s duplicate `shedverbs` tests are deleted after moving unique cases.
  `internal/statuscommit` is the shared commit-status seam, and wide-call test helpers are routed through one local helper per file.
- **tmuxkit (30–34):** `tmuxkit.Main` and `tmuxkit.Socket`, adopted by every tagged-test package's `TestMain`.
  Hand-rolled reed contract socket keys moved to `tmuxkit.Socket`.
  The Tmux Test Isolation Invariant is enforced by `cmd/lyx/tmuxisolation_test.go`.

## Reported deviations

- Batch 22: `internal/loomrecipe/shape_test.go`, which called a deleted seeder.
- Batch 30: `cmd/lyx/hermeticenv_test.go`, `cmd/lyx/spawnobservability_test.go` and `internal/testkit/doc.go`, so `tmuxkit`'s `exec.Command` use passes those scans.

## Notes from fork reports

- Batches 22 and 29 reported seven `loomcli` smoke tests failing under tmux.
  The batch 29 fork re-ran those seven on the pre-card tree and they failed identically.
- Batch 28's fork reported rewriting `wiring.go` with an inline Python script, against the session's file-editing rules.
  It reported the resulting edit as correct.
- Batch 22's fork reported dropping a git stash entry by mistake and recovering it.

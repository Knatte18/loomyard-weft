# Share test fakes in internal/testkit and git helpers in gitkit

A small production change used to cost edits in many test files, because seam fakes, builders and git helpers were copied per consuming package.
This change moves them to one place each.
It carries the full scope of #324 and #325.
Production behaviour is unchanged, except that `internal/fabricengine` now exports its refusal sentinels; operator-facing error text is byte-identical.

Closes #324
Closes #325

## What changes

- **`internal/testkit/`** is new, one package per kit: `shuttlefake` (the `shuttleengine.Engine` and `ReedOps` fakes), `shedfake` (`Shuttle`, `BurlerRunner`, merge-shuttle, webster placeholder seams, producer `Call` helpers), `envkit` (builders for `shedrecipe.Env`, `landingshed.Deps` and the loom recipe `Deps`), `logcapture`, `envelope`, `lyxbin` and `locationkit`.
  Consumer packages lose their local copies.
  `internal/shuttleengine`, `shedrecipe` and `landingshed` keep one local copy each where an import cycle forbids the kit.
- **`gitkit`** gains the git query and commit helpers (`Git`, `RevParse`, `CurrentBranch`, `BranchExists`, `IsAncestor`, `RevListCount`, `LsFiles`, `ExcludeLines`, `CommitFile`, `CommitFileOnBranch`).
  The private copies in `fabricengine`, `gitrepo`, `fabriccli`, `cmd/lyx`, `hubforge`, `ideengine`, `loomcli`, `landingshed` and the webster pair are gone.
- **`hubforge`** is the only hub fixture vocabulary in `fabricengine` tests: the `fabricFixture` shim is deleted, and `AddPairWith` and `OpenFabric` are new.
- **`fabricengine`** exports `ErrInvalidSlug`, `ErrNoWarpBinding` and `ErrWarpBindingMismatch`; "which refusal" tests use `errors.Is`.
  Each remedy string is pinned in one engine test, and the CLI asserts once that the engine's message reaches the envelope unchanged.
- **Wiring tests**: `loomcli` and `battencli` each get one reflective test that walks `shedrecipe.Env` and fails on any nil seam not named in `intentionallyNil`.
- **Other consolidations**: one scripted `fakeTmux` in `reedengine`; one scenario table for the `reedcli` reap cycle; the gitrepo oracle moved to `internal/gitrepo/internal/gitoracle`; one shuttleengine Runner/Run fixture; one shedadapters bouncer fixture; `slices` replaces `containsPath` and `equalStringSlices`.
- Comments citing the old "duplicate test helpers rather than share them" policy are removed.
  At the plan-fixture sites left to `test-suite-derived-guards`, only the comment changes.

## Design decisions to review

- **Testkit Invariant** (new in `CONSTRAINTS.md`, enforced by an AST scan in `internal/testkit`): kits are imported only from `_test.go` files, import no `internal/*cli` package, and import none of `os/exec`, `gitexec`, `gitkit` or `hubforge`.
  `lyxbin` is the one exemption and may only run `go build` of `./cmd/lyx`; that part is review discipline.
  Kits sit outside the packages they fake so the per-package seam scans stay unchanged.
- **Tier Purity and Hermetic Git Test Environment** now share one rule: every `gitkit` export except `HermeticGitEnv` is a git spawn.
  Neither scan lists individual helpers any more, and `lyxbin.` joins the Tier Purity banned tokens.
  This tightens the scans: untagged tests that called `gitkit.MustRun`, `SeedConfig` or `GitStatusPorcelain` moved to `integration`-tagged files and no longer run under plain `go test ./...`.
- The gitkit Leaf and hubforge Fabric-Fixture sections gain review-discipline clauses; their import rules are unchanged.
- Protected tests (invariant scans, finding regressions, smoke/integration tests, the oracle, `livestate_*`) keep every assertion, except where D7 or the #325 duplicate deletions name another test as owner.
- `SkipPush` in `hubforge` suppresses only the weft push.
- `docs/overview.md` and the kit package docs are updated; no `manifest/designs/` doc, since `testkit` is not a CLI module.

## To check

- **Smoke tier was not run by the plan-level verify.**
  Forks reported 7 `go test -tags smoke ./internal/loomcli/...` failures; two were reproduced on the unchanged tree, so the rest are unconfirmed as pre-existing.
  A later fork reported 3 more smoke failures in `reedcli` and `orchcli`.
  Run the smoke tier for every package whose smoke files changed, and compare against `main`.
- **Test count**: the leaf count went from 9457 to 9498.
  Check that each drop is an intended deletion or consolidation (reflective wiring test, #325 duplicates, reap-cycle table).
- Deleting the `validateWorktreeSlug` call in `Add` should still fail a test.
- Planted-violation proofs for the Testkit, Tier Purity and Hermetic Env scans were part of the plan; re-plant one violation per rule if the scans matter to the review.
- Some batches touched files beyond their declared set: `fabricengine/sentinel.go` and `clone.go`, `reedengine/directory_test.go`, `websterengine/integration_test.go`, `landingshed/export_test.go` and `loomshed/gates_test.go` with several `shedadapters` and `shuttleengine` test files.
- At HEAD `69a7acbf2`, `go build ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./...` passed.

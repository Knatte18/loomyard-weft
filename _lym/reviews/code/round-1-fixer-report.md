---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Both refusals sit in markedScope, not one of them in runTest after planTierRuns, so one place decides why a marked key cannot run and the refusal comes before planning. The upkeep stencil also says what the agent files for a refused call, since without that every go hub with no marked test would get a bug note at each tidy."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

# Fixer report, round 1

## Changes

- F1 (fixed differently): `markedScope` refuses a marked test of a tier no listed tier is or covers, naming the tests and `gate.gates.upkeep`.
  It also refuses a module with no test marked for the key, and a scanned directory with no `go.mod`.
  Both rows are in `TestGateTest_Scopes`.
  The upkeep stencil files one `bug` note per refused call, except the "no test is marked" refusal.
  The template comment, `gatecli/doc.go`, `PATTERN-test-speed` and the change description follow.
- F2: `gate.gates` and `gate.gates.upkeep` are written in full in `gatecli/doc.go`, `template.yaml` and the change description.
- F3: `scopeInvocations` plans nothing for a marked scope; a `--no-carry` row is in `TestPlanTierRuns`.
- F4: the prime renders no upkeep call under a scoper other than `go`.
  The no-upkeep fragment, the board-tidy stencil and the comments no longer say that only a beam gets that step.
- F5: both scans carry `//lym:guard` and `//lym:only parent-align finalize`, the finding's second option, so no pattern text changes.
- F6: the change description calls the door one-way and names both effects with their recoveries.
- F7: tests for the held guard, the `held` field of a recorded and of an inert-carried green entry, `UpkeepCall`'s fallback and its prime-only wiring.
- F8: tests for `skippedInside` over a covered tier, `greenRecord.with` over two held sets, an unlistable module, the `none` scoper, `-C`, and explicit targets on a marked key.
- F9: `withOnlyTests` holds nothing back when the words after `--` carry a filter flag.
- F10: `ModuleOnlyTests` takes a context, and `runTest` checks the pre-tier ceiling after the marker scan.
- F11: the `dobbyengine` doc bullet is corrected and the timeout line is back under the spawn bullet.

Commits: `a1e8da43f` (F1-F4, F7-F11), `bc4cd85fe` (F5), `619d9ffb5` (self-check) and `b83ef7ad5` (comment breaks).
F6 changes only the run's own record under `_lym`, so it is in no commit.

After this report was first written, the round's gate found two wrapped comment lines, in `internal/gatecli/only.go` and `internal/gatecli/run.go`.
`b83ef7ad5` breaks both at sentence ends and changes no code.

`lym gate test --gate code-burler-fix` passed over tier0, tier1, tier2 and the guard step at `619d9ffb5`.
On `b83ef7ad5` only `go vet ./internal/gatecli` was run, and it passed.

This hub's gate config sets no `gates.upkeep`.
After landing, the upkeep call is therefore refused until `gate.gates.upkeep` is set to `{tiers: [0, 1], scope: marked}` from the prime.
That config is outside this round's write surface.

## Self-check

- fix-code-correctness, MEDIUM (the unlistable-module row passed with the defect in place): fixed as proposed.
- fix-code-correctness, LOW (a held test of the same name drops a marked one): fixed as proposed; a marked scope skips `withoutHeld`, with a regression assertion, and the comment's claim is narrowed.
- fix-consistency, MEDIUM (the no-test refusal names a config step without the config route): fixed with an own change; only the no-`go.mod` case is a config refusal, and the unmarked case names marking a test alone.
- fix-consistency, LOW (a hub with no tier exits 0): fixed with an own change; `gatecli/doc.go` limits the statement to a hub that declares tiers.
- fix-consistency, LOW (template lines on the `go` scoper and on covered tiers): fixed as proposed.
- fix-proposal, MEDIUM (bare-name drop in a marked scope): fixed as proposed, the same change as the correctness LOW.
- fix-proposal, LOW (the no-test refusal's way forward): fixed as proposed, the same change as the consistency MEDIUM.
- fix-affected-set, LOW (the template's upkeep line lacks the scoper condition): fixed as proposed.

## Disputed

None.

## Deferred

None.

## Rulings

None.

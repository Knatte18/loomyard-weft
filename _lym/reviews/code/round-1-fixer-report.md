---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "A killed package is told by go test's own `signal: killed` line, not by the directory of the killed binary. A test's chdir moves the binary's working directory, and an invocation directory without a go.mod maps no package. The record.Failed == 0 check is as proposed."
  - id: F3
    disposition: fixed-differently
    reason: "verifytree got a second field, MemoryPressure, and Result.Unjudged covers both, instead of one renamed flag. The messages differ by cause, so each site needs to know which one it has."
  - id: F4
    disposition: fixed-differently
    reason: "The closing line moved into unjudgedRuns and is pinned by a table test with stopsCall, instead of integration rows. The integration rows run the gate as its own process, which has no seam to force a memory guard pressure kill."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed-differently
    reason: "Every attempt stopped for slice pressure is marked, including a rerun that follows a shortage or a crash and a first run of a cancelled call. The proposed two sites left those unmarked. The field has a json tag only, as RunRecord has no yaml tags."
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed-differently
    reason: "The proposed `ownCgroup + \"x\"` query passes with the trailing slash dropped, since the own cgroup does not start with it. The test queries the own cgroup path less its last character, which the own path does start with."
---

## What changed

- `457101cab` (F1, F2, F4, F6, F8, F9):
  - The memory guard sums the gate call's whole process tree for the own share and excludes it from the named holders.
    A call that cannot be sampled gets an ordinary pressure kill.
  - A failed package that no kill explains keeps a run judged.
  - Run records carry `memory_pressure`.
  - New tests cover the closing line, `--stop-on-fail`, the mixed stops and the unreadable holder name.
- `c67e0187f` (F3): `verifytree` reads the memory pressure line, and the loom, red-first, landing, weaver, webster and `lym webster verify` sites stop on `Result.Unjudged`.
  The dotnet red-first judge reads the line too.
  The landing fixer end `crash` is now `unjudged`.
- `2653bd903` (F5): `TestMergeCopies` rows for the slice-pressure fold.
- `15a9cfe7e` (F10): the root and shared-prefix cases of `ResidentInCgroup`.
- `89583786f` (F2, F8, from the self-check): the killed package is told by its `signal: killed` line, pinned over a real `go test` by `TestGoJSONCounter_MarksTheFailureOfAKilledTestBinary`.
  Every pressured attempt is marked, and a stale comment and a test row name are corrected.
- `a7e2a8ac8` (gate findings): the comment lines this round added that wrapped mid-sentence or reached 120 characters are broken at sentence boundaries or rewritten.
  Only comments changed.
  The gate had stopped on these lines before running any test.
- `_lym/landing/summary.md` (not committed):
  - F7: the `cmd/lym` test binary running `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` alone peaked at 2011532 KB resident, one run.
    That is above the 1024 MB pressure kill floor and below the 4096 MB cap.
  - The verify-site bound is removed (F3), and the bound on memory that no resident size shows is added (F1).
  - The Tests and What changes sections follow the code.

Gate runs: tiers 0 and 1 of every changed Go package passed, `internal/proc` at tier 0 only.
After `a7e2a8ac8`, tier 0 of `gatecli`, `gateslot`, `verifytree` and `websterengine` passed.
Tier 2 and the packages the change does not touch were not run.

## Self-check

- fix-affected-set, LOW, `PATTERN-gate-slot.md` names only the runner crash as getting no fixer: deferred.
- fix-consistency, LOW, `noTestRan` field comment omits memory pressure: fixed as proposed.
- fix-consistency, LOW, the summary's Tests section is stale: fixed as proposed.
- fix-consistency, LOW, a `fixerrun_test.go` row name still says crash end: fixed as proposed.
- fix-code-correctness, LOW, a changed cwd or a run directory without go.mod loses the rerun: fixed.
- fix-proposal, MEDIUM, a test's chdir loses the rerun: fixed, with the `signal: killed` option it named.
- fix-proposal, LOW, a run directory below the module root loses the rerun: fixed by the same change.
- fix-proposal, LOW, some pressured attempts stay unmarked: fixed.

## Disputed

None.

## Deferred

- `pattern/PATTERN-gate-slot.md:154` (self-check, fix-affected-set) says "a runner crash gets none" of the cold fixer, and a memory pressure stop now gets none either.
  The sentence stays true but incomplete.
  The review names no PATTERN change and I know of no declared one for this run, so the parent decides whether to amend it.
- `internal/gatecli/cli.go` comments on the rerun delay and `--stop-on-fail` do not mention the memory pressure case.
  The fix-consistency fork noted this as outside its scope, since the fix did not touch that file.

## Rulings

None.

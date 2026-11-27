```yaml
lens: code-scope
outcome: reported
```

**Decisions** (decision record alone; no `Added after Discussion` entries in the record)
- G1 — The guard lets a darn run with profile `docs` create and remove bench pairs itself.
- G2 — Only `lym fabric add` and the removal verb, run from the task's own worktree, so each pair forks from the task branch.
- G3 — Only pairs whose slug starts with the task's slug, such as `<slug>-b1`, so a bench never touches another task's pair.
- G4 — Worktree-Teardown removes every bench pair the run left behind.
- G5 — The bench body's cap on runs bounds the count; whether lym also caps it is open.

**Per change**
- C1 `internal/fabriccli/roleguard.go` — `benchPairVerbs`, `benchPairVerb`, the `sandboxedRoleRefusal` admission and way-forward clause, the `pairLocation`/`ownPairLocation`/`strandHubWorktree` refactor → G1, G2, G3 (way-forward clause forced by PATTERN-refusal-way-forward).
- C2 `internal/fabriccli/roleguard.go` — `isBenchPairPath` in `injectedPathsAreOwn` → forced by G1: the detached push `add` spawns (`fabric.go` `runAdd` → `spawnPairPush`) would otherwise be refused.
- C3 `internal/darnengine/bench.go` — `IsBenchPairSlug`, `BenchPairSlugInfix` → G3; `IsWriter` → G1. `internal/darnengine/mergetask.go` `IsMergeTaskWriter` reuse → forced by C3 (no behaviour change).
- C4 `internal/runprofile/runprofile.go` `DocsProfile` → G1 (profile name as one named constant per PATTERN-code-conventions); `internal/retroengine/retro.go` `TaskProfile = runprofile.DocsProfile` → forced by the same rule once the constant exists.
- C5 `internal/shedrun/usage.go` `ReadRunUsageRecord` → G1 (reading the profile without the board, per PATTERN-fabric-boundary); header reflow forced by the comment lint.
- C6 `internal/pairteardown/leftover.go` `RemoveBenchPairs`, `runNamingReason` extraction from `requireStillOrphan` → G4.
- C7 `internal/battencli/wire.go` `Teardown.Remove` → G4.
- C8 doc comments in `fabriccli/doc.go`, `battenshed/doc.go`, `darnengine/doc.go`, `pairteardown/doc.go` → forced by C1/C3/C6.
- C9 tests: `bench_test.go` → forced by C3; `roleguard_test.go` subtest → forced by C1/C2; `TestRemoveBenchPairs` → forced by C6.
- C10 commit 3a3fdc1 (comment-line re-breaks) → forced by the comment lint.

**Per decision**
- G1 → C1, C3, C4, C5 (C2 forced).
- G2 → C1.
- G3 → C1, C3 (strict `<slug>-b<N>`, narrower than "starts with"); guard side only partly built (see F1).
- G4 → C6, C7.
- G5 → none; the record leaves the cap open and none is added — no finding.

**Findings**

1. F1 — LOW · scope · `internal/fabriccli/roleguard.go:272` (`benchPairVerb`)
   Summary: G3's bound "a bench never touches another task's pair" is not enforced by the guard: `add`/`remove` of a `<slug>-b<N>` slug that another task's board entry names is admitted; only the teardown checks (`internal/pairteardown/leftover.go` `runNamingReason`). Commit 53987e2 states the reason (fabric learns nothing of the board), but no `Pattern-Departure:` trailer names PATTERN-fabric-boundary for this difference from the answer key.
   Evidence: `benchPairVerb` body (no board or batten-run-dir read); commit 53987e2 message (no trailer).

2. F2 — LOW · scope · `internal/fabriccli/roleguard.go:233-240` (`isBenchPairPath`)
   Summary: the push admission requires only a darn writer and the bench slug form — it reads no run record, so a darn writer on any profile (not just `docs`) may push a pair named `<slug>-b<N>`; G1 bounds the new access to a darn run with profile `docs`.
   Evidence: `isBenchPairPath` checks `darnengine.IsWriter` and `IsBenchPairSlug` only; contrast `benchPairVerb` lines 295-296 which check `run.Recipe`/`run.Profile`.

Tool calls: none — the inherited context held everything this lens needed.

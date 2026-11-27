---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: "The fixer's reason holds. Config.Gate refuses a hub that declares tiers but no tier0 (internal/gateslot/tiers.go:392-393), and runRedFirst calls TierList before goModuleOf (internal/gatecli/redfirst.go:165-177), so the neither-tier0-nor-tier1 refusal cannot be reached through red-first. TestGateRedFirst_RefusesWhatItCannotDiffOrBuild covers every reachable refusal, way forward included. It passed at tier 1 through lym gate test."
  - id: F2
    status: resolved
    answer: "_lym/landing/summary.md under Merge danger now names the effect on older binaries: an older binary fills a missing reports list from its own template, and the summary gives the explicit reports: [] or leave-alone step. The two-way call stands under contracts/specs/loom-final-summary-spec.md, since gate.yaml is config and not a record or board format, and no hubreconcile reinterpretation ships."
  - id: F3
    status: resolved
    answer: "_lym/landing/summary.md under Hub config names the hand deletion of the board directory's .lym/testcover and each worktree's own .lym/testcover."
  - id: F4
    status: resolved
    answer: "pattern/PATTERN-test-speed.md:222-223 now matches cmd/testtiming/resources.go:122-174, which holds one slot through gateslot.HolderEnv in every mode, and main.go:215-250, which sets LYM_TEST_TIER from its tags."
  - id: F5
    status: resolved
    answer: "internal/gateslot/doc.go:46 lists the cardverify run, the drift command, the plan lint's type load and cmd/testtiming. These are exactly the production callers of gateslot.HolderEnv at HEAD."
  - id: F6
    status: resolved
    answer: "The re-run mark clause is gone from internal/gatefixer/rowgate.go:205, and no remaining comment in the package names the mark."
  - id: F7
    status: resolved
    answer: "TestLoadConfig gains the row retired report-step keys still load (internal/gateslot/config_test.go:276), which expects the baseKeys config and no error."
---

Every round-1 finding is resolved, and this incremental round raises no new BLOCKING finding on a changed hunk.
`go vet` passes on every touched package.
`TestGateRedFirst_RefusesWhatItCannotDiffOrBuild` passes at tier 1 through `lym gate test`.

## Focus verdicts

- **Red-first refusals.**
  The no-parent-branch, `--base` naming no commit and tier-dir-without-`go.mod` refusals are each asserted with exit 1 and their way forward.
  No hub can reach the neither-tier0-nor-tier1 refusal through red-first: `Config.Gate` refuses a tiered hub without tier0 before `goModuleOf` runs.
  Not a defect in the test.
- **Deleted tests.**
  I checked every deleted test of behaviour that survives the diff, and each surviving behaviour is still asserted:
  - covering-test ordering, by `TestJudge`'s third row;
  - the gate-entry template fill, by `TestLoadConfig`'s row where a tuned key keeps the template's lints;
  - the batch gate entry names, by `TestRun` (`internal/weaverengine/run_test.go:93`);
  - the writer-only re-run reply, by the kept `TestWrapRowGate` subtest.
  The other deletions pinned removed code only: the build timeout, `LinkedPackages`, the carve-outs, the report markers and the re-run mark.
- **Summary, Merge danger and Hub config.**
  The door call, the effect on older binaries and the named steps match the code.
  Reconcile carries a per-name `gates` entry whole (`internal/yamlengine/doc.go:72`), so on this hub it drops nothing.
  The `reports: []` lists stay until an entry is rewritten, as the summary says.
- **Prose.**
  The statements in `PATTERN-test-speed.md`, `PATTERN-gate-slot.md`, `internal/gateslot/doc.go` and `internal/gatefixer/rowgate.go` agree with HEAD.
- **Retired keys.**
  A hub `gate.yaml` that sets a retired top-level key loads: `warnUnknownKeys` warns, and nothing refuses it.
  Row `retired report-step keys still load` pins this.

## Rejected

- `goModuleOf`'s tier1 fallback and its neither-tier0-nor-tier1 error (`internal/gatecli/gomodule.go:136-149`) are unreachable now that red-first is the only caller.
  This is below BLOCKING and sits outside this round's changed hunks, so it is not raised.
- `internal/weaverengine/export_test.go:15` still exports `BatchGateSpec`, though the deleted reach test was its only caller.
  This is below BLOCKING and outside the changed files, so it is not raised.
- The summary's Hub config names only `.lym/testcover` as leftover data.
  Each worktree's `.lym/gate` also keeps `reach.json`, `redundancy.json` and `mutation.json` with their locks, and `.lym/verify` keeps the reach-listed files.
  These are never-tracked caches that nothing reads.
  This is below BLOCKING and not in the changes file, so it is not raised.

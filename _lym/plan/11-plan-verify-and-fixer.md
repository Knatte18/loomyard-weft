# Card 11 — plan-verify-and-fixer

**Create:**
- `plan:internal/weaverengine#RunDeps` -> `type RunDeps struct`
- `plan:internal/weaverengine#verifyOutcome` -> `type verifyOutcome struct`
- `plan:internal/weaverengine#runVerify` -> `func runVerify(deps RunDeps, st *State, evaluation int) (verifyOutcome, error)`
- `plan:internal/weaverengine#fixerStore` -> `type fixerStore struct`
- `plan:internal/weaverengine#fixerStore.Live` -> `func (s *fixerStore) Live() (*gatefixer.Record, error)`
- `plan:internal/weaverengine#fixerStore.Save` -> `func (s *fixerStore) Save(rec *gatefixer.Record) error`
- `plan:internal/weaverengine#fixerSpec` -> `func fixerSpec(deps RunDeps, st *State, fixer *FixerState, findings string) (gatefixer.Spec, error)`
- `plan:internal/weaverengine#writeSummary` -> `func writeSummary(deps RunDeps, st *State, flaky []string) error`
- `plan:internal/weaverengine#TestRunVerify` -> `func TestRunVerify(t *testing.T)`
- `plan:internal/weaverengine#TestWriteSummary` -> `func TestWriteSummary(t *testing.T)`

**Edit:**
- `internal/gatefixer#RoleVerifyFixer`
- `internal/gatefixer/doc.go`

**Extends:**
- `internal/weaverengine/doc.go`

**Uses:**
- `internal/runplan#VerifyGateReport`
- `internal/runplan#WriteVerifyGateReport`
- `internal/runplan#RenderVerifyGateFindings`
- `internal/runplan#ParseVerifyRun`
- `internal/runplan#CardHint`
- `internal/gatefixer#SpawnOrAttach`
- `internal/gatefixer#Await`
- `internal/gatefixer#RenderPrompt`

**Intent:** The plan-level verify, its fixer and the run record (D10; PATTERN-gate-slot).

- `RunDeps` is everything the row's producer is told, declared here for its first user and read by card 12's `Run`:
  `Geom`; `Config`; `Ladder`, the resolved rung choices (card 2); `Batcher`; `Gate`, the row's `gates` entries;
  `Runner StrandRunner`; `Fixer gatefixer.Fixer`; `Reed shuttleengine.ReedOps`;
  `Verify`, the verify seam; `CommitRecords func(msg string) error`; `WeftHead func() (string, error)`;
  `ParentBranch`, which names the parent branch a clean merge-in comes from (card 10);
  `Deliver func(strandName string, delivered []string) ([]string, error)`, which sends a strand the decision entries recorded since it began whose headings `delivered` lacks and returns the headings it sent (card 21 fills it; nil sends nothing);
  `LanguageSkills`, `FrictionDir`, `ReentryStep` and `Clock`.
  A nil `Verify` or `Clock` takes the production value and a nil `Deliver` or `ParentBranch` is legal; every other seam is required, and `Run` refuses a nil one as a wiring error naming the field.
- `runVerify` renders `verify_gate`'s gate call (`GateCall(cfg.VerifyGate)`), runs it through `verifytree.Verify` in held gate slots with the geometry's pause hold, through the told `RunDeps.Verify` seam (`verifytree.Verify` in production), and reruns a failure once: a pass on the rerun is a pass whose failing identities are flaky.
  A timeout or a hang is no rerun.
  A runner crash (`RunnerCrash`) is its own outcome, which starts no fixer and stops the row.
  A dirty tree (`StatusDirty`) is a failed evaluation whose findings are the uncommitted paths.
  A failure writes the verify-failure record (`runplan.WriteVerifyGateReport`, the card hint from the batches' card trails) and returns its findings rendered by `runplan.RenderVerifyGateFindings`.
- `fixerStore` adapts `State.Fixers` to `gatefixer.Store`: `Live` is the newest fixer whose `Record` is set and not terminal, and `Save` sets the record on the fixer state the number names.
- `fixerSpec` renders the fixer's prompt through `gatefixer.RenderPrompt` with `weaver-body-verify-fix`, the findings and the journal path, the reply path `FixerReplyPath`, the role `gatefixer.RoleVerifyFixer`, the rung's choice, and `FixerTimeoutMin` as its timeout.
  It carries no shuttle gates: Go's re-run of the verify judges the fixer (Added after Discussion, Q18).
- `writeSummary` writes `summary.md` in the shared final-summary format: a `# ` title naming the task, the batches with their outcomes, `## Integration suite triage` listing the flaky identities, and `## Verify gate fixes` listing each fixer's findings and its `fix:` commits.
- `gatefixer.RoleVerifyFixer`'s comment and `gatefixer/doc.go`'s Roles section say the role is the cold fixer of an implementer's plan-level verify, Webster's or Weaver's.

`TestRunVerify` is tier0, over a fake verify seam: a pass; a fail then pass on rerun is a pass with flaky identities; two fails write the record and return its findings; a dirty tree returns the uncommitted paths as findings; a runner crash returns the crash outcome and writes no record; a timeout is not rerun.
`TestWriteSummary` is tier0: the title, both sections, and no section when there is nothing to list.

**ImpactSummary:** Rewords one gatefixer role comment and its package doc; no gatefixer behaviour changes.

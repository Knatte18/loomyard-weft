# Card 11 — plan-verify-and-fixer

**Create:**
- `internal/weaverengine#RunDeps`
- `internal/weaverengine#verifyOutcome`
- `internal/weaverengine#runVerify`
- `internal/weaverengine#fixerStore`
- `internal/weaverengine#fixerStore.Live`
- `internal/weaverengine#fixerStore.Save`
- `internal/weaverengine#fixerSpec`
- `internal/weaverengine#writeSummary`

**Edit:**
- `internal/gatefixer#RoleVerifyFixer`
- `internal/gatefixer/doc.go`
- `pattern/PATTERN-gate-slot.md`
- `pattern/PATTERN-test-speed.md`
- `internal/gateslot/doc.go`
- `cmd/lym#planVerifySites`

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
  A nil `Verify` or `Clock` takes the production value and a nil `Deliver` or `ParentBranch` is legal;
  `runVerify`'s file declares `Verify`'s production value, `verifytree.Verify` called with the geometry's gate slots and its pause hold (`Hold:`), which `Run` takes for a nil `Verify`; every other seam is required, and `Run` refuses a nil one as a wiring error naming the field.
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
- The verify-site docs and tripwire name Weaver's plan-level verify:
  `PATTERN-gate-slot.md`'s Verified-tree site list names it;
  `PATTERN-test-speed.md`'s Gate keys table names Weaver's plan-level verify beside Webster's end gate on the `webster` row;
  `gateslot/doc.go`'s tier plan gains Weaver's end-gate row;
  `planVerifySites` gains `runVerify`'s file, which holds both the `verifytree.Verify` call and the `Hold` the tripwire looks for.
  The site is listed here, not in card 9, because the tripwire fails on a listed file it does not find.

These unexported helpers are tested through `Run`: card 12's `TestRun` holds their rows.

**ImpactSummary:** Rewords one gatefixer role comment and its package doc, names Weaver's plan-level verify in two pattern docs and gateslot's tier plan, and adds one plan-verify site to the tripwire's list; no gatefixer behaviour changes.

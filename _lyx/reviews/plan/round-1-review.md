---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/03-pointer-launch.md
    summary: card 3's pointer launch breaks internal/loomcli/integration_driverbootstrap_test.go, whose stub reads the prompt from $1, and no card targets that test
  - id: F2
    severity: LOW
    location: _lyx/plan/03-pointer-launch.md
    summary: card 3 drops the prompt-size refusal without stating what now bounds prompt size, and leaves driverprompt.go's cap comment and its cap test asserting the retired bound
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/16-webster-verify-verb.md
    summary: a call-site source scan is not the verdict-parity check the Gate Self-Check Parity Invariant requires for a new gate; neither gate closure is reachable by one
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/14-webster-verify-gate.md
    summary: card 14 bundles three independently testable units (D4 report and card hint, D3 gate, D5 fixer side) into one card
  - id: F5
    severity: MEDIUM
    location: _lyx/plan/15-retire-integration-stage.md
    summary: card 15 deletes accumulatedCardSHAs, the only SHA-to-NN-slug trail, which card 14's card hint needs
  - id: F6
    severity: MEDIUM
    location: _lyx/plan/14-webster-verify-gate.md
    summary: the pre-fix head is recorded only after a verify failure survives the rerun, so fixer commits after a dirty-tree failure escape D5's commit check
  - id: F7
    severity: LOW
    location: _lyx/plan/14-webster-verify-gate.md
    summary: loomcli's new RunDeps.ParentBranch wiring has no test
  - id: F8
    severity: LOW
    location: _lyx/plan/14-webster-verify-gate.md
    summary: card 14 does not settle the attach-path note for a resumed Merriam, nor a Webster row naming verify beside Run's own entry; websterEntry's doc goes stale
  - id: F9
    severity: LOW
    location: _lyx/plan/13-plan-write-reworded-messages.md
    summary: ImpactSummaries of cards 13, 15 and 16 list what the card changes instead of concluding who is affected
  - id: F10
    severity: LOW
    location: _lyx/plan/17-docs-and-specs.md
    summary: card 17 leaves final-summary-spec.md describing bisect's summary sections and does not add lyx webster verify to the overview's webster verbs
  - id: F11
    severity: NIT
    location: _lyx/plan/03-pointer-launch.md
    summary: card 3 leaves the Shell.ReadFile interface method with no caller
---

### [BLOCKING] Card 3 breaks the driver-bootstrap integration test

**Location:** `_lyx/plan/03-pointer-launch.md`; `internal/loomcli/integration_driverbootstrap_test.go:58`.

**Issue:** `integrationWriteStubDriverScript` stands in for the claude binary and greps the drive-report path out of `"$1"`, which today is the expanded prompt.
After card 3, `$1` is the pointer line, which holds no drive-report path, so the stub writes no report and `waitForDriveReport` fatals.
The test is integration-tagged in `internal/loomcli`; card 3's own gate (D1) runs only its own packages' tagged tests, so the break surfaces at the plan-level gate.
The decision record's open risk names exactly this ("a test or tool reading the first message for content would break"), and no card takes the test into scope.

**Fix:** add `internal/loomcli/integration_driverbootstrap_test.go` to card 3's Edit targets: the stub extracts the absolute `prompt.md` path from `$1` and greps the drive-report path from that file's content.

### [LOW] Card 3's removed guard has no stated bound, and its stale reach is left behind

**Location:** `_lyx/plan/03-pointer-launch.md`; `internal/loomcli/driverprompt.go:28`, `internal/loomcli/driverprompt_test.go:122`.

**Issue:** card 3 drops the 30000-byte refusal on `spec.Prompt` for every role.
Its Intent says a prompt of any size launches, but not what the guard refused before (any prompt that would overflow the Windows command line) or what bounds prompt size now (nothing at launch; only the agent's own read of `prompt.md`).
`driverPrompt`'s doc comment says the engine caps the prompt and a long prompt would fail at launch, and `TestDriverPrompt_StaysWellUnderLaunchPromptCap` justifies its bound by that cap; both become false.

**Fix:** state the before/after and the bound in card 3's Intent, and add `driverprompt.go` and `driverprompt_test.go` to its targets so the comment and the test's rationale stop citing the launch cap.

### [MEDIUM] Card 16's source scan is not a parity check

**Location:** `_lyx/plan/16-webster-verify-verb.md`; `CONSTRAINTS.md` Gate Self-Check Parity Invariant; `internal/loomcli/parity_test.go`.

**Issue:** D15 asks for "the parity test", and the invariant says "Adding a mechanical gate means adding its verb and its parity check in the same task".
The repo's parity checks (`TestGateParity_*`) drive the gate closure and the verb over one fixture and compare a three-way verdict.
Card 16's `cmd/lyx/verifiedtree_test.go` only greps each file for a `verifytree.Verify` call: it passes when the call sits in a helper the closure never reaches, and when the verb and a gate map the same `Result` to different verdicts (a dirty tree passing one side, a missing `## verify:` failing the other).
The scan does carry D16's tripwire (no other `verifyrun.Run` caller, no `verify-pending`).
Neither closure can be driven by a parity test as planned: card 8 builds the Webster-Burler closure inline in `shedrecipe`, unlike every other plain gate (`loomshed.NewPlanGate`, `landingshed.NewDescriptionGate`), and card 14's closure is unexported.

**Fix:** card 8 declares the Webster-Burler closure as an exported constructor beside `NewPlanGate` (`loomshed.NewVerifyGate`), which `resolvePlainGateClosure`'s `verify` case returns;
the webster gate's card exports its constructor;
card 16 adds an integration-tagged verdict-parity test that drives the verb and both closures over one scratch-repo fixture (passing command, failing command, dirty tree, no `## verify:`) and compares three-way verdicts.
The source scan stays as D16's tripwire.

### [MEDIUM] Card 14 bundles three independently testable units

**Location:** `_lyx/plan/14-webster-verify-gate.md`.

**Issue:** card 14 carries D3, D4 and D5 at once: the gate closure and Run's outcome mapping, the findings text with its card hint and the `verify-gate.yaml` writer, and the fixer side (new stencil, Merriam's gate-failure section, the `forkOwnReport` remap, record-batch's audit skip).
The card hint and report are a pure computation over recorded SHAs, changed paths and failing packages, testable with no gate.
The fixer side is testable through `recordbatch_test.go`, the fork-audit test and the template tests, given only `VerifyGateReportPath`.
Each is reviewable alone; the gate itself is inseparable from Run's outcome mapping and the loomcli `ParentBranch` wiring its commit check needs.
`verify_gate_attempts` in card 10 is not a defect: card 10 already edits the config, template and config test for `poll_wait_s`, and the key is inert until the gate reads it.

**Fix:** split card 14 into three cards in order: the verify-gate report and card hint (D4), the gate on Merriam's strand (D3, the commit check, the outcome mapping, the loomcli wiring), and the fixer fork (D5), renumbering the later cards and their references.

### [MEDIUM] Card 15 deletes the trail card 14's card hint reads

**Location:** `_lyx/plan/15-retire-integration-stage.md`; `internal/websterengine/runlevel.go:1647`.

**Issue:** D4's hint names cards "by `NN-slug`" whose recorded `CardSHAs` commits touched a failing package.
The one function pairing each recorded SHA with its `NN-slug` label is `accumulatedCardSHAs`, which card 15 deletes "outright".
Card 14 does not say what builds the trail, so it either calls `accumulatedCardSHAs` (and card 15 breaks the build) or reimplements it (and the code duplicates).

**Fix:** the card-hint card moves `accumulatedCardSHAs` beside the hint and uses it, with `commitChangedPaths` for each SHA's paths; card 15 drops it from its deletion list.

### [MEDIUM] Fixer commits after a dirty-tree failure escape the commit check

**Location:** `_lyx/plan/14-webster-verify-gate.md`, steps 3 and 6.

**Issue:** D5: "Go records the HEAD at the gate's first failure as the pre-fix head."
Card 14 records it only in step 6, after a verify failure survives the rerun, and runs the check "from the second arrival on".
A first arrival that fails the clean-tree check records no pre-fix head, Merriam forks a fixer on those findings, and every commit until the next verify failure, merges included, is never checked.

**Fix:** record the HEAD as the pre-fix head at the first failed evaluation of any kind (dirty tree or surviving verify failure), and run the commit check at every arrival once a pre-fix head is recorded.

### [LOW] loomcli's ParentBranch wiring has no test

**Location:** `_lyx/plan/14-webster-verify-gate.md`; `internal/loomcli/wiring.go`.

**Issue:** card 14 fills `RunDeps.ParentBranch` in loomcli so a clean parent merge passes the commit check under loom, but no loomcli test pins it; a nil there silently rejects every parent merge during a fix.

**Fix:** add `internal/loomcli/wiring_test.go` to the gate card with a case asserting the wired `RunDeps.ParentBranch` is non-nil and returns the pair's parent branch.

### [LOW] The resumed-Merriam note and the Webster row's gate vocabulary are unsettled

**Location:** `_lyx/plan/14-webster-verify-gate.md`; `internal/shedrecipe/entries_simple.go:142`.

**Issue:** the decision record asks the plan writer to check that a resumed Merriam carries the same gate spec; card 14 is silent.
Merriam has no attach path (`reclaimEntryTimeStrands` stops the old strand and `Run` starts a fresh one through its one `StartMaster` call), which the card should say.
Once card 8 adds `verify` to the closed vocabulary, a Webster row naming it would put a second `verify` entry beside the one `Run` appends, and `websterEntry`'s doc ("no Webster validator exists to name") goes stale.

**Fix:** card 14 states that `StartMaster` is Merriam's only start, so every start carries the spec; `Run` refuses a `deps.Gate` already holding `verify`; `entries_simple.go` joins the card to update `websterEntry`'s doc.

### [LOW] Three ImpactSummaries restate the change

**Location:** `_lyx/plan/13-plan-write-reworded-messages.md`, `_lyx/plan/15-retire-integration-stage.md`, `_lyx/plan/16-webster-verify-verb.md`.

**Issue:** card 13's "gains one card rule and one claim pin", card 15's list of what each package loses and card 16's "gains one verb … one source-scan guard" describe the edit, not its reach.
Cards 4 and 7 conclude ("no existing caller changes", "no consumer reads it yet") and pass.

**Fix:** card 13: no code consumer; only future Plan-Write plans change, gaining test targets. Card 15: no surviving production caller outside the edited packages; an in-flight run with a fix record or `integration.yaml` needs `--fresh`. Card 16: additive verb with no existing caller; the source scan fails any future plan-verify site that bypasses `verifytree.Verify`.

### [LOW] Card 17 misses two stale or missing doc mentions

**Location:** `_lyx/plan/17-docs-and-specs.md`; `contracts/specs/final-summary-spec.md:47`; `docs/overview.md:339-344`.

**Issue:** `final-summary-spec.md` says webster appends "the integration-failure section webster's own bisect appends" and the integration-triage section; after the plan, bisect is gone and the summary gains "Verify gate fixes".
`docs/overview.md`'s webster entry names its verbs but card 17 does not add `lyx webster verify`.

**Fix:** add `contracts/specs/final-summary-spec.md` to card 17 with the gate's summary sections, and name the new verb in the overview bullet.

### [NIT] Shell.ReadFile loses its only caller

**Location:** `_lyx/plan/03-pointer-launch.md`; `internal/shell/shell.go:19`.

**Issue:** `buildLaunchCmd` is the only caller of the `Shell.ReadFile` interface method; `pwshShell` uses its own method internally.
After card 3 the interface method and `posixShell.ReadFile` are dead.

**Fix:** card 3 drops `ReadFile` from the `Shell` interface and `posix.go`, keeps pwsh's as an unexported helper, and updates `shell_test.go` and `command.go`'s header comment.

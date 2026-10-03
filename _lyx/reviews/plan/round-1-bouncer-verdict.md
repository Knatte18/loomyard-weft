---
verdict: BLOCKING
rationale: "All eleven round-1 findings are fixed in the 19-card plan, but card 15's fixer-commit check must call parentMergeRejection, which takes a *gitrepo.Repo that only gitrepo.New builds; cmd/lyx/rawgitmutation_test.go bans gitrepo.New( in every websterengine production file except gitwrap.go, and card 15 lists gitwrap.go under Uses only, so the card cannot be built as scoped without breaking an untagged guard or editing an untargeted file"
---

# Plan-Review round 1 — judge verdict

## Round-1 findings: all resolved

I checked each finding against the current plan, not against the fixer report's claims.

- **F1 (BLOCKING):** card 3 now targets `internal/loomcli/integration_driverbootstrap_test.go`, and its Intent says how the stub reads the `prompt.md` path from the pointer in `$1`.
  No other test reads prompt content off the launch line: the remaining `ReadFile`/launch-cap test files (`command_test.go`, `settings_test.go`, `driverprompt_test.go`, `shell_test.go`) are all in card 3.
  `TestMasterPromptPointer_StaysShort` sits in `runlevel_test.go`, which is also card 3's.
- **F2:** card 3's Intent says what the guard refused (any `spec.Prompt` over 30000 bytes, because of the Windows 32,767-character command line), that nothing bounds prompt size at launch now, and that the launch line stays bounded.
  `driverprompt.go` and its test are now targets.
- **F3:** `loomshed.NewVerifyGate` (card 8) and `websterengine.NewVerifyGate` (card 15) are exported.
  Card 18's `verify_parity_integration_test.go` drives the verb and both closures through a three-way verdict over four cases, in the shape of `loomcli`'s `TestGateParity_*`.
  `go list -deps ./internal/loomshed` shows no `webstercli` import, so the test file causes no import cycle.
- **F4:** the old card 14 is now three cards: 14 (report and hint), 15 (gate), 16 (fixer).
  Each is independently reviewable, and the cross-references are renumbered.
- **F5:** card 17 keeps `accumulatedCardSHAs` (`runlevel.go:1647`), and card 14 reads it.
- **F6:** card 15 records the pre-fix head at the first failed evaluation of any kind, and a test pins the dirty case.
- **F7:** card 15 targets `internal/loomcli/wiring_test.go`.
  At HEAD, the only `ParentBranch` in `loomcli/wiring.go` (line 585) is `DescribeInputs`, so the claim that loomcli does not yet fill `RunDeps.ParentBranch` holds.
- **F8:** card 15 states that `StartMaster` is Merriam's only start, refuses a `deps.Gate` that already names `verify`, and edits `entries_simple.go`.
- **F9:** the ImpactSummaries of cards 13, 17 and 18 now state a conclusion about reach.
- **F10:** card 19 adds `final-summary-spec.md` and names `lyx webster verify` in the overview.
- **F11:** card 3 drops `Shell.ReadFile` from the interface and from `posix.go`.
  `pwsh.go:60`'s `Source` keeps its own use.

The report has no `## Focus departures` section, so there is nothing to ratify.

## New finding

### [MEDIUM] Card 15's commit check needs `gitwrap.go`, which it does not target

**Location:** `_lyx/plan/15-webster-verify-gate.md`, closure step 3; `internal/websterengine/gitwrap.go:158`; `cmd/lyx/rawgitmutation_test.go`.

**Issue:** step 3 checks every commit from the pre-fix head to HEAD on the first-parent chain: each must be a non-merge commit or a clean parent merge "as `gitwrap.go`'s `parentMergeRejection` accepts it".
`parentMergeRejection(repo *gitrepo.Repo, …)` and `resolveParentTips(repo *gitrepo.Repo, …)` both take a `*gitrepo.Repo`, and `gitrepo.New` is that package's only constructor.
No existing helper walks a mixed run of non-merges and merges.
`reconcileHead` accepts only merges between a head and HEAD, and the retiring `integrationfix.go` builds its check from `commitsBetween`, `commitParentCount` and `reconcileHead`.
So the walk is new code that has to call `gitrepo.New(`.
`TestNoRawGitMutation_WebsterProductionSource` (untagged, `cmd/lyx`) fails any websterengine production file containing `gitrepo.New(` except the allowlisted `gitwrap.go`.
Card 15 lists `gitwrap.go` under `Uses` only, and its new `verifygate.go` is not allowlisted.
As scoped, the implementer can only trip that guard or edit a file outside the card's targets.

**Fix:** add `internal/websterengine/gitwrap.go` to card 15's Edit targets.
Its Intent should name a gitwrap helper that walks the pre-fix-head-to-HEAD first-parent chain, accepting non-merges and `parentMergeRejection`-clean merges, which `verifygate.go` calls.

### [NIT] Card 17 leaves gitwrap.go's integration-fix leftovers

**Location:** `_lyx/plan/17-retire-integration-stage.md`; `internal/websterengine/gitwrap.go` (`integrationFixHeadRefusal`, `commitsBetween`, `commitParentCount`); `cmd/lyx/rawgitmutation_test.go:10`.

**Issue:** outside `gitwrap.go`, those three are used only by `integrationfix.go`, which card 17 deletes, so they lose their last caller unless card 15's new helper reuses the two commit helpers.
`integrationFixHeadRefusal`'s doc names the retired fix strand.
`rawgitmutation_test.go`'s header comment names the deleted `FabricBisector`.

**Fix:** whichever card ends up editing `gitwrap.go` drops `integrationFixHeadRefusal`, plus any commit helper the new walk does not reuse.
Card 17 updates `rawgitmutation_test.go`'s header comment.

## Checked and clean

- Verify coverage: `## verify:` runs `go test ./...` and `go test -tags integration ./...`, which cover every targeted package; `go vet -tags smoke ./...` only compiles smoke tests.
- No card is `Custom`, and card 19's `Prosa` targets are all files.
- Fidelity: every Decision D1–D16 and every Constraint is carried by a card, including the degrading config, no migration, in-memory attempt counts and the fabric-side naming.
  The plan writer's notes are settled: `forkOwnReport` (card 16), the attach path (card 15), and `waitingNote` (card 9).
  `LoomVerifyOutputPath` retires rather than becoming the log path; the note said "can", so that is allowed.
- Card 17's rename and deletion reach: every production and test file that names `IntegrationFailure`, `IntegrationFix`, `FabricBisector`, `OpenBisector`, `FixStarter` or `IntegrationReportPath` is in card 16 or card 17.
  The `recordbatch` pair is card 16's.
- The `verifyrun.Run` tripwire in card 18 matches HEAD.
  After the plan, the only production callers left are `cardverify.go` and `verifytree`: `integration.go` is deleted, and `landingshed/verifygate.go` moves onto `verifytree`.
- Every new verb, flag, routing edge and removed guard has a stated bound in its card's Intent.
  That covers cards 1, 2, 3, 4, 5, 8, 10, 12, 15, 16 and 18.

# Fixer report — plan, round 1

F9 is `decision`-class and goes to the parent; this report leaves it out.
`lym loom validate-plan` passes after every change, reporting only the informational `create-new-unit` on card 6.

## Changes

- **F1** (BLOCKING), fixed as proposed.
  - Card 8: `New` no longer refuses a recipe that lacks `contract: loom`. It stamps `DarnVerifyAttempts` on a recipe whose build row's engine is `DarnWrite`. The fail-loud-guard sentence is removed.
  - Card 11: adds `internal/loomrecipe#New` under Edit and `internal/loomrecipe/doc.go` under Uses. Its Intent adds the contract refusal, naming `NewPawl` or `lym shed`, and switches the stamp onto `verify_source: tree_scope`.
- **F2** (MEDIUM), fixed as proposed.
  - Card 17: creates `commitDescription`, adds `internal/loomcli#darnDeps` under Edit, and adds an Intent bullet saying `wire`'s `CommitDescription` and `darnDeps`' `Commit` both call it (D8). The ImpactSummary now names the merge.
  - One departure from the fix key, which said existing tests keep covering it: no test drives either closure (verified by grep). The card instead says the function extracts two identical bodies and gains no test of its own.
- **F3** (MEDIUM), fixed as proposed. Card 13's Intent and test bullet have battencli's `intentionallyNil` list `DiscussionHistory`, with the reason that batten wires no loom record.
- **F4** (MEDIUM), fixed as proposed, test wording refined in the self-check.
  - Card 6's `perchHardCap` bullet loads `LoadTapestryConfig` and applies `WithTapestry` for the plan perch. Its doc names `tapestry.yaml`.
  - `TestPerchHardCap` is added under Edit. Its existing Plan-Bouncer row moves onto `tapestry.yaml`.
- **F5** (MEDIUM), fixed as proposed and widened in the self-check (see below).
  - Card 6 adds under Edit `TestDispatchLayers`, `engineMinimalConfig`, `implementerEntry` and `internal/shedrecipe/doc.go`.
  - A new Intent bullet says how each of them follows.
- **F6** (MEDIUM), fixed as proposed.
  - Card 9 moves `internal/reportengine/doc.go` from Uses to Edit.
  - Card 9 adds under Edit `VerifySeedOwnership`, `internal/loomrecipe#New`, `lockFree` and `pattern/PATTERN-cwd-resolution.md`, each with its doc change in the test-and-docs bullet.
  - Card 8 lists `internal/reportengine/doc.go` under Edit, and its Intent says how that file's darn sentence changes.
- **F7** (MEDIUM), fixed as proposed. Card 13 adds `pattern/PATTERN-gate-slot.md` under Edit. Its Pairs section names `loomshed.ValidateDiscussion` and drops the Darn-Write pair.
- **F8** (MEDIUM), card text fixed as proposed.
  - Card 16's Not-committed refusal now follows the current row. At Record-Write it names `lym loom resume`. On any later row it names the goto to Record-Write, then resume, then re-running the verb.
  - `TestDecisionVerb_Refusals` gains rows at Record-Write and at Darn-Write.
  - The decision-record entry the fix asks for is deferred (see below).
- **F10** (LOW), fixed as proposed. Card 11 moves `internal/reportengine/doc.go` from Uses to Edit, and its darn setup sentence names Record-Write.
- **F11** (LOW), fixed as proposed. In card 9's test bullet, the row's seam now cancels the context and then returns an error. The row asserts the cancellation error and an outcome other than Stuck.
- **F12** (LOW), fixed as proposed. Card 5 adds `internal/loomcli/doc.go` under Edit, and its three sentences name `shed.yaml` and the profile block's `shed` object.
- **F13** (NIT), fixed as proposed. Card 11 now cites card 15.

## Disputed

None.

## Deferred

- **F8, the decision-record entry.**
  - Card 16's new text widens D4 in two places. D4 says "only a crash inside Record-Write leaves this", and its acceptance criterion says the not-committed refusal names `lym loom resume`.
  - Recording that change with `lym loom decision add` is the parent's call. This fixer may neither write the record nor message another session about the review.
  - Until the parent records it, card 16 and D4 disagree. The self-check's fix-consistency finding 2 raised the same gap.

## Self-check

- fix-consistency, MEDIUM: `New`'s new contract refusal in card 11 has no reaching test or label. **Fixed.** Card 11 says a batten seed reaches the refusal until card 19, and `TestNewDarn_Routing` gains a row in which `New` with batten is refused.
- fix-consistency, MEDIUM: card 16 disagrees with D4 and its criterion. **Deferred** with F8, for the reason above.
- fix-consistency, LOW: in card 9, `PATTERN-cwd-resolution.md`'s tier count and the "tier 3" line go stale. **Fixed.** The file now lists two tiers, and an orchestrator needs tier 2. This merges fix-claims 2 and fix-proposal 3.
- fix-consistency, LOW: card 11's ImpactSummary is stale. **Fixed.** It names the refusal and the stamp's key.
- fix-claims, MEDIUM: card 9 justifies `New`'s status-path check by "Shed's own status alone", a claim the code does not support. **Fixed with own change.**
  - The doc now drops the `tapestryPreflightEntry` rationale and states the check as keeping `env`'s copy equal to Shed's paths.
  - Removing the now-unread `Env.StatusPath`/`StatusLockPath` would be a wider change than this finding licenses.
- fix-claims, LOW: duplicate of fix-consistency's tier finding, merged there.
- fix-proposal, MEDIUM: the new `TestPerchHardCap` row duplicates the existing Plan-Bouncer row and leaves it stale. **Fixed with own change.** The existing row moves onto `tapestry.yaml` with a value its budget admits, and the raise names `lym config tapestry`. No row is added.
- fix-proposal, LOW: `TestDispatchLayers`' row still seeds loom's template. **Fixed.** The row seeds the tapestry template.
- fix-proposal, LOW: duplicate of the tier finding, merged.
- fix-affected-set, MEDIUM: more tests of the moved keys break. **Fixed.**
  - Card 6 adds them under Edit: `TestImplementerEntry_RefusesAnUnusableSetting`, `TestSettingLookup`, `TestFullEnv_FillsEverySeam`, `TestModules_LocalLayerRefusals` and battencli's `TestPrepareRunSettings`.
  - The readers bullet says how each changes.
- fix-affected-set, LOW: stale comments, help texts and fakes. **Fixed.**
  - Card 6 adds `planWriteEntry`, `TestBuild_EntryGate`, `TestBuild_Errors`, `TestReviewBudget_NegativeRefused`, `internal/configcli/doc.go` and `boardcli.Command` under Edit.
  - `internal/boardcli/doc.go` is added under Uses, which `doc-go-unlisted` requires.
- fix-affected-set, LOW: card 9 misses `internal/loomrecipe/doc.go` and `gatefindings_test.go`'s header. **Fixed.**
  - `internal/loomrecipe/doc.go` is added under Edit.
  - The header change rides on the deleted `TestTapestryPreflight_StuckSurfacesItsFailures`, since validate-plan refuses a `_test.go` file under Edit (`edit-test-file`, `redundant-file-target`).

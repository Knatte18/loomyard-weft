---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/loomshed/gates.go:1
    summary: the file header lists five gate closures but pairs only four functions respectively, so NewDarnDiscussionGate has no counterpart and the pairing reads shifted
    fix: In the file header comment of internal/loomshed/gates.go, add ValidateDarnDiscussion to the parenthesised list so each closure has its function in closure order, reading discussionparser.Validate, ValidateDarnDiscussion, ValidatePlan, ValidatePlanLensReview and ValidateReworkPlan, respectively. Keep one sentence per line.
  - id: F2
    severity: NIT
    class: consistency
    location: contracts/specs/refusal-spec.md:285
    summary: the new validate-discussion commit-failed row sits between two circling rows and splits the circling group
    fix: In contracts/specs/refusal-spec.md, move the validate-discussion commit-failed-on-a-darn-run row out from between the circling not-awaiting row and the circling not-at-a-Bouncer-row row, to just before the decision rows. The row text stays unchanged.
---

### [LOW] Gates file header pairs five closures with four functions

**Location:** internal/loomshed/gates.go:1
**Issue:** Card 4 added `NewDarnDiscussionGate` to the header's closure list, but the parenthesised list of functions each closure shares with its CLI verb still names four, followed by "respectively". The darn gate's function, `ValidateDarnDiscussion`, is the pair `PATTERN-gate-self-check-parity` now lists, so the header understates the parity it states.

### [NIT] Refusal row placed inside the circling group

**Location:** contracts/specs/refusal-spec.md:285
**Issue:** The loom section groups its rows by verb. The new `validate-discussion` row lands between two `circling:` rows, so a reader scanning the circling refusals meets an unrelated verb. The row's content matches the behavior in `internal/loomcli/validate.go`.

## Verdicts on the focus questions

- Cards 4-6: the darn gate (`loomshed.NewDarnDiscussionGate`) runs the section check, then the history check, then `commit`, and passes only after a successful commit; a commit error is returned wrapped with the record path. `validate-discussion` shares `ValidateDarnDiscussion` and commits on a clean run. A loom build leaves `Env.DiscussionHistory` nil, so `resolvePlainGateClosure` returns `NewDiscussionGate` and `wireLightweight` leaves both seams nil. Not a defect.
- Cards 2-3: `CommittedAnchoredFileHistory` walks every revision of the path through `gitrepo.PathRevisions(path, 0)`, which is uncapped, so it reads all versions, not the latest commit. `CheckAddedDecisionsKept` keys entries by author, date and title and reports each dropped one. The dropping-copy-is-newest case is pinned in `TestNewDiscussionGate`. Not a defect.
- Card 7: `decisionVerb` keeps the caller, `--by` and Discussion-Write checks first, then `checkDarnRecord` requires the record to exist, be committed, equal its committed copy and keep its history's entries. It re-reads the committed record after the commit and refuses a lost entry. The way-forward texts match the behavior and `refusal-spec.md` rows exist for each. Not a defect.
- Cards 10-13: `landingAlignmentDeps` takes the recipe, and only `RecipeDarn` gets an empty `PlanDir` and `MeasureDarn`. Every other recipe keeps the plan directory and `MeasureLoom`. `Prompt` fills `plan_dir` as `none` only when empty and refuses an empty `MeasureStencil`. The darn recipe's PR-Align row sits between Publish and PR-Gate with no `on_stuck`. Not a defect.
- Per-card mechanical check: no card carries a `Rename` or `Delete` group; every group is `Create` or `Edit`. Each Create target exists and each Edit target changed. The touched packages' tests pass under `lyx gate test` for the default tier and `--tags integration` for `fabricengine` and `loomcli`.
- Test economy: the only new top-level test is `TestCheckAddedDecisionsKept`, and the budget row for `internal/discussionparser` was raised to match. The darn-gate cases in `TestNewDiscussionGate`, `TestResolveGateSpec_ValidOneEntryPerVocabularyName`, `TestValidateDiscussionCmd` and `TestGateParity_DiscussionGate` each pin a different seam (the gate closure, the resolver's choice of form, the verb's envelope and the verb/gate parity), as the cards prescribe, so none is raised as redundant.

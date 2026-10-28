# Card 7 — test-tier-check

**Create:**
- `internal/loomshed#testTierFindings`
- `internal/loomshed#TestValidatePlan_CardChecks`
- `internal/loomcli#gateTierTags`

**Edit:**
- `internal/loomshed#PlanContext`
- `internal/loomshed#planContextFindings`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#planContextOf`
- `internal/shedrecipe#TestPlanWriteEntry_GateConfig`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.validatePlanCmd`
- `internal/loomcli#planParityCase`
- `internal/loomcli#TestGateParity_PlanGate`
- `internal/loomcli#TestValidatePlanCmd`
- `internal/loomcli#TestValidatePlanCmd_Overlay`
- `internal/loomcli#TestValidatePlanCmd_ReworkReportsFindingsForFirstCardMismatch`

**Extends:**
- `contracts/specs/loom-plan-spec.md`
- `internal/loomshed/doc.go`

**Uses:**
- `internal/shedrecipe/doc.go`
- `internal/planparser#PathEntryPath`
- `internal/testkit/plankit#Write`
- `internal/loomshed#TestNewPlanGate`
- `internal/websterengine#SaveState`

**Intent:**
D7, and the D9 plumbing the plan checks share.

`PlanContext` gains `TierTags []string`, the hub's `gate.yaml` tier keys; nil skips the tier check, which is the state with no `gate.yaml`.
`shedrecipe.Env` gains the field `TierTags []string`, documented as read by the plan gates' `test-tier-unstated` check, and `planContextOf` copies it into the `PlanContext`.
`gateTierTags` returns the sorted keys of `cfg.Tiers`; `loomCLI.wire` sets `Env.TierTags` from it where it already reads `RefuseRaw` from the loaded `gate.yaml`, and `validatePlanCmd` sets `PlanContext.TierTags` from it where it reads `refuse_raw`, so the gate's and the verb's builders fill the field the same way and an absent `gate.yaml` leaves it nil on both.

`planContextFindings` appends `testTierFindings(plan, cards, pc.TierTags)` after the verify-line findings; it runs over the cards it is told, so `ValidatePlan` skips the done cards and `ValidateReworkPlan` checks the fresh generation, and it returns nothing unless `plan.GoToolchain()` holds and `pc.TierTags` is non-empty.
For each ref of a card's `Create` group that `planparser.PathEntryPath` reads as a path ending `_test.go`, it collects the card's Intent lines that name the file, by that worktree-relative path or by its base name as a token of its own, never the tail of a longer path, and the tier tags those lines name as whole words, collected as a set of distinct tags, so the same tag stated on several naming lines counts as one tag.
The file passes when the lines name exactly one tag; or several, each pair of consecutive ones joined by `||` on the same line, the form of a helper several tiers share; or the word `untagged` and no tag, a support-only file.
Otherwise it is one blocking `test-tier-unstated` finding naming the card, the file and the tags its naming lines hold, with "way forward: add an Intent line naming the file and its `//go:build` line, such as `//go:build tier0`, or the word `untagged` when the file holds support only (TestMain, helpers) and carries no build line, on a line that names no other test file; where the lines name several tags, join them with || only for a helper several tiers share, else keep one; drop untagged or the tag when both appear".
A line that also names another test file carries that file's tags into the file's collection, so the way forward puts each file's statement on a line that names no other test file.
Bound, stated in the spec: a test file created as a member handle rather than a file target is not seen; whether the stated tier fits stays with `cmd/lym/tiertags_test.go` and review; the check takes each tier key as its tier's build tag, as `PATTERN-test-speed` requires.
It adds a refusal, never a bypass.

Tests:
`TestValidatePlan_CardChecks`, tier0 in the existing test file beside `TestNewPlanGate`, is a new top-level table rather than rows of `TestNewPlanGate`'s context table, because that table's `seedPlanContextFixture` writes a `language: none` plan, which the three card checks skip, and its rows share no worktree quarry can resolve, which the D6 and D8 rows need.
It is a table that writes a `language: go` plan with `plankit.Write` under a temp anchor, runs `ValidatePlan` with `planglyph.NewIndex(fabricengine.NewReferenceRule())` and a `PlanContext` holding `TierTags` `tier0` to `tier3`, and counts findings by check, asserting each finding's Detail holds its way-forward clause.
Its D7 rows: a card creating `internal/x/new_test.go` with no Intent line naming it fails with `test-tier-unstated` naming the card and file, its Detail holding the `untagged` clause of the way forward; an Intent line naming the file and `//go:build tier1` passes; one naming it by base name passes; one naming it on two Intent lines that both state `//go:build tier1` passes; `tier0 tier1` on the naming line fails, its Detail naming both tags and the join-or-keep-one clause; `tier0 || tier1` passes; `untagged` passes, the support-only fix the no-statement row's Detail names; `untagged` with `tier0` fails, its Detail naming `tier0` and the drop clause; a card creating `internal/x/a_test.go` and `internal/x/b_test.go` whose single Intent line names both files with `tier0` and `tier1` fails both files with `test-tier-unstated`, each Detail naming both tags and the own-line clause, and the same two statements split over two Intent lines pass; a card creating `internal/x/x_test.go` with `tier0` and `internal/x/helper_test.go` with `untagged` whose single Intent line names both files fails both with `test-tier-unstated`, each Detail naming `tier0` and the own-line clause, and the same two statements split over two Intent lines pass; a card creating `internal/x/a_test.go` with `tier0` and `internal/y/a_test.go` with `tier1`, each stated on its own Intent line by its full path, passes; nil `TierTags` passes the missing statement; a `language: csharp` plan passes it; the same card recorded done (a webster run record holding batch 1 done, written with `websterengine.SaveState` as `TestNewPlanGate`'s inline `saveState` closure does) passes; and `planglyph.NewIndex(fabricengine.NewReferenceRule()).ValidateDispatch` over the failing row's plan reports no `test-tier-unstated`, since webster's dispatch runs only the index.
`TestGateParity_PlanGate` gains a field on `planParityCase` for a `gate.yaml` the fixture hub carries and the tier tags the gate is told, and a case whose plan creates a test file with no tier statement: `NewPlanGate`, `NewLensReviewPlanGate`, the verb's no-flag and `--lens-review` modes all report `test-tier-unstated`.
`TestValidatePlanCmd`, `TestValidatePlanCmd_Overlay` and `TestValidatePlanCmd_ReworkReportsFindingsForFirstCardMismatch` run without a `gate.yaml` today, which leaves `TierTags` nil, so each new row first writes a `gate.yaml` declaring tiers `tier0` to `tier3` into its fixture's config directory.
Over that fixture, `TestValidatePlanCmd` gains a pair of rows over the same plan: the no-flag mode reports `test-tier-unstated` and `--require-approved` does not; `TestValidatePlanCmd_Overlay` gains a row overlaying such a card that reports it; `TestValidatePlanCmd_ReworkReportsFindingsForFirstCardMismatch` gains an assertion that `--rework` reports it for such a fresh-generation card.
`TestPlanWriteEntry_GateConfig` gains a row: an `Env` with `TierTags` set makes the Plan-Write gate built through `planContextOf` report `test-tier-unstated`.

Docs: `contracts/specs/loom-plan-spec.md` gains the `test-tier-unstated` validation-check row with its rule, the rules that a base name matches only as a token of its own, that a file's tags are collected as a set of distinct tags, so the same tag on several naming lines counts as one, and that a line naming another test file carries that file's tags into the file's collection, the way-forward string the finding carries, and the bound, names it among the checks `loomshed` appends in every mode but `--require-approved`, and its card rules tell the writer to name a new test file as a `**Create:**` file target and state its `//go:build` line in the Intent, on a line that names no other test file.
`internal/loomshed/doc.go` lists the check among the `PlanContext` checks; `validatePlanCmd`'s Long help rewrites its sentence starting "Every mode but --require-approved also runs rename-tool-missing and pattern-changes-mismatch, which read language.yaml and the decision record": it lists `test-tier-unstated` among the checks that sentence names, and moves the clause on what they read so it describes only `rename-tool-missing` and `pattern-changes-mismatch`.

**ImpactSummary:** two struct fields (`PlanContext.TierTags`, `Env.TierTags`) filled at the gate's and the verb's one builder each; the only new refusal fires on a go plan that creates a `_test.go` file target without stating its tier, at every plan gate and `validate-plan` mode but `--require-approved`.

**Commit:** `7: Refuse a created test file whose card states no tier`

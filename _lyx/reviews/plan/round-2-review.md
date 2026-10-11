---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/08-begin-batch-builds-pack.md:54
    summary: The breakPlanAfterValidate row names no overview text, and the obvious unparsable text is refused upstream by restampFingerprint, so begin-batch refuses before buildKickstartPack's parse-error degrade branch runs
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/08-begin-batch-builds-pack.md:60
    summary: TestBeginBatch_KickstartPack's plan makes card 1 an Edit card but says ValidateDispatch passes because every ref resolves, leaving out the ImpactSummary that card-missing-field requires, without which every row refuses with ErrPlanDrifted
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/05-bind-trace.md:71
    summary: TestRecordBatch_BindTrace runs over the record fixture, whose plan dir is empty and whose batch holds one card, so the bound row's RewriteRefs fails on ParsePlan and the mixed done/not-done row has no second card; the card names no on-disk plan
---

### [MEDIUM] The parse-error degrade row's input is refused upstream by the fingerprint re-stamp

**Location:** `_lyx/plan/08-begin-batch-builds-pack.md:53-54`, and the row at line 69.

**Issue:**
Card 8 prescribes a guard branch: when `buildKickstartPack`'s fresh `planparser.ParsePlan(deps.Plan.Dir)` fails, it logs at Warn and builds the pack from `deps.Plan`.
The input that reaches it is `fakeIndex.ValidateDispatch`, which, with `breakPlanAfterValidate` set, "overwrites the plan's `00-overview.md` with text `planparser.ParsePlan` refuses".
In `BeginBatch` (`internal/websterengine/beginbatch.go:422`), `restampFingerprint(deps.State, deps.Plan.Dir, …)` runs right after `ValidateDispatch` and before anything else.
Its `restampBaseline` calls `overviewFrameHash` (`internal/websterengine/fingerprint.go:225`), which goes through `planparser.OverviewWithoutCardIndex` and fails on an overview with no well-formed frontmatter or no `## Card Index` (`fingerprint.go:105-106`, `internal/planparser/parse.go:295-310`).
With `resolveErr` nil, that error is returned and `begin-batch` refuses there.
So the natural reading of "text `ParsePlan` refuses", such as garbage or an empty file, never reaches `buildKickstartPack`.
The row's assertion that `begin-batch` succeeds would fail, or a test writer would have to find the narrow input by trial.
Under the rubric's reachability rule this is a named input that an upstream check rejects, and that upstream check is `restampFingerprint` → `overviewFrameHash`.

**Fix:**
Make the override name an overview that keeps well-formed frontmatter and a `## Card Index` while `ParsePlan` still refuses it.
One example is a Card Index line for a card number whose file does not exist, which `parseCardFile` refuses while the batch's own card files still hash.
Then state that `restampFingerprint` accepts it.

### [LOW] The KickstartPack test plan's premise for passing ValidateDispatch omits the Edit card's ImpactSummary

**Location:** `_lyx/plan/08-begin-batch-builds-pack.md:60`, and the rows at lines 62-69.

**Issue:**
Every row's plan makes card 1 an `Edit` card on a member glyph.
The card says that because "every ref resolves against the fixture worktree, … `ValidateDispatch` passes".
`ValidateDispatch` first runs `planparser.ValidateFormat` (`internal/planglyph/planglyph.go:134`), and `checkCardMissingField` requires `ImpactSummary:` on an `Edit` or `Delete` group (`internal/planparser/validate.go:843-851`).
`plankit.Write` renders `**ImpactSummary:**` only when the field is set (`internal/testkit/plankit/plankit.go:161`).
`seedRewritingPlanDir`'s `Edit` card sets it for this reason.
A plan written to the card's stated premise gets a blocking `card-missing-field` and `ErrPlanDrifted`, and no row reaches `buildKickstartPack`.
Under that same premise, the draft-handle row's card 1 is a mixed `Create` + `Edit` card and needs the field as well.

**Fix:**
In the paragraph at line 60, say that card 1 carries an `ImpactSummary`, so `ValidateFormat` reports nothing blocking, beside the existing "every ref resolves" condition.

### [LOW] TestRecordBatch_BindTrace's fixture cannot reach the bound and mixed-batch rows as named

**Location:** `_lyx/plan/05-bind-trace.md:71-81`.

**Issue:**
The card drives `RecordBatch` "over the record fixture".
`newRecordFixtureOver` (`internal/websterengine/recordbatch_test.go:133-160`) gives it an empty plan directory, with the comment "No card here declares a handle, so nothing is ever written into it", and one single-card batch.
For a row where a handle binds, `BindHandles` hands its substitutions to `planparser.RewriteRefs`, which calls `ParsePlan(planDir)` whenever substitutions exist (`internal/planparser/rewrite.go:44-52`).
On the empty directory that errors, `postBatchChecks` returns `bindErr` (`recordbatch.go:808-810`), and the "all handles bound" row logs no line.
The "done card and a not-done card in one batch" row needs a two-card batch, plus state whose begin record names both cards, and the fixture has neither.
The no-line rows the focus asks about all reach their branches:
- `Geom.Index` nil reaches `in.Geom.index()` at `recordbatch.go:764`, and `RecordBatch` calls `index()` nowhere earlier.
- `doneErr` and a plain `deltaErr` reach `recordbatch.go:769` and `:792`.
- `bindErr` reaches `:808` through the fake, so it no longer depends on `RewriteRefs`.

The gap is only in the rows that need real plan files.

**Fix:**
Name the row fixture in the card.
Each row writes its plan with `plankit.Write`, as `writeRecordPlanDir` does, sets `Deps.Geom.PlanDir`, `Deps.Plan`, `Deps.Batches` and the begin record's cards to it, and re-stamps with `websterengine.RestampPlanBaseline`, so `PlanEditError` accepts it.

## Verdicts on the focus directives

- Card 8 (focus's card 7), whether `buildKickstartPack` can refuse `begin-batch`: it cannot.
  It returns no error, the parse-error and `KickstartPack`-error paths both degrade with a Warn line, and `BeginBatch` gains no refusal, so `refusal-spec.md` needs no new row and `PATTERN-refusal-way-forward` is not engaged.
  The degrade branch's reachability is F1.
- Card 8's `TestBeginBatch_KickstartPack` rows: the fixture reads `deps.Plan.Dir`, the directory `ParsePlan` was given, and the row repoints `Geom.PlanDir` and the fingerprint as `TestBeginBatch_RestampsFingerprintEvenWhenPlanDrifts` does.
  The batches' slugs match, so `batchCardHashes` reads the row's card files.
  Under `language: none` a member glyph maps to no disk path (`internal/planparser/shape.go:229-233`), so `path-missing` is silent and that row reaches the nil-pack branch.
  The draft-handle row matches the existing canonicalization fixture minus its blocking card.
  The remaining gaps are F1 and F2.
- Card 5 (focus's card 4) no-line rows: every one reaches its branch, as F3's Issue states.
  The `BindHandles`-error row now goes through `fakeDelta.bindErr`, not through `RewriteRefs`.
- Cards 3 and 4: `planparser.OwnedHandles` has its own card 4, which creates it, deletes `cardOwnHandles` and rewrites every mention.
  I checked those mentions: `handle.go` (doc and body comments), `doc.go:86` and `handle_test.go:664`.
  Card 3 keeps `cardOwnHandles` until card 4 replaces it.
  The granularity rule is met, and the order is sound: card 5 `Uses` the handle card 4 creates.
- Card 7 (focus's card 6) re-sign arrows: `RenderForkPrompt` and `renderFork` carry arrows matching the new trailing `pack` parameter.
  `RenderRecoveryPrompt` and `BeginBatch` keep their signatures and carry none.
  Outside `template_test.go:338`, no other code calls `RenderForkPrompt`.
  The only callers of `renderFork` are `TestRenderPrompts`, `TestRenderPrompts_FrictionDirective` and `TestRenderPrompts_ParentDirective`, and all three are listed under `Edit:`, so no defect.

## Focus departures

- All five directives: "Card 7 (`_lyx/plan/07-begin-batch-builds-pack.md`)", "Card 4 (`_lyx/plan/04-bind-trace.md`)" and "Card 6 (`_lyx/plan/06-pack-stencils-and-render.md`)" name card numbers and files from before the `owned-handles` card was inserted.
  I judged them as the same cards under their current names: `08-begin-batch-builds-pack.md`, `05-bind-trace.md` and `07-pack-stencils-and-render.md`.
  The "Cards 3 and 4" directive I judged against the current cards 3 (`bind-outcomes`) and 4 (`owned-handles`).

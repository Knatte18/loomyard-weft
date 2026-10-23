---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/27-publish-failure-note-trace.md:12
    summary: Card 27 shape-checks traced glyphs with a local regex (tracedGlyph), which PATTERN-glyph-conversion-chokepoint forbids
    fix: "In 27-publish-failure-note-trace.md, drop the Create target `plan:internal/loomshed#tracedGlyph` (line 12). Reword the Intent sentence at line 17 so a traced glyph is kept only when it is non-empty and holds no whitespace, control character or backtick, checked with `strings.ContainsFunc` over `unicode.IsSpace`/`unicode.IsControl` and `strings.ContainsRune(g, '`')`. Add that this is a prompt-safety character check, not a glyph grammar, so no local regex reads a glyph string (PATTERN-glyph-conversion-chokepoint). Keep `tracedLocation`, which reads a path:line, not a glyph. The TestPublishFailure row that drops a glyph holding a backtick stays as written."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/11-amendment-record.md:3
    summary: Card 11's YAML amendment record makes internal/loomshed import gopkg.in/yaml.v3, which loomshedAllowedImports omits, so TestToldGeometryInvariant_AllowlistOnly fails
    fix: "In 11-amendment-record.md, add an `**Edit:**` group after the Create group with `- `internal/loomshed#loomshedAllowedImports``. Add one Intent sentence: the allowlist gains `gopkg.in/yaml.v3` with a comment that the amendment record is YAML and the package still resolves no geometry. Add an `**ImpactSummary:**` line, which the Edit group requires, such as: one test-only allowlist gains `gopkg.in/yaml.v3`; the rest is new code with no caller until card 13. Checked with validate-plan --overlay: no new blocking finding."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/25-failuretrace-package.md:14
    summary: The new internal/failuretrace test binary links tree-sitter through planglyph, and card 25 does not add it to the tree-sitter link guard's allowed list, so TestTreeSitterLink_OnlyAllowedTestBinariesLinkIt fails
    fix: "In 25-failuretrace-package.md, add `- `cmd/lyx#treeSitterAllowedLinks`` to the Edit group (after `docs/overview.md`). Add one Intent sentence: the list gains `{Key: \"internal/failuretrace\", Why: \"traces a failed verify through planglyph's EncloseAt\"}`. The guard in cmd/lyx/treesitterlink_integration_test.go fails any test binary outside it that links tree-sitter. Checked with validate-plan --overlay: no new blocking finding."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: _lyx/plan/04-plan-gate-pattern-checks.md:41
    summary: pattern-removes-prescribed counts done cards, and its only way forward (drop the fenced line from the card) is refused by done-card-edited, so a removes amendment over an entry a done card built cannot clear the gate
    fix: "In 04-plan-gate-pattern-checks.md, run `pattern-removes-prescribed` over the cards not recorded done only. Pass those cards to the check, for example re-sign the Create head of `patternPlanFindings` to `func patternPlanFindings(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string, pc PlanContext) ([]planindex.Finding, error)`, with `planContextFindings` passing its `cards` argument. Keep the projection over every live card. Add an Intent sentence: a done card's fenced line is already built, and the projection applies `removes` after the prescribed lines, so the entry is still dropped. Add a TestNewPlanGate row where the fencing card is recorded done: no `pattern-removes-prescribed`."
  - id: F5
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/12-settle-round-start.md:23
    summary: round_started_at is said to be nanosecond-precise, but the review half's start comes from shuttle's CreatedAt in whole-second RFC 3339, so the recorded start is floored to the second
    fix: "In 12-settle-round-start.md, state that the start's source (shuttle's run CreatedAt, written by time.Now().UTC().Format(time.RFC3339)) is whole-second, so round_started_at is floored to the second. In 11-amendment-record.md and 14-settled-guard-amendment.md, state that `WrittenAt` keeps full precision against the floored start. A same-second tie therefore reads the amendment as after the start: the guard admits the goto and AmendmentSettledBy keeps the record, so the floor costs at most an extra round and never loses an amendment. Write the test rows' round starts as whole seconds."
  - id: F6
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/04-plan-gate-pattern-checks.md:48
    summary: pattern-line-unprescribed's way forward names the card that edits PATTERN.md, which on the amendment route or a no-parent base may be done and frozen
    fix: "In 04-plan-gate-pattern-checks.md, reword the way forward at line 48. Prescribe the entry's whole overview line in a fenced block on a pending card that edits `PATTERN.md`, or on a card inserted with `lyx loom insert-card` after the last begun batch when that card is done. Make the TestNewPlanGate row that asserts this way forward check the new wording."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/13-decision-amends-plan.md:39
    summary: Card 13 puts the darn --amends-plan refusal after checkDarnRecord, which refuses a darn run first unless its committed record matches, but its reaching-input line says every branch's fake is a loom run
    fix: "In 13-decision-amends-plan.md, move the darn refusal under `amendsPlan` right after `deps.darn()` answers true and before `checkDarnRecord`, since `--amends-plan` has no plan to amend on any darn run. Reword line 53 so the darn row's fake is a darn run whose record check is never reached."
  - id: F8
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/05-plan-gate-pattern-base-wiring.md:23
    summary: OverviewAt's refusal of an empty revision names no input or test row that reaches it
    fix: "In 05-plan-gate-pattern-base-wiring.md, add a TestPatternBaseReader row whose merge base answers the empty string and expects the returned error. This is the input that reaches OverviewAt's empty-revision refusal through PatternBaseReader."
  - id: F9
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/11-amendment-record.md:24
    summary: The undecodable-record way forward offers a restore from HEAD that can return the same broken bytes a status commit landed, names no actor who may edit the task worktree, and names no step after the commit
    fix: "In 11-amendment-record.md, reword the way forward. The operator removes the file and runs `lyx fabric commit`, then re-runs the refused step: the goto, `lyx loom resume` for a failed Plan-Bouncer commit seam, or the `decision add` call. Drop the restore-from-HEAD alternative. Keep the clause that a session lyx refuses the verb from reports status: FAILED. Card 11's TestAmendmentRecord row asserts the reworded text."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/12-settle-round-start.md:27
    summary: A round_started_at that does not parse is an error naming no way forward, and card 14's guard turns it into a goto refusal the parent cannot act on
    fix: "In 12-settle-round-start.md, make SettleRoundStart read a round start that does not parse as no known round start: log it at Warn and return the zero time and false, the same as an absent field. The guard then admits and the commit seam keeps the record. Change the TestSettleRoundStart row from an unparsable start being an error to it reading false."
  - id: F11
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/13-decision-amends-plan.md:40
    summary: An approvedPlan error is refused with decisionWayRetry, but planApprovedReader parses the whole plan, so a plan that does not parse fails every re-run
    fix: "In 13-decision-amends-plan.md, split the `approvedPlan` error. A plan parse failure is refused with a way forward naming `lyx loom validate-plan` to show the defect, and recording the decision without `--amends-plan` until the plan parses. Any other read error keeps `decisionWayRetry`. Add a TestDecisionVerb_AmendsPlan row for the parse failure."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/06-extends-label.md:10
    summary: Card 6 updates the spec's label lists but leaves other lists of the seven labels stale, including the user-visible card-type-missing detail
    fix: "In 06-extends-label.md, add these to the Edit group: `- `internal/planparser#checkCardTypeMissing`` (the detail at validate.go:324 lists Create/Edit/Delete/Rename/Move/Prosa/Custom), `- `internal/planparser#CardTypeCustom`` (its comment says the other six types) and `- `tools/sandbox/SANDBOX-WEBSTER-SUITE.md`` (the seven-name set). Add an Intent sentence that each names Extends. The seven counts in parse.go's package comment and the label const block's comment lie outside any member span. parse.go's file self glyph would trip redundant-file-target beside the parse.go members card 6 already lists, so the Intent names those two comments as part of the parse.go edit. Checked with validate-plan --overlay: no new blocking finding."
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/06-extends-label.md:43
    summary: Card 6 defers planparser's doc of the dependency rule to card 8, but card 8 edits websterengine's doc.go and the spec, not internal/planparser/doc.go, where DependencyEdge lives
    fix: "In 08-card-pair-rule.md, add `- `internal/planparser/doc.go`` to the Edit group. Extend the Intent's doc sentence so planparser's doc.go states `CardRefKind`, `DependencyEdge` and the rule that two Extends on one ref make no edge unless both fence a line for one PATTERN entry ID. Checked with validate-plan --overlay: no new blocking finding."
  - id: F14
    severity: NIT
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/13-decision-amends-plan.md:41
    summary: With --amends-plan the verb reads the decision record before the append, so an absent record is refused with decisionWayRetry instead of the existing decisionWayNoRecord way forward
    fix: "In 13-decision-amends-plan.md, state that a not-exist read of the decision record before the call is not refused there. The verb goes on to the append, whose existing fs.ErrNotExist case refuses with decisionWayNoRecord, after restoring the amendment record."
  - id: F15
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/plan/18-recipe-amendment-target.md:12
    summary: loomrecipe.AmendmentTarget takes a darn bool, where the package's other per-recipe readers (Rows, RowNames) take the recipe name and refuse an unknown one
    fix: "In 18-recipe-amendment-target.md, re-sign the Create head to `func AmendmentTarget(recipe string) (string, error)`, answering for shedrun.RecipeLoom, RecipeDarn and RecipeBatten and refusing any other name as Rows does. TestAmendmentTarget asserts loom's answer, darn's and batten's empty answers, and an unknown name's error. In 19-arbiter-amendment-target.md, change the call to `loomrecipe.AmendmentTarget(c.recipe)`. Checked with validate-plan --overlay: no new blocking finding."
---

### [BLOCKING] Card 27's tracedGlyph regex breaks the glyph-conversion chokepoint

**Location:** `_lyx/plan/27-publish-failure-note-trace.md:12`, Intent at line 17

**Issue:** `checkedPublishFailure` is to keep a traced glyph only when it matches `tracedGlyph`, a `regexp` of "glyph characters only".
`pattern/PATTERN-glyph-conversion-chokepoint.md` lists "a local regex over a glyph string" as forbidden, and `glyph.Parse` plus `Glyph.String` as the only glyph grammar.
The check exists to keep free text out of the prompt, so a character-class test does the same work without defining a glyph grammar outside quarry.

### [MEDIUM] Card 11 needs gopkg.in/yaml.v3 on loomshed's import allowlist

**Location:** `_lyx/plan/11-amendment-record.md:3`

**Issue:** D7 fixes the record at `_lyx/loom/amendment.yaml`, and card 11 encodes it in YAML inside `internal/loomshed`.
`internal/loomshed/seam_enforcement_test.go:24-63` lists every non-stdlib import the package may use (`scankit.isStdlib` decides what counts), and no loomshed production file imports `yaml.v3` today.
`TestToldGeometryInvariant_AllowlistOnly` fails on card 11's own gate unless the card names the list.

### [MEDIUM] Card 25 misses the tree-sitter link guard

**Location:** `_lyx/plan/25-failuretrace-package.md:14`

**Issue:** `internal/failuretrace` imports `internal/planglyph`, so its test binary links tree-sitter.
`cmd/lyx/treesitterlink_integration_test.go:18-27` holds `treeSitterAllowedLinks`, and `TestTreeSitterLink_OnlyAllowedTestBinariesLinkIt` fails any test binary outside it that links tree-sitter.
This is a tier-1 test, so the batch gate fails with nothing in the plan pointing at the fix.

### [MEDIUM] pattern-removes-prescribed counts done cards it cannot ask to edit

**Location:** `_lyx/plan/04-plan-gate-pattern-checks.md:41` (and line 36: "The new checks read `plan.Cards`, every live card")

**Issue:** The check's only way forward is to drop the fenced line from the card.
The card itself notes that no verb withdraws a `removes` bullet.
On the amendment route this plan builds, a parent may record `--pattern-action removes … --amends-plan` for an entry a done card fenced and built.
The re-opened fix round's gate then raises a finding that only an edit to the done card clears, and `done-card-edited` (`internal/loomshed/gates.go`, `ValidatePlan`) refuses that edit.
The rewrite already happened in the tree, so dropping the entry afterwards is a coherent sequence; the check needs only the pending cards.

### [LOW] The settle round start is whole-second, not nanosecond

**Location:** `_lyx/plan/12-settle-round-start.md:23`

**Issue:** `halfUsage.Start` is set from shuttle's `Result.StartedAt`.
That is `time.Parse(time.RFC3339, run.state.CreatedAt)` (`internal/shuttleengine/wait.go:1713-1714`), and `CreatedAt` is written as `time.Now().UTC().Format(time.RFC3339)` (`internal/shuttleengine/run.go:619`).
This round's own `round-1-usage.yaml` shows `start: 2026-10-22T18:44:11Z`.
The cards rest on a premise the source does not give.
The floor errs toward re-admitting and keeping, so it costs an extra round at worst, but the cards should say so, and their rows should not imply sub-second ordering.

### [LOW] pattern-line-unprescribed points at a possibly frozen card

**Location:** `_lyx/plan/04-plan-gate-pattern-checks.md:48`

**Issue:** "On the card that edits `PATTERN.md`" can name a done card.
That happens when an amendment declares a `changes` entry after that card's batch finished, or in the no-parent HEAD-base case the card describes at line 51.
Editing the done card is refused by `done-card-edited`, so the stated step does not progress.

### [LOW] Card 13's darn refusal sits behind checkDarnRecord

**Location:** `_lyx/plan/13-decision-amends-plan.md:39`, reaching inputs at line 53

**Issue:** `decisionVerb` runs `checkDarnRecord` inside its `if darn {` block and refuses there unless the committed darn record matches.
The new refusal placed "after its existing caller, form and darn checks" is reached only from that narrower state.
Line 53 says every branch's fake is "a loom run", which contradicts the darn row.

### [LOW] OverviewAt's empty-revision refusal has no reaching row

**Location:** `_lyx/plan/05-plan-gate-pattern-base-wiring.md:23`

**Issue:** `PatternBaseReader` passes `HEAD` or `mergeBase()`'s answer, and no `TestPatternBaseReader` row feeds an empty revision.
The rubric requires a named input and test for each prescribed refusal.

### [LOW] The undecodable amendment record's way forward does not reliably progress

**Location:** `_lyx/plan/11-amendment-record.md:24`

**Issue:** `statusCommitPathspec` stages `_lyx/loom` whole, so HEAD may already hold the broken bytes, and "restore from HEAD" then changes nothing.
The orch stencil forbids edits in a task pair (`contracts/stencils/orch/orch-template-role.md:109`), and the arbiter edits only its verdict, so no named role can carry it out.
On card 15's commit-seam path the text names no `lyx loom resume` after the fix.

### [LOW] An unparsable round start refuses with no way forward

**Location:** `_lyx/plan/12-settle-round-start.md:27`

**Issue:** Card 14 returns this error from the guard, and Shed turns it into a refusal.
PATTERN-refusal-way-forward requires that refusal to name its way forward, and it names none.
The decision record already reads an absent or zero start as no known round start, which is the safe direction for an unreadable one too.

### [LOW] A plan that does not parse makes --amends-plan's retry loop forever

**Location:** `_lyx/plan/13-decision-amends-plan.md:40`

**Issue:** `planApprovedReader` parses the plan through `planparser`.
A parse failure is persistent, yet it is refused with `decisionWayRetry` ("re-run … nothing is appended"), and every re-run fails the same way.

### [LOW] Stale seven-label lists outside the spec

**Location:** `_lyx/plan/06-extends-label.md:10`

**Issue:** Card 6 edits the spec's label lists and counts, but other lists stay stale.
`internal/planparser/validate.go:324`'s user-visible `card-type-missing` detail lists seven labels.
`internal/planparser/plan.go:32` says "the other six types".
`internal/planparser/parse.go:10` and `:437` count seven.
`tools/sandbox/SANDBOX-WEBSTER-SUITE.md:44` names "the seven-name set".
`lyx quarry resolve` confirms `checkCardTypeMissing` (314-330) and `CardTypeCustom` (32-33) enclose their lines.

### [LOW] planparser's doc of the pair rule is promised but not planned

**Location:** `_lyx/plan/06-extends-label.md:43`

**Issue:** Card 6's planparser `doc.go` paragraph ends "the dependency rule follows on card 8", but card 8's Edit group lists `internal/websterengine/doc.go` and the spec only.
`CardRefKind` and `DependencyEdge` are planparser's exported surface, which D4 places there so the batchifier can call it, and CLAUDE.md asks for the package doc in the same commit.

### [NIT] --amends-plan shadows the no-record refusal

**Location:** `_lyx/plan/13-decision-amends-plan.md:41`

**Issue:** The new pre-read of the decision record turns an absent record into `decisionWayRetry`, ahead of the append's `fs.ErrNotExist` case and its `decisionWayNoRecord` way forward.
It is rare, since an approved loom plan implies a record, but the refusal names the wrong step.

### [NIT] AmendmentTarget's bool departs from loomrecipe's recipe-name convention

**Location:** `_lyx/plan/18-recipe-amendment-target.md:12`

**Issue:** `loomrecipe.Rows` and `RowNames` take the recipe name (`shedrun.RecipeLoom`, `RecipeDarn`, `RecipeBatten`) and refuse any other.
`AmendmentTarget(darn bool)` cannot answer for batten, and card 19 has to build the bool from `c.recipe`, which it could pass as is.

## Focus verdicts

- Fidelity: D1 to D9 and every Constraint are carried, by cards 2–5 (D1, D2), 6–7 (D3), 8–9 (D4), 1 (D5), 10 (D6), 11–17 and 21 (D7), 18–20 (D8) and 22–27 (D9). The glyph background file is card 23, and the told seam is card 26. Card 21's change to `loomRerouteCapWayForward` is licensed as the same hand-edit way forward D7 replaces. No card adds unlicensed scope.
- Card order: no card uses a symbol a later card creates. Card 19 uses `AmendmentTarget` (18) and `planApprovedReader` (13); card 16 does not use `AmendmentTarget`. Cards 24–26 come after `EncloseAt` (23), and 25 after 22 and 24.
- Cards 1 to 10: none prescribes a PATTERN entry line or edits `PATTERN.md`, per `no-overview-growth`. No card uses `**Extends:**`, which `docs-under-edit` settles because the validating binary does not know the label yet. Not a defect.
- Reachability and attack surface: `--amends-plan` (13), the guard's admission (14), the skip seam (16) and the pattern checks (4) each state their bound and name reaching rows, bar F7 and F8.
- Verify lines and test economy: no card carries a `**Verify:**` line. No planned test duplicates an existing public-surface test, since each new test covers a new exported symbol or a new package.

## Rejected

- lens:guard-reach, card 25's `TestTrace` row "a card Webster holds no record of gets batch 0 and an empty range" guards an unreachable state: D9 names this case explicitly ("empty for a card Webster holds no record of"), so the row is licensed by the answer key.
- lens:way-forward rated the undecodable-record way forward (F9) MEDIUM: kept at LOW, because the record is Go-written through one writer and only a hand edit or a torn write breaks it.

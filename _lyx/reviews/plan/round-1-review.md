---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-shared-relaunch-admit.md:42
    summary: the row named for a step after the goto is refused by the producer-is-target conjunct, so no arbiter row reaches the every-entry-is-a-goto conjunct D1 prescribes, and dropping that conjunct from admitRelaunch would pass every planned row
    fix: In card 2's TestConsumeArbiterAttempt row list, rewrite the row "moved by a goto and then stepped is refused" so only the every-entry-is-a-goto conjunct refuses it. Its mutate appends a `Plan-Write` goto entry, then a `Plan-Write` stuck entry (`shedengine.Stuck`), then a second `Plan-Write` goto entry, and sets `CurrentProducer` to `Plan-Write` and `State` to paused, with envelope goto `Plan-Write` and want "the run moved". Producer, state and last-goto-names-target then all hold, and only the step entry since the spawn refuses. Add one sentence saying the row isolates that conjunct.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/02-shared-relaunch-admit.md:45
    summary: TestConsumeTacklerAttempt gains the spawn column, but none of its three named rows sets it, so the card plans a dead test column
    fix: In card 2's TestConsumeTacklerAttempt paragraph, replace "gains the same two columns" with "gains the `wantHistory int` column" and drop the `spawn` column from the tackler test. Leave the arbiter test's `spawn` column as planned, since its pause rows use it.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/04-peers-row.md:29
    summary: card 4 plans a logger.Warn fallback in hubPeersDeps for a parse error of the embedded loom recipe, a branch no input reaches and no test names, and it turns a defect of the binary into a silent unknown order for every peer
    fix: In card 4's Intent, replace the clause starting "a parse error there is a defect of the binary itself, so it logs one `logger.Warn`" through "rather than refusing the verb" with "a parse error there is a defect of the binary itself, so it panics with the error; `loomrecipe`'s `TestRowNames_FollowEachRecipesRowOrder` already pins that the embedded recipe parses". This drops the fallback branch and its untested degrade path.
---

### [MEDIUM] The step-after-goto refusal has no row that reaches its own conjunct

**Location:** `_lyx/plan/02-shared-relaunch-admit.md:42`

**Issue:**
D1 admits the verdict-in-effect case only when the run stands paused at the goto target, every history entry since the spawn is a `goto` entry and the last names the target.
The bound lists "a step run after the goto" as refused, as today.
The planned row "moved by a goto and then stepped is refused" leaves `CurrentProducer` at `Plan-Bouncer` while the envelope's target is `Plan-Write`.
`admitRelaunch` therefore refuses it on `current.CurrentProducer == target` alone, which is the same reason the row "moved by a goto to another row is refused" pins.
No planned row has a non-goto entry since the spawn while the producer, state and last-goto conditions all hold.
A history like that is reachable: the parent goes to a row, the run steps and gets stuck, and the parent goes to the target again.
It passes `CheckGotoTarget`, since a paused run admits its own reference row (`internal/shedengine/goto.go`, `gotoAdmitted`), so `Admit` is reached.
An implementation that dropped the every-entry-is-a-goto check would pass every row card 2 plans.

### [LOW] The tackler test gains a column none of its rows uses

**Location:** `_lyx/plan/02-shared-relaunch-admit.md:45`

**Issue:**
The tackler test gains "the same two columns", `spawn` and `wantHistory`, and keeps only three new rows.
"Verdict already in effect" uses `wantHistory`.
"A pause added since the spawn" changes the status through `mutate`, not the attempt through `spawn`.
"Moved by a goto to another row" needs neither column.
The `spawn` column would be dead code in `internal/loomcli/tacklerconsume_test.go`.

### [LOW] The peers row fallback handles a state the binary's own test rules out

**Location:** `_lyx/plan/04-peers-row.md:29`

**Issue:**
`loomrecipe.RowNames(shedrun.RecipeLoom)` parses the embedded `recipes.LoomRecipe`.
`internal/loomrecipe/recipe_test.go`'s `TestRowNames_FollowEachRecipesRowOrder` already pins that it parses, so a built binary never takes the planned `logger.Warn` branch.
The card names no input that reaches the branch and no test for it.
If a defective binary did reach it, every `row` and `own_row` would read 0.
The alignment agent would then report every overlap as an unknown order instead of anything failing loudly.
The card's own text calls the error a defect of the binary, which is a programmer error to fail loud on, not an operational one to degrade on.

## Rejected

- Fidelity walk, no finding.
  Each decision has a card: D1 cards 1–2, D2 card 3, D3 card 5, D4 cards 6–8, D5 cards 4–5.
  Each constraint is carried too: refusal-spec rows in cards 2 and 3, the generic `shedengine` signal in card 1, `RefScope` through `refUnit`/`goPackageDir` in card 6, the stencil pointer rules in cards 5 and 8, tolerant digest records in card 7, the `# Peers` doc in card 4, the test budget in card 6, and every named doc.
  No card's scope lies outside the record; card 8's `loom-plan-spec.md` paragraph is a doc the D4 change carries.
- Card order, no finding: card 2 uses card 1's `inEffect` result, card 5 reads card 4's fields, card 7 reads card 6's `Finding.File`, and card 8 follows card 7.
- Card 2's in-effect admit is reachable: `Relaunch` runs `CheckGotoTarget` before `Admit`, and a paused run at the target admits its own row.
- Card 3's match uses `battenshed.RecordMatchesHalt`, which exists with the stated signature.
  The rows that stay eligible spawn without reading the producer list, since `CheckGotoTarget` runs only on the verdict.
- Edit glyphs and re-sign arrows, no finding.
  `postBatchChecks` carries the arrow and lists both callers.
  Every other changed member keeps its signature or is a struct field, and every constructor of `peersDeps`, `arbiterDeps` and the `Admit` closure is listed.
- CLI shape, no finding: card 4 adds output fields to an existing command, and no command, `Use` or audience changes.
- Verify, no finding: the section runs `./...` across every tier in template order.
- ImpactSummaries carry blast-radius conclusions, each matching the callers found in the code.
- Card 7's recover-batch row reaches the real guard, since `fakeDelta.ScopeGuard` calls `planglyph.ScopeGuard`.

---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-loom-config-zero-rounds.md:85
    summary: >-
      The yamlengine retag covers only a null leaf set to a value; lyx config --set of a per-perch budget back to null over a hub file holding an integer writes an explicit integer tag on null text
    fix: >-
      In card 3's `internal/yamlengine` bullet, state the rule on the text being set, not only on the leaf's tag:
      in `SetValues`, a value whose own text resolves to null, or a leaf whose tag is `!!null` at the point `.Value` is assigned, takes the tag the text resolves to (null for `~` and `null`, int for `2`);
      the template's tag cannot be the test in `SetValues`, since `applyExistingOverrides` has already copied the existing leaf's tag onto the working tree.
      `SetSparse` already takes the template leaf's tag, so it needs the same text-resolved tag for both directions.
      Add one row to `TestSetValues`, setting `plan_max_bounces=~` over an existing `plan_max_bounces: 2` and asserting the written document holds a null for the key with no `!!int` tag and decodes into an `*int` as nil;
      and make the `TestSetSparse` row set both `2` and `~` over the `~` template leaf.
      No target changes, so no validate-plan overlay is needed.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/03-loom-config-zero-rounds.md:83
    summary: >-
      The yamlengine SetValues and SetSparse change is a public-surface change in another package, reused by every module, bundled into the loom config card
    fix: >-
      Run `lyx loom insert-card` to add a card before card 3 (for example slug `yamlengine-null-leaf-set`) with Edit targets `internal/yamlengine#SetValues`, `internal/yamlengine#SetSparse`, `internal/yamlengine#TestSetValues` and `internal/yamlengine#TestSetSparse`, carrying the yamlengine bullet, its two test sentences and the F1 rule as its Intent, and a Commit line;
      then remove those four targets, the yamlengine bullet and the `TestSetValues` and `TestSetSparse` sentence from card 3, and drop the `yamlengine` mention from card 3's ImpactSummary.
      insert-card renumbers the later cards, so afterwards update every hard-coded card number in the Intents and Commit lines (for example "card 3", "card 8", "cards 3 and 4", "card 1's hard-below-soft refusal").
      Card 3's Intent should then say its `--set` of the null budgets relies on the earlier card.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/01-reviewbudget-package.md:61
    summary: >-
      CheckLocal as worded passes a null for any budget key, so a run_settings review_max_bounces null decodes to 0 and silently gives every perch zero rounds
    fix: >-
      In card 1's `CheckLocal` paragraph, say a nil value passes only for a per-perch soft key (`SoftKey(p)`); a nil run-wide key (`RunSoftKey`, `RunHardKey`) or a nil per-perch hard key is refused with `key "<key>": must be an integer`.
      Add a row to `TestCheckLocal` refusing a null `review_max_bounces`.
      Card 3's `TestPrepareRunSettings` needs no further row, since the bound matrix is `TestCheckLocal`'s.
      No target changes, so no validate-plan overlay is needed.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/07-arbiter-hold-reading.md:62
    summary: >-
      The TestHoldReading row for the window after the arbiter records its decision names no layout for the circling decision file, though the card names the escalation layout for the same reason
    fix: >-
      In card 7's last `TestHoldReading` row, name the decision file as `round-1-circling-decision.md` in Plan-Bouncer's run directory, with the frontmatter `round: 1` and `decision: continue`, which makes the escalation non-pending, as `circlingDecisionPath` and `readCirclingFile` in `internal/shedadapters` read it.
      The test sits outside `shedadapters` and cannot call those unexported helpers, so the card must give the name and keys, as it does for `round-1-review.md` and `round-1-escalation.md`.
---

### [MEDIUM] --set back to null writes an integer-tagged null

**Location:** `_lyx/plan/03-loom-config-zero-rounds.md:85`

**Issue:** Card 3 fixes `SetValues` and `SetSparse` so that setting `plan_max_bounces=2` over the template's `~` yields the integer 2 instead of `!!null 2`.
That direction is covered by the card's Edit targets (`internal/yamlengine#SetValues`, `#SetSparse`, `#TestSetValues`, `#TestSetSparse`) and by a named row in each test.
The reverse direction is not.
The settled design gives inherit as a null value, and the plan's own refusal wording (card 1's per-perch message "or to null to take review_max_bounces", card 3's refusal-spec row "null for the run-wide budget") tells the operator to set a budget to null.
In `SetValues` the working tree is the template tree, and `applyExistingOverrides` (`internal/yamlengine/reconcile.go:298`) copies the existing leaf's tag onto it before the pairs apply (`internal/yamlengine/set.go:97` and `:134`).
So a hub default file holding `plan_max_bounces: 2`, set with `--set plan_max_bounces=~`, reaches the assignment with an `!!int` leaf, and the card's rule, keyed on a leaf whose tag is `!!null`, does not fire; the encoder then keeps the explicit tag because the text resolves to a different one, writing `!!int ~`.
The base layer is written without validation (`internal/configengine/set.go:81`), so the hub's loom.yaml no longer reads as null for that key.
The card's sentence "an existing null leaf's tag is also carried by `applyExistingOverrides`" describes only the null-to-value case, and every test row starts from a null leaf.

### [LOW] yamlengine change bundled into the loom config card

**Location:** `_lyx/plan/03-loom-config-zero-rounds.md:83`

**Issue:** `yamlengine.SetValues` and `SetSparse` are public symbols of a package that every config module's `--set` reaches, with their own tests, and the change is independently reviewable: it needs nothing from `reviewbudget` or from the loom keys.
The rubric's granularity rule gives such a symbol its own card even when one card is its only consumer.
Card 3 is the largest card (27 Edit targets across five packages), and the yamlengine fix is the one unit in it with no dependence on the rest.

### [LOW] CheckLocal passes a null for every budget key

**Location:** `_lyx/plan/01-reviewbudget-package.md:61`

**Issue:** The card says a per-perch soft key whose value is nil passes, and that "any budget key whose value is neither an `int` nor nil is refused".
Read literally, nil passes for `review_max_bounces` too, and the test rows name only the per-perch null.
A task's `run_settings` of `{"loom": {"review_max_bounces": null}}` then clears `lyx board upsert`, merges over the hub value as null, and decodes into the `int` field `Config.ReviewMaxBounces` as 0 (yaml.v3 leaves a non-pointer int at zero for a null), which the plan gives the meaning zero rounds for every perch without its own key.
Before this change the same input reached load as 0 and was refused below 1.
The record accepts a deliberate 0 per run (point 8); a null is not a deliberate 0, and only the per-perch soft keys are pointers that give null a meaning.
A blanked key in a hand-edited hub file reads 0 the same way; that residual is not in the plan's scope, and the fix limits itself to the new per-run channel.

### [LOW] Decision-file layout unnamed in the hold-window test row

**Location:** `_lyx/plan/07-arbiter-hold-reading.md:62`

**Issue:** The row that reaches the window the focus names (decision recorded, marker removed, resume not yet applied) needs a circling decision on disk that makes the escalation non-pending.
`internal/arbiterstop`'s test cannot call `shedadapters`' unexported path helpers, and the card names the escalation files precisely for that reason but writes only "a round-1 circling decision file".
The file is `round-1-circling-decision.md`, and `readCirclingFile` (`internal/shedadapters/circling.go:180`) refuses one whose `round` disagrees with the filename or whose `decision` is neither `accept` nor `continue`, so a test author guessing the layout gets a malformed-decision error, which `Check` reads as an unreadable escalation and the exception does not fire.

## Rejected

- Focus 1, the null-to-integer path: setting an integer over a null leaf through `lyx config --set` (`configengine.Set`, `yamlengine.SetValues` for the base layer, `SetSparse` for the layers above) is covered by card 3's Edit targets and by one row in each of `TestSetValues` and `TestSetSparse`; the only gap is the reverse direction, F1.
- Focus 2, the hold across the arbiter's own decision: not a defect.
  `resumePrecondition` (`internal/loomcli/arbiter.go:423`) requires the circling decision recorded before a resume, and `PendingEscalation` then reads the escalation as non-pending, so `Check` refuses; card 7's exception (an attempt at the stop key, no applied mark) keeps `Eligible` true, and `finish` (`:460`) removes the marker before writing the envelope while `MarkArbiterApplied` runs only at consume (`internal/loomcli/arbiterconsume.go:59`).
  Card 8's `stopHold` then holds on `AppliedAt` zero, `HandOnAt` zero for a `resume` envelope and no park marker, until the applied mark or the backstop.
  Two named tests reach the window: the last row of `TestHoldReading` for the reading and the "arbiter marker is gone and the attempt's envelope carries `resume`" row of `TestNotice_EligibleStopHold` for the hold.
  The one remaining gap is F4.
- Focus 3, the Migrate scope: the card does narrow the settled design's point (4), and it states the narrowing as a departure with its reason (card 3 lines 67-70).
  The reason holds: `configsync.reconcileSparseFile` (`internal/configsync/configsync.go:408`) and `PrepareRunSettings` (`internal/configreg/runsettings.go:60`) both call `Module.Migrate` on bytes alone, a sparse worktree file and a `run_settings` document are indistinguishable there, and a `run_settings` 0 must keep its new meaning.
  Keying the rewrite on the old schema's top-level `skip_merge_review` is a sound way to fire it on the hub's default file; not a defect.
- Focus 4, Routing against the progress projection: consistent.
  A zero-round segment's rows carry `MaxBounces` 0, `shedengine.effectiveMaxBounces` (`internal/shedengine/run.go:634`) reads 0 as inherit, and `loadRouting` leaves the shed-level default at 0, so `lyx loom status` prints the engine default for such a perch; card 2 states this in the `Routing` bullet, in `doc.go`, and as an exception to the `Routing` doc's "never reports the engine default" sentence.
  The Bouncer's own `budgetReached` is not reached at zero, since the `Skip` seam runs first (`internal/shedadapters/bouncer.go:319`).
- Test economy for `TestEscalationEligible`: it overlaps the three new `TestArbiterVerb_SpawnPath` rows on the cause rule, but at a different public surface (the Go driver's fact against the verb), so no finding.
- Verify coverage, card order, member glyphs, re-sign arrows, `ImpactSummary` content and test-budget counts were each checked against the repo and raised nothing; the `arbiterstop` import set links no tree-sitter (`go list -deps` for each named package).

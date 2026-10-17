---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/06-review-rows-read-segments.md:112
    summary: Card 6 grows PATTERN.md past the 15564-byte headroom line, so its card gate (lint-pattern) and every later verify gate refuse the branch
    fix: >-
      In card 6, after the paragraph that prescribes the `PATTERN.md` entry line, add one sentence to the `**Intent:**`
      saying that `PATTERN.md` ends at no more than 15564 bytes (`pattern.HeadroomBytes`),
      so the card also shortens existing entry lines by at least 264 bytes in total,
      keeping the declared entry line verbatim and dropping only words the entries' linked background files already say.
      The basis is that `PATTERN.md` is 15560 bytes today, the declared line is 268 bytes with its newline, and 15828 minus 15564 is 264.
      `lyx loom lint-pattern` confirms the result.
      `PATTERN.md` is already a target of the card, so no target changes.
      The combined `lyx loom validate-plan --overlay` run of this fix with F2 to F4 reported no new blocking finding against the `{}` baseline.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-configreg-setting-lookup.md:31
    summary: Card 1 has the unregistered-module row assert the key in the error, but the message it prescribes for that case names only the module
    fix: >-
      In card 1, replace the sentence "The unregistered-module and unknown-key rows assert an error containing the module, the key and `way forward`."
      with "The unregistered-module row asserts an error containing the module and `way forward`, and the unknown-key row an error containing the module, the key and `way forward`."
      Item 1 of the Intent keeps its message, which carries no key.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/05-wire-segments-and-setting.md:24
    summary: Card 5 calls loomSegmentPerches the one table a new loom perch adds a row to, while card 8 keeps loomPerches as a second perch table
    fix: >-
      In card 5's Loom wiring bullet for `loomSegmentPerches`, replace "its comment names it the one table a new loom perch adds a row to"
      with "its comment names it the table `loomSegmentSettings` and `loomSegmentBudgets` read, and says a new loom perch also adds a row to `loomPerches`, which the hard-cap lookup keeps".
      Card 7's and the decision record's way-forward texts for `Routing` stay as they are.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/03-shedrecipe-segment-settings.md:23
    summary: Card 3 rewrites the Env opening comment, but contracts/specs/shed-recipe-spec.md line 42 still states the replaced rule that Env never holds a value that differs between two rows
    fix: >-
      In card 3's `**Edit:**` list, add the file target `contracts/specs/shed-recipe-spec.md` after `internal/testkit/envkit#nilLegal`.
      In the `**Intent:**`, add that the spec's sentence beginning "The rule that makes the `Env`-versus-`Config` split decidable" is reworded to the new rule:
      `Env` holds roots, run-wide values and the per-segment settings of `Segments`, keyed by segment so a segment's rows share one entry,
      and anything per-row is a relative path or scalar in `Config`.
      The spec is not a registered spec, so no verify line changes.
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/01-configreg-setting-lookup.md:29
    summary: Card 1 writes the default-file path as ConfigFile of the board _lyx dir, but ConfigFile takes the directory that holds _lyx
    fix: >-
      In card 1's `TestSettingLookup` paragraph, replace `configengine.ConfigFile(<board _lyx dir>, "loom")` with `configengine.ConfigFile(<board dir>, "loom")`,
      since `ConfigFile` joins `_lyx/config/<module>.yaml` onto its first argument.
---

### [BLOCKING] Card 6 grows PATTERN.md past the headroom line

**Location:** `_lyx/plan/06-review-rows-read-segments.md:112`, with `_lyx/plan/00-overview.md` (`## PATTERN changes`).

**Issue:** Card 6 inserts the declared `PATTERN-recipe-setting-lookup` line into `PATTERN.md`.
`PATTERN.md` is 15560 bytes at `HEAD`, and `pattern.HeadroomBytes` is 15564 (95% of the 16384-byte cap in `internal/pattern/check.go`).
`pattern.Headroom` reports a finding when the head is longer than both the base and the headroom line, and names the bytes to cut.
The declared line is 268 bytes with its newline, so the card leaves the file at 15828 bytes and the finding asks for a cut of 264.
The finding is mechanical and fires at two sites:
- The card gate: the plan spec's Tier 1 and `PATTERN-test-speed` run `lyx loom lint-pattern` after the comment lint on every card, and card 6 is the only card that edits the overview.
- Every later verify gate, through `patternlint.Lint` in `internal/loomshed/verifygate.go`, which refuses the branch with "so no test ran".

No card trims the overview, and the writer's stencil names only the per-line cap (`pattern-entry-line-cap`), which the 265-character line passes, so the plan cannot reach green at card 6.
The declared line is fixed by the decision record and must stay verbatim, so the room has to come from other overview lines.
The format check `pattern-changes-mismatch` is unaffected, because the overview's `## PATTERN changes` bullet already matches.

### [LOW] Card 1 asserts a key the unregistered-module error does not carry

**Location:** `_lyx/plan/01-configreg-setting-lookup.md:31`.

**Issue:** Item 1 of the Intent gives the unregistered-module message as `configreg: setting module "lom" is not a registered config module; way forward: name one of <Names()>`, which names the module and no key.
The test paragraph then says the unregistered-module and unknown-key rows both assert an error containing the module, the key and `way forward`.
An implementer who follows both either adds the key to the message item 1 specifies or writes a row that fails.
The unknown-key message does name the key, so only the unregistered-module row needs the narrower assertion.

### [LOW] Two perch tables remain after card 8

**Location:** `_lyx/plan/05-wire-segments-and-setting.md:24`, against `_lyx/plan/08-retire-row-maps.md:33`.

**Issue:** Card 5 has the comment on `loomSegmentPerches` call it the one table a new loom perch adds a row to.
Card 8 keeps `loomPerches` (Bouncer row name to perch) because `perchHardCap` in `internal/loomcli/circling.go` resolves a CIRCLING verb's hard cap through it, and that function refuses a Bouncer row the table lacks.
A new perch therefore needs a row in both tables, and the comment card 5 prescribes would be false from the day it lands.
The decision record's Routing way forward (add the segment's row to the segment-to-perch table) stays correct for the budget path, so only the comment claim changes.

### [LOW] shed-recipe-spec.md keeps the rule card 3 replaces

**Location:** `_lyx/plan/03-shedrecipe-segment-settings.md:23`, against `contracts/specs/shed-recipe-spec.md:42`.

**Issue:** Card 3 rewrites `Env`'s opening comment, which today says `Env` carries roots and run-wide values only and never a value that differs between two rows.
`contracts/specs/shed-recipe-spec.md` line 42 states the same rule as the contract of the `Env`-versus-`Config` split, and no card touches it.
After landing, the contract says `Env` never holds a per-row value while `Env.Segments` holds per-segment ones, and the spec is a "kept on landing" document.
The decision record's scope lists `internal/shedrecipe/doc.go` and the comment, and the fix adds the one contract sentence that restates the comment.
The spec is not registered in `contracts/specs/specs.go` (the registry holds the plan, final-summary and discussion specs), so no test reads it and `verify:` needs no new line.

### [NIT] Card 1 names the wrong base for ConfigFile

**Location:** `_lyx/plan/01-configreg-setting-lookup.md:29`.

**Issue:** `configengine.ConfigFile(baseDir, module)` returns `<baseDir>/_lyx/config/<module>.yaml`.
The card's `<board _lyx dir>` read literally yields `_lyx/_lyx/config`; the argument is the board dir that holds `_lyx`, which the same sentence already says each fixture dir has.

## Rejected

- Granularity of `shedrecipe.SegmentSettings` and `envkit.TemplateSetting`: each sits on its own card beside the `Env` fields or `FullEnv` fill it serves, and `TestFullEnv_FillsEverySeam` stays green at every card boundary (card 3's `nilLegal` entries cover the unfilled fields until card 4, card 8 removes the `Row*` entries together with their fields).
- Card order and Card Index: every `plan:` Use resolves to a symbol an earlier card creates (cards 4, 5, 6 and 7 against cards 1, 2, 3, 4 and 5), and the index lines match the eight card files.
- `verify:` coverage: every package a card targets, `internal/battencli` and `internal/battenrecipe` included, appears on both `lyx gate test` lines, and the hub list is `none`, so only the coverage rule applies.
- Attack surface of the `_setting` keys: card 6's Intent states the read surface (any template key of any registered module, a guard key included) and its bounds (embedded recipes only, read-only lookup, `configengine.Load` refusing a guard key in the local layer, the exact-reference pin in the D7 test).
- Refusal reachability on cards 6 and 7: each prescribed refusal has a named input and a named test, the `configString` row asserts only the text `configString` emits, the `Load`-propagated rows in card 1 assert the module and key and no way forward, and the `segmentBudgets` and `checkSegmentBudgets` rows assert the way forwards the cards prescribe.
- Round 1's fixes and its fixer report (read after this review was saved): none regressed, and the report disputes and defers nothing, so no item is re-evaluated.
  F2 above sits beside round 1's F7 split, which prescribed the same assertion for the unregistered-module row; the message it asserts against was already in card 1.
- The `Constructor` re-sign in card 2 lists all registry entries and every test member whose body calls an entry; no caller outside `internal/shedrecipe`, `internal/shedbuild` and the loomcli integration fixture exists.

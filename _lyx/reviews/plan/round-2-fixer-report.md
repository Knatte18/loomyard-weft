# Fixer report — plan, round 2

## Changes

- **F1 — fixed differently.** I confirmed the premise: the declared `PATTERN-recipe-setting-lookup` line is 321 runes against `MaxEntryLineChars` 320 (`internal/pattern/check.go:27`, `:117`).
  Card 15's Docs paragraph now writes the declared text with the one word "told" left out, which makes a 316-character line.
  The paragraph quotes the written text, names the departure and the cap as its reason, and leaves the overview's `## PATTERN changes` on the decision record's text.
  I did not make the proposed "asks `ly:orch` to amend the decision record's declared text".
  `patternChangeMismatches` (`internal/loomshed/plancontext.go`) needs the overview and the record to carry identical text, and the proposal leaves the overview as it is.
  So an amended record alone would raise `pattern-changes-mismatch` at the next plan gate.
  The departure is reported here instead, so the loop owner and `ly:orch` see it.
  No mechanical check binds `PATTERN.md`'s written text to the declared text: `DeclaredPatternChanges` feeds only prompt directives.
- **F2 — partly fixed, rest deferred (see Deferred).** Card 10's open obligation now holds the measured arithmetic and a named cut:
  - The plan grows `PATTERN.md` by 371 bytes: 70 + 112 + 189.
  - The headroom lint, measured against the merge base (`internal/loomshed/verifygate.go:103-110`), admits 8 bytes, so 363 bytes are owed.
  - The card lists a 152-byte wording-only cut across 17 entries outside the three declared ones, each with its new text.
    Every entry keeps its rule, its `(test)` marker and its background link.
  - That leaves 211 bytes owed, so the card now stops explicitly before any `PATTERN.md` edit and asks `ly:orch` for the room.
    It no longer offers a mid-card question as a fallback.
- **F3 — fixed as proposed.** Card 15's last test bullet now states each of these:
  - `envkit.nilLegal` gains `Command.Setting` and `Command.ExecutableSetting`.
  - `FullEnv` fills `ExecutableSetting`, the func fields `Command.TaskDir`, `LogPath` and `Run`, and the string facts.
  - `Forced` is a `[]string`, which `walkNil` never reports. Card 14 now types it as `Forced []string`.
  - battencli's `wire` fills the func fields, so it needs no `intentionallyNil` entry.
  - loom's `intentionallyNil` drops the six switched entries and gains `ExecutableSetting`, `Command.TaskDir`, `Command.LogPath` and `Command.Run`.
- **F4 — fixed as proposed (drop the link).** Card 10's tally counts the `PATTERN-command-row` line with no background link.
  Card 15 says the entry carries no link, with the reason: `Check` refuses a link to a missing file (`check.go:127`).
- **F5 — fixed as proposed.** `./internal/loomrecipe` is appended to both `go test` lines of the overview's `## verify:`.
  `lyx loom validate-plan` reports only its two informational findings.
- **F6 — fixed as proposed.** Card 8's Intent now says:
  - A flag stores the side's path and the two slugs only.
  - A holder's respawn sets `WorkDir` to `BoardDir` of the hub, the parent of the records worktree, and ignores any stored value.

  `TestCoalesceFlagged_PendingFlags`'s respawn row asserts that `WorkDir`, and `internal/fabricengine#BoardDir` joins the Uses list.
- **F7 — fixed as proposed.** Card 15's Intent wraps `configString`'s and `configInt`'s typed-accessor errors with the row, the key and the `way forward:` clause.
  `TestCommandEntry` gains a non-string `switch` (`3`) row.

## Self-check

- fix-consistency, LOW: in card 8, step 4 and the "What this lets through" line still said the holder respawns the flag's request as stored.
  Fixed: both now say the respawn's `WorkDir` is set to the board directory.
- fix-consistency, LOW: card 15 names `configString` and `configInt` but its Uses list did not.
  Fixed: both are added to Uses.
- fix-proposal: no findings.
- fix-claims: no findings.
- fix-affected-set: no findings.
  Its out-of-lens note named `internal/shedrecipe/conformance_test.go:84-86` and `internal/battencli/run_test.go:122` as untargeted switched-seam references.
  Not a finding: they sit inside `minimalEntries` and `newFakeReceiver`, which card 15 already edits.

## Disputed

None.

## Deferred

- **F2, the remaining 211 bytes of `PATTERN.md` room.** Every rule-preserving wording cut I found adds up to 152 bytes, and the plan owes 363.
  The rest can only come from removing or merging entries, or moving `HeadroomBytes` in `internal/pattern/check.go`.
  Each of those changes a rule or a guard that the decision record, which declares only three PATTERN changes, does not license.
  So it is an operator decision for `ly:orch`, and card 10 stops for it before editing `PATTERN.md`.
  Amending the plan with that decision beforehand removes the stop.

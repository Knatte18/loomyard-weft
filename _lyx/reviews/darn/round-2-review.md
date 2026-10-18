---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/landing/summary.md:25
    summary: The change description claims a fixed flaky creel test, but the range holds no creel hunk; the fix was reverted in a later commit.
    fix: In _lyx/landing/summary.md delete the bullet "A flaky creel test is fixed." together with its two continuation lines, so the description matches the diff, which touches nothing under internal/creelengine.
  - id: F2
    severity: LOW
    class: scope
    location: PATTERN.md:11
    summary: The diff rewords three unrelated invariants (durable-vs-ephemeral-state, config-strictness, push-both-sides) and drops "exactly" from config-strictness, a change the board entry does not license.
    fix: In PATTERN.md restore the original text of the PATTERN-durable-vs-ephemeral-state, PATTERN-config-strictness and PATTERN-push-both-sides lines, and keep the overview under its headroom line some other way, for example by shortening the new PATTERN-shed-recipe-registry line, or leave the three rewordings and add a sentence to the change description naming them as headroom trims that keep each rule's meaning ("exactly one" restored for config-strictness).
---

### [MEDIUM] Change description claims a creel fix the diff does not contain

**Location:** `_lyx/landing/summary.md:25`
**Issue:** The "What changes" list says `TestScheduler_Reminders/HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry` was fixed.
The range holds commit `4edda33eb` adding that change and commit `01b4e9061` ("creel: fix F5 — drop the hold-test probe change, unrelated to the implementer seam") removing it.
`git diff --stat` over the range shows no file under `internal/creelengine`.
The rubric makes a description claim the diff does not bear out a finding.
The `internal/creelengine/deliver_test.go` hunk the focus asks about is absent from the final diff, so the scope check on that file is clean; only the description is stale.

### [LOW] Unrelated PATTERN.md entries reworded

**Location:** `PATTERN.md:11`
**Issue:** Besides the `PATTERN-shed-recipe-registry` entry the task needs, the diff rewrites `PATTERN-durable-vs-ephemeral-state`, `PATTERN-config-strictness` and `PATTERN-push-both-sides`.
They look like headroom trims from the round-1 F1 self-check fix.
The board entry licenses none of them, and `config-strictness` loses "exactly" in "adopts exactly one of", which weakens the stated rule.
The change description does not mention the trims.

## Checked and found sound

- `map[string]Constructor` declarations in `internal/shedrecipe` production code: `registry` in `registry.go` and `implementers` in `entries_implementer.go`.
  The `PATTERN-shed-recipe-registry` entry ("bar the `Implementer` table"), the `registry.go` header and the `registry` comment describe exactly those two truthfully.
  The one other literal is test-only (`entries_gate_test.go`).
- Pass-through: `implementerEntry` clones the Config and deletes `implementer_setting` before calling the constructor.
  `TestImplementerEntry_ForwardsTheRestOfTheRowConfig` pins the accepted case (`gates` plus `implementer_setting` builds, which could not pass if the key reached `configRejectUnknown(cfg, "gates")`) and the refused case (`bogus` is refused by the constructor).
- Non-string and empty `implementer_setting` are refused before `readSetting` with messages naming the way forward; `TestImplementerEntry_RefusesAnUnusableSetting` has a row for each, plus absent, unknown, empty, non-string and null setting values, lookup error and nil lookup.
- `TestConfigTemplate_ImplementerDefaultsToWebsterAndAcceptsALocalValue` pins the template default `webster` and the declared category none, checking presence so a dropped entry fails.
- The refusal's `lyx config loom --set implementer=<implementer>` form matches the usage `configcli` prints elsewhere.

---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: PATTERN.md:33
    summary: Commit 6 rewrote nine unrelated PATTERN entries to shorter text, dropping qualifiers; only PATTERN-recipe-setting-lookup is declared
    fix: In PATTERN.md restore to their text at merge-base 89b854c85 (equal to main) the entries PATTERN-mutation-record, PATTERN-plan-generation, PATTERN-gate-self-check-parity, PATTERN-creel-store, PATTERN-friction-capture, PATTERN-edit-directive, PATTERN-wait-mark-display-only, PATTERN-spawn-observability, PATTERN-testkit and PATTERN-documentation-lifecycle; keep only the added PATTERN-recipe-setting-lookup line and the pattern/PATTERN-config-strictness.md change.
    origin: lens:code-pattern-fit
  - id: F2
    severity: LOW
    class: design
    location: internal/shedrecipe/entries_settings.go:92
    summary: fanSetting refuses a fan list holding a null entry, which the old typed decode treated as an empty name meaning solo
    fix: In the []any case of fanSetting treat a nil element as the empty string instead of refusing it, and add a row for a list such as [null, "x"] to the existing fan-setting table test.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: consistency
    location: internal/loomcli/wiring.go:946
    summary: Comment on loomSegmentPerches claims loomPerches keeps the hard-cap lookup, but that lookup was deleted and loomPerches is now read only by circling
    fix: Reword the comment to say loomPerches keys the circling lookup by Bouncer row, and drop the hard-cap claim.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: scope
    location: internal/shedrecipe/entries_settings.go:55
    summary: New refusals for a malformed module.key reference, a wrong-shape fan setting and a non-bool auto-accept setting have no test
    fix: Add rows to the existing shedrecipe refusal tables for a reference with no dot, an empty module and an empty key, for a fan setting holding an int and a mixed list, and for an auto-accept setting holding a string, each asserting the message.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: consistency
    location: internal/shedrecipe/entries_burler.go:194
    summary: FixFanKey is set to the citation "key in module.yaml", and burlerengine words it as a profile field to remove or a key to move
    fix: Set opts.FixFanKey to the setting's key name (fixFanRef.Key) so the burlerengine refusals at fix.go:322 and fix.go:325 read as a config key, and adjust the test that pins FixFanKey.
    origin: handler
---

### [BLOCKING] Unrelated PATTERN entries rewritten

**Location:** PATTERN.md:33 (and the entries listed in the fix)
**Issue:** Commit 6 shortens ten existing entries, and the matching pattern/ background files are untouched. The task declares only PATTERN-recipe-setting-lookup, and main's PATTERN.md still carries the longer texts. Landing it would drop qualifiers such as "orch stencils are exempt" and "never duplicated" from invariants.

### [LOW] Null entry in a fan list refused

**Location:** internal/shedrecipe/entries_settings.go:92
**Issue:** `[~, "x"]` used to decode to an empty name, meaning that round runs solo. It now fails the row's construction.

### [LOW] Stale comment on loomSegmentPerches

**Location:** internal/loomcli/wiring.go:946
**Issue:** The comment names a hard-cap lookup that this diff deleted.

### [LOW] Untested refusals

**Location:** internal/shedrecipe/entries_settings.go:55
**Issue:** The malformed-reference, wrong-shape and non-bool refusals are behavior the diff adds with no test. The registry's passing of Segment into RowIdentity (internal/shedbuild/build.go:43) is also not asserted directly, only through the review-entry refusals, so it is not raised separately.

### [LOW] FixFanKey carries a citation

**Location:** internal/shedrecipe/entries_burler.go:194
**Issue:** The value `webster_fixer_fan in loom.yaml` is spliced into messages that expect a key, such as "remove %s from the profile".

## Rejected

- Card 8 leftovers: grep over the repo finds no reference to the removed Env maps, RowModelKeys or burlerRowModels, including battencli and battenrecipe; the Delete group's assert-no-callers holds.
- Declared PATTERN change: PATTERN-recipe-setting-lookup lands at PATTERN.md:49 with the declared text.
- No Pattern-Departure trailer appears in the range's commits.
- Env.Segments and Env.Setting are in envkit's nilLegal, and TemplateSetting fills FullEnv.
- Budgets keyed by segment use the loomcli segment names with no new shared constant.

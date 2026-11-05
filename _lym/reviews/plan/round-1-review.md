---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/plan/07-report-config.md:11
    summary: Card 7's Create target plan:internal/reportconfig#TestLoadConfig names the wrong package; the landed test is declared in the external package reportconfig_test, so Webster's recover-batch 07 refuses again with create-not-done
    fix: "In _lym/plan/07-report-config.md, replace the Create line `plan:internal/reportconfig#TestLoadConfig` -> `func TestLoadConfig(t *testing.T)` with `plan:internal/reportconfig_test#TestLoadConfig` -> `func TestLoadConfig(t *testing.T)`, the external-package form cards 13, 14 and 17 already use. Withheld under the overlay rule: validate-plan --overlay reports create-already-exists/7-report-config for the corrected target, which the baseline does not, because the test already exists (commit 5ec85b9c). The baseline reports the same create-already-exists finding for every other card-7 target, so the constraining check is create-already-exists, and the fixer applies this line together with however it clears those siblings."
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/07-report-config.md:13
    summary: Card 7 makes internal/reportconfig a LoadOrTemplate caller but does not list pattern/PATTERN-config-layers.md, whose Degrading callers set names sibling hibernateconfig and not reportconfig
    fix: "In _lym/plan/07-report-config.md, add `pattern/PATTERN-config-layers.md` under **Edit:** after `cmd/lym#configStrictnessDegradingSet`, and add one Intent line: `pattern/PATTERN-config-layers.md`'s Strictness section, whose Degrading callers set lists the packages that load with LoadOrTemplate, gains `reportconfig`. Checked with validate-plan --overlay: no finding beyond the baseline."
---

The amendment base `cde2a832` is reachable in the weft repository (`git -C _lym`), so this review judged the real diff.
The committed part of the change drops the `plan:` prefix and signature from the Create and Uses targets of the done cards 02–06 and from card 26's Uses.
That is Webster's rebaseline of landed work and raises nothing.
The uncommitted part changes cards 07, 08, 23 and 25:

- Card 07 moves the config module to `internal/reportconfig`.
- Card 08 points its Uses and Intent at `reportconfig.LoadConfig`.
- Card 23 moves its rewritten doc files from **Extends** to **Edit** and names each recipe list they gain.
- Card 25 moves `pawlengine/doc.go` to **Edit** and names its changed sentence.

The amendment entry "The report config module lives in internal/reportconfig" is carried by cards 07 and 08.
Card 07 sits in failed batch 7, so it can still be edited.

### [BLOCKING] Card 7's test target names the internal package

**Location:** `_lym/plan/07-report-config.md:11`

**Issue:** `lym quarry resolve internal/reportconfig#TestLoadConfig` answers `not_found`, and `internal/reportconfig_test#TestLoadConfig` answers `found`: `internal/reportconfig/config_test.go:3` declares `package reportconfig_test`.
Batch 7 halted on create-not-done for card 7's targets (`_lym/loom/friction/loom-halt.md`).
The amended card fixes every target but this one, so `lym webster recover-batch 07` would refuse again on the same check.
The `validate-plan --overlay` baseline lists create-already-exists for card 7's other Create targets and omits this one, which shows it does not resolve.
The decision record's amendment entry spells it `reportconfig#TestLoadConfig`, but the entry is about which directory the module lives in.
The card has to name the test where the landed code declares it.

### [LOW] PATTERN-config-layers background omits the new degrading caller

**Location:** `_lym/plan/07-report-config.md:13`

**Issue:** `pattern/PATTERN-config-layers.md:63` lists the degrading callers of `configengine.LoadOrTemplate`.
The list includes `hibernateconfig`, whose shape card 7 copies, but not `reportconfig`.
`cmd/lym/configstrictness_test.go:64` pins `internal/reportconfig` in the degrading set, and no card lists the background file.
The list was already incomplete, since `sheddriver` and `codexengine` are missing, and no test pins it, so this is LOW.

## Rejected

- repo-invariants BLOCKING: card 23 fills neither `envkit.FullEnv` nor the seam markers for its new `Env` fields. Out of amendment scope: the amendment changed only card 23's doc-file lines (moving them from Extends to Edit, plus the two doc-text Intent lines). Its `Env` fields, `**Uses:**` envkit lines and test list are unchanged since `cde2a832`, and this round reports nothing about the settled remainder.
- guard-reach F-GR1: no test row for `raddleWriteEntry`'s unknown-key refusal. Out of amendment scope: card 23 lines 99 and 110 are unchanged since `cde2a832`.
- guard-reach F-GR2: no test row for `createBeamFromParent`'s prime fallback. Out of amendment scope: card 23 lines 103 and 111 are unchanged since `cde2a832`.

## Focus departures

- "Amendment scope: base commit cde2a832 is not an object in this worktree, so `git diff` cannot name the change. Establish the changed span from file mtimes and content instead…": the base is an object in the weft repository that `_lym` belongs to. The changed span therefore comes from `git -C _lym diff cde2a832`, not from mtimes. That diff also shows rebaseline-only edits to cards 02–06 and 26.
- "23-raddle-recipe (largest card, changed): settle whether its targets form one reviewable unit or bundle independent surfaces, whether any `Edit` glyph lacks the re-sign arrow…, and whether each new verb, flag or guard states in `Intent` what it can skip": the card's change is limited to its doc-file targets and two doc-text Intent lines. Granularity, signatures and guards are settled remainder, so this round judged only the change: the move from **Extends** to **Edit** is correct, because both docs reword existing text.
- "25-pawl-raddle-handoff (changed): settle whether each prescribed refusal or guard branch names an input and a test that reach it": the card's change only moves `pawlengine/doc.go` to **Edit** and names its sentence, and it adds no refusal. The guard-reach lens found every card-25 guard reached by a named row.

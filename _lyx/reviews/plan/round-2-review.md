---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/06-webster-wiring.md:36
    summary: Card 6 makes language.yaml reachable through the unreadable-config refusal and the Master-stop rung, but updates only the invalid row, so the unreadable row and the websterengine doc.go section still name only webster.yaml and batcher.yaml
    fix: >-
      In card 6 (_lyx/plan/06-webster-wiring.md) make three edits.
      (a) Add the line "- `internal/websterengine/doc.go`" to the **Edit:** list, after "- `internal/webstercli#configLoadError`".
      (b) In the Intent paragraph that updates `contracts/specs/refusal-spec.md` (the one starting "What the new load can refuse"), also extend the webster row "config file unreadable" so its trigger names `language.yaml` beside `batcher.yaml` and `webster.yaml`; its class (transient) and way forward (re-run the verb) stay as they are.
      (c) In the same paragraph add that the "A broken config stops Master" section of `internal/websterengine/doc.go` names `language.yaml` beside `webster.yaml` and `batcher.yaml`: a present file with unusable content refuses with `config_invalid: true` and stops Master, an unreadable one refuses transient.
      Checked with `lyx loom validate-plan --overlay`: card 6 with the doc.go target added reports no blocking finding beyond the `{}` baseline.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-language-config-module.md:46
    summary: Card 1 claims loom's existing transient handling classifies an unreadable language.yaml as transient, but loom has no such handling for a wire error
    fix: >-
      In card 1 (_lyx/plan/01-language-config-module.md) replace the tail of the bullet "Any other `LoadOrTemplate` error ..." ("so webster's `configLoadError` default case and loom's existing transient handling classify it as transient") with "so webster's `configLoadError` default case classifies it transient, and loom's `wire` returns it as the plain arming error, which names the unreadable file and carries no `lyx config language` clause".
      In card 5 (_lyx/plan/05-loom-wiring.md), after the Intent sentence that adds the refusal row, add one sentence: an unreadable `language.yaml` is not that row's trigger, it surfaces as the arming error that the `step arming refused` row of the shed section already covers.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/05-loom-wiring.md:35
    summary: Card 5 puts the new row in the loom section beside hub config missing, but that row sits in the shed section and the loom section has no such row
    fix: >-
      In card 5 (_lyx/plan/05-loom-wiring.md) change "`contracts/specs/refusal-spec.md`'s loom section gains a row beside "hub config missing"" to "`contracts/specs/refusal-spec.md`'s shed section gains a row after "hub config missing"".
      That row sits in the shed section and covers loom arming; the loom section starts further down and has no row of that name.
      Keep the row's own text, whose trigger already names a loom verb's wiring.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/05-loom-wiring.md:35
    summary: The triggers of the new loom row and the extended webster row omit the local-override case that Load also turns into this refusal
    fix: >-
      In card 5 (_lyx/plan/05-loom-wiring.md) extend the new row's trigger, and in card 6 (_lyx/plan/06-webster-wiring.md) extend the trigger wording given for the "config file content invalid" row, each with "or a worktree local override file that holds `skills`, which the Guard classification refuses".
      Card 1's `Load` already returns that case as `ErrUnusable`, and its `TestLoad_InvalidFailsNamingKey` local-override row reaches it, so the rows' triggers should list it.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/01-language-config-module.md:44
    summary: The way-forward clause of Load's error lacks the way forward label that the refusal contract makes the carrier of the way forward
    fix: >-
      In card 1 (_lyx/plan/01-language-config-module.md) change the clause the error ends with from `fix it with "lyx config language" from the prime` to `; way forward: fix it with "lyx config language" from the prime`, the shape `batcher`'s retired-key error has.
      The refusal contract says the message carries its way forward as a trailing `way forward: ...` clause.
      Card 6's `configLoadError` message then reads `webster: <file> did not load: <cause>, then re-run the verb` with the label inside the cause, as the retired-key case does.
      Card 5's and card 6's row cells keep quoting the clause's content without the label, as the other rows do.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/08-burler-profile-key.md:22
    summary: Card 8 updates the burlerFixProfile doc comment but leaves burlerRoundProfile's comment stating seven recognised keys and six duplicated key names
    fix: >-
      In card 8 (_lyx/plan/08-burler-profile-key.md) extend the sentence "The fix-row doc comment on `burlerFixProfile` names its third recognised key." with: "`burlerRoundProfile`'s doc comment states eight recognised keys with `language-skills` among them, and that seven of the eight key names duplicate `internal/burlercli`'s `profileYAML`".
      The current comment says "exactly seven keys" and "Six of these seven key names", which the new key makes false.
---

### [MEDIUM] Card 6 leaves the unreadable-config row and the Master-stop doc naming two files

**Location:** `_lyx/plan/06-webster-wiring.md:36`
**Issue:** Card 6 sends a `language.yaml` load failure through `configLoadError`.
Its invalid case is covered: the card extends the "config file content invalid" row.
Its default case is reached too, since `Load` returns a present but unreadable file unwrapped and unmarked, and `configLoadError` then answers `webster: language.yaml could not be read: ...; way forward: transient, re-run the verb`.
The webster row "config file unreadable" (`contracts/specs/refusal-spec.md:49`) names only `batcher.yaml` or `webster.yaml`, so this refusal has no row naming its file, which `PATTERN-refusal-way-forward` asks for.
The same two-file wording sits in `internal/websterengine/doc.go:578` ("A broken config stops Master"), which the refusal makes stale for both branches.
CLAUDE.md puts the doc change in the same commit as the behavior change.
Card 3 already edits `internal/websterengine/doc.go`, but card 6 introduces the behavior, so the sentence belongs there.

### [LOW] Card 1's claim of loom transient handling has no counterpart in the code

**Location:** `_lyx/plan/01-language-config-module.md:46`
**Issue:** Loom's `wire` returns a load error unchanged, `armAt` returns it before any hook, and `shedtransient.Mark` runs only inside `loomPreStep`, after arming.
No loom code classifies an unreadable config file as transient; `landing.yaml` unreadable is a correctness halt in the shed section.
The webster half of the sentence is right, and the implementer of card 5 would otherwise look for handling that does not exist.

### [LOW] Card 5 places its row in a section that lacks the row it is placed beside

**Location:** `_lyx/plan/05-loom-wiring.md:35`
**Issue:** `contracts/specs/refusal-spec.md` holds "hub config missing" in the `## shed` section (line 174); the `## loom` section (from line 224) has no row of that name.
The card's own instruction contradicts itself, and an implementer must guess which section.

### [LOW] Row triggers omit the local-override failure

**Location:** `_lyx/plan/05-loom-wiring.md:35`
**Issue:** `configengine.Layers` runs `ValidateLocal` on a worktree's `.lyx/config/language.local.yaml` whenever the anchor is told, and refuses `skills` as a Guard key with an `ErrInvalid` error.
Card 1's `Load` turns that into `ErrUnusable` and its test row reaches it, so wiring in loom (card 5) and webster (card 6) refuses on it.
Neither card's row trigger lists it; both name only the hub file's content failures.

### [LOW] The way-forward clause has no `way forward:` label

**Location:** `_lyx/plan/01-language-config-module.md:44`
**Issue:** The refusal contract's "What a way forward is" section says the message carries its way forward as a trailing `way forward: <...>` clause, which the row cells quote.
`batcher`'s retired-key error (`internal/batcher/config.go:58`) and `hubConfigWayForward` both carry the label; card 1's clause does not, so the loom message and the webster message built from it lack it.

### [LOW] Card 8 leaves a stale key count in burlerRoundProfile's comment

**Location:** `_lyx/plan/08-burler-profile-key.md:22`
**Issue:** `internal/shedrecipe/entries_burler.go` documents `burlerRoundProfile` as recognising "exactly seven keys" and as duplicating six of them from `profileYAML`.
Card 8 adds an eighth key to both mappers and to `profileYAML`, and names only `burlerFixProfile`'s comment for update.

## Rejected

- Focus 1, card order across cards 4 and 5: not a defect.
  Card 4 gives `landingDeps` the `languageSkills` parameter, passes `nil` at its production callers and a non-nil list in `TestLandingDeps_EveryFieldPopulated`, so the reflection walk stays green in card 4's commit.
  Every `landingDeps` caller (`arm.go:429`, `sharedbootstrap.go:335`, and the five test callers) is listed.
  No other test reflects over `landingshed.Deps`, `websterengine.RunDeps` or `RecoverDeps`, and `envkit.NilSeams` ignores a nil `[]string`.
- Focus 3, attack-surface bounds: not a defect.
  Card 1's Intent states what the `skills` hatch lets through (raw build-skill commands outside the gate slots) and what bounds it (template comment, per-hub choice, Guard classification, verify gates).
  Card 8's Intent states that the `language-skills` key only adds skills to the seats' load turn and is bounded by D5, the Guard list and the absent-key default.
- Focus 4, reachability of `TestLoad_InvalidFailsNamingKey`'s local-override row: not a defect.
  `load` drops the anchor layer only when `FindBaseDir(anchor)` fails on a missing `_lyx/`; the row creates `configengine.ConfigDir(anchor)`, which creates `_lyx/`, passes both `Board` and `Anchor`, and `LayerFile` then applies the local layer for a hub-wide module, so `ValidateLocal` refuses `skills`.
- Focus 2, error classification: the webster path is right, since `configLoadError`'s new `ErrUnusable` case precedes the `ErrInvalid` case and the default case keeps an unmarked read error transient; the gaps found are F1, F2 and F4.
- Verify section, test budget and glyph arrows: no finding.
  The `## verify:` order matches the writer's template, every card's new top-level tests fit its package's budget row (card 4 raises `internal/landingshed`, card 1 adds the new row), and each changed signature carries the re-sign arrow.

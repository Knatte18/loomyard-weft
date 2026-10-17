---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:120
    summary: D7 pins only that rows naming a fan setting name the same reference, so a fix row or Bouncer can drop or keep its setting independently of the review row, and the fix row's recorded-round guard becomes bypassable by omission with no build-time check
    fix: In D7, extend the agreement test so that within each review segment of the embedded loom and darn recipes the BurlerReview row names fixer_fan_setting exactly when the BurlerFix row does, and names reviewer_fan_setting exactly when the Bouncer does, besides naming the same reference; state in D5 (beside the sentence at line 91) that a BurlerFix row naming no fixer_fan_setting reads no recorded fans and runs the single fixer, bounded by D7's presence agreement over the embedded recipes; add the presence half to the acceptance scenario that the rows of each segment name the same fan settings.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:133
    summary: DarnRouting's refusal way forward points at the wiring's segment-to-perch table, but D8 says the Darn-Review budget comes from darn.yaml through the darn wiring, not from that table
    fix: In D8 line 133, give the two refusals their own way forward, Routing naming the loom segment-to-perch table and DarnRouting naming the darn wiring's Darn-Review budget entry from darn.yaml's review_max_bounces.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:129
    summary: D8 says the negative-budget refusal names the key and file as reviewbudget.Caps records it, but Caps records only SoftKey and HardKey, no file
    fix: In D8 line 129, replace "as `reviewbudget.Caps` records it" with "the key from `reviewbudget.Caps` and the file from the segment's module (`loom.yaml` or `darn.yaml`)", matching the key-and-file fields D1 lists in SegmentSettings.
---

### [MEDIUM] Fan-setting agreement test misses presence, leaving the fix row's recorded-round guard bypassable

**Location:** `_lyx/discussion/decision-record.md:120` (D7), with D5 at line 91.

**Issue:** Today the wiring fills `RowFixFans` for every review row and `RowModelKeys.FixFan` for every fix row (`internal/loomcli/wiring.go` `reviewRowModelKeys`, `internal/loomcli/darnwiring.go` `wireDarnReview`), so every fix row is built with `RequiresFans` and every review row records its round: the pairing holds by construction.
Under D5, recording and requiring move to whether each row names its `_setting` key, and D7 checks only that the rows naming a key name the same reference.
A recipe whose BurlerFix row omits `fixer_fan_setting` while its review row names it passes D7.
Its fix row then reads no recorded names and runs the single fixer (`internal/shedadapters/doc.go`: a row not told `RequiresFans` reads no names), so a configured fixer fan is silently dropped, and the unrecorded-round refusal, a guard, is gone with nothing at build to catch it.
The reverse, a fix row naming the key under a review row naming neither, refuses every round at run time instead of at build.
The same holds for a Bouncer naming `reviewer_fan_setting` beside a review row that names no fan setting.
D7's rationale, that the test keeps the agreement the wiring gave by construction, therefore holds for reference only, not for presence.
Under the rubric's attack-surface bullet, D5's absent-means-today's-absent-entry rule makes this guard bypassable without a stated bound.

### [LOW] DarnRouting's way forward names a table the darn budget does not come from

**Location:** `_lyx/discussion/decision-record.md:133` (D8).

**Issue:** D8's first paragraph keeps `Darn-Review` out of the loom segment-to-perch table: the darn wiring adds that entry from `darn.yaml`, and the budget source sentence lists `darn.yaml`'s `review_max_bounces` beside `loomengine.Config.Caps`.
The shared refusal sentence still tells both `Routing` and `DarnRouting` to add the missing segment's row to the segment-to-perch table, which for a darn run points at the wrong place.

### [NIT] Caps does not record a file

**Location:** `_lyx/discussion/decision-record.md:129` (D8).

**Issue:** `reviewbudget.Caps` (`internal/reviewbudget/reviewbudget.go`) holds `SoftKey`, `Soft`, `HardKey` and `Hard`; the file is not part of it.
D1 already has `SegmentSettings` carry the key and file, so only the attribution in D8 is off.

## Focus verdicts

- Default-config fan path: not a defect.
  The template sets every perch fan key to `""` (`internal/loomengine/template.yaml`, `internal/darnengine/template.yaml`), which `ValidateFanKeys` accepts.
  D5 has a BurlerReview row naming a fan setting record its round whatever the value, and a BurlerFix row naming `fixer_fan_setting` require that record, so an empty value runs solo, records the round and the fix row runs the single fixer from it, as the acceptance scenario states.
  The review row and the fix row agree on that path; F1 covers the case where a recipe names the key on only one of them.
- D8 budget parity: not a defect for the path the record describes.
  Both the status read and the build take loom budgets from one segment-to-perch table through `loomengine.Config.Caps`, and darn's from `darn.yaml`'s `review_max_bounces`; the full wiring hands the status path the same loaded `Config` (`internal/loomcli/arm.go` `reviewBudget`), and the lightweight read differs only where the build would itself refuse an unreadable file.
  `Routing` and `DarnRouting` refuse a missing segment instead of defaulting it, so a status read cannot report a budget the build would not apply; F2 concerns only the darn refusal's wording.
- Null fan consistency: consistent.
  D5 treats null, `""` and an all-empty list as absent, D6 admits null as a shape, the acceptance scenario matches, and D5's hub-run statement holds: yaml.v3 decodes a null into a nil `FanList`, which `ValidateFanKeys` refuses as an empty list in both the loom and darn loaders.

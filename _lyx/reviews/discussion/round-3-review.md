---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:59
    summary: D2 refusal for a round whose usage record holds no fan names names re-stepping the review row, but that row hands an accepted review on without spawning and so never writes the names
    fix: In D2, replace the last bullet sentence about a fix half or Bouncer whose round's usage record holds no fan names. State that the refusal first archives the round's review, fixer report and acceptance record together, as the fix producer's changed-review exit already does through its whole-round archive, so that re-stepping the review row spawns a fresh review half that writes the names before its spawn. Alternatively state that the review half also writes absent names on its accepted-review hand-on path. Name which of the two it is, and add the matching acceptance criterion row with a test that re-steps a round whose accepted review has a record without names and ends with names recorded.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:57
    summary: D2 does not say how a reader tells absent fan names from a recorded empty name, nor that a later writer or an undecodable record can erase the names
    fix: In D2, state the presence rule the record uses. The existing fan field is a plain string without omitempty, so a solo round already decodes as an empty string and equals an absent key. Name a presence marker, for example pointer fields or a names-recorded flag, in the record's shape. State that a record which fails to decode when the fix half or Bouncer reads it counts as holding no names and takes the same refusal. State that the shared merge path keeps the recorded names across every later writer's merge, and that a record replaced because it did not decode leaves the names absent rather than empty. Add one acceptance criterion row for the decode-failure case.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:57
    summary: D2 says the names are written before the reviewer spawns but leaves the live-reviewer resume path and the existing acceptance-time write of the fan field unaddressed
    fix: In D2, state that the pre-spawn write happens only on a path that spawns a new reviewer, after the live-reviewer probe finds none, so a resumed live reviewer keeps the names it was spawned under. State that the acceptance-time merge stops writing the fan and lenses from the producer's profile and writes only the lenses and the review half, so a hub edit between spawn and acceptance cannot overwrite the recorded name. Add both to the acceptance criteria as rows of the existing fan-list-changed criterion.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:59
    summary: D2 makes a Bouncer refuse on absent names, which contradicts the Bouncer's tolerant lens-facts reading, and does not scope the refusal to segments that have fan keys
    fix: In D2, scope the refusal to the Discussion and Plan segments, whose review rows are handed the fan lists, and say what the review half of the Webster, Merge and Darn rows records, either nothing or empty names. Replace Bouncer in the refusal sentence with the one Bouncer read that needs the names, the judged round's fan for the exclude_lenses decision, and say whether it refuses or degrades. The lens-facts reader and the lens-eligibility reader today record a missing record on the facts file and treat it as ineligible without failing. Align the acceptance criterion at the end of the record with the choice.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:134
    summary: D7 hands the fixer fan list only to the BurlerReview row, so the BurlerFix row has no carrier for the key names its engine refusal must name, and the standalone refusal names nothing
    fix: In D7, state how the BurlerFix row learns the fixer model key and the fixer fan key it names in the fork-less engine refusal, for example a told per-row key pair beside the existing per-row model lists. Today the fix constructor takes no fan and the run options carry only a reviewer model key. State what the refusal names in lyx burler run, whose profile has a fix-fan key and no loom.yaml key, so the Burler spec row can quote one wording for both callers.
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:152
    summary: D8 fix-affected-set asks a fork for one search per finding while the inherited self-check block caps each fork at six further tool calls, and Open risks names the cap only against the Plan reviewer lenses
    fix: In D8, make fix-affected-set state what a fork does when the round's set-premised findings exceed its call cap, for example rerun the searches for the most severe findings first and list each finding left unchecked. In Open risks, extend the sentence on the six-call fork cap to cover the fixer fork cap against fix-affected-set, and state the cap once in D7 where the self-check block is adopted, since the record never states the number outside Open risks.
---

### [BLOCKING] D2 refusal's way forward cannot take effect

**Location:** `_lyx/discussion/decision-record.md:59`
**Issue:** D2 refuses a fix half or Bouncer whose round's usage record holds no fan names, "with the way forward to re-step the segment's review row".
In `internal/shedadapters/burlerreview.go`, `Call` hands an already accepted review on with `handOn` when `readAcceptance` reports it accepted and the review file exists, and spawns nothing on that path.
D2 writes the names "before it spawns the reviewer", so the hand-on path never writes them.
The refusal therefore returns the operator to a row that changes nothing, and the next fix half refuses again.
The case is reachable: a run in flight across the binary change has a record from the old binary and an accepted review, and a record replaced by the merge path (F2) is the same state.
The fix producer's existing way forward (`goto` the review row, then `resume`) works only because every exit that names it archives the whole round first (`archiveWholeRound` on a changed review or a failed gate); D2's new refusal has no such archive.
`PATTERN-refusal-way-forward` requires the message to name a way forward, and one that loops is not one.

### [MEDIUM] Absent names are indistinguishable from a recorded empty name

**Location:** `_lyx/discussion/decision-record.md:57`
**Issue:** D2 writes an empty name for a solo or single round and refuses on a record that "holds no fan names", but names no rule that separates the two.
`roundUsage.Fan` in `internal/shedadapters/roundusage.go` is a string tagged `yaml:"fan"` without `omitempty`, so a solo round decodes as an empty string and an absent key decodes the same.
`mergeRoundUsage` also archives an undecodable record and continues from a fresh one holding only the current writer's part, so a record that loses its names after the pre-spawn write comes back as a plausible, name-less record.
`readRoundUsage` returns an error for a malformed record, which D2 does not say is the same refusal.
D2 does state when the names are written and that a failed write fails the half, so those two parts of the question are settled; the reader side is not.

### [MEDIUM] Pre-spawn write against the live-reviewer resume and the acceptance write

**Location:** `_lyx/discussion/decision-record.md:57`
**Issue:** `Call` probes for a live reviewer (`probeLiveReview`) before its spawn loop and resumes it through `acceptedExit`, which today merges `u.Fan = p.profile.ClusterFan`.
D2 says the round's fans are "fixed when its review half starts" and read "from there, never from `loom.yaml`".
A review half that starts again on resume and writes before the probe records the edited fan for a reviewer spawned under the earlier one.
The acceptance-time merge, left as it is, takes the fan from the producer's profile, which D2's notes say becomes a per-round pick of the current config, and overwrites the recorded name.
Either way the record the fix half and the Bouncer trust can name a fan the reviewer did not run.

### [MEDIUM] Bouncer refusal against the Bouncer's tolerant reading and unscoped rows

**Location:** `_lyx/discussion/decision-record.md:59`
**Issue:** `computeLensFacts` in `internal/shedadapters/bouncerfacts.go` records a missing or unreadable usage record as `UsageErr` and never fails, and `lensesRunUnderFan` in `focus.go` logs a warning and treats the round as ineligible.
D2 makes the same condition a refusal for the Bouncer without saying which of these reads it replaces.
D2 also does not say whether the Webster, Merge and Darn rows, which have no fan key and no fixer key, record names; if they do not, the refusal reads as applying to every Bouncer and stops those segments.

### [LOW] No carrier for the fix row's key names

**Location:** `_lyx/discussion/decision-record.md:134`
**Issue:** `burlerFixEntry` in `internal/shedrecipe/entries_burler.go` takes no fan and sets no model key, and `RunOpts` has only `ReviewKey`.
D7 routes the fixer fan list to the BurlerReview row and has the BurlerFix row read the recorded name, which carries the fan end to end for a shed run.
The refusal D7 specifies also names the fixer model key and the segment's fixer fan key, and nothing in D7 or D2 delivers those to the fix row.
For `lyx burler run`, which D7 says reads `fix-fan` from its profile, the refusal has no loom.yaml key to name and the record does not say what it names instead.

### [MEDIUM] Fixer fork cap against fix-affected-set

**Location:** `_lyx/discussion/decision-record.md:152`
**Issue:** D8 `fix-affected-set` reruns "the finding's own search, one tool call per finding".
D7 adopts the bench's self-check block, whose fork boilerplate allows "at most 6 further tool calls" per fork (`docs/research/burler-twill-fixer-bench-2026-10-16/setup/selfcheck-block.md`).
A round with more than six set-premised findings cannot be covered, and the record does not say what the fork does then.
Open risks names the six-call cap only "against the Plan lenses, `affected-set` above all", which is the reviewer's cap in D6; it does not name the fixer fork cap against the new, unbenched `fix-affected-set`.
The number appears nowhere in D4, D6, D7 or D8.

## Rejected

- Focus 1, whether the D7 refusal's steps take effect for the refused round: not a defect. The step it names, moving the fixer model key to an engine with forks and resuming, takes effect, because `loomcli/wiring.go` resolves the model lists from the stacked config on every wiring and the fix producer picks the model per attempt. The fixer fan stays as recorded, so the twill fixer then runs. The clause that setting the fan entry to an empty string reaches only later rounds is accurate and keeps that edit from being offered as a way forward for the refused round.
- Focus 2, the carrier for a shed run: not a defect beyond F5. D2 and D7 agree that the BurlerReview row receives both lists, records both picks before its spawn, and the BurlerFix row reads the recorded name; `lyx burler run` reading `fix-fan` from its own profile is stated.
- Focus 4, the pre-read against `affected-set`: not a defect. The added checklist line (every test, caller, doc and fixture that reads or asserts each changed output, key, string, file shape or behaviour) is the pre-read `affected-set` needs, and the lens text points at it. Open risks does name the six-call cap against `affected-set`; only the fixer side is missing (F6).

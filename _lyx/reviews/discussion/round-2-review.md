---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D7 (fork-support refusal bullet) and D2 (round fan pinning bullet)
    summary: D7's way forward to set the round's fixer fan entry to an empty string cannot take effect, because D2 makes the fix half read the fixer fan from the usage record and never from loom.yaml
    fix: In D7's last bullet, make the way forward for the refused round move the fixer (`discussion_fix`, `plan_fix` or `fix`) to an engine with forks, and state that setting the segment's fixer fan entry to an empty string reaches only rounds whose review half starts after the edit, per D2. Change the spec-row sentence in that bullet and the matching Acceptance criterion wording to the same way forward. Do not loosen D2's pinning.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D7 (`fix-fan` profile bullet) against D2 (round fan pinning bullet)
    summary: D7 hands the fixer fan to the BurlerFix row and lets the BurlerFix recipe profile set `fix-fan`, but D2 has the review half record the fixer fan, and the review row never sees the BurlerFix row's wiring or profile
    fix: Rewrite D7's `fix-fan` bullet so that in a shed run the wiring hands the round's `discussion_fix_fan` or `plan_fix_fan` list to the segment's BurlerReview row, beside its reviewer fan, and the review half records the picked entry per D2; the BurlerFix row reads the recorded name. Move the recipe-profile `fix-fan` key and its conflict refusal to the BurlerReview row's profile. State that `lyx burler run`, which has no shed usage record, reads `fix-fan` from its own profile.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (round fan pinning bullet)
    summary: D2 makes the round usage record the sole source of a round's fans, but that record is a best-effort measurement written today only at acceptance, whose write failure is only logged and whose undecodable copy is replaced; D2 leaves the absent-record case undefined
    fix: In D2's pinning bullet, state that the review half writes both resolved fan names to the round's usage record before it spawns the reviewer, that a failed write fails the review half before any spawn, and that a fix half or Bouncer finding no recorded fan names for its round refuses with a way forward (re-step the segment's review row) and a spec row. Add the matching refusal to the Acceptance criteria.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 (fork reading part) with D6 `affected-set` and D8 `fix-affected-set`
    summary: The bench fork boilerplates D4 and D7 adopt cap each fork at 6 tool calls over an inherited pre-read, but D4's pre-read checklist holds nothing for the Plan fan's affected sets, while `affected-set` and `fix-affected-set` each ask for repository-wide searches
    fix: In D4's first bullet, add a checklist change for the fork reading part, rendered in every fanned round, that reads, for each output, key, string, file shape or behaviour the target changes, every test, caller, doc and fixture that reads or asserts it. In D8's `fix-affected-set` text, limit the lens to rerunning each finding's own search, one tool call per finding. Name the 6-call cap against the Plan lenses as unbenched in Open risks.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Open risks (first bullet)
    summary: The list of unbenched parts omits that production drops today's Go fork boilerplate and holistic paragraph, which still rendered above the bench's fork rules, so the fork prompt as built differs from what the bench ran
    fix: Append to Open risks' first bullet that the bench ran its fork rules under today's Go boilerplate and holistic paragraph, which D4 removes, so the fork prompt without them is unbenched.
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D4 (third sub-bullet of the first bullet)
    summary: D4 lists the manifest never ending the turn as a change to the bench section, but the bench text already says so
    fix: Delete D4's third sub-bullet of the first bullet, the one that says the manifest never ends the turn and the reviewer goes straight on to the spawn in the same turn.
---

### [BLOCKING] The fixer-fan way forward of D7 cannot succeed under D2's pinning

**Location:** `_lyx/discussion/decision-record.md` D7, last bullet; D2, seventh bullet.

**Issue:** D2 fixes a round's reviewer and fixer fans when its review half starts and says the fix half and a resumed half "read them from there, never from `loom.yaml`; a hub edit reaches only rounds whose review half starts after it".
D7 refuses a twill fixer on an engine without fork support with the way forward "move the fixer to an engine with forks or set that round's fixer fan entry to `""`".
A refused fixer stops the run at the fix half, after that round's review half recorded the fan.
The operator then edits the round's fixer fan entry and resumes.
The fix half reads the recorded twill fan, spawns with forks on the same engine and is refused again.
One of the two named steps leaves the run where it was, which `PATTERN-refusal-way-forward` and the rubric's attack-surface question forbid.
The model half of the way forward still works, since `p.models.Pick(round)` (`internal/shedadapters/burlerreview.go:233`) reads models from the current config and D2 pins only fans.

### [MEDIUM] D7 sends the fixer fan to the row D2 says never reads it from config

**Location:** `_lyx/discussion/decision-record.md` D7, `fix-fan` bullet; D2, seventh bullet.

**Issue:** D2 says the review half writes both resolved names, the fixer fan included, into the usage record, and the fix half reads them from there.
D7 says "the wiring hands the round's `discussion_fix_fan` or `plan_fix_fan` entry to the segment's BurlerFix row", and lets the recipe's BurlerFix profile set `fix-fan`, with a conflict refusal at construction.
Today each row is constructed separately: `burlerReviewEntry` reads its own `profile` and `env.RowClusterFans[name]` (`internal/shedrecipe/entries_burler.go:81`), and `burlerFixEntry` reads only its own profile (`internal/shedrecipe/entries_burler.go:107-167`).
So a fixer fan set in the BurlerFix row's profile never reaches the review half, and the review half cannot record it.
The record then either lacks the fixer fan or holds the loom one, and the plan writer has two incompatible carriers to pick from.
This is the focus directive's fourth question: D7 is the Decision that carries the fixer fan to the BurlerFix row and `lyx burler run`, but its carrier disagrees with D2's.

### [MEDIUM] The usage record is made authoritative without a hard write or an absence rule

**Location:** `_lyx/discussion/decision-record.md` D2, seventh bullet.

**Issue:** Today the review half's part of `round-<N>-usage.yaml` is written only in `acceptedExit` after the review is accepted (`internal/shedadapters/burlerreview.go:282-296`), and a failed write is logged, not returned: `logger.Warn("shedadapters: burler review's usage record was not written", ...)`.
`mergeRoundUsage` archives a record that does not decode and writes a fresh one holding only the current writer's part (`internal/shedadapters/roundusage.go:101-121`).
D2 turns that measurement record into the only source the fix half, a resumed half and the Bouncer read the round's fans from.
It says the names are fixed "when its review half starts" but never says whether that write happens before the spawn, whether its failure fails the half, or what a reader does when the names are missing.
A failed or replaced record therefore leaves the fix half's mode undefined.
D2 has to settle that, because `Plan-Write` never reads the support log.

### [MEDIUM] The Plan lenses cannot do repository-wide searches under the 6-call cap

**Location:** `_lyx/discussion/decision-record.md` D4 first bullet; D6 `affected-set`; D8 `fix-affected-set`.

**Issue:** D4 adopts the bench's fork rules as the only fork rules.
Their boilerplate allows each fork "at most 6 tool calls, each only for something your procedure needs that the manifest lacks" (`docs/research/burler-fan-tuning-bench-2026-10-16/setup/iter1/burler-step-review.md`).
D7 adopts the self-check block, whose fork boilerplate sets the same cap (`docs/research/burler-twill-fixer-bench-2026-10-16/setup/selfcheck-block.md`).
D6's `affected-set` asks a fork, for every output a card changes, to "search the whole repository for everything that reads or asserts it", and D8's `fix-affected-set` reruns each set finding's search over the whole repository.
D4's pre-read checklist, built from the bench's Discussion fixture, covers symbols' callers, state writers and readers, PATTERN backgrounds and failure paths, but not the tests, docs and fixtures that assert a card's outputs.
A Plan fork therefore has to find a multi-card affected set within six calls.
The bench measured 2-5 calls per Discussion fork (`report.md:94`), which says nothing about this lens.
The lens most tied to the Plan rounds' late BLOCKING findings is the one the cap starves.

### [LOW] Open risks omits that the bench's confound is removed unbenched

**Location:** `_lyx/discussion/decision-record.md` Open risks, first bullet.

**Issue:** The bench report names its confound: "Go's own fork boilerplate (\"after exploring the target fully\", \"fetch only what your lens needs\") still renders above the override" (`docs/research/burler-fan-tuning-bench-2026-10-16/report.md:66`).
D4 removes that boilerplate and the holistic paragraph, so production forks get a prompt the bench never ran.
The support log's rejected alternative relies on that removal, but Open risks lists the other unbenched parts and not this one.
This is the focus directive's fifth question.
The rest of the list matches the lens and fork-rule changes: `walkthrough`, the widened `existing-paths`, the dropped escape, the board-entry line (dropped in the bench, `report.md:60`), the Plan fan, `fix-proposal` and `fix-affected-set`.

### [NIT] D4 lists a bench line as a change

**Location:** `_lyx/discussion/decision-record.md` D4, first bullet, third sub-bullet.

**Issue:** The bench section already ends "Write it as text in the same turn and go straight on to step 2: the manifest never ends your turn." (`setup/iter1/burler-step-1-explore.md`), and the report calls it "the same-turn fix" (`report.md:58`).
Listing it among "these changes" tells the plan writer the bench text needs an edit it does not need.

## Rejected

- Focus 1, D4 spawn mechanics: held. D4's fanned part keeps "every fork spawned in one message through the Agent tool with `subagent_type: \"fork\"`, one per listed lens, never with a `name`", which matches `auditClusterRound`'s count check and named-spawn failure (`internal/burlerengine/cluster.go:35-58`). Acceptance criterion 1 pins it.
- Focus 2, `fan_review` loading in a local layer or run_settings: held. `fan_review` is `CategoryNone` today (`internal/loomengine/config.go:601`). A retired key keeps that category for the local check, and every load drops it (`internal/configengine/doc.go`, `RetiredKeys` paragraph). `PrepareRunSettings` runs the module's `Migrate` and then `ValidateLocal` (`internal/configreg/runsettings.go`), so a task's run_settings holding `fan_review` loses it in `MigrateConfig` and passes validation. D3's second bullet states this.
- D10 against D2: the next round's fan comes from the config at judgment time, while D2 pins only the judged round's fans. An edit between the judgment and the next review start is already caught by `excludedLenses`, which drops exclusions when the previous round's record names a different fan (`internal/shedadapters/burlerreview.go:119-125`). Not a defect.

---
round: 4
focus:
  - "D2's refusal for a round whose usage record holds no fan names: does its stated way forward (re-stepping the segment's review row) end with the names recorded? Check against `Call` in `internal/shedadapters/burlerreview.go`, which hands an already accepted review on without spawning, and against what the fix producer's changed-review exit archives before it names a goto."
  - "D2's reader side for the usage record in `internal/shedadapters/roundusage.go`: the fan field is a plain string without omitempty, and `mergeRoundUsage` replaces an undecodable record with a fresh one. Does the record state how absent names differ from a recorded empty name, and what a fix half or Bouncer does on a record that fails to decode?"
  - "D2's pre-spawn write of the round's fan names against the live-reviewer probe (`probeLiveReview`) and the acceptance-time merge in `acceptedExit` that today writes the fan from the producer's profile: can the recorded name differ from the fan the reviewer ran under?"
  - "D2's refusal against the Bouncer's existing reads in `internal/shedadapters/bouncerfacts.go` (`computeLensFacts`) and `focus.go` (`lensesRunUnderFan`), which record a missing usage record and treat the round as ineligible without failing: does the record say which read the refusal replaces, and which segments' rows record names?"
  - "D7's BurlerFix row against `burlerFixEntry` in `internal/shedrecipe/entries_burler.go`, which takes no fan: does the record say how the row learns the key names its fork-less engine refusal names, and what `lyx burler run` names instead?"
  - "D8's `fix-affected-set` (one search per finding) against the six-further-tool-calls cap in the inherited self-check fork boilerplate: does the record say what a fixer fork does when set-premised findings exceed the cap, and does Open risks name that cap on the fixer side?"
---

The entries name sites to settle; none caps severity or states an outcome.

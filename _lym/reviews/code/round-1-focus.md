---
round: 1
focus:
  - "internal/battencli/cancel.go: trace every path a `--remove` of a beam can take (fresh run, finished cancel with the pair kept, resumed or interrupted cancel, unfinished teardown, flag left alone, end-only with origin unreachable). Does each one that removes the beam also clear the flag, or leave a re-run that does?"
  - "clearRunPhase and refuseNothingLeft: after a failure at or after EndBeam, or at any earlier step, does the named re-run get past every refusal (owner, landed, stopped origin entry, members, nothing left) and finish? Check what isBeamProject reads once the pair and records are gone."
  - "The issue beam had members, and the expected result is a plain project with them untouched. Which members make the beam cancel refuse, and can the scenario in the board entry reach EndBeam?"
  - "internal/battencli/cancelcmd.go StopBeam: does skipping on an absent or unmarked origin record also skip an orch or watcher that can be running, or swallow a failure that should warn? Settle what test covers this new behaviour."
  - "internal/boardengine/beamend.go EndBeam: are the cleared and kept fields right for a plain project against the board's invariants and other writers of these fields (status, owners, holds, front tasks, commit and push of the write)?"
  - "cancelNotice, cancelEnvelope and the help and doc text: do they report a cleared flag correctly in every outcome, including one with no pair removed? Did shortening the cancel help drop a statement a user needs?"
  - "_lym/landing/summary.md: check each claim against the diff, and check the `## Merge danger` door and blast radius against everything the diff changes, including the fields EndBeam erases and the boardengine surface."
---

Seed focus for round 1, set from the board entry, the diff over the review range, the commit messages and the change description.
The entries are questions to settle; the rubric's full review still applies.

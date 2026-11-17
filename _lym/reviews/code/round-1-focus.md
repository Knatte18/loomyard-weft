---
round: 1
focus:
  - "internal/boardengine/holdwrite.go, internal/adoptcli: does the one pull-first Bolt write refuse on a rejected push as D6 step 1 says (owner of these subjects, or a hold another machine wrote since), and does the same-set own-hold case write only the owner and registry entry while any other set, an overlapping one included, refuses naming the hold's subjects?"
  - "internal/adoptcli, internal/fabricengine/takeoverwrite.go: are steps 2 and 3 skipped for a run whose owner before the adopt was this machine with no takeover entry naming another, so a killed adopt re-runs to the same end; does every step from step 2 on re-read the hold and stop when it is gone or names another machine?"
  - "internal/adoptcli check and green paths: does a green check clear the hold, drop this machine's bolt-hold pause entries and run orchcli.BeamUp per subject; does a failing check under an own adopt-fix hold or --take-hold file no second task and keep kind, fix task and try counts; are try counts read from the hold on every machine?"
  - "internal/shellguard/rows.go and its tests: does the --take-hold deny row's steer and notice avoid spelling the flag and name asking the operator, and does every refusal an agent reads avoid naming the flag while still naming a way forward?"
  - "Retirements: is every reader and writer of the adopt record, the adopt and adopt-hold callers, the worker, --keep-local, --drop-local, --discard and the adopt pause step gone, with no leftover reference (assert-no-callers on each Delete group), and do the dobby handover, orch watcher and lym beam status read the Bolt hold instead?"
  - "internal/adoptcli --here path: are no_subject and owned_here kept, with owned_here ending ok after one Bolt pull and writing no hold?"
  - "Test economy and PATTERN: new tests against the redundancy report's candidates and the rubric's bullets, and whether the PATTERN-dobby-runs change lands with the declared text and no other pattern entry drifts."
---

Focus set from the decision record's D6 and its acceptance criteria; the plan cards were read only as scope.

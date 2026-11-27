```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's change set agrees with the sibling cards, decisions and cross-references it touches (fix-consistency).

- **MEDIUM**: card 14, the action-list bullet ("re-reads the answer's `Reread` fields under the latest phase, the zero `Phase` when none was read yet") and the new ReadFailure clause ("for a re-read the phase the re-read ran under").
  - Summary: the fix now appends a phase to `Phases` only once its read succeeds. That leaves "latest phase" with two readings: the phase requested last, whose lock is held, or the last phase in `Phases`.
  - Evidence: say a rule asks for `PhaseOrigin`, the read fails as a transport failure, and the rule answers with an action list. Under one reading the re-read runs under `PhaseOrigin`, since it was requested and locked. Under the other it runs under the last successful phase, or the zero `Phase`. The new clause about which phase a re-read failure records depends on this choice. So two implementers can still record different `ReadFailure.Phase` values, which is F16's premise.
  - The untouched sentence "the zero `Phase` when none was read yet" reads as "none read successfully" under the new rule. The row "an action list as the start answer re-reads under the zero phase" does not separate the two readings.
- **LOW**: overview, decision "the foundation's tests shrink", the unchanged sentence "a test whose assertion another test already holds, or that guards no rule of its own, is deleted in the card that touches it".
  - Summary: card 15 now deletes about 20 engine tests and parts of tests in files that no other part of card 15 touches. It does this only to pay for the net shrink, so "in the card that touches it" no longer describes where they are deleted.
  - Evidence: card 15's new "Further engine tests go…" section targets files such as `destroy_test.go`, `mergecrucible_integration_test.go` and `warpjunction_test.go`, none of which card 15's reader work touches. The overview's new estimate sentence names "card 15's further engine test deletions", but the rule sentence above it was left as it was.
- **LOW**: overview Card Index, line 15, which still reads "snapshot-reader — add the phased snapshot reader with its git fallback for non-files ref storage".
  - Summary: card 15 now also carries the part's main batch of test deletions, as its ImpactSummary says, but the index line and the card's **Intent** opening do not mention them.
  - Evidence: card 15's ImpactSummary now ends "…and engine tests whose assertions others hold are deleted". The Card Index line is unchanged.

Not a finding: card 2 edits `TestConfigTemplate` so it reads the structural sets through the layout functions. The lines that edit touches (`template_test.go:91-104`) include the tail that card 15 now deletes, so card 2 does work card 15 later undoes. They do not contradict each other, and card 3 line 119's reliance on the empty `Dirs()` check at line 87 still holds.

Tool calls:
1. grep of plan cards 0–14 for the test names card 15 now deletes or edits, to find twins outside card 15 (my context held only some of the cards).
2. `template_test.go:80-105`, to see whether card 2's `TestConfigTemplate` edit and card 15's tail deletion overlap (my context did not hold the test body).

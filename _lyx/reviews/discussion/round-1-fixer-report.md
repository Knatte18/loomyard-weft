# Fixer report — discussion round 1

## Changes

- F1 (BLOCKING), fixed as proposed: D2 now requires the new entry to land with a cut of at least its own byte count elsewhere in `PATTERN.md`, by moving entry-line detail into background files, so the overview ends at or under the headroom line.
  Constraints gains a bullet for the `lyx loom lint-pattern` headroom lint run by every card gate, and Acceptance criteria gains `lyx loom lint-pattern --base <merge base>` passes.
  Premise checked: `PATTERN.md` is 15560 bytes and `HeadroomBytes` is 15564 (`internal/pattern/check.go`).
- F2 (MEDIUM), fixed as proposed: D4's second sentence now widens the Plan stencil's post-Discussion route to name a drop of a decided own-entry step, recorded by the parent with `lyx loom decision add` as an `Added after Discussion` entry, with the stop-and-report path kept, and states that the route admits a drop of an own-entry edit only.
- F3 (MEDIUM), fixed as proposed: D6's Plan-review sentence now accepts a missing step only when it edits this task's own board entry and the record carries an `Added after Discussion` entry recording the drop; any other uncarried Decision stays a finding.
- F4 (LOW), fixed as proposed: the Goal and D2's PATTERN text now say "change(s) ... own board entry's body or scope", and D2's closing clause reads "its landing marks it done and the board prune removes it".
  The Goal also gains one sentence defining "editing the own entry" as such a change, with the landing's status write outside it, so the "edit" wording in D1, D3, D4, D6 and the acceptance criteria carries the narrowed meaning without a rewrite of each.
- F5 (LOW), fixed as proposed: D5 now says the darn writer leaves out a requested own-entry step and asks its parent to record the drop through `darn-template-write.md`'s existing scope-correction route (`lyx loom decision add`), and D6 says the darn-review line accepts the drop once the record carries that entry.

## Disputed

None.

## Deferred

None.

## Observed outside the findings

`internal/pattern/doc.go` states that PATTERN text states rules for the code only and never says how an agent, a run, a stencil or a workflow uses it.
D2's entry text ("Planning or doing a task's work: no decision, card or step ...") describes run steps, which that rule may exclude.
No finding raised it, so the record is unchanged on this point; the next round may weigh it.

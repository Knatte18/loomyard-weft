---
round: 1
focus:
  - "internal/landingshed/prreview.go `wake` and `sessionStopWanted`: settle whether skipping the second `readTree` when no stop is wanted leaves any path where `judge` ends the review, or acts on the operator's ending, on a tree the agent could still change between the read and the verdict."
  - "internal/landingshed/prreview.go `sessionStopWanted`: settle whether the new `rec.ReviewEnded` case and the `rec.Continue` case (clean tree at the gated head) still stop the session in every state the old `rec.Continue` case did, and whether `judge` can end on Continue with a dirty tree or an ungated head."
  - "internal/loomcli/prcontinue.go `prContinueVerb` against `gateDue` in internal/landingshed/prreview.go: settle whether clearing `MailedHead` makes the next wake gate the failed head again on every path, including a record with Continue set and a mailed head, and whether the idempotent no-write branch stays correct."
  - "Race between `pr-continue` clearing `MailedHead` and `gateHead` writing it: settle whether `updateRecord` in `gateHead` can overwrite the cleared value or the new Continue flag, or lose the retry."
  - "internal/landingshed/prreview.go `gateHead` and `gateRetryHint`: settle whether the hint mailed with the failure is sent only for a failure that `pr-continue` can retry, and whether it reaches the agent once per head as before."
  - "Test coverage in internal/landingshed/prreview_wait_test.go and internal/loomcli/commitrecords_test.go: settle whether the cycling-session fix and the retry-on-continue path each have a test that fails without the change, and whether any behavior the diff adds is uncovered."
  - "Scope against the board entry's Expected section: settle whether the diff licenses its docs and stencil edits (docs/overview.md, internal/landingshed/doc.go, internal/loomcli/doc.go, the pr-review landing stencil) and whether each one states what the code now does."
---

Seed focus from the committed diff over `f98e284c..HEAD` and the board entry under `## Goal`.
The decision record's other sections are empty, so the entry's `Expected` paragraph is the measuring stick.

---
round: 1
focus:
  - "internal/shuttleengine/wait.go and gate.go (card 1): does the pending gate state hold only at a turn boundary as Shared Decisions define it, and do the liveness, stall and deadline checks keep firing while a gate entry is waiting? Is a pending result or a Final closure on a non-PassOnCap entry refused at run time?"
  - "internal/parentreview/store.go and gate.go (cards 4-5): are round transitions (BeginRound, request open, delivered, approve, reject) atomic under the lock directory, and are they consistent with the round lifecycle (verbs, closure and status act on the latest round only; an attach continues it)? Does a reject spend exactly one re-prompt, after which the next arrival re-runs discussion and passes through?"
  - "internal/parentreview/gate.go send path: when the delivery send fails or delivered is never stamped, does the closure re-send only at a turn boundary, surface SendFailedWayForward, and stop at parent_review_wait_min's bound with a pass-on-cap rather than a stall or a fail?"
  - "internal/loomcli/review.go (card 7): do the four review verbs refuse every row the card and contracts/specs/refusal-spec.md name, return JSON errors through internal/output, and stay off the loom parent's arm (no slug read as a run-id)? Is each new command's Short non-empty and covered by the help-tree test, per CONSTRAINTS.md's CLI/Cobra Invariant?"
  - "internal/shedverbs/status.go and human.go (card 8): with the Waiting hook nil, is every status envelope and human rendering byte-identical to before the change?"
  - "internal/shedrecipe/entries_gate.go and entries_discussionwrite.go (card 6, 12): is parent-review the only gate name allowed to carry Final or return pending, enforced at construction, and does a seed with no Parent pass the entry without opening a request?"
  - "Provider-wording seam: do internal/shuttleengine, internal/reedengine and internal/parentreview stay free of SendMessage/provider words, with the instruction living only in the loom-template-parent-review-delivery stencil, and does seam_enforcement_test.go pin that?"
  - "Comment-convention compliance across every new or changed Go comment in the diff: CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks with no column limit; are any new comments fixed-column wrapped?"
  - "Per-card mechanical check: the cards carry only Create and Edit groups; for each Edit group, is the blast radius on the changed symbols (GateResult, GateEntry, Run.Wait, specFor, RenderStatusHuman, resolveGateClosure, the batten running arm) confined to what the card's ImpactSummary states, and for each Create group, is there no pre-existing equivalent the new code duplicates?"
---

Review range: no `_lyx/loom/rework/` directory exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (merge base `218ba2a68`), from `product.parent` = `main` in `_lyx/shed/parent-review/status.json`.
The diff spans the fourteen card commits.
The focus weights the concurrency- and state-heavy code (wait loop, round store, gate closure) over prompt-text and doc edits.

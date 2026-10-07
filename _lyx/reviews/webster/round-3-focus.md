---
round: 3
focus:
  - "Comment line breaks across the diff: in every new or changed Go comment, and in new or changed `.md` rows, does each sentence sit on one line, breaking only before an independent clause with its own subject and verb? Check `internal/websterengine`, `internal/webstercli`, `internal/fabricengine`, `internal/gitrepo`, `internal/landingshed`, `internal/planglyph`, `internal/planparser` and `cmd/lyx/destructiveguard_test.go`, plus the new rows in `contracts/specs/loom-plan-spec.md`; a changed line inside a legacy wrapped sentence makes the whole sentence a changed comment."
  - "`internal/websterengine/state.go` file header: does any live text anywhere in the diff's packages still name Master's last-asserted model, now that `State.AssertedModel` is removed?"
  - "`contracts/stencils/loom/loom-rubric-plan-review.md`: does its Do-not-flag range name every spec check this round's gate enforces without bounding the list at a last-row ID, and is `delete-target-gone` (emitted only at Webster's dispatch) named as a check the gate does not run alongside `plan-unapproved`?"
  - "`internal/landingshed/publish.go` `pushRejectedReason`: when `Deps.RemoteOnlyCommits` returns an empty tip, meaning the remote has no task branch, does the reason avoid asserting that the branch exists, does the matching `contracts/specs/refusal-spec.md` row's condition cover that case, and does a `TestPublish_StuckBeforePullRequest` row reach it?"
  - "`internal/websterengine/triage.go`: do `expiredShellWarning` and `backgroundShellSentence` both survive as separate functions with one wording between them, and is the friction note's call site the single user of the shell wording?"
---

The first entry is the broadest; round 2's range of sites was long, so a fresh sweep of the whole diff is needed rather than a spot check of the listed files.

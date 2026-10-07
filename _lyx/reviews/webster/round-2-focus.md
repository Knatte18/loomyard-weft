---
round: 2
focus:
  - "Where the diff adds or changes Go comments, in particular `internal/gitrepo/doc.go`, `internal/webstercli/cli_test.go` and `internal/websterengine/beginbatch_test.go`: does every new or changed comment line follow `PATTERN-comment-line-breaks` (one sentence per line, no mid-sentence column wrap)?"
  - "In `internal/websterengine/doc.go`, do the bracket-verbs section and the one-model-per-run section agree on what begin-batch does with Master's model?"
  - "In `contracts/stencils/loom/loom-rubric-plan-review.md` against the diff's `loom-plan-spec.md` validation-checks section: does the stencil name any check count or last-row ID that the spec's rows no longer match?"
  - "In `internal/landingshed/publish.go` `ErrPushRejected` handling and the matching `refusal-spec.md` row: for a rejected push where `RemoteOnlyCommits` returns zero commits, or the remote branch is absent, does the reason and way forward name a step that clears the rejection, and does a test row reach that case?"
---

Each entry restates the sites it depends on and asks a question about the current diff.

```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: checking that each sentence the fix added or rewrote holds against the code at HEAD.

**MEDIUM**: card 14's description of the stop seam does not match the code.
- **Changed text:** card 14, "The loom side", the `prReviewGotoAdmitted` bullet: "`stopLiveSession` stopping the strand through `pullRequestDeps`'s `Sessions.StopStrand` only when its `SessionLive` reads true".
- **Summary:** `pullRequestDeps` fills neither field, so the sentence describes a step that cannot be carried out as written.
- **Evidence:**
  - `internal/loomcli/landingdeps.go:375` says "SessionLive and RegisterSession read reed's strand table, which only the run's wiring holds, so the wiring fills them". The seam is set only at `internal/loomcli/sharedbootstrap.go:435` (`c.env.Landing.PullRequest.SessionLive = sessionLiveness(c.reed.Status)`).
  - `Sessions` is the `runner` argument (`landingdeps.go:396`). A `pullRequestDeps(..., nil)` call, the form card 13 uses, leaves it nil.
  - The hook must instead take liveness from `sessionLiveness(c.reed.Status)` and the stop from a real shuttle runner, or from the `c.env.Landing.PullRequest` the shared bootstrap fills.

**LOW**: card 6's `doc.go` bullet leaves out one ended state.
- **Changed text:** card 6, the `doc.go` bullet: "under it the parent agreement entry's timeout or unsent ask halts Drift-Verify instead of passing".
- **Summary:** the halt also covers a no-parent end, which "unsent" does not clearly include, so the doc sentence would understate the halt.
- **Evidence:** card 6's `endedUnanswered` bullet names `ParentAskTimeout`, `ParentAskNotSent` and `ParentAskNoParent`. Today's `doc.go` words the pass as "a timeout, or an ask that never reached the parent" (`internal/driftengine/doc.go:92`).

The other new claims hold:
- Both consumers copy the base request (`request := base`).
- `Goto` and `Relaunch` hold the run lock until they return (`goto.go:80-87`, `:217-224`).
- `Relaunch`'s comment says "It commits nothing".
- `driverGoto` goes through `c.spec`, which `specFor` builds.
- `resumeSession` goes through `ensureSession`, which writes a new `StrandName`.
- The BeamAwait row of `simpleEntryCases` has `validatedFields`.
- `TestGoto_Guard` and `TestRelaunch` exist.
- The "awaits an answer" text is at `parentreview.go:165-168`.

Calls made:
1. Read the `pullRequestDeps` signature, the `specFor` and `c.spec` sites, `resumeSession`, the BeamAwait row of `simpleEntryCases`, and the run-lock lines in `goto.go`. My context lacked these spans.
2. Grepped for the `SessionLive` wiring, since my context did not show where loomcli fills it.

```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. For each function and branch the diff changes, run one concrete case through it.

**Cases run**

1. `fabricSync` with `open == nil` (standalone) returns `(false, nil)`. The failure branch is not entered, and `pushResetRecords` returns nil because `pushRecords` is nil (`wiring.go:370`). Correct.
2. `fabricSync` with `SkipGit` returns `(false, nil)`. The success path runs and `pushResetRecords` skips the push. Correct.
3. `fabricSync` when `open()` fails returns `(false, err)`, and `!committed` gives the refusal with no push (`sync.go:34-36`). Correct.
4. `CommitWithTrailers` refusing a bad trailer, a merge record or a secret path returns a zero `CommitResult`, so `committed` is false and the verb refuses with no push (`commit.go:89-93`, `commit.go` `commit`). Correct.
5. The weft commit lands but recording it in the corrindex fails. `commitWeftLocked` returns `(sha, true, err)` (`recordsgit.go:269`), `commitBothSides` sets `WeftCommitted` and `partialErr` (`commit.go:212+`), and `commit` returns that result with `partialErr`. `fabricSync` then gives `(true, err)` and the branch pushes. Correct. The non-nil error is a real `*PartialCommitError`, not a typed nil, because `commit` returns `nil` explicitly when there is none.
6. The warp side lands and the weft side fails without landing. `Committed()` is true through `WarpCommitted`, so the records push runs even though no records commit landed. Under the `_lym` pathspec `warpFiles` is empty, so this case cannot occur from the reset. Correct in practice.
7. Failure branch, push returns nil: `fail(msg)` refuses with exit 1 and no `push_error`. Correct.
8. Failure branch, push fails with something other than a hand-over: it is logged once (`reset.go:256-257`), and `ErrFields` writes `ok:false`, `error` and `push_error` with exit 1 (`output.go:34-42`). Correct.
9. Failure branch, push returns `ErrHandedOver`: nothing is logged, and `push_error` carries the hand-over text. The `error` field is the sync refusal, whose `fabricSyncWayForward` says "the next bracket verb's own sync carries it" (`beginbatch.go:27`). That points the caller at more work on a run already handed to another machine, while the success path makes the hand-over itself the error (`reset.go:214-215`). This is a misreported outcome (see the finding).
10. `pushResetRecords` cases:
    - nil `pushRecords`, `SkipGit`, `SkipPush`: returns nil. Correct.
    - Success: returns nil. Correct.
    - Hand-over: returned without logging, as before. Correct.
    - Other failure: logged and returned, as before. Correct.
11. Success path after the extraction: hand-over still gives `fail`, any other failure still sets `push_error`, and the warning is still logged inside the helper. Behaviour unchanged. Correct.
12. `SkipPush` with the commit landed and the sync failed: refuses with no push, matching the success path's skip. Correct.

**Findings**

- **LOW, design** (`internal/webstercli/reset.go:200-205`): on the sync-failure path a hand-over reaches the caller only in `push_error`. The `error` field and its way forward still tell the caller to let the next bracket verb's sync carry the state. That contradicts the hand-over, which means nothing more is pushed for the run (`internal/fabricengine/takeover.go:22`). The success path reports the hand-over as the error instead (`reset.go:214-215`). The change description makes this choice on purpose (`_lym/landing/summary.md`, Design decisions), but the refusal text still names a way forward that no longer applies.
  - Evidence: `reset.go:195`, `beginbatch.go:27`, `takeover.go:22`.

**Tool calls**

1. Read the returns of `commitWeftLocked` in `internal/fabricengine/recordsgit.go` and the `output.ErrFields` body in `internal/output/output.go`. The manifest did not cover how a corrindex failure maps to `(sha, true, err)`, or the shape of the error envelope.

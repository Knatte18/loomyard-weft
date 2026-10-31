```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens: what the diff's changes force outside the lines they touch.

**List: per change, the set found, covered or not**

1. `PushRebaseFree` now returns `fmt.Errorf("%w: %s", ErrPushRejected, line)` instead of the bare sentinel, and the sentinel text drops `(remote diverged)`.
   - **Identity or `==` comparison callers:** none. Every non-test hit uses `errors.Is`, in `pushclassified.go` lines 127, 197 and 305, `pushanchored.go` line 32, and `loomcli/commitrecords.go` line 57. Covered.
   - **`coalesce.go`:** the only `ErrPushRejected` match is in `pushclassified.go`. The old comment's "identity check in `coalesce.go`" was stale and the diff removes it. Covered.
   - **Text assertions on `remote diverged`:** the only one was `loomcli/commitrecords_test.go`, which the diff updates. Covered.
   - **Callers that classify by sentinel text:** none found.
   - **`gitrepo/doc.go` lines 174–194:** these describe `Push`'s rebase-retry and the exhausted-retry case, not `PushRebaseFree`'s rejection error. Nothing in them is wrong. The scope asked for a `gitrepo` doc update, and the diff makes none. The decision-record scope is not binding on the diff, so I raise no finding.

2. Class change: a bare rejection is now judged by `failRejection`, which records `other` or `not_fast_forward`.
   - **Tests asserting `not_fast_forward` for a declining hook:** `boltpullfirst_integration_test.go` line 619 and `pushanchored_integration_test.go` line 367 are updated. The `pushclasses` rows are in the diff stat, which I did not re-read. Covered.
   - **Tests that still assert `not_fast_forward` plus `ErrPushRejected`:** `boltpullfirst_integration_test.go` lines 563 and 586. They are real-divergence or replay cases and stay valid.
   - **Consumers keyed on class `not_fast_forward`:** `landingshed/finalize_test.go` line 158 and `orchcli/rules_test.go` line 95 build records by hand and are unaffected. I did not check non-test consumers of `PushFailureNotFastForward` or `FailedPushWayForward` beyond the doc.
   - **`fabricengine/doc.go` lines 937–947:** updated for the judged class. Covered.
   - **`doc.go` lines 931–936:** unchanged and still accurate.
   - **`doc.go` line 757:** "A caller treats the `gitrepo.ErrPushRejected` that remains … as a human-decidable condition" still holds.
   - **`loomcli/commitrecords.go` `pushFailureWayForward`:** it only runs when `HasUnclassifiedFailure` is true. A `*PushFailedError` is classified, so the `other` record's own way forward is used and the merge advice is not shown for it. Its text also covers a remote rule ("clear what a remote rule objects to"). Covered.

3. `PushAnchored` doc comment at `pushanchored.go` lines 76–77: updated from "UNWRAPPED" to "satisfies errors.Is". Covered.

4. Stale "unwrapped" wording is left behind (finding F1 below).

5. `PATTERN.md` and `pattern/`: the changes touch `PATTERN-fabric-git`, `PATTERN-gitrepo-client-boundary` and `PATTERN-push-both-sides`. The fetch and `HasUnpulled` go through `gitrepo.Repo`, with no raw git. The failed-push record and its one notice are kept, and only the class and way forward change. No declared PATTERN changes were visible in the supplied directive, so none are to land. Covered.

6. Commit message: there is no `Pattern-Departure:` trailer and none is needed. The trailer says `Co-Authored-By: Claude Opus 5.5`, which differs from this session's attribution line. That is not a repo rule.

**Findings**

- **F1 — LOW, consistency** — `internal/fabricengine/pushanchored_integration_test.go:6`
  - The file header still says a diverged weft remote "surfaces gitrepo.ErrPushRejected unwrapped".
  - After the diff the error is wrapped, and `errors.Is` matches it through `*PushFailedError`. The test at lines 152–159 is also named `…ReturnsErrPushRejectedUnwrapped`.
  - The sibling comment at `pushanchored.go` lines 76–77 was fixed and this header was missed.
  - Fix: reword the header to say the error satisfies `errors.Is(err, gitrepo.ErrPushRejected)`. Consider renaming the test to drop `Unwrapped`.
  - Evidence: grep hit at `pushanchored_integration_test.go:6`, and the `PushAnchored` diff hunk.

- **F2 — LOW, consistency** — `internal/fabricengine/spawn.go:264–266` and `pushanchored.go:133–134` (`PushPairAnchored` doc)
  - Both say "the rejection that remains is a human-decidable condition" and "errors.Is holds for a rejected side".
  - A non-divergent rejection is now recorded as class `other`, and the plain-retry way forward is the right advice for it. "Human-decidable" is mildly misleading but not wrong.
  - Fix: optional, leave as is or add a clause saying the record's class says whether origin holds commits the branch lacks.
  - This is the weakest finding here.

No MEDIUM or BLOCKING findings in this lens.

**Tool calls**
1. A grep over `*.go`, `*.md`, `PATTERN.md` and `pattern/`, plus reads of `commitrecords.go`, `gitrepo/doc.go`, `coalesce.go`, `fabricengine/doc.go`, `spawn.go` and `pushanchored.go` for comments on rejection and "unwrapped" wording. My context lacked these call sites.
2. Reads of `gitrepo/doc.go` lines 30–38 and 174–194, `HasUnclassifiedFailure`, and `fabricengine/doc.go` lines 925–936. My context lacked these, and they settle whether the docs and the way-forward path are still right.

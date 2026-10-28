```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. The diff leaves three comments describing the old retry. I found no other stale references.

**List: per change, the set found, covered or not**

1. **`pushRebaseFreeRetryingOnce` removed.**
   - Set: the search returns no Go or Markdown reference outside the `_lym` and `.git` paths I excluded from it. Covered.
   - Set: the `PushAnchored` doc (`pushanchored.go`) was updated. Covered.
   - Set: `pushanchored.go:69` still names the "stale remote-tracking ref" scenario. It describes the race itself, not the retry, so it stays correct. Covered.
2. **A rejection of any side is retried over the backoff.** This changes `PushWarpRebaseFreeAt`, `PushRecorded` and the package doc, and the set is not covered.
   - Set: `spawn.go:265`, the `PushWarpRebaseFreeAt` doc. Not covered; see F1.
   - Set: `bolt.go:116`, the `PushRecorded` doc. Not covered; see F2.
   - Set: `doc.go:753-754`. Not covered; see F3.
   - Set: the `doc.go` push classification section and `pushClassified`'s own doc. Covered.
   - Set: tests that keep a remote declining. The diff injects `BackoffSleep` in `pushanchored_integration_test.go`, `boltpullfirst_integration_test.go` and `pushclasses_integration_test.go`. I did not find other callers beyond those, but I did not run a full search. Partly covered.
3. **`recordsConflictWayForward` wording.**
   - Set: the "another machine wrote" and "paths: none" searches match no remaining Go or Markdown source. Covered.
   - Set: the `FailedPushWayForward` callers and the `TestFailedPushRecord` table were updated. Covered.
4. **PATTERN.**
   - Set: `pattern/PATTERN-push-both-sides.md:12` says "still refused after its retry", which still holds. The directive declares no PATTERN changes, and the diff lands none. Covered.
5. **Commit messages.** There is no `Pattern-Departure:` trailer, and none is needed. Covered.

**Findings**

- **F1 — LOW, consistency.** `internal/fabricengine/spawn.go:265`.
  - Summary: the `PushWarpRebaseFreeAt` doc says a rejection is "never retried here". It now goes through `pushClassified` with a nil retry, which runs `repushOverOriginContained` over the backoff.
  - Evidence: `spawn.go:265` and `spawn.go:275`.
  - Fix: reword to say a rejection is retried over the push backoff only while origin holds no commit the branch lacks, then recorded.
- **F2 — LOW, consistency.** `internal/fabricengine/bolt.go:116`.
  - Summary: the `PushRecorded` doc says a rejected push is "retried once". `settleRejection` now reruns `retryBoltPush` over the backoff while it fails with a bare rejection or an unnamed error.
  - Evidence: `bolt.go:116` and `bolt.go:125`.
  - Fix: say the replay retry reruns over the push backoff until it lands, defers or fails with a named class.
- **F3 — MEDIUM, consistency.** `internal/fabricengine/doc.go:753-754`.
  - Summary: the package doc still says a code-side rejected push "is retried once, after a fetch".
    - The same file's updated section (about line 929) now says the retry runs over the backoff, so the doc contradicts itself.
    - The `doc.go` hunk also leaves this sentence untouched, so it falls short of the focus item that the doc match the code.
  - Evidence: `doc.go:753`.
  - Fix: reword, or delete it as a duplicate of the new `repushOverOriginContained` sentence.

**Tool calls (4)**
1. `grep` for stale retry and wording phrases across `.go`, `.md` and `.yaml`. My context held only the diff.
2. `sed` on `bolt.go` 60-135 and `spawn.go` 255-290, to read the `PushRecorded` and `PushWarpRebaseFreeAt` docs and callers.
3. `sed` on `doc.go` 748-758, to read the `doc.go` lines the first grep matched.
4. `grep` for other "retried once" wordings and for `pattern/PATTERN-push-both-sides.md` retry text.

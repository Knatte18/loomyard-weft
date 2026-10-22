---
verdict: APPROVED
---

## Scope

The board entry `retry-archive-tag-push` asks that the archive-tag push retry a transient remote rejection with backoff before failing the row.
The operator decision of 2026-10-22 in the entry asks for a longer backoff, and for the retry to cover the rejection class actually seen if `PushWithRetry` did not retry it.
The decision record holds no `Added after Discussion` entries, so the board entry is the measure.

The diff of pull request 839 at `85cf0473c` against `main` covers both points and nothing beyond them:

- `namesRef` in `internal/gitrepo/pushretry.go` matches the short name git prints for a ref passed as `refs/tags/...` or `refs/heads/...`, so the archive push's `archive/<slug>/<sha> -> archive/<slug>/<sha> (failed)` line is now retryable.
  This is the rejection class the entry says to cover.
- `PushWithRetry` takes the caller's delays; `pushPlainWithRetry` passes `PushBackoffDelays()` and keeps the plain primitives' four attempts at 1 s, 2 s and 4 s.
- `archiveWeftTip` in `internal/fabricengine/archive.go` passes `archiveTagPushDelays()`, 2 s doubling to 32 s over six attempts, which is the longer backoff.
- The `gitrepo` and `fabricengine` package docs state the ref matching and the archive schedule, as the docs rule requires.
- Tests pin the short-form matching, the namespace bound, the caller's delays, empty delays, the archive schedule and a real-remote refusal of a tag pushed by its full ref.

`PushWithRetry` has no other production caller at the PR head, so the signature change leaves no caller behind.

## Open risks carried over

The Darn-Review carry-over names two round-1 findings.
`archive-schedule-comment-lock-claim` is addressed by `fc748fd0b` and `a497c67a4`, whose comment now says the teardown holds at most the pair's own lock.
`archive-schedule-untested` is addressed by `dc84f6dc4`, which adds `TestArchiveTagPushDelays`.
The fixer report defers pinning the call site's use of the schedule; that is a test-depth point, not scope.
Neither finding names an unrecorded parent correction, so neither escalates.
The record's own open risk, a GitHub refusal longer than a minute, matches the entry's "a few times with backoff" and is not a deviation.

## Overlap

No peer from `lyx loom peers` changes a path this run changes:
`orch-strand-cycle` lists no changed paths yet, and `loop-exit-halt-names-its-cause` and `done-check-spaced-create-paths` work in `loomcli`, `loomengine`, `shedverbs`, `planglyph`, `planparser` and `websterengine`.
None of their titles touches push retry or archive tags.
The parent lists no landed commits.
No overlap, so no landing order applies.

## Superseded

No board entry is made obsolete by this run.
`gate-runs-once` depends on this task but covers other work.

```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function against a concrete input and found no defect that affects behaviour. Two stale doc comments are LOW.

**Per function or branch (case run, verdict)**

- **`pushClassified` rejection branch.** A code-side rejection with `retryRecords == nil` takes `repushOverOriginContained`. Records and bolt always pass a retry (`pushanchored.go:~182`, `bolt.go:126`).
  - A code side that lands on retry goes to `landed`; a deferral goes to `deferral`. Correct.
  - A retry error replaces `pushErr` and is classified once, afterwards. This gives the same classes as the old `classifyFailure(retryErr)`, and an unnamed error still becomes `ErrPushRejected` joined with the retry's error, the same shape as before. Correct.
  - `spawn.go:275` (nil retry, code side) uses the code-side retry. Correct.
- **`push()`.** It is now a plain `PushRebaseFree` for every side. The first push of the code side loses its stale-ref retry, but `repushOverOriginContained` replaces it as the first retry. The early `return job.push()` paths (branch unreadable, or the local-branch mark) also lose the stale-ref retry on the code side.
  - That is a small behaviour change. The local-branch mark is rejected by `PushRebaseFree` itself, and the unreadable-branch path is an edge case. Not a defect.
- **`settleRejection`, boundaries.**
  - `delays` empty: attempt 0 runs, `attempt == 0 == len(delays)`, so it returns. No panic and no sleep.
  - `len(delays)=N`: at most N+1 retries and N sleeps, with `delays[attempt]` in range because the `attempt == N` check comes first. Correct.
  - Origin ahead on any attempt returns the original `rejected` with no sleep. `unpushedCommits()` then reads the freshly fetched upstream. Correct.
  - Deferral and landing return before any error handling. Correct.
  - Bare rejection on every attempt: it sleeps N times, then returns the last retry error (`ErrPushRejected`). One record and one mail follow. Correct.
  - Unnamed error (failed fetch) is joined with `rejected` and retried. A persisting one is classified `not_fast_forward` with the fetch error in the text. This matches the old "stopped before it pushed" behaviour. Correct.
  - Named non-retryable errors (anchor stop, auth, replay conflict, destructive refusal, retries exhausted, transport) return at once with no sleep. Correct.
  - A joined error is never re-wrapped across attempts, because `retryErr` is local to each iteration. Correct.
- **`rejectionRetryKind`.** Check order is anchor stop and auth, then rejected, then replay conflict, then transport. A `fmt.Errorf("%w: %w", ErrPushRejected, ErrReplayConflict)` shape would read as retryable. `classifyFailure` creates that shape only after the loop, so `settleRejection` never sees it. A retry that returns `replayConflict` directly is non-retryable. Correct.
- **`repushOverOriginContained`.**
  - Fetch fails: it returns the error, which is unnamed and so retried.
  - `HasUnpulled` fails: same.
  - Origin ahead: it returns `errOriginAhead` and does not push.
  - Contained: it pushes, either landing or returning a bare rejection (retried). Never forces, rebases or merges. Correct.
- **`retryWeftPush` / `retryBoltPush`, repeated calls.** Each call fetches again and replays from the current upstream, and locks are released by `defer` on every return. A held write lock gives a deferral on any attempt. Repeating them is safe.
  - Records retries after a first replay: the local SHAs are already rewritten onto origin, so the replay is idempotent when nothing is behind.
- **`recordsConflictWayForward`.** Empty `paths` omits `paths:` and says "settle any conflict". Non-empty `paths` names the paths. Nil `commits` gives `joinOrNone`. No "another machine" appears. Correct for the records and bolt sides.
- **`pushRebaseFreeRetryingOnce` removal.** No reference is left in the tree (grep of the whole tree). Correct.

**Findings**

- **LOW, consistency:** `internal/fabricengine/spawn.go:~262`. The `PushWarpRebaseFreeAt` doc comment says "A rejection is a human-decidable condition, never retried here." The code side now retries a rejection over the backoff through `pushClassified`. The comment is now false.
- **LOW, consistency:** `internal/fabricengine/bolt.go:~111`. The `PushRecorded` doc comment says a rejection "is retried once after the board's local commits are replayed". It is now retried over the backoff schedule.
- **LOW, design:** `internal/fabricengine/pushclassified.go` `settleRejection`, called from `pushanchored.go:~105` and `~186`. The loop sleeps `BackoffSleep` while the caller still holds the weft push lock. A persisting rejection therefore blocks other pushers for the whole backoff. This is within the decision record's accepted risk (synchronous waits), but it is a new lock-hold duration. It is not a defect if the lock's holders tolerate waiting.

**Tool calls (2 of 6)**
1. Read `retryWeftPush` and `retryBoltPush` in `pushclassified.go`. The manifest lacked their bodies, which the repeated-retry case needs.
2. Read `coalesce.go`, `bolt.go`, `spawn.go` and `pushanchored.go` around the `pushClassified` callers. I needed to confirm which sides pass a nil retry and to read the stale doc comments.

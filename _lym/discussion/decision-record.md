# Decision record — async-push-race

## Goal

A push that loses a race with another push of the same branch lands on a later attempt instead of reporting a divergence, and a `not_fast_forward` notice never claims another machine wrote paths it cannot name.

## Scope

- `internal/fabricengine/pushclassified.go`: a rejected push re-checks origin, and pushes again, before it is recorded as `not_fast_forward`, for every side `pushClassified` pushes (code, records, bolt).
- `internal/fabricengine/pushanchored.go`: the code side's one-shot stale-tracking-ref retry is folded into that re-check.
- `internal/fabricengine/failedpush.go`: the records and bolt `not_fast_forward` way forward drops "another machine" and, with no conflicting paths, drops the `paths:` clause.
- `internal/fabricengine/doc.go`: the push classification section states the new retry and wording.
- Tests in `internal/fabricengine` for the retry and the wording.

Out of scope: a machine name on commits, and any change to how a replay conflict, an anchor stop, an auth refusal or a transport failure is classified.

## Decisions

### A rejected push re-checks origin before it is reported

Decision: when a push of any side is rejected, `pushClassified` runs a bounded re-check loop before it writes a `not_fast_forward` record.
On the code side each attempt fetches the branch and, while origin's tip is contained in the local tip, pushes again;
an attempt that finds origin holding commits the local branch lacks stops at once and reports the rejection.
On the records and bolt sides each attempt is the side's existing replay retry (`retryWeftPush`, `retryBoltPush`), which fetches, replays and pushes.
The loop runs again only after a bare rejection or an unclassified failure such as a fetch that failed, and stops on a landing, a deferral, a replay conflict, an anchor stop, an auth refusal or a transport failure.
Attempts are spaced by `gitrepo.PushBackoffDelays()`, slept through the injectable `SyncOptions.BackoffSleep`.

Rationale: the seen notices came from two concurrent pushes of one branch on one machine; the loser's rejection, or a fetch it made while the winner held the tracking ref, named a divergence the winner's landing had already removed.
Re-checking against a fresh fetch, more than once, tells such a race from a real divergence.

Bound: the loop never forces, rebases or merges, and a code-side push runs only while origin's tip is contained in the local tip, so it can only fast-forward origin.
It is bounded by the backoff schedule's length, so a rejection that persists, such as a declining hook, is still recorded and mailed after at most that many attempts.

### The code side's one-shot retry is replaced

Decision: `pushClassified` pushes the code side with a plain `PushRebaseFree`, and `pushRebaseFreeRetryingOnce` is removed, since the re-check loop does what it did and more.

Rationale: two retries of one shape on one path would differ only in their count.

### The not-fast-forward notice names no machine and no empty paths

Decision: `recordsConflictWayForward` states that origin holds commits this branch lacks, never that another machine wrote them.
With conflicting paths it names them and says origin's commits change the same paths;
with none it omits the `paths:` clause and says to settle any conflict instead of the named paths.

Rationale: no commit lym writes carries a machine name, so the notice cannot tell another machine's commits from this machine's own racing push; the condition the entry sets for naming another machine never holds.
`paths: none` contradicted the sentence it sat in.

## Constraints

- `PATTERN-fabric-git`: every push stays inside `internal/fabricengine`, rebase-free, with no destructive primitive outside `destroy.go`.
- `PATTERN-push-both-sides`: a push still refused after its retry leaves a failed-push record and a creel notice.
- `PATTERN-gitrepo-client-boundary`: git runs through `gitrepo` and `gitexec` only.
- `PATTERN-test-speed`: the new tests inject the backoff sleep and run in the lowest tier that fits, beside the existing push-class tests.

## Auto-mode assumptions

- The race cause is not pinned to one code path; the fix is the re-check the entry's Idea names, applied to every side, rather than a lock change.
- A persisting rejection keeps the `not_fast_forward` class even when origin is contained in the local tip, since reclassifying it is not asked for.

## Open risks

- A synchronous push whose rejection persists now waits out the backoff schedule before it returns, the same wait a gone branch or an unreachable origin already takes.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - doc-go-retry-once-sentence (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- A code-side push rejected more than once while origin holds no commit the local branch lacks lands, with no failed-push record and no mail.
- A records push whose retry is rejected again while origin holds no commit it lacks lands on a later attempt, with no record and no mail.
- A push rejected because origin holds commits the local branch lacks still writes a `not_fast_forward` record at once, with no backoff sleep on the code side.
- A rejection that persists through every attempt still writes one `not_fast_forward` record and one mail.
- The records and bolt way forward never contains "another machine" and, with no conflicting paths, never contains `paths:`.

## Notes for the plan writer

- `pushJob.awaitOrigin` is the existing backoff loop the re-check loop mirrors.

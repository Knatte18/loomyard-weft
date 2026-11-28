```yaml
lens: fix-consistency
outcome: reported
```

Task: consistency check of the fix's change set against its surroundings.

1. **MEDIUM**: `internal/fabricengine/add.go`, `Add` doc ("A read of origin that fails in transit runs again after each delay of a short backoff. That covers each probe, lookup and fetch of a branch the pair is made from.")
   - The first sentence claims every read of origin is retried, but one read `Add` makes is not.
   - `add.go:462` calls `warpDefaultBranch`, which falls through to `remoteDefaultBranch`, an unwrapped `ls-remote --symref origin HEAD` (`addadopt.go:68`). It runs when `AdoptLocal &amp;&amp; warpAdopted &amp;&amp; !AdoptOnly` and the local `origin/HEAD` is unreadable.
   - Worktree-Create does not set `AdoptLocal`, so only the doc's claim and the summary's "covers Add's later origin reads" list are affected.
   - Either wrap that call, or narrow the first sentence so it states only the covered reads.

2. **LOW**: `_lym/landing/summary.md`, the added bullets under `## What changes`, `## Design decisions` and `## Tests`
   - Several added lines run well past the 120-character limit in the prose rule, with two sentences or clauses that should be broken.
   - Examples are the "The same retry, … only on origin." line (about 224 characters) and the "A retried read adds seven seconds … by up to that." line. `awk` shows lines of 224, 168 and 198 characters. Some long lines, such as line 10, may predate this change.

I found nothing else inconsistent:
- No reference to the old names `retryOriginProbeInTransit` or `originProbeBackoffDelays` remains.
- All `resolveWeftBranch` callers pass the new argument.
- The "origin probe" wording in `battenshed/create.go`, `wire.go` and the tests still holds.

Tool calls (3 of 6):
- `git grep` across the whole repository for stale references. The output was too large to use, and my context lacked the repository-wide mentions.
- A narrowed `git grep`, plus a grep for `warpDefaultBranch`/`remoteDefaultBranch` and `awk` line lengths on the summary. My context lacked those call sites and measurements.
- `sed` on `battenshed/create.go` and `beamfork_test.go`. My context had not shown the lock-hold and probe wording there.

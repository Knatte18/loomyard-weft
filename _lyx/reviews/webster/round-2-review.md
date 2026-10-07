---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/pushanchored.go:65
    summary: New and changed comments still carry independent clauses joined by a comma on one line, against PATTERN-comment-line-breaks
  - id: F2
    severity: LOW
    class: consistency
    location: internal/loomcli/wiring.go:57
    summary: commitStatusDeps.Push comment says only a busy push lock names no side, but any error from taking the lock names none
  - id: F3
    severity: LOW
    class: scope
    location: internal/fabricengine/pushanchored.go:71
    summary: The retry helper's branch where the post-rejection fetch fails, so the bare rejection is returned without a second push, is reached by no test
---

Review range: `git merge-base main HEAD..HEAD` (no rework round exists, `product.parent` is `main`).
Verified by running `go build ./...` and the `integration` tests for `internal/fabricengine` with a coverage profile over `pushanchored.go`.

### [BLOCKING] Comments keep independent clauses on one line

**Location:** `internal/fabricengine/pushanchored.go:65`, `:87`, `:122`, `:125`; `internal/fabricengine/doc.go:584`, `:586`, `:587`; `internal/fabriccli/spawn.go:2`; `internal/fabriccli/weft_verbs.go:318`; `internal/loomcli/commitrecords.go:48`; `internal/loomcli/wiring.go:57`; `internal/statuscommit/statuscommit.go:43`; `internal/gitrepo/push.go:225`; and the test comments at `internal/fabricengine/pushanchored_integration_test.go:296`, `:377`, `internal/fabriccli/hubmutation_integration_test.go:289`, `internal/fabriccli/weftskippush_integration_test.go:79`, `internal/loomcli/wiring_commitstatus_integration_test.go:427`.
**Issue:** `PATTERN-comment-line-breaks` requires a break at every independent-clause boundary in a new or changed comment.
Each location above has a second clause with its own subject and verb after `, and`, `, so` or `;` on the same line, for example `LockWaitUnbounded blocks, and a holder lasts as long as git does` and `it never rebases, merges or forces, and a remote holding commits ... is left alone`.
The round-1 reflow fixed the sentence boundaries and left these clause boundaries.
**Fix:** Break each line at the clause boundary named above, leaving the wording unchanged.
A comma that joins a list item or a compound predicate stays on its line.

### [LOW] commitStatusDeps.Push comment names the exception too narrowly

**Location:** `internal/loomcli/wiring.go:57`
**Issue:** The comment reads "A failed push's error names the side that failed, and a push lock that stayed busy names none."
`PushPairAnchored` also returns `fabricengine: acquire push lock: %w` for a failure to create the lock directory or to take the lock, and that error names no side either.
The claim is loom-only and so does not collide with batten's bare `PushAnchored` rejection, but it is wrong for the loom wiring itself.
The statuscommit `Deps.Push` comment and the push-warns bullet are accurate for both wirings: they promise only that every push error warns and that the warning carries the push's own error, and both hold for loom and batten.
**Fix:** Say that an error from one side's push names that side and an error from taking the push lock names none, in two lines.

### [LOW] Fetch failure after a rejection is not covered

**Location:** `internal/fabricengine/pushanchored.go:71`
**Issue:** The coverage profile shows the `fetchErr != nil` branch of `pushRebaseFreeRetryingOnce` unexecuted.
Plan card 3 states the behavior: the second push happens only when the fetch succeeded, and otherwise the bare rejection is returned.
The unborn-HEAD skip, the second push rejected again, the diverged side and the stale-ref retry are each reached by `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`.
**Fix:** Add one step to that test: with an always-declining hook on the code side's bare and the code side's `remote.origin.url` pointing at a missing path while `remote.origin.pushurl` keeps the real bare, the push is rejected, the fetch fails, and `IsPushRejected` holds with the bare unmoved.

### Focus directive answers

- Comment line breaks: not met, see the first finding.
- `statuscommit` `Deps.Push` comment and push-warns bullet: both hold for the loom and batten wirings, since neither promises a side name; the loom-only comment in `wiring.go` is the second finding.
- `AcquireWriteLockWithin`: both remaining branches are reachable, since `TryLockContext` returns `(true, nil)`, `(false, ctx.Err())` or `(false, err)`; the deadline case and the lock-held case are covered by `TestAcquireWriteLockWithin`.
- New push helpers: the unborn-HEAD skip and the second push rejected again are reached by the scenario test; only the fetch-failure branch is not, see the third finding.
- `lyx loom` help text for `commit-records`: it says the verb pushes the task branch along with the records, which is what `docs/overview.md` says and what the verb does in a task pair; not a defect.

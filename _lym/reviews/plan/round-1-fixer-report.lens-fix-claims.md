Checked the fix's claims about the code at HEAD.

**LOW** — Card 3, the sentence on the package-`fabricengine` grep hits: "reach no seam read, since they drive `coalescePush`'s loop, the spawn gating, the paused `DetachedPush` exit, or `pushClassified` of a code side."
- The conclusion holds, but the reason given for `export_test.go` is not why it matched.
- Its matches for the search pattern are `spawnDetachedPushFn` lines 333, 344, 392 and 399–406 of `swapPushRecorder`, a stub that records a spawn and never pushes.
- The `pushClassified` code-side accessor (line 1092) does not match the search.
- So the list of reasons leaves out "a stubbed detached spawn".

**LOW** — Card 10, `recordedRuns`: "a verdict error fails the reconcile wrapped as "decode &lt;path&gt;", as today". The same "as today" sits under card 2's `StatusRecorded`.
- Today `readRunStatusRows` runs `json.Unmarshal` into the narrow `runStatusRows` (`replay.go:199-205`, `231-245`). The error text is "decode &lt;path&gt;: &lt;json error&gt;".
- `StatusRecorded` runs `ParseStatusTolerant`, which is `json.Unmarshal` into the full `shedengine.Status` (`status.go:241-247`). Card 2 also wraps its error "naming the status record".
- So the error text gains a second naming of the record.
- A record that decodes into the narrow struct but fails on a type mismatch in a field only `Status` declares now fails where today it reads recorded.
- Both cases are edge cases, but "as today" overstates it.

Claims checked that hold:
- **Card 7, protected refusal:**
  - `rejectionRetryKind` classes a protected refusal as named and not retryable (`pushclassified.go:361`).
  - `failRejection`'s judgment cases and the "a fetch found origin holding nothing this branch lacks" text match `pushclassified.go:664-690`.
  - `protectedParentWayForward` is set in `fail` on an `other` record when the push carried landings and `ProtectedBranchRefusal(cause)` holds (`pushclassified.go:563-571`).
- **Card 7, exhausted push:** `retryTransportPush` re-pushes only while `gitexec.IsTransportFailure` holds (`pushclassified.go:213-220`), and `classifyFailure` records an exhausted push as transport (`pushclassified.go:647`).
- **Cards 7 and 8, the pre-lock refusal:**
  - `coalescePushSides` takes its lock inside `step`, after `ensureWeftLockDirAt` (`coalesce.go:136-170`), so "first, before it takes the push lock" can be carried out.
  - `CoalescePushBothAt` runs through it too.
- **Card 9:** every scope runs `recordedPairs` and `recordedRuns`, judge-only included, after the judge has returned and when origin was reachable (`originreconcile.go:563-570`). So the codec refusal for every scope is justified.
- **Card 10:** `reconcileEnded` runs after `judgeAgainstOrigin` has returned and with no lock of its own (`originreconcile.go:558-581`).
- **Card 12:** `resetUnnamedTip` reads the transition trailer (`lastTransitionCodeTip`), `holdsCommit` and the reflog (`unmadeOriginCommit`), never the planted machine (`pushclassified.go:417-447`). `TestReturnToPreviousTransition` asserts the planted machine (`codetipsettle_integration_test.go:128-133`).

Tool calls:
1. Read `originreconcile.go` 547–600, the `export_test.go` hits for the card 3 search, and `coalesce.go` 100–175. My context lacked the order of the reconcile steps, what `export_test.go` matched, and where `coalescePushSides` takes its lock.
2. Read `gitrepo/push.go` 285–296, `ProtectedBranchRefusal`, `resetUnnamedTip`, `ParseStatusTolerant`, `runStatusRows` and the lock order in `PushPairAnchored`. My context lacked these bodies, which the protected, owner-guard and codec claims rest on.

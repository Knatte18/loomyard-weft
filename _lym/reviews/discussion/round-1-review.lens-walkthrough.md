```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end against the code it reuses.

**Decisions walked**
- D1: a batten Run-Shed transition and a loom skipped row. The seam commits with its trailers and returns. Purpose met.
- D2: adopt reads origin's tip and re-runs that row. Purpose met only if D6 fires, which fails for batten (see F1).
- D3, batten case: the prime's pusher pushes after a batten transition. Purpose missed (F1).
- D3, loom case: the pusher is registered under the pair worktree's name, then the run's processes are ended. Purpose missed (F3).
- D4: a transition commit while the pusher replays. The exclusion already exists today, so D4 decides nothing new (F5).
- D5, owed case: origin unreachable for longer than the push backoff, then the act rule runs. Purpose missed (F2).
- D5, refused case: a rejected push that is not a takeover. Record and notice are written. Purpose met.
- D6, loom case: a takeover is met by a pusher running outside the pair. What happens to the pair is undefined (F4).
- D6, batten case: a takeover is met by the prime's pusher. Purpose missed (F1).
- D7: the reconcile finds EntryStopped, the run is paused, and the pusher makes its next pass. Purpose missed (F6).
- D8: origin unreachable while the dobby schedules the run. Purpose missed (F2).

**Findings**

F1. BLOCKING, design. D3 and D6 (batten): the prime's pusher has no run slug, so a takeover is never read and is overwritten.
- D3 keys the pusher by worktree. The prime's records hold the commits of every batten run, so no single slug exists to pass to `pairteardown.PushRun` as `RunSlug`.
- With an empty slug, `takeoverStop` returns nil (`internal/fabricengine/takeover.go:110-111`). `retryWeftPush` then replays the local commits onto origin's tip (`internal/fabricengine/pushclassified.go:818-826`, and replay from 863).
- The stale local batten record then lands on top of the other machine's takeover entry, and no hand-over runs. The record never says how the prime's pusher picks the slug or slugs whose takeover it reads.

F2. BLOCKING, design. D5 and D8: an owed push from the pusher still writes a failed-push record, so the run is still held.
- D3 has the pusher push through `PushPairAnchored`, and its bound says it pushes "only the sides `PushPairAnchored` already pushes".
- `PushPairAnchored` writes owed-class records on three paths:
  - `PushFailureDeferred` once its backoff is spent (`recordOwedRecordsPush`, `internal/fabricengine/pushanchored.go:365-385`).
  - `PushFailureTeardownLock` when its teardown wait runs out (`internal/fabricengine/pushanchored.go:296-318`).
  - `PushFailureTransport` through `pushClassified` (`internal/fabricengine/pushclassified.go:532,627`).
- `FailedPushesOwedBy` reads exactly those classes (`internal/fabricengine/failedpush.go:283-286`). `ReadRun` turns them into `PushOwed` (`internal/pairteardown/runread.go:233-240`), and the act rule then answers wait.
- So the D8 run is held and the dobby stops scheduling it. The record states no change to `PushPairAnchored` or to that reader.

F3. MEDIUM, design. D3: registering "under its worktree, not under a run's slug" is the same key for a loom pair.
- A pair's worktree name is its slug (`internal/loomcli/seedinput.go:77-78`, and `statusPushRequest` sets `TeardownSlug` to `WorktreeName` in `internal/fabricengine/spawn.go`).
- runproc keys records by an owner directory (`internal/runproc/record.go:148`). `runproc.End(slug)` from a cancel or a teardown therefore ends the pair's pusher. That breaks D3's claim and its acceptance criterion "ending a run's processes leaves its worktree's pusher running".

F4. MEDIUM, design. D6: "with the effect it has today for the worktree's kind" leaves the pusher's input undecided.
- `HandOver`'s effect depends on the caller's `RunPush.InsidePair`, not on the worktree's kind (`internal/pairteardown/handover.go:146-152`).
- Today loom passes `InsidePair: true` (`internal/loomcli/wiring.go`, `commitStatusDepsGuardedBy`) and batten passes false. A pusher running outside the pair could pass either, which means the pair is either kept for the dobby or removed under a run that may still be acting.
- The record leaves this to the implementer.

F5. LOW, design. D4 decides nothing new.
- `CommitAnchoredPathsWithTrailers` → `CommitRecordsPaths` already takes the weft write lock with a blocking acquire (`internal/fabricengine/commitweftpaths.go:49-57`).
- `retryWeftPush` try-acquires the same lock (`internal/fabricengine/pushclassified.go:835-841`).
- D4's rationale, "without the lock a transition commit could land in the middle of a replay", is a false premise. The record should state this as existing behaviour, not as a decision.

F6. MEDIUM, design. D7: after the pusher pauses a run, its next pass pushes unchecked.
- `skipsCheck` returns skip when `pauseCovers` reports true, and `guard` then returns nil, so the push goes (`internal/loomcli/prepush.go`, `skipsCheck` and `guard`).
- In the pusher, commits the run made before the pause (D1 lets it go on) are pushed on the next pass. That contradicts D7's "the pusher pushes nothing more for that pair until the run resumes", and the record does not state the skip rule's change.

Tool calls:
1. `takeoverStop` and runproc owner keying: needed to check the prime pusher's slug-less push and how the pusher is registered, which the manifest lacked.
2. `seedSlug` and `ReadRun`'s owed-push seam: needed to confirm that a pair's worktree name equals its slug, and where `PushOwed` comes from.
3. `FailedPushesOwedBy` and the transport-failure recording sites: needed to confirm which record classes hold the run.

```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked each claim and bound in the decision record against the code already in context, plus two reads one hop further.

**Claims attacked**
- D1: the transition takes no admission and the pusher still passes the gate. **Held**: `PushPairAnchored` admits itself (`pushanchored.go:166`).
- D3: at most one pusher per records worktree, guarded by its lock. **Broken** by D5's "holds no lock while it sleeps" (F6).
- D3: the exit protocol (release, re-read, retake). **Held**, provided a spawned pusher try-acquires the lock and exits when it is held.
- D3: registering under the worktree, not the slug, means ending a run's processes does not end the pusher. **Broken** for a pair (F4).
- D3 bound: "pushes only the sides `PushPairAnchored` already pushes". **Held**.
- D4: `retryWeftPush` try-acquires the weft write lock and a replay that finds it held defers. **Held** (`pushclassified.go:835-840`).
- D4: without the decision a transition commit could land mid-replay. **Broken**: the commit already takes that lock (F5).
- D5: an owed retry writes no failed-push record. **Broken** (F1).
- D5: teardown's exclusive take never waits on the backoff. **Partly broken**: `PushPairAnchored`'s own records backoff (`pushRecordsSide`, `pushanchored.go:213-240`) sleeps while holding the teardown lock shared (`:183`). That wait is bounded, so it is noted and not filed.
- D5: the dobby tick catches a branch ahead of its tracking ref. **Broken** for a branch with no upstream (F7).
- D5: Worktree-Teardown is the one wait. **Breaks** a hub-wide lock bound (F8).
- D6: the pusher reads the takeover and calls `HandOver`. **Broken** for the prime's pusher (F2).
- D6: `HandOver` has "the effect it has today for the worktree's kind". **Broken**: the effect follows the caller's `InsidePair` flag, not the worktree's kind (F9).
- D6: unpushed transitions go under archive tags and are never replayed. **Broken** for batten through the prime's pusher (F2).
- D7: the pusher runs "the same exceptions and verdicts", then "pushes nothing more for that pair until the run resumes". **Broken** (F3).
- D8: `runfsm`'s push-owed input reads no owed state of the pusher. **Broken** (F1).
- Scope Out: a board status write spawns a detached sync. **Held** (`boardengine/board.go:79`, `:199`).
- D6: PR-Review's and Finalize's own origin checks. **Not attacked**: the code is not in context and the tool budget went elsewhere.
- Notes for the plan writer: the "unused `spawnStatusPush` variable". **Broken** (F10).

**Findings**

**F1 — BLOCKING, design. D5 and D8 (`decision-record.md`, D5 first bullet and D8).** The pusher's owed outcomes write failed-push records, so `runfsm` holds the run anyway. This breaks D8's bound and acceptance criteria D5 and D8.
- The record says the pusher pushes through `PushRun` and so `PushPairAnchored`. That function writes a failed-push record with `Runs=[slug]` itself for every owed class:
  - `PushFailureDeferred`, when the records side is still unpushed after its backoff (`pushanchored.go:230-233`, `:365-385`);
  - `PushFailureTeardownLock` (`pushanchored.go:296-318`);
  - `PushFailureTransport`, per its doc comment.
- `ReadRun` reads exactly those classes through `FailedPushesOwedBy` (`failedpush.go:276-288`) into `PushOwed` (`runread.go:232-238`). The act rule then answers wait.
- Counterexample: a loom darn walks skipped rows quickly. Each commit lands while the pusher's push is in flight, and commits made during a push keep the records side from reading as landed (`pushanchored.go:137-138`). Once the push backoff is spent, a `Deferred` record names the run, and the next dobby, pre-run or loom read holds it.
- With origin unreachable, a `Transport` record does the same, which breaks D8's criterion that "the dobby still schedules it".
- The record never says that `PushPairAnchored`'s record-writing, or the reader, changes.

**F2 — BLOCKING, design. D3 and D6 (`decision-record.md`, D3 "a prime's records side alone" and D6).** The prime's pusher has no run slug, so a batten run's takeover is never read, and its unpushed commits get replayed.
- `retryWeftPush` reads a takeover only when `runSlug != ""` (`pushclassified.go:818-826`). `PushRun` needs a single `slug` (`handover.go:74-86`).
- One pusher per prime records worktree serves every batten run on the hub, and the record names no slug for it.
- Counterexample:
  1. Batten run A on machine M has unpushed transitions in the prime's records.
  2. Machine N adopts A and pushes a takeover entry to the prime's records branch.
  3. M's prime pusher gets a rejected push and calls `retryWeftPush` with no slug.
  4. `takeoverStop` is skipped and the replay rebases A's local commits onto origin's tip and pushes them.
- Result: no hand-over, and A's stale transitions are replayed, against D6 ("never replayed") and `PATTERN-run-fsm`'s declared "hands over, never replays".

**F3 — BLOCKING, design. D7 (`decision-record.md`, D7 "pushes nothing more for that pair until the run resumes").** The "same exceptions" include "a pause covers the run", which makes the check skip, so the pusher pushes the paused run.
- `skipsCheck` returns skip when `pauseCovers()` (`loomcli/prepush.go:99-101`), and `guard()` then returns nil, so the push goes (`:116-118`).
- Today a paused run makes no further transitions, so the skip was harmless.
- Counterexample: under D1 the run committed rows past the point the reconcile judged. The pusher's reconcile pauses the run through `PausePair`. On the pusher's next pass, `skip()` reads the pause and returns true, and it pushes the stopped run's commits to a branch origin rewound or ended.
- The same skip applies at Finalize (`prepush.go:103-107`), so a pusher reconciling a Finalize row never checks it.

**F4 — MEDIUM, design. D3 (`decision-record.md`, D3 last bullet, and acceptance criterion "ending a run's processes leaves its worktree's pusher running").** For a task pair the worktree's name is the run's slug, so "under its worktree, not under a run's slug" names the same `runproc` owner.
- Today's pair push registers under `req.TeardownSlug = l.WorktreeName` (`fabricengine/spawn.go:124-127`, `:167-170`). The pre-push check also takes the slug from `location.WorktreeName` (`prepush.go:51`).
- Counterexample: pair teardown runs `runproc.End` for the pair's slug first (`PATTERN-pair-remote` background, Pair teardown). A cancel or teardown therefore ends the loom pusher mid-push.
- The record names no owner key that differs from the slug.

**F5 — MEDIUM, consistency. D4 (`decision-record.md`, D4 and its rationale).** The decision is existing behaviour, and its rationale states a hazard the code already excludes.
- Every transition commit goes `CommitAnchoredPathsWithTrailers` → `CommitAnchoredPaths` → `CommitRecordsPaths`, which takes the weft write lock with a blocking acquire (`commitweftpaths.go:49-57`, `trailercommit.go:92`).
- "Without the lock a transition commit could land in the middle of a replay" is a false premise.
- D4's acceptance criterion pins nothing new. The record should say the exclusion already holds and state that the pusher's commits must keep going through this path.

**F6 — MEDIUM, consistency. D3 against D5 (`decision-record.md`, D3 "holds a lock … while it runs" and D5 "the pusher holds no lock while it sleeps").** Read literally, D5's no-lock sleep includes the pusher lock, which allows two pushers for one worktree.
- Counterexample: the pusher sleeps in the owed backoff with its pusher lock released. The next transition's seam finds the lock free (D3: "starts a pusher after each commit when the lock is free") and spawns a second pusher. Both then loop on the same owed push.
- The record should say which locks are released during the sleep and keep the pusher lock out of that set.

**F7 — MEDIUM, design. D5 dobby tick (`decision-record.md`, D5 third bullet).** "A commit its origin tracking ref lacks" is undefined for a records branch with no upstream.
- A new pair's branch has none until its first push creates it (`fabricengine/spawn.go:144-146`).
- Counterexample: Worktree-Create commits the seeded record, and the seam's spawn of the pusher fails or the process dies before the first push. The branch has no tracking ref, the dobby's local read finds nothing ahead, and the commit stays unpushed until the next transition. That breaks the D5 rationale, "every path that can leave a commit behind has a restart".

**F8 — MEDIUM, design. D5 Worktree-Teardown (`decision-record.md`, D5 fifth bullet).** The teardown row holds the hub's prime lock for its whole run, and an owed push inside it can wait without end.
- `worktreeTeardownProducer` holds the prime lock for both calls (`battenshed/teardown.go`, the producer's doc comment).
- D5 puts an unbounded owed backoff in front of the removal without saying whether it runs inside that lock.
- Counterexample: origin is down during one run's teardown, and every other batten run's prime-lock user on the hub blocks until origin answers.
- The bound "Worktree-Teardown is the one row that waits" does not state this.

**F9 — MEDIUM, design. D6 (`decision-record.md`, D6 "with the effect it has today for the worktree's kind").** `HandOver`'s removal is decided by the caller's `RunPush.InsidePair`, not by the worktree's kind.
- Today loom passes `InsidePair: true`, which leaves the pair for the dobby (`loomcli/wiring.go:233`; `pairteardown/handover.go:146-148`). Batten's call from the prime passes false, which removes the pair (`handover.go:149-151`).
- The record does not say which flag the pusher passes. A pusher "outside the pair" with the default false removes a loom pair that today is left standing: a changed effect the record calls unchanged.

**F10 — LOW, consistency. Notes for the plan writer (`decision-record.md`, first note).** The note calls the `spawnStatusPush` variable unused, but `battencli/cleanup.go:89` calls it in `fabricCleanupDeps.Spawn`. A plan that deletes it as dead code breaks `lym cleanup`'s push.

**Tool calls**
1. `pairteardown/teardown.go`, grepped for archive and `RemovePair`, and the owed-push reader in `runread.go`: my context lacked which reader fills `PushOwed`.
2. `runread.go:140-160` and `fabricengine/failedpush.go` `FailedPushesOwedBy`: my context lacked which failed-push classes count as owed for the act rule.

```yaml
lens: interactions
outcome: reported
```

**Interactions lens: pairs checked**

- D1 × D5 (no step waits / Worktree-Teardown waits): broken. Two statements of the rule conflict.
- D3 × D5 (pusher pushes through `PushPairAnchored` / an owed retry writes no failed-push record): broken.
- D5 × D8 (no owed record / `runfsm` push-owed input reads nothing): broken, from the same cause as the pair above.
- D4 × D5 (a deferred replay is retried / no owed record): broken, from the same cause.
- D3 × D6 (one pusher per worktree, the prime's serving batten / hand-over of "the run" through `PushRun(slug)`): broken.
- D7 × D5 (a pause stops the pair's pusher / the dobby tick and the commit seam restart a pusher on any pair whose tip is ahead): broken.
- D7 × D3 (the pusher stops after a pause / the exit protocol re-reads the tip and goes on while it is ahead): broken.
- D3 × D6 and D5 (registered "under its worktree, not under a run's slug" / a pair's worktree name is its slug): broken.
- D3 × D5 acceptance criteria (a pusher starts whenever the lock is free / none starts while a refused record stands): broken, LOW.
- D1 × D2 (commit-only transitions / a takeover re-runs from origin): held.
- D2 × D6 (redone work accepted / the hand-over window bound): held.
- D3 × D4 (pusher replay / commit takes the weft write lock): held. The commit already takes the lock at `internal/fabricengine/commitweftpaths.go:53`.
- D3 lock × D5 dobby and seam starts (concurrent spawners): held, since the lock admits one pusher.
- D5 resume × D7 (resume starts the pusher / the pause lasts until resume): held.
- D6 × D7 (owned elsewhere hands over through the catch-up path inside the pusher): held.
- Constraints: `PATTERN-pair-remote` shared teardown lock × D5's lock-free backoff, held. `PATTERN-daemon-wakeups` × D5's backoff, held. `PATTERN-tolerant-records` × D1's retired `push_pending`, held.

**Findings**

1. BLOCKING · design · D5 first bullet, D8, D3 first bullet
   An owed push writes a failed-push record in the very path D3 picks, so D5's and D8's premise is false.
   `PushPairAnchored` writes a record for a deferred records side (`recordOwedRecordsPush`, `internal/fabricengine/pushanchored.go:365-385`) and for a held teardown lock (`teardownLockOwedError`, `pushanchored.go:296-318`). `IsPushOwed` reads exactly those classes as owed (`pushanchored.go:46-58`).
   `ReadRun` sets `PushOwed` from those records (`internal/pairteardown/runread.go:232-240`), so the act rule holds the run, which D8 says no longer happens.
   D5's restart gate ("no refused-push record stands") would also read the owed record as blocking unless the gate separates records by class, and the record never says so.
   Fix: state that the pusher's path stops writing owed-class records, or that the readers and the restart gate filter them by class.

2. BLOCKING · design · D3 (the batten pusher is the prime's) against D6
   The prime's pusher serves every batten run's records, but the hand-over D6 relies on reads a single run.
   `PushRun` takes one slug, and `retryWeftPush`'s takeover check reads that `runSlug` alone (`internal/pairteardown/handover.go:74-91`, `internal/fabricengine/pushclassified.go:818-826`).
   A per-worktree prime pusher has no slug, so the record leaves open which run's takeover a rejected prime push reads and hands over.
   D6's acceptance criterion holds for a loom pair only.

3. BLOCKING · design · D7 against D5's dobby and seam restarts and D3's exit protocol
   D7 says a paused pair's pusher "pushes nothing more until the run resumes", but no other Decision stops the pusher from coming back:
   - D3's exit protocol takes the lock again and goes on while the tip is ahead.
   - D5's dobby tick starts a pusher for any pair whose tip is ahead and has no refused record.
   - The reused check skips itself, and so lets the push through, when a pause covers the run (`skipsCheck`, `internal/loomcli/prepush.go`, the `pauseCovers()` branch).
   So the pushes D7 means to stop go through on the next tick.
   Fix: add the pause to the conditions on the exit protocol, the dobby tick and the seam's start, and state that the pre-push check's skip-on-pause no longer lets a pusher's push through.

4. MEDIUM · design · D3 last bullet against D6 and `PATTERN-child-processes`
   D3 says "registers under its worktree, not under a run's slug". A pair's worktree name is its slug (`statusPushRequest` uses `l.WorktreeName` as the teardown slug and pause slug, `internal/fabricengine/spawn.go`), and runproc owners are slugs or `HubOwner` (`internal/runproc/record.go:24`).
   So a pair pusher registered "under its worktree" sits under the run's slug, and the run's process end (`runproc.End` for the slug, and `EndSubjectProcesses` in `HandOver`, `internal/pairteardown/handover.go:103`) ends it. That contradicts D3's acceptance criterion "ending a run's processes leaves its worktree's pusher running".
   Fix: name the owner key the pusher registers under.

5. MEDIUM · consistency · D1 against D5 and the Auto-mode assumptions
   D1, an operator decision, says "no step waits for a push", with no exception.
   D5 makes Worktree-Teardown wait, and an auto-mode assumption calls that "the one wait D1 allows", which D1 does not state.
   Fix: state the exception once, in D1 or in D5's bound, and drop the duplicate assumption, or put it to the parent if it departs from the operator's wording.

6. LOW · consistency · Acceptance criteria D3 #1 against D5 #2
   "After a commit with the worktree's pusher lock free, one pusher starts" contradicts D5's "no commit starts that worktree's pusher" while a refused record stands.
   D3's seam bullet also omits the refused-record condition.
   Fix: add the condition to D3's seam bullet and to its criterion.

**Tool calls**

1. Grepped `internal/pairteardown/teardown.go`, `internal/fabricengine/*.go` and `internal/runproc` for the teardown lock's exclusive take and runproc's owner keys. My context lacked the teardown lock's order and `HubOwner`'s definition. The lock order stayed unconfirmed, so I raised no finding on it.

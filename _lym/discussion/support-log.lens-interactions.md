```yaml
lens: interactions
outcome: reported
```

Task: interactions lens over the decision record's D1–D6, the acceptance criteria and the constraints.

**Pairs checked**
- D1 × D2 (commit-only seam, then the seam spawns the pusher): held.
- D1 × D4 (reconcile removed from the transition and moved into the pusher; pause met by the local pause gate D1 keeps): held.
- D1 × D3 (`push_pending` forcing retired; resume and dobby restarts replace it): held, except for the prime-anchored records (F4).
- D1 × D5 (no fence before the act; takeover detected by the pusher): broken (F3).
- D2 × D3 (pusher exits on refusal; the next commit and the dobby restart it): broken (F1).
- D2 × D3, the at-most-one-pusher bound against two spawners: broken (F2).
- D3 × D6 (owed retried silently; the overview shows the push owed): broken (F5).
- D3 × D6 (resume starts the pusher; act rule's PushOwed): broken (F6).
- D3, teardown ends the pusher × teardown archive push: held (LOW F7).
- D4 × D5 (two hand-over routes from the pusher, owned-elsewhere and takeover): held, because the hand-over mark stops a second one.
- D5 × D2 (hand-over run from inside the pusher removes the pair the pusher may sit in): held (LOW F8).
- Constraint `PATTERN-pair-remote` shared teardown lock × D3's endless owed retry: held, as long as the backoff sleeps outside the lock. Not stated, LOW, folded into F7.
- Constraint `PATTERN-tolerant-records` × D1's retirement of `push_pending`: held.

**Findings**

F1 — MEDIUM — D2/D3: a persistent refusal loops.
- D3: a refused push writes a failed-push record and creel notice, and the pusher exits.
- The next commit restarts it (D2/D3).
- The dobby's tick restarts it as well, because a refused push leaves the tracking ref behind, so the "ahead of tracking ref" test stays true.
- A non-takeover refusal that keeps recurring, such as diverged history, therefore produces a refusal, record and notice on every transition and every dobby tick, without bound.
- Neither decision bounds restarts after a refusal.

F2 — MEDIUM — D2/D3: the at-most-one-pusher bound has no exclusion.
- D2 states the bound, and the acceptance criterion "with one alive, none starts" depends on it.
- The seam (D2), the dobby tick (D3) and a resume (D3) each start a pusher after a separate live-process read.
- That read-then-spawn is not atomic, and no decision names a lock or pid file taken under `lock`.
- The constraints mention `PATTERN-lym-files` only as "any marker or pid file".

F3 — MEDIUM — D1 × D5: the takeover window is unbounded during a long row.
- D2's pusher exits once origin is level. During a long agent row there is no transition, so no pusher runs.
- After another machine adopts, the original machine therefore learns of it only at its next commit, through the dobby's catch-up, or through an owner check. That can be hours of concurrent agent work on two machines.
- The transition into Finalize also no longer pushes first (D1). Finalize's own guard now races the pusher spawned by that same transition.
- D5's bound names the window only as "between the takeover push and its pusher's next rejected push". It does not state that a pusher may not exist then.
- Open risks covers only part of this: Finalize's owner-check reading.

F4 — MEDIUM — D3 × the batten records anchored at the prime: the dobby restart misses batten's transitions.
- Batten's status is anchored at the prime (`PATTERN-pair-remote`, "Batten bookend"), and D2 itself allows "in a prime records only".
- D3's dobby restart covers only "every pair of its runs whose local records branch" is ahead.
- If a batten pusher dies, for example during a reboot between Land and Worktree-Teardown, its prime records commits are restarted only by the next batten commit or a resume, not by the dobby.
- So D3's claim "every path that can leave a commit behind has a restart" does not hold for the prime side.
- Related: D2's exit test "origin holds the local tip" on the shared prime records branch can be kept false by other runs' commits.

F5 — MEDIUM — D3 × D6 and its acceptance criterion: the overview cannot show an owed push.
- D3: an owed push is retried inside the pusher, and only a refusal leaves a failed-push record.
- `ReadRun` fills `runfsm`'s PushOwed from failed-push records (`PATTERN-run-fsm`, "An owed push holds the run").
- So no record exists while a push is owed, yet D6 and its criterion say the batten overview shows the push owed.
- Either the pusher writes an owed record, or the criterion cannot pass.

F6 — MEDIUM — D6 × D3's resume and the act rule: a recorded owed push may still block the resume that is meant to restart the pusher.
- D6 keeps the PushOwed input "for the verbs and views that read it".
- `runfsm`'s act rule refuses on PushOwed, naming `FailedPushWayForward`.
- D3 makes a resume the restart path, and the resume is judged by the act rule.
- If the pusher does write failed or owed records (see F5), a resume after a crash is refused by the very state the pusher is meant to clear. D6 then contradicts itself: the run is still held.
- The record does not say whether the act rule drops PushOwed as a refusal reason.

F7 — LOW — D3, teardown ends the pusher:
- Teardown kills the pusher mid-push, and its archive push sends the weft branch alone.
- The code side of the last transitions is not carried. That is harmless after a landing, but not stated for a cancelled run whose pair is kept or archived.
- The backoff's relation to the shared teardown lock is not stated either: it must sleep outside the lock, or teardown's exclusive take waits on it.

F8 — LOW — D5 × D2, the hand-over run from inside the pusher:
- `HandOver` removes the local pair while the pusher calling it may have its working directory or locks in that pair.
- D5 covers process self-exclusion only, not the pusher's own working directory or held push lock during the removal.

Further tool calls: none.

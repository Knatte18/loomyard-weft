---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/09-creel-deliver-marked-probe.md:37
    summary: awaitMarkedProbe waits for the marked pause read before it advances and takes seen after the post, so a wake-driven probe that outruns the caller's mark or seen read leaves the helper waiting for a read or wait that never comes, and its 8 s bound contradicts its own retried passes
    fix: |-
      Rewrite the `awaitMarkedProbe` protocol in `_lyx/plan/09-creel-deliver-marked-probe.md` as one loop with no non-advancing first step; a variant of the card with these edits passes `lyx loom validate-plan --overlay` with no blocking finding the baseline lacks.
      1. Drop the `plan:internal/creelengine_test#env.awaitPauseRead` line from `**Create:**` and its bullet; nothing calls it once step 2 is made.
      2. In the `awaitMarkedProbe` bullet remove "it waits for the marked read (`awaitPauseRead(t, receiver, mark+1)`)" and the sentence "It then loops under the real-time deadline until `PausedCalls(receiver)` reaches `mark+2`". Say instead that the helper is one loop under a real-time deadline that only bounds a failing test, with no wait of its own for the marked read, because the marked read can depend on an advance the loop makes.
      3. Move `seen` before the post: say the caller takes `seen`, the probe-wait count (`e.clock.Waits()` filtered by `isProbeWait`), before the post (or interrupt) call, and `mark` after it returns. Replace the paragraph that says `seen` is taken "after the post (or interrupt) returns and so before the marked read can start" with this reason: the post's wake runs before the post returns, so a probe it starts can record its wait between the post's return and the caller's reads, and a wait counted into `seen` would leave pass 1 waiting for a wait that never comes. Keep the sentence that a wait an earlier unmarked probe records after `seen` only adds a retried pass.
      4. Replace the Bound sentence ("the first new probe wait is the receiver's own and the helper advances at most 8 s") with: in a subtest whose only live worker after the post is the receiver's, the helper makes at most three passes, 24 s: one for a probe in flight at the post, one for a wake-driven probe whose pause read came before `mark`, one for the marked probe's own wait. Keep "each probe wait another live worker records before the receiver parks adds one pass" and the per-check assertion that the total advanced is shorter than the shortest hold, TTL, back-off or stall freshness the check runs under (`stallFreshness` is 16 s).
      5. In the marker bullet add that the wake a post sends runs before the post returns, so the probe it starts can read the pause before the caller reads `mark`; that probe is not the marked one, and the marked probe then comes only after the clock advances past the wait that probe records.
      6. In the reshaping bullet change "takes its mark after the post and calls `awaitMarkedProbe`, advancing the clock past the worker's wait first where the post leaves nothing typed and the probe comes only then" to "takes `seen` before the post and its mark after it, and calls `awaitMarkedProbe`, which advances the clock itself where the post leaves nothing typed and the probe comes only then; the caller never advances first".
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/09-creel-deliver-marked-probe.md:54
    summary: the two helpers that post before e.start are told to take the same reshaping although no wake can precede their first probe, and awaitMarkedProbe would break the draft-read-once contract of holdAfterAReminderProbeReadTheDraft
    fix: |-
      In `_lyx/plan/09-creel-deliver-marked-probe.md`, in the bullet that ends "and the helpers `holdAfterAReminderProbeReadTheDraft` and `orchStrandIsRemindedAndTypedInto` take the same reshaping", replace that clause with: the helpers `holdAfterAReminderProbeReadTheDraft` and `orchStrandIsRemindedAndTypedInto` post before `e.start`, so `Start` creates the worker with no wake signal and its first probe snapshots every post; they keep their `probeWaits(1)` wait and neither calls `awaitMarkedProbe`. This leaves the target lists unchanged.
---

### [BLOCKING] awaitMarkedProbe can wait for a read or a wait that never comes

**Location:** `_lyx/plan/09-creel-deliver-marked-probe.md:37`

**Issue:**
The `Store.do` wake hook runs in the poster's goroutine after the store mutex is released and before `post` returns (`internal/creelengine/pool.go`), so the probe a post starts, or wakes, can read the pause on another core before the caller reads `mark`.
That probe is unmarked by the card's own rule, and with nothing typed it records a wait and parks, so the marked probe starts only after the clock advances past that wait.
The card's helper first runs `awaitPauseRead(t, receiver, mark+1)`, which polls `PausedCalls` and advances nothing, so in that interleaving it runs to its real-time deadline and fails the subtest.
The other reading of the bullet, that the caller advances first ("advancing the clock past the worker's wait first"), names no state the caller's advance waits on.
`creelfake.Clock.Advance` fires only timers already registered, and the worker registers `clock.After` only after a probe returns (`internal/creelengine/deliver.go`, `worker.run`), so an unguarded advance can land between the pause read and the `After` and be lost; the loop inside the helper is guarded by waiting for a newly recorded probe wait, but the caller-side advance is not.
The card also takes `seen` "after the post (or interrupt) returns".
A probe the post's wake started can finish, and record its wait, between the post's return and the caller's reads, so that wait sits inside `seen` and pass 1 waits for a wait that never comes.
The card covers only the converse, a wait recorded after `seen`.
Both interleavings are the ones `-cpu=2,4` provokes, and the acceptance criterion is that the deliver subtests pass under `-count=1000 -cpu=1,2,4`.
The Bound sentence says a lone worker's helper advances "at most 8 s", while the same bullet says an earlier unmarked probe's wait adds a retried pass.
Tracing a lone worker with a probe in flight at the post gives a second pass, and a wake-driven probe that read before `mark` gives a third, so up to 24 s; the negative-check assertion that the total is shorter than the stall freshness (`stallFreshness`, 16 s, in `internal/creelengine/pool.go`) would fail on two passes.
The fix replaces the two-step helper with one loop, takes `seen` before the post, and restates the bound; the loop cannot hang, because every pass waits for a wait recorded after `seen`, every advance fires every registered probe timer (all at most 8 s), and a worker with a pending post records a new wait after each probe.

### [LOW] Helpers that post before Start gain a reshaping they do not need

**Location:** `_lyx/plan/09-creel-deliver-marked-probe.md:54`

**Issue:**
`holdAfterAReminderProbeReadTheDraft` and `orchStrandIsRemindedAndTypedInto` post before `e.start(t)`.
`Scheduler.Start` creates the worker through `wake`, which sends no wake signal when it starts a worker, so the first probe snapshots every post and `probeWaits(1)` already waits for a probe whose snapshot includes them.
The card still says both "take the same reshaping".
For `holdAfterAReminderProbeReadTheDraft` the marked-probe reshaping is harmful as well as unneeded: its comment says the draft is read once by the reminder probe and the hold then stops the pane read, and `awaitMarkedProbe` ends only at a second pause read, which advances the clock and runs a second probe that reads the pane again before the hold is placed.
The decision record's D6 wording ("a probe whose snapshot was taken after the post") is already met by `probeWaits(1)` in these two helpers, so keeping their shape carries it.

## Rejected

- Card 3, `Repo.CherryPick` against `git cherry-pick --empty=drop`: not a defect. On this host's git 2.53 the card's probes (CHERRY_PICK_HEAD present, nothing unmerged, index equals HEAD, then `diff-tree --quiet --root`) reproduce `--empty=drop` for each outcome the card prescribes: an already-carried commit and a parentless commit over HEAD's own tree are dropped with no pick left, a commit empty from the start and a parentless empty-tree commit stop with the pick in progress, a failing `prepare-commit-msg` hook leaves CHERRY_PICK_HEAD set with the index differing from HEAD, a nonexistent object leaves no pick in progress, and `--skip` keeps an uncommitted tracked change. Every prescribed branch has a named step or is a propagated git error that the rubric exempts.
- Card 8, the unmarked and marked row assertions: not a defect. The unmarked render asserts only each range once, the client's lit full name and no comma, which hold in session-name and session-id order alike; the order is asserted only by the two marked renders, where the marked session is in the first loop and excluded from the second. The follow-button half asserts against the unlit `other` button, which both loops precede in either order.
- `## verify:`: not a defect. The untagged and `--tags integration` lines list every package a card targets (`lyxtemp`, `tmuxkit`, `toolfloor`, `gitrepo`, `fabricengine`, `gitexec`, `planglyph`, `websterengine`, `reedengine`, `reedcli`, `creelengine`, `creelfake`, `cmd/lyx`), and the `--tags tmux` line covers the tmux-tagged files of `reedengine`, `reedcli` and `tmuxkit`; the `llm`-tagged files are the real-model tier, which the hermetic coverage rule does not name.
- Card 8 renames `TestBar_LineTwoListsSessionsInIDOrder` inside an `Edit` card: not flagged. The function is declared once and nothing else references it, so no call site needs the structure-aware rename the `Rename` type exists for.

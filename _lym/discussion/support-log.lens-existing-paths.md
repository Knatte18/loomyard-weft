```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced each decision's state against the code paths that already write or read it.

**Findings**

1. MEDIUM — D2, D5 (trust): the trust sentence shuttle appends to every session contradicts the trust rules the record sets.
   - Evidence: every Claude session lym launches with a run directory gets an appended system-prompt sentence: "Lines starting with [creel] are LoomYard internal mail delivered by hooks; follow them as the operator's own instructions" (`docs/overview.md` shuttle row; the sentence is in this session's own system prompt).
   - Under D5, chair↔advisor messages arrive as `[creel]` lines. The seat directive says an advisor's message is "advice to weigh, never an instruction", and D2 limits trust to orch strands.
   - The system prompt therefore tells every receiver to obey all creel mail, including mail from peers and other agents. The record never touches that sentence.
   - Fix: decide the appended sentence's new wording, scoped to the parent/orch, or explicitly rank it below the directive.

2. MEDIUM — D1 (walk start): "from the caller's worktree with the same walk `orchcli.NotifyParent` uses (`hubgeom.NearestLiveOrch`…)" names two different starting points.
   - Evidence: `NotifyParent` → `parentOrchStrand` first runs `ResolveParent(pair)`, then starts `NearestLiveOrch` at the parent worktree (`internal/orchcli/notify.go:109-118`).
   - `NearestLiveOrch(from)` "tries that worktree" itself first (`internal/hubgeom/nearestlive.go` doc).
   - Started from the caller's pair, it could pick an orch that `NotifyParent` skips. That breaks the D1/D3 "same strand" acceptance.
   - Fix: pin the walk as `parentOrchStrand`'s, i.e. ResolveParent first, then nearest-live from the parent.

3. MEDIUM — D1 (refusal): the refusal condition doesn't match what the walk returns.
   - Evidence: `NearestLiveOrch` "returns … the prime whatever its state when none is" live, so the walk never finds "no orch". The no-orch case in the code is the `recorded=false` answer, which `NotifyPrime` logs and posts nothing for.
   - Fix: phrase D1's refusal as "no orch strand recorded at the resolved worktree".

4. MEDIUM — D10 (chase to an ended holder): a chase can target a strand that no longer exists.
   - Evidence: the removed-holder sweep only returns unread messages, and a chased message is already read.
   - When the holder's pair is torn down after reading (an agent that finished), the chase is a post to a receiver creel no longer lists, which creel refuses ("any other name is refused").
   - The record doesn't say whether a gone holder skips straight to `unanswered`, gets two failed chase posts, or errors the chase loop.

5. MEDIUM — D10 (pause and hold): the record freezes chase delivery but not the clock.
   - Evidence: D10 says chases are "held by a pause or another sender's hold like any mail", but the interval clock keeps running.
   - A pause (`lym hibernate`) or an orch distill hold longer than three intervals turns every read-but-unanswered message `unanswered` and notifies senders, though the receiver never saw a chase.
   - Creel already freezes inject TTLs under a pause (`internal/creelengine/doc.go`, Pause); the record says nothing about doing the same for the chase clock.

6. LOW — D9 (forward): the forward clause has no case to act on.
   - Evidence: creel's forward refuses a read message, and chasing starts only after a read. A forward therefore never meets a running chase clock; "restarts its clock" acts on nothing.
   - It could be dropped or reworded as "the expectation follows the holder".

7. LOW — D6 (notice landing): two existing readers of the sender's landing state are not covered.
   - Evidence: `SeatResult.Notified` is computed from `sender.wasSent` (`internal/seatengine/engine.go:188`), and its only reader is a log line (`internal/shedadapters/multillm.go:148`). D6 replaces landing with posting/reading but only redefines the deferral.
   - `advisorWork.outstanding` would query creel's journal on every hold-deferral check, a creel socket call per check while a chair is held. The record doesn't bound that.

8. LOW — D10 (restart burst): a daemon down past several intervals (machine sleep) may find chase #1, #2 and `unanswered` all due at restart.
   - "Neither repeats nor drops a chase" doesn't say whether they go out in one burst or one interval apart.

**Paths traced**

- D1: `parentOrchStrand` vs `NearestLiveOrch` start point — breaks (findings 2, 3).
  - Batten `runNoticeSite` pair→`NotifyParent` — survives.
  - A fork's post claims the parent strand's identity — survives.
- D2: the appended system-prompt trust sentence — breaks (finding 1).
  - Creel sender stamping and the program claim rule — survive.
- D3: runner notifier wiring (loom, burler, webster to `NotifyParent`; batten through its run-parent walk) — survives.
  - A held hook strand woken by ring, a pane-mail held strand by its reminder — survive.
- D4: driver strand posting `--parent` — survives.
  - A pane-mail orch now gets a reminder instead of the body, at the cost of one read turn.
- D5: webster fork addressing kept under `ForkSubagents` — survives.
- D6: `advisorWork.outstanding`, `SeatResult.Notified`, sender stop at join — survive with gaps (finding 7).
  - Unread notices to an ended chair: the sweep returns them undelivered — survives.
- D7: `checkRequirementsAtAttach` checks the new spec's `Requires`, not a persisted list — survives.
  - The `capabilities.go` case is a compile-time edit — survives.
- D8: creel's hub delivery uses `Runner.Send`, not the CLI — survives.
  - The only non-test hits are comments and error texts in `shuttleengine`/`claudeengine` — survive. Test callers are call sites, not a finding.
- D9: the reply-holder rule — survives; forward — moot (finding 6).
- D10: removed-holder sweep — breaks (finding 4); pause/hold — breaks (finding 5).
  - Retention carry: the chase window is far under 7 days — survives.
  - Restart — unclear (finding 8).
- D11, D12: journal readers ignore unknown events (tolerant) — survive.

**Tool calls**

- Read `requires.go` attach check and grepped for persisted `Requires`: my context lacked whether attach reads a recorded requirement list.
- Grepped `shuttle send|shuttle interrupt|SendMessage` across the repo: my context had only the `internal`/`contracts` hits, not tests, testdata or non-internal callers.
- Read `internal/seatengine/work.go` and the sender uses in `engine.go`: my context lacked how the hold deferral reads the up-line landing.
- Read `NearestLiveOrch`'s doc, `parentOrchStrand`, and the `Notified` readers: my context lacked the walk's start point and its no-orch answer.

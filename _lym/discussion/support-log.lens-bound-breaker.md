```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I attacked the record's claims, bounds and guarantees with concrete counterexamples. Four MEDIUM findings and four LOW findings broke a claim.

**Findings**

1. MEDIUM — D6 (and D3/D1 by extension): seat notices signed as a program may be refused by creel's identity rule.
   - Rule: `internal/creelengine/identity.go` refuses a program claim from any pane but an orch strand's ("only an orch strand's pane may claim a program name"). A Go client outside every pane may claim one.
   - Counterexample: on the LLM-driver path (`lym loom start`), the driver strand runs the step's `lym` process in its own pane. A seat engine posting the advisor up line as the program LOOM from that process is refused on Linux.
   - Effect: the chair never gets its up line. D6's deferral ("advisor whose up post the chair has not read") then holds until shuttle's 10-minute maximum. The chair, told to send nothing before the up line, stalls.
   - The record never states which process the seat poster runs in, or what happens when the post is refused.

2. MEDIUM — Constraints, `PATTERN-daemon-wakeups` claim ("the chase timer is folded into the delivery loop's existing waits").
   - The delivery loop runs "one worker per receiver with a queued inject or unread mail", and the worker "ends when nothing is pending" (creelengine doc, Delivery loop).
   - A marked message that was read and not answered is neither unread mail nor a queued inject, so no worker exists to fire its chase.
   - The claim about existing code is false. D10 needs a timer source of its own, and the record does not state that one is bounded.

3. MEDIUM — D10: a pause does not stop the chase clock.
   - Creel freezes an inject's TTL under a pause. D10 holds chase delivery under a pause but keeps the interval running.
   - Counterexample: a hibernate or hub pause over 45 minutes. Every marked message read just before the pause runs both chases into the held queue, then turns `unanswered`, posts the sender a notice and ends a waiting `--wait-reply` with `unanswered`, while the receiver never had a turn.
   - Another sender's hold on the receiver gives the same result.
   - The record also says nothing for a holder that goes away after reading. The removed-holder sweep only returns *unread* messages, so the chases post to a gone strand.

4. MEDIUM — D2 bound (sender stamp "off Linux from the strand-name environment, which every lym-launched session's shell guard refuses to set").
   - The shell guard is a Claude `PreToolUse(Bash)` hook (`internal/shellguard/rows.go`). The record's own Out list says no Codex hook is rendered, and Codex roles are this task's reason to exist.
   - The guard also matches only commands naming the variable; a script file that sets it passes.
   - Counterexample: off Linux, a Codex agent runs a script exporting `LYM_STRAND_NAME=<shortname>:orch` and posts. Creel stamps it as the orch. Under D2's widened trust, every agent with a parent obeys it as the operator's delegate.
   - The bound holds only for Claude on any OS, or for any engine on Linux.

5. LOW — D1 "same walk `orchcli.NotifyParent` uses (`hubgeom.NearestLiveOrch` …, from the caller's worktree)".
   - `NearestLiveOrch(from)` "tries that worktree" first, while `NotifyParent` starts its walk at the pair's *parent*. The record names both as one walk.
   - Counterexample: an agent running in a beam, which has its own live orch. A `--parent` started from the caller's worktree resolves to the beam's own orch, while `NotifyParent(pair)` resolves to the beam's parent. The question and the hold notice then split, which breaks the D1/D3 acceptance line.
   - Also, `NearestLiveOrch` returns the prime "whatever its state" and never "no orch". So D1's refusal "the walk finds no orch strand" is not a case the walk produces. It reduces to creel refusing an unknown receiver, or to a dormant prime orch silently holding the question.

6. LOW — D10 "a `post --reply` to a chase is taken as a reply to the message it chases".
   - This bypasses "only its holder may send one". The holder X gets chase #57 for #42, then forwards #42 to Y. X replies to #57, and creel re-addresses that as a reply to #42 although X no longer holds it.
   - The re-addressing needs the holder check at reply time.

7. LOW — D10 sender notice to a program sender.
   - A receiver must be a strand "or a program that has read once". A program sender that posts with `--expect-reply` and has never read gets its `unanswered` notice refused.
   - D10 excepts only EXTERNAL.

8. LOW — D13 scope versus fixtures.
   - `internal/seatengine/testdata/smoke-chair.md` and `smoke-advisor.md` still name `SendMessage`. They are outside `contracts/stencils/` and are not Go literals, so the scan passes.
   - The seat smoke prompts would then still drive real agents over `SendMessage` after D5/D7 move the seat channel to creel.

**Claims attacked**

- D1 "adds no privilege": held.
- D1 "same walk as NotifyParent": broken (finding 5).
- D1 refusal "no orch strand": broken; the walk never answers none (finding 5).
- D2 widened trust bound: broken off Linux for Codex (finding 4).
- D3 "every wiring points at NotifyParent or batten's run-parent walk": held. Loom, burler, webster and batten all do; batten's `NotifyFrom` from the parent site is equivalent.
- D4 watched variant never repeats batten's notices: held. The three cases are unchanged.
- D5 webster fork line stays under `ForkSubagents`: held. The Merriam spec sets `ForkSubagents: true`.
- D6 poster seam works from the step's process: broken on the LLM-driver path (finding 1).
- D7 old `peer-messaging` refused only as unknown, with no breakage at attach: held. Attach rebuilds the spec from the new binary.
- D8 creel verbs cover the shuttle verbs: held for the operator (EXTERNAL may inject or interrupt anywhere). A non-orch strand loses cross-strand interrupt. Nothing uses it, so not a finding.
- D10 at most two chases, restart-safe through the journal: held for restarts.
- D10 pause behaviour: broken (finding 3).
- D10 chase-reply re-addressing: broken (finding 6).
- D10 sender notice: broken for a program that never read (finding 7).
- Constraint on daemon wakeups and the existing loop: broken (finding 2).
- D12 report through Log from the prime: held.
- D13 allowlist rule: held for stencils; gap in fixtures (finding 8).

**Tool calls**

- `internal/hubgeom/nearestlive.go` doc, `creelengine/identity.go` program-claim lines and `shellguard/rows.go` strand-name rows: my context lacked the walk's start point, the program-claim pane rule and the guard's engine scope.
- creel store receiver lines (no match) and `battencli/notices.go` `noticeSite`: my context lacked batten's walk start, to compare it with D1.

# Support log — shuttle-gate-early-done

## Interview

Autonomous run: no operator answers; every pick below is the writer's, recorded in `## Question ledger`.

1. Board entry read; the board list holds no other entry that plans work on the shuttle turn-start hold or the Plan gate.
   `twill-fork-timing`, `twill-plan-followups` and the `twb-ftp-*` bench pairs are context: the bug was found by the fork-timing bench.
2. Evidence read: the two bench traces show `turn end waiting on background work` per fork, then a `busy` / `turn` state, `a turn started after the boundary, holding the gate send`, the release Warn 10.2 s later, the Plan gate pass, and `session state disagrees ... loop=done state=busy cause=turn`.
   The `twb-ftp-r-1` transcript's last entry is the task-notification user entry at 10:34:32.640Z; no assistant entry follows before the strand was removed at 10:34:45, so the model was thinking.
3. Code read: `boundaryIdle` clears the boundary on an unmatched turn start; `startHoldReleased` restores it once `turnStartHold.holds` releases; `holds` releases on a reported interrupt or after the pane reads idle on every poll for `turnStartIdleOverride`.
   The idle reading is `Startup == StartupReady && IdleSession(capture)`, whose running-turn test is the `esc to interrupt` needle.
   The hook-derived state is built from the same unmatched `turn_start`, so it cannot settle the question.
4. Release history: the override arrived in `e12fb6afb` (2026-10-08) for interrupted turns and missing `Stop` lines; the transcript interrupt prober arrived in `86982fa43` (2026-10-11).
5. Real-run audit: grep over every surviving `.lyx/logs` in the hub found the release Warn only in the two bench Plan-Write runs (and an old `orch watch` process's send, unrelated to a gate).
   Weft branches with twill Plan commits: `logging-cleanup` and `reed-multi-window` have `lens-review.md`; only the bench pairs lack it.
6. Design settled: D1–D4 in the record.
   A pane-capture loop was started over this session's own pane for the fork-notification turns, to learn what the pane shows while a notification turn thinks.
7. Pane capture result (Claude Code 2.1.296, 91 columns): with forks outstanding and the turn ended, the footer read `⏵⏵ bypass permissions on · 1 shell · ← 4 agents · ↓ to manage` over a fork list drawn below it; when a notification turn started, the footer read `… · /tasks to see subagents · esc to interrupt · ← 4 …`, cut at the right edge.
   So the running-turn hint shares a footer that grows with background-work hints and is truncated at the pane width; in this session it stayed visible, and the bench pane's idle reading was not reproduced.
   Answer to the board's first question: the pane test cannot be relied on to see a thinking model, and D1 removes the dependence.
8. Real-run audit widened after the lens review: every `loom: plan artifacts` commit in the records repository since 2026-10-12 was checked per task.
   Every task's first plan commit carries `lens-review.md` except `lyx-halt` (2026-10-12 13:46, `982ac406`) and the two bench pairs; `lyx-halt`'s pair is torn down, so its cause cannot be confirmed.
9. Lens review (six forks, all returned):
   - scenarios, BLOCKING, D1/D2 resume after a death mid-turn would hold every send to the deadline: fixed — a session start after the turn start lifts the veto; acceptance criterion added.
   - scenarios, MEDIUM, `Runner.SessionIdle` also decides through the hold: fixed — named in Scope, D1's bound and an acceptance criterion.
   - scenarios + pattern-fit, MEDIUM, D3 twill detection by a fresh `plan_producer` read can disagree with the spawn and leaves the loader unnamed: fixed — taken from the spawn's recorded fork-subagent authorization, no new config read.
   - premises, MEDIUM, Plan-Write and Plan-Burler-Fix share the `plan` gate kind: fixed — D3 says the wiring tells the Plan-Write row apart.
   - goal-scope + board-fit + premises, MEDIUM, real-run audit too narrow: fixed — widened (turn 8); `lyx-halt` found.
   - goal-scope, MEDIUM, the pane question left dangling: fixed — answered in turn 7, Scope Out keeps the classifier unchanged.
   - leanness, MEDIUM, Auto-mode assumptions repeated Scope Out and held evidence: fixed — replaced by the two assumptions the plan relies on.
   - board-fit, MEDIUM, overlap with `twill-fork-timing` on the directive stencil and `twill.go`: fixed — named in Constraints.
   - premises, LOW, D4 needs a second directive value and pinned marker tests change: fixed — D4 and Constraints.
   - pattern-fit, LOW, parity row is rewritten, not only added: fixed — Scope docs bullet.
   - pattern-fit, LOW, `docs/overview.md` missing from docs: fixed.
   - pattern-fit, LOW, refusal-spec row for the new finding: fixed — D3 states it is a validation finding, not a refusal.
   - pattern-fit, LOW, `SessionProber` method vs optional interface: fixed — D2 picks a separate optional capability.
   - board-fit, LOW, a second engine without the reading inherits the bug: fixed — Open risks.
   - board-fit, LOW, per-poll logging against `logging-cleanup`: fixed — D1 says the veto logs nothing per poll.
   - scenarios, LOW, partial last line and hook-before-transcript race: fixed — D2 reads complete lines only, Open risks names the race, acceptance criterion covers malformed and unterminated lines.
   - scenarios, LOW, "empty" undefined: fixed — whitespace-only counts as empty.
   - goal-scope, LOW, D3 overstated ("whatever ends the turn early"): fixed — D3's bound states presence, not timing.
   - goal-scope, LOW, D4 not licensed by the board: rejected — it is the parity verb's one caller in the writer's own loop, and costs one directive value.
   - goal-scope, LOW, Discussion has no backstop when the reading is unknown: rejected — unknown is today's behaviour, already stated in Scope Out and Open risks.
   - leanness, LOW, Goal walkthrough, D1 rationale arguing rejected options, padding constraints, duplicated evidence paths: fixed — Goal cut to one sentence, rationale states the positive reason, two constraints dropped, Notes cite the scratch copies only, the split-message scenario moved into Acceptance criteria.
   - board-fit, LOW, `fan-master-preread` and `fan-prompt-tuning` touch nearby stencils: rejected — merge risk only, no design consequence.

## Rejected alternatives

- **Fix the pane classifier to see a thinking model.**
  The pane shape that read idle is unknown, and any needle is tied to one Claude Code release; the transcript reading answers the question the hold asks without it.
- **Drop the idle-pane override.**
  A genuinely orphaned turn start (a missing `Stop` line) would then hold every send until `send_ready_timeout_s` and the gate boundary until the run deadline.
- **Lengthen `turnStartIdleOverride`.**
  Thinking time has no fixed bound, so any length fails on a long enough think, and every genuine orphan waits longer.
- **Trust the hook state over the pane.**
  The hook state's `busy` / `turn` is derived from the same unmatched `turn_start`, so it reads an orphan and a running turn alike.
- **Add `lens-review.md` to the Plan spec's `OutputFiles`.**
  `00-overview.md` is the documented done-sentinel, and output files drive completion classification and attach matching across producers; the gate is the narrower place.
- **A lens-review check for twill Discussion-Write.**
  The disposition is a turn inside a file the writer already wrote, so presence cannot be checked without a new parsed marker; D1 removes the cause seen.
- **Gate-only lens-review check with no verb mode.**
  `PATTERN-gate-self-check-parity` requires the verb.
- **Read `plan_producer` afresh in the gate.**
  A config edit between spawn and gate would make the gate demand a file the prompt never asked for, or drop the check for a twill spawn.
- **Add the in-flight reading as a `SessionProber` method.**
  Every implementer and fake would change; an optional capability gives "no capability, today's behaviour" directly.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): why was the held turn start released? Picked: the idle-pane override fired because the pane classifier read a thinking model as idle; fix with a transcript in-flight veto (D1, D2).
- Q2 (resolved, auto-pick): should the twill gate refuse Done until the lens review is written? Picked: yes for Plan-Write, with the parity verb mode (D3, D4); not for Discussion-Write.
- Q3 (resolved, auto-pick): did real runs lose their lens review? Picked: `lyx-halt` landed a plan without one, cause unconfirmed; every other twill Plan-Write since 2026-10-12 has one; no backfill.
- Q4 (resolved, auto-pick): change the pane classifier? Picked: no; the capture shows the hint shares a width-truncated footer, and the outcome no longer depends on it.
- Q6 (resolved, auto-pick): does a session start after an orphaned turn start lift the veto? Picked: yes, any source.
- Q7 (resolved, auto-pick): where does the gate learn the spawn was twill? Picked: the spawn's recorded fork-subagent authorization.
- Q5 (resolved, auto-pick): unknown in-flight reading? Picked: keep today's override, so an engine or transcript that cannot be read stalls nothing new.

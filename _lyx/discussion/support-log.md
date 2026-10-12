# Support log: go-driver-assistant

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

1. Read the board entry and the related notes `arbiter-when`, `judge-cap-call`, `orch-split`, `arbiter-relaunch-moved`, `report-timing`, `alignment-landing-order` and the running task `pause-esc-rewind`.
   The entry fixes the three tiers, the tackler's name, its cold spawn, the arbiter's three triggers and the case brief; open were where the Go driver lives, how the choice is made per task, how notices reach the parent, how a binary change is watched and how the Go driver avoids racing batten.
2. Explored the go arm (`lyx loom run`, detached, spawned by `lyx loom start`), the LLM driver stencil, guide and notify stencils, the arbiter verb and stencil, batten's child-driver resolution, Seed-Child's run-settings write, batten's InnerRun notices and publish retry, `orchcli.NotifyPrime` and `PATTERN-driver-choice-single-site`.
   Finding: today's go arm steps and exits with no settling.
3. Settled the driver choice as a loom config key reached through run settings (D1), the rule table mirroring the LLM driver's stop handling (D3), and the tackler as an arbiter-shaped cold agent (D7).
4. Found batten's publish retry acts on verify failure halts with a recorded target, and a Go goto on the same halts would be a second mover; whether batten's retry was the mover in `arbiter-relaunch-moved` is a hypothesis, not established. Settled D4: one actor per halt.
5. Found a Go driver that outlives its stop would keep an old build and hold the executable; settled D5 with a hand-over at the stop and batten's InnerRun owning the watch after hand-back.
6. Found a go-seeded halted run makes `lyx loom resume` refuse toward `lyx loom start`; settled D10 so resume spawns the Go driver.
7. Lens review: six forks (premises, goal-scope, board-fit, pattern-fit, scenarios, leanness), all returned. Findings merged across lenses, with dispositions:
   - premises + scenarios, BLOCKING, D2 liveness: the go arm is alive only by the run lock, free while settling. Fixed: D2 adds a driver lock read by start, resume, batten adoption and driver-dead notice; `drive` refuses over a held one.
   - scenarios + goal-scope + premises, BLOCKING/MEDIUM, D3/D4 gap for batten-covered halts, and premises' point that only Publish writes the failure record. Fixed: D3 row for verify failure halts (watched: batten's; unwatched: one Go retry), `driver_goto_max` dropped, D4 rewritten.
   - scenarios, BLOCKING, D7 no `applied` mark. Fixed: D7 adds it; acceptance covers a kill between re-step and settle record.
   - scenarios, MEDIUM, tackler `resume` at `blocked` without goto. Fixed: reads as `hand-back`, cause `precondition`.
   - scenarios, MEDIUM, batten retry vs binary re-step. Fixed: D5 excludes halts the retry takes or is taking.
   - scenarios, MEDIUM, stale park marker. Fixed: D2, `drive` removes it and its resume-report file.
   - scenarios + board-fit, MEDIUM, pause during a live agent and `pause-esc-rewind`. Fixed: D11 and D7 exclude paused intervals from the tackler deadline; R2 names `pause-esc-rewind` and the arbiter's deadline.
   - scenarios, MEDIUM, `parent_notice` relay after arbiter. Fixed: D3 and D8.
   - scenarios, MEDIUM, `busy` exit leaves nothing. Fixed: settle record, and a driverless running go child is noticed through the driver lock.
   - scenarios, MEDIUM, invalid inputs. Fixed: D1 invalid `driver_kind`, D7's malformed cases, acceptance criteria.
   - scenarios, LOW, moved run now running. Fixed: D12 exits.
   - scenarios, LOW, daemon-wakeups sentence omitted the pause hold. Fixed.
   - scenarios, LOW, "differs" vs "newer". Fixed: D5 states a rollback counts.
   - goal-scope, MEDIUM, relaunch after a parent decision unplaced. Fixed: D10 names batten's awaiting arm through start's go arm.
   - goal-scope, MEDIUM, D5 touches LLM children. Fixed: bounded by "no driver strand recorded".
   - goal-scope + board-fit, MEDIUM, R3/Q11 open and `orch-split`/`creel` routing. Fixed: D15 decides it and names both departures.
   - goal-scope + premises, LOW/MEDIUM, absent `child_driver` default is `go`. Fixed: D1 states the change.
   - goal-scope, LOW, `interrupted` under `handback` narrows the entry. Fixed: stated in D3.
   - goal-scope, LOW, D6 wider than the board's trigger until `judge-cap-call` lands. Fixed: stated in D6.
   - board-fit, MEDIUM, divergence from `orch-split`'s Run-Shed-steps-in-Go idea. Rejected: the hub-level loop is out of scope and the support log's rejected alternatives record why the task-level driver is the go arm; the record needs no cross-reference to act on it.
   - board-fit, LOW, `arbiter-when` case 2 differs. Fixed: stated in D6.
   - board-fit, LOW, tackler spawn failure and `run-window-bugs`. Fixed: D7 cause, R3.
   - board-fit, LOW, `report-timing` vocabulary. Fixed: D9 says report-timing maps the settler and the record carries its time.
   - board-fit + leanness, LOW, the race attributed to batten's retry. Fixed: worded as a hypothesis here; D4 no longer claims it.
   - premises, LOW, D8 parent. Fixed: the parent is the prime orch.
   - premises, LOW, `unseeded` escalate ineligible. Fixed: acceptance criterion.
   - premises + leanness, LOW, R1 in-process vs loop. Fixed: D2 names the `shedverbs` loop body; R1 cut.
   - pattern-fit, MEDIUM, config-layers and strictness. Fixed: D1 and Constraints.
   - pattern-fit, MEDIUM, shed-verb-set. Fixed: D2 adds `lyx loom drive` and leaves `lyx loom run` generic.
   - pattern-fit, LOW, resume refusal change. Fixed: D10 and Constraints.
   - pattern-fit, LOW, binary check backoff. Fixed: D5.
   - pattern-fit, LOW, tackler in-flight marker. Fixed: D7.
   - pattern-fit, LOW, process seam under `go test`. Fixed: Constraints.
   - leanness, MEDIUM, D5 and D7 rationales re-argue rejected alternatives. Fixed.
   - leanness, LOW, R3/R4 restate scope. Fixed: replaced.
   - leanness, LOW, repair-verbs deliberation in Scope. Fixed: cut.
   - leanness, LOW, generic constraints. Fixed: cut tmux, agent-name, tolerant-records, pane-text-injection and orch-pane-single-writer lines.
   - leanness, LOW, test-budget note. Fixed: cut.
   - leanness, LOW, Goal's last sentence. Fixed: cut.

## Rejected alternatives

- **Batten's InnerRun as the Go driver** (the shape `orch-split`'s go-driven-shed idea sketches) instead of the go arm: batten watches only batten-started runs and already carries notice, retry and parent-lock duties; the go arm already owns a go-seeded run's spawn and handshake.
- **Turning `lyx loom run` into the settle loop**: it is the generic `shedverbs` run body; changing it for loom is the reimplementation `PATTERN-shed-verb-set` bars.
- **Holding the run lock while settling** for liveness: it would block the orch's goto, batten's retry and the arbiter's re-step while an agent judges.
- **A `run_settings`-only switch read straight from the board map**: a hub could not set a default, and the key would exist in `run_settings` but nowhere in loom's config contract.
- **Parsing the goto target out of the envelope's reason text**: fragile against any wording change; the failure record carries the target as data.
- **A `driver_goto_max` cap on Go gotos**: the only structured target is the verify failure record, already bounded to one retry per run by batten's design.
- **The Go driver taking every verify failure goto, batten's publish retry included**: two movers on one halt; making batten stand down by driver kind would read the recorded driver outside the bootstrap verbs.
- **A parked Go driver that stays alive to watch its binary**: it would keep an old build in memory, hold the executable on Windows, and need a new resume channel since it has no pane.
- **A warm tackler on standby for the whole run**: rejected by the operator on the board (2026-10-12); a standby session knows nothing of a stop before it happens and must be watched for the whole run.
- **Giving the tackler the guide's repair verbs**: no repair in the 2026-10-12 count needed an LLM, and a cheap model with write powers widens the attack surface for no measured gain.
- **A `lyx loom tackler` CLI verb**: only the Go driver spawns it, and an in-process call adds no public surface.
- **Escalating a tackler timeout to the arbiter**: a silent cheap agent is no evidence the stop needs opus; the parent is the conservative way out.
- **Sending `interrupted` under `handback` to the tackler**: the policy says a live agent may still run, and no reading of logs makes a re-step safe.
- **Reworking the LLM driver to use the tackler**: outside the entry's scope, which keeps the LLM driver as the unchanged option.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Where does the Go driver run? A new hidden role verb `lyx loom drive`, spawned by start's go arm, holding a driver lock (D2).
- Q2 (auto-pick): How is the driver chosen per task? Loom key `driver_kind`, set through `run_settings`, typed `--child-driver` first (D1).
- Q3 (auto-pick): How does the Go driver reach the parent? `orchcli.NotifyPrime`, with the LLM driver's watched and unwatched rules (D8).
- Q4 (auto-pick): Who watches the binary after a hand-back? Batten's InnerRun, for a halted child with no driver strand and no driver or run lock (D5, A7).
- Q5 (auto-pick): Which gotos may the Go driver run by rule? The verify failure record's target, once per unwatched run (D4).
- Q6 (auto-pick): Tackler defaults: `haiku`, 10 minutes, 4 spawns per run (A4).
- Q7 (auto-pick): Does `lyx loom resume` spawn a Go driver? Yes, for a driverless go-seeded halted run (D10).
- Q8 (auto-pick): Tackler timeout or malformed verdict: hand back, not escalate (A5).
- Q9 (auto-pick): Does the Go driver get a strand or pane? No (A6).
- Q10 (auto-pick): The empty-reason `paused` stop: trace and fix in shed when found there, and a Go friction note at every sighting (D13).
- Q11 (auto-pick): How does the orch address a go-driven run? Through verbs; the orch role stencil gains one line (D15).

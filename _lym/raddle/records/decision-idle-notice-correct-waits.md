## Goal

The idle and held notices skip correct waits and name what an agent waits on

Takes: D11 whole: the quiet notice in `internal/dobbyengine/runnotice.go`, `readAgentActivity` in `internal/battencli/wire.go`, `Store.AwaitsReply` in `internal/creelengine/verbs.go` and `creelcli.AwaitsReplyProbe`, the held notice in `internal/shuttleengine/hold.go` and `Runner.SetReplyProbe` in `internal/shuttleengine/run.go`, and `verifyBudgetNotice` in `internal/loomshed/verifybudget.go`.
Delivers: The quiet notice raises nothing for a `Raddle-Write` run whose proposal passes with a live raddle agent or for an agent with an open question in creel, and names an unread reply; `AwaitsReply` read as two probes for the quiet notice while the held notice keeps the combined one; an idle notice names the live command, its run time and its last output, or the Finalize step and what it waits on; the held notice is not sent while a question is open or a reply unread, in every runner; the verify gate's past-budget notice names the failing runs or says none failed.
Exports: Fixed for consumers: `creelcli.OpenQuestionProbe` and `creelcli.UnreadReplyProbe`, which dobby-stall-chase's question chase reads.

## Scope

The part changes these packages:

- internal/dobbyengine (about 300 changed lines)
- internal/battencli (about 130 changed lines)
- internal/creelengine (about 130 changed lines)
- internal/creelcli (about 50 changed lines)
- internal/shuttleengine (about 160 changed lines)
- internal/loomshed (about 40 changed lines)

## Decisions

### D11 — The idle notice skips correct waits and names what an agent waits on

The quiet notice in `internal/dobbyengine/runnotice.go`, through `readAgentActivity` in `internal/battencli/wire.go`, reads these as correct waits and raises nothing for them:

- A run at `Raddle-Write` whose proposal passes its checks at the tick and whose raddle agent's session is live, waiting for the operator's approval;
  the dobby's wait reminders also remind of it while the operator is away.
  A proposal that does not pass, or an agent whose session ended, is no wait, so a dead raddle agent still shows.
- An agent holding its turn end with an open question in creel: a message of its own that expects a reply, neither answered nor settled.
  An agent that holds an unread reply is no wait; the notice names the reply it has not read.
  The creel chase and D10's chase still cover a question nobody answers.

`creelengine.Store.AwaitsReply` answers true both for an open question of the strand's own and while the strand holds an unread reply to one of its messages.
The quiet notice reads it split in two: a probe that answers only an open question of the strand's own,
and a probe of its own for the unread reply, so the notice can name it.
The held notice keeps the combined probe, since an unread reply starts a turn in a live session and the hold sends no notice for it (`internal/shuttleengine/doc.go`).

An idle notice that still fires names what the agent waits on instead of only "idle":
a live command the agent's pane runs, with how long it has run and when it last wrote output, or the Finalize step a run with no agent is at, with the lock holder or GitHub check it waits on.
The notice "ended a turn without its output files and is held", `internal/shuttleengine/hold.go`, is not sent while the agent has an open question in creel or holds an unread reply;
every runner that sends it reads the combined probe, not only the ones whose wiring sets a reply probe today.
The verify gate's past-budget notice, `internal/loomshed/verifybudget.go`, names the failing runs, if any, or says none has failed.

Bound: a wait is skipped only while its proof stands at the tick: a passing proposal with a live agent, an open question in creel; a wait with no proof still raises the notice.

Rationale: the study found the quiet notice firing for an operator's approval at `Raddle-Write` and for a held turn end awaiting a creel reply, both correct waits.

### Added after Discussion (parent, 2026-11-27): A stop that leaves no history entry counts once and is recorded (F37)

Decision: Adopt F37's fix: in D3, a stop that leaves no history entry (a producer error such as launch never ready or agent died) counts once too. The driver gives each stepped stop an identity that stays the same across a driver restart; every settle record of it, hand-back and arbiter restep records included, carries that identity; the count at a row is the distinct stops over the halt entries, the handed-on Awaiting entries and the identities of entry-less stops D3 does not exclude. A hand-back or arbiter restep record excludes only a stop already counted or excluded. In D7, the four facts of such a halt live in a stop record the driver writes through internal/state and dobby's child read reads, never in an earlier row's newest entry, and the stop identity is part of the halt key, so two successive failed halts at one row get different keys for D8's dedup, D7's post and D10's chase. Tier0 criteria: an agent launch that fails three times at one row with no history entry (one fresh start, a hand-back, a resume, a third failure) reaches settle: operator; a failed-state hand-back records the four facts, and a second one gets a new key and a second post.

Rationale: Without it the third halt at a row never turns severe for the commonest entry-less stops, which defeats the rule table's bound; the fix stays within D3 and D7.

### Added after Discussion (arbiter, 2026-11-27): A stop with no history entry is counted and recorded through its stop identity

Decision: The Discussion review's round-3 finding F37, that a producer error leaving the run failed appends no history entry and so is neither counted toward the third halt at a row nor recorded, is answered by D3 and D7 as they stand: D3's stop identity in the driver's stop record counts such a stop once, across a driver restart, and every settle record carries it; D7 records the four facts on that stop record and keys the halt by its row and stop identity, which D8's dedup and D10's chase read. The design needs no further change for it; the plan writer builds D3, D7, D8 and D10 and their criteria as written.

Rationale: appendHistory in internal/shedengine/run.go skips an empty outcome and StopKey in internal/shedrun/settle.go is the producer, history length and state, so the premise holds; the round-3 fix adds each part the finding's fix asked for: the identity, the narrowed nothing-count bullet, the stop record as the home of the four facts, the identity in the halt key, and the three-failure launch, second failed halt and restart criteria.

## Constraints

- PATTERN-config-layers: `beam.autonomy` stays read only through `dobbyengine`'s one lookup; its booleans keep their meaning, so no reinterpretation ships (D1).
  `beam.halt_chase_min` is a new key and needs no move table.
- PATTERN-config-key-names: every message and comment names `beam.autonomy`, `beam.halt_chase_min` and `slack.remind_after_min` in full through `configengine`'s key-name helper.
- PATTERN-shed-driver: the new driver rows sit in `internal/sheddriver`'s one rule table and stop chain; the park wait is a wait in the verb, not a new stop chain.
- PATTERN-run-fsm: the driver's and the dobby's settles act through `internal/runfsm`'s act rule; a resume past a bounce budget is the same transition the parent's resume makes.
- PATTERN-daemon-wakeups: the park wait and the chase are waits that wake at most once a second and back off while idle;
  the park wait is one an agent blocks on, so its cap comes from the keep-warm resolver; a test injects their clocks.
- PATTERN-agent-addressing: lym reaches Slack only by creel mail to `<shortname>:SLACK`, and only `internal/slackengine` calls Slack (D7, D8).
- PATTERN-refusal-way-forward: the refusal of an unknown level, the park wait's refusal past its cap and D12's warning each name a way forward a test reaches, and none names a step agents' settings deny.
- PATTERN-stencil-ownership and PATTERN-heddle-instructions: the role texts and the glossary are changed in their stencil and heddle files, read at call time.
- PATTERN-leaf-packages: `internal/stopclass` (new) imports only the standard library and does no I/O.
- PATTERN-tolerant-records: the stop record's new facts and the dobby's chase and post records refuse no unknown key and are written through `internal/state`.
- This task builds on `push-retry-not-halt`, which changes `internal/sheddriver/driverules.go`, the dobby's run watch and the halt classifier, and adds the driver's wait record (its D6), which D5 reads.

## Auto-mode assumptions

- Under `true` the severe classes stay as today's role text lists them, scope change and cancel included; only `unattended` narrows them.
- A severe-halt post mentions the operator under both levels, since both levels relay severe halts to Slack today.
- The fixer report's `deferred` disposition is what "a finding the fixer deferred" means.
- `beam.halt_chase_min` defaults to 10 minutes; the study shows 65 of 73 questions answered within two minutes.
- The bounds in the table are the study's proposed ones, with lym's board-claim bound one, so that every row reads lym, then orch, then severe.

## Open risks

- `push-retry-not-halt` changes the files D4, D5, D7 and D11 change; this task's runs must start after it lands, so the raddle promotion names it in `depends_on`.

## Acceptance criteria

- D11: a test pins that the quiet notice raises nothing for a run at `Raddle-Write` whose proposal passes with a live agent and for an agent with an open question in creel,
  and still raises it for a `Raddle-Write` run whose proposal does not pass or whose agent ended, and for an agent holding an unread reply, naming that reply; tier1 or the lowest that fits.
- D11: a test pins that an idle notice for an agent running a live command names the command, its run time and its last output time; tier1 or the lowest that fits.
- D11: a test pins that the held notice is not sent while the agent has an open question in creel, in a runner whose wiring sets no reply probe today among them,
  nor while it holds an unread reply, and that the verify gate's past-budget notice names the failing runs or says none failed; tier1 or the lowest that fits.

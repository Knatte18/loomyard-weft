# Card 2 — One admit check admits a re-launch whose verdict is already in effect

**Create:**
- `internal/loomcli#admitRelaunch`

**Edit:**
- `internal/loomcli#consumeArbiterAttempt`
- `internal/loomcli#consumeTacklerAttempt`
- `internal/loomcli#TestConsumeArbiterAttempt`
- `internal/loomcli#TestConsumeTacklerAttempt`
- `internal/loomcli/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:**
D1: the arbiter's and the tackler's consume share one admit check, so the two cannot drift apart.

`admitRelaunch` lives in `internal/loomcli/arbiterconsume.go`.
It takes the status under the lock, the attempt's recorded stop key and pause conditions, the envelope's goto target and the actor word (`arbiter` or `tackler`) the refusal texts name, and returns either `inEffect` or a refusal reason; an empty reason with `inEffect` false is today's unmoved admit.

- Unmoved (`loomengine.StopKeyOf(current) == key`): today's equality check; pause conditions other than `pauseBefore`/`pauseAfter` return "the run was paused since the <actor> spawned", so a pause condition cleared by hand since the spawn still refuses.
- Moved, verdict in effect: `target` is non-empty, `current.CurrentProducer == target`, `current.State` is `paused`, `len(current.History) > key.HistoryLength`, every entry from index `key.HistoryLength` on has outcome `shedengine.OutcomeGoto`, and the last names `target`.
  The pause check then refuses only a condition present now that the spawn did not record: `current.PauseBefore != "" && current.PauseBefore != pauseBefore`, and the same for `PauseAfter`, with the paused reason above; otherwise it returns `inEffect` true.
- Any other move returns "the run moved since the <actor> spawned: its stop key is no longer the attempt's".

Both refusal texts are today's, so no exact-string assertion elsewhere changes.

Each consume's `Admit` closure calls `admitRelaunch` with its attempt's fields and `request.Target`, refuses through its own `refuse` with the reason, and otherwise writes its applied mark (`loomengine.MarkArbiterApplied` or `loomengine.MarkTacklerApplied`) and returns `inEffect` with the mark's error, so card 1's `Relaunch` skips the second goto.
The consumes' doc comments name the in-effect admit.

Bound: besides the verdict's own move, this admits a parent's goto to the same row, which the re-launch then steps without the parent's resume, as the Go driver's moved-run recheck already does for an orch goto.
A step after the goto, a different row, a resume with no goto and an already-applied attempt are refused as today, under the same status lock; the applied mark is written in that same hold, so a second re-launch on the attempt is refused as already applied.
The loop that then steps the run is the one every consume returns into (`shedverbs`' `--until-stop --arbiter-attempt` waiter, or the Go driver's re-step), so a hub pause holds it before stepping exactly as it holds any step; this card changes nothing after `Admit`.

Test rows, each reached through the real `shedengine.Relaunch` over a temp status file:

`TestConsumeArbiterAttempt` gains a `spawn func(a *loomengine.ArbiterAttempt)` column, applied to the attempt record before it is written, and a `wantHistory int` column asserted against `len(got.History)` on success rows when non-zero.
- "verdict already in effect is consumed with no second goto": envelope goto `Plan-Write`; mutate appends a `Plan-Write` goto entry, sets `CurrentProducer` to `Plan-Write` and `State` to paused; wants producer `Plan-Write`, the applied mark and `wantHistory: 3`.
- "a pause the goto cleared does not refuse the in-effect admit": as above with `spawn` setting `PauseBefore: "Plan-Bouncer"` and the mutated status carrying none; consumed.
- "a pause added since the spawn refuses the in-effect admit": as the first row with mutate also setting `PauseAfter: "Plan-Write"`; wants "paused since the arbiter spawned".
- "an unmoved run whose pause was cleared by hand is refused": `spawn` sets `PauseBefore: "Plan-Bouncer"`, no mutate, resume envelope; wants "paused since the arbiter spawned".
- "moved by a goto to another row is refused": envelope goto `Plan-Write`; mutate appends a `Plan-Bouncer` goto entry and sets state paused; wants "the run moved".
- "moved by a goto and then stepped is refused": envelope goto `Plan-Write`; mutate appends a `Plan-Write` goto entry, a `Plan-Write` stuck entry (`shedengine.Stuck`) and a second `Plan-Write` goto entry, sets `CurrentProducer` to `Plan-Write` and `State` to paused; wants "the run moved".
  Producer, state and last-goto-names-target all hold, so the row isolates the every-entry-is-a-goto conjunct.
The existing "stop key moved by a goto" row stays the resume-with-no-goto-over-a-moved-run case, and "already applied" the applied case.

`TestConsumeTacklerAttempt` gains the `wantHistory int` column and, since both consumes call the one `admitRelaunch`, only the rows that prove the tackler's wiring of it: "verdict already in effect is consumed with no second goto", "a pause added since the spawn refuses the in-effect admit" and "moved by a goto to another row is refused", with "tackler" in the wanted texts.
Its refusal branch's unmoved check compares `got.CurrentProducer` against the status the row wrote before the call (the mutated one when `mutate` is set) instead of the literal `Plan-Bouncer`, since the in-effect refusal rows start from a run already at `Plan-Write`.

`contracts/specs/refusal-spec.md`'s row "step --arbiter-attempt: run moved or pause conditions changed since the arbiter spawned" narrows its trigger: a stop key other than the attempt's, unless the run stands paused at the verdict's goto target with only goto entries since the spawn and the last naming that target; or pause conditions other than those recorded at the spawn, where on that verdict-in-effect admit only a condition present now that the spawn did not record refuses.
The tackler's consume has no refusal-spec row of its own, being the Go driver's in-process call, so no second row changes.
`internal/loomcli/doc.go`'s `# Arbiter` and `# Tackler` paragraphs on the re-launch say the same in one sentence each.

**ImpactSummary:** Two consume closures in `loomcli` and their two table tests, one spec row and two doc paragraphs; refusal wording is unchanged, so no other test reads differently.

**Commit:** 2: Admit a re-launch whose arbiter or tackler verdict is already in effect

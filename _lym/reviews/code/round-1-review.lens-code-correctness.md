```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function and branch with a concrete input.

**Per-item list**
- `arbiterstop.Eligibility`, awaiting with pending `decision`: returns nil, correct.
- `arbiterstop.Eligibility`, awaiting with pending `budget`: refusal ends with `HandBackWayForward`, correct.
- `arbiterstop.Eligibility`, awaiting with `EscalationErr` set or nothing pending: the earlier cases still refuse first, correct.
- `arbiterstop.Check`, awaiting `decision`: `briefPath` is the escalation brief, correct.
- `noticeArbiterSettled`, `OrchRecorded` returns an error: logged, nothing posted, driver goes on, correct.
- `noticeArbiterSettled`, nothing recorded: logged, nothing posted, correct.
- `noticeArbiterSettled`, `Notify` returns an error: warned, driver goes on, correct.
- `noticeArbiterSettled`, `Notify` returns `sent=false` with a nil error: dropped with no log, report line or friction note. This is acceptable because the notice is advisory.
- `noticeArbiterSettled`, arbiter resume with no goto: the notice still reads "resumed the run", correct.
- `noticeArbiterSettled`, arbiter resume with the consume refused in the next step: **not correct** (finding 1).
- `arbitrate`, moved run: returns before the notice, so no notice, correct.
- `arbitrate`, hand-on: no outcome notice, and the existing relay and park path is unchanged, correct.
- Stencil `decision` rule, `continue` at the hard cap: the stencil limits `continue` to below the cap, correct.
- Stencil `hard-cap` rule, "record it as a decision and accept": **not correct** (finding 2).
- Stencil `decision` rule, an accept the bouncer's refusal check holds: **gap** (finding 3).

**Findings**

1. MEDIUM, `design`: `internal/sheddriver/drive.go:553-559`, `noticeArbiterSettled` runs before the verdict is applied.
   - The arbiter's `resume` is only consumed in the next step, through `pendingAttempt` and `ConsumeAttempt` (`internal/shedarbiter/consume.go:50-59`). That step can refuse: an already-applied attempt, a changed stop key, or changed pause conditions.
   - On a refusal the run is handed back, but the orch has already been sent "the arbiter settled the review escalation … and resumed the run … Nothing to answer."
   - The orch then gets a hand-back notice after being told to take no action, and the settled claim is false.
   - The tackler path consumes before it settles (`drive.go:482-495`).
   - Fix: send the notice only once the re-step has consumed the attempt, or word it as the arbiter's verdict rather than a resumed run.

2. MEDIUM, `design`: `contracts/stencils/loom/loom-template-arbiter.md:85-88`, the `hard-cap` rule says to record the ruling "as a decision and accept, so the run's own agents carry it out".
   - At the hard cap `continue` is refused, and an accept ends the review segment. No fixer or reviewer round follows to carry the recorded decision out.
   - An open design-wrong finding is therefore accepted over with nothing that acts on the decision.
   - The escalation brief's rule "never accept over an open design-wrong finding unless a recorded operator decision covers it" (`bouncer-template-escalation.md`) is overridden by the stencil's line that its prompt wins.
   - The same flaw sits in the `circling` rule at the cap (`loom-template-arbiter.md:90-91`, "accept at it").

3. LOW, `design`: `contracts/stencils/loom/loom-template-arbiter.md:92-94`, the `decision` rule ignores the refusal check on an accept.
   - `decisionRefusal` (`internal/shedadapters/bouncerdecision.go:191-196`) can hold an arbiter's accept, via `refusalLeavingOut`. It re-judges, re-escalates below the hard cap, or halts with `DecisionHaltPrefix` at the budget at or past the cap.
   - The stencil never tells the arbiter to read the brief's "Refusal" section before choosing accept.
   - A held accept re-escalates to the arbiter and uses another spawn. At the cap it becomes a parent-only decision halt after the notice said nothing was left to answer.
   - Evidence: `bouncerdecision.go:157-175` and the `decision` rule's wording.

Tool calls: none; the inherited context already held everything needed.

```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set I found and whether the diff covers it.**

1. **`decision` made eligible in `arbiterstop.Eligibility`.**
   - Covered:
     - the `arbiterstop` doc comment and `doc.go`
     - `loomcli/doc.go` at lines 207 and 263
     - `sheddriver` rule table (`EscalationPending`, via the one `arbiterstop` rule)
     - `loomcli/drivewiring.go:141`
     - `adoptcli` (`EscalationEligible`, which reads the same `Check`)
     - batten hold (shares the rule)
     - `TestEscalationEligible`
   - Not covered: see findings 1 to 3.

2. **Arbiter stencil rewrite.**
   - The repair verbs (`circling accept|continue`, `decision add --by arbiter`) already exist in `loomcli/arbiterprompt.go:17-20`.
   - The `--by arbiter` guard already exists in `loomcli/decision.go:690`.
   - The resume precondition in `shedarbiter/call.go:325` already requires a circling decision first.
   - Covered.
   - Not covered: finding 2.

3. **Outcome notice.**
   - Covered:
     - the `drive.go` code
     - the `sheddriver/doc.go` line
     - `orch-role.md:122`
     - the `TestGoDriver_Settle` row
   - Not covered: finding 4.

4. **Bouncer escalation brief and `EscalationCause` comment.** The brief (`bouncer-template-escalation.md`) and the `EscalationCause` comment (`shedadapters/escalation.go:18`) were updated.
   - Not covered: finding 1.

5. **Orch role.** `orch-role.md:131` was updated.
   - It still says at line 125 that a `decision` escalation is a run-level call the orch settles, and at line 154 that a review escalation without the arbiter's own hand-on is a fault.
   - Those two lines now fit, since a `decision` escalation now reaches the orch only as a hand-on. Covered.

6. **PATTERN declared changes.** `PATTERN.md` and `pattern/` were not touched, and I found no declared change in the constraints section. Nothing to land.

7. **Commit messages.** Neither carries a `Pattern-Departure:` trailer, and I found no departure that needs one.

**Findings**

1. **MEDIUM, consistency.** Prose that defines a `decision` finding or escalation as "only the run's parent can settle" was left unchanged.
   - Locations:
     - `contracts/stencils/bouncer/bouncer-fragment-decision.md:1,7-8`
     - `internal/shedadapters/doc.go:108`
     - `internal/loomcli/circling.go:300`
   - Evidence:
     - The fragment is the `cause_detail` of the brief the arbiter reads. It says "Each finding below is a question only the run's parent can settle. Put each to the operator, unless a recorded operator decision answers it."
     - This contradicts the changed `bouncer-template-escalation.md` line (the review "cannot settle itself") and the arbiter stencil's `decision` rule (record the answer, then accept).
   - Not changed, and not in this finding: `loom-rubric-discussion-review.md:66,71` defines the `decision` finding class for the discussion review, and `shed-recipe-spec.md:61` still says "escalate to the run's parent". Both read as class and recipe text rather than who settles the escalation, so I leave them to the handler.

2. **MEDIUM, design.** The arbiter stencil's "What you may change" paragraph says the escalation brief's steps "are the parent's procedure, which counts only operator decisions; yours is this prompt, which wins where they differ."
   - Location: `loom-template-arbiter.md:60`.
   - Evidence:
     - The brief it reads, `bouncer-template-escalation.md:35-46`, says "Settle the question only when a recorded operator decision answers…" and "Never … accept over an open design-wrong finding unless a recorded operator decision covers that finding."
     - It also says the brief "never counts" a `--by arbiter` entry.
     - The stencil's override is a single sentence, with no list of which brief rules it replaces. Which brief rules still bind the arbiter is left open.
     - The new `hard-cap` rule says "record it as a decision and accept", which is accepting over an open design-wrong finding on the arbiter's own entry. That is what the brief forbids.
     - The brief also keeps the wording "report back to your parent… the run stays awaiting" for the fork.

3. **LOW, consistency.** The `escalation.go:18` comment says a `decision` escalation is "a call for the arbiter and, when it hands on, the run's parent". The same doc.go line (`shedadapters/doc.go:108`) was not updated, so the two now disagree.
   - This is the same set as finding 1, listed so the handler can cite it.

4. **LOW, consistency.** The `drive.go` comment and `sheddriver/doc.go:75` say the notice goes out "after an arbiter `resume` at an awaiting stop".
   - The notice goes out in `arbitrate` before the re-step consumes the attempt, and `ConsumeAttempt` can still refuse (`consume.go:52-57`: already applied, a changed stop key, changed pause conditions). The line says "resumed the run … Nothing to answer."
   - On a refused consume the parent gets that line and then a hand-back notice. This is a timing and claim mismatch.
   - Code-correctness may judge the behaviour; I list the wording and the unmatched claim in the doc.

**Tool calls**
1. `grep` over `contracts/specs`, `orchrole`, `parent`, `darn`, `webster` and `docs/*.md`: my context lacked those docs. It found `docs/overview.md:635` (no change needed) and the `orch-role.md` lines.
2. `grep` in `shedarbiter/doc.go`, `prompt.go` and `shedadapters/bouncer.go`: my context lacked the arbiter package prose and the parent-notice source. Nothing stale found there.

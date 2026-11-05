```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer applied as proposed (F1, F2, F3, F5, F6). Lens list: F1, F2, F3, F5 and F6 were all carried out as proposed, and I challenged each.

**Findings**

1. **MEDIUM** — `contracts/stencils/loom/loom-template-arbiter.md`, the F6 sentence "Every other line of the brief, its materials list and its refusal text included, still binds you."
   - The proposed override list is incomplete, so the blanket "still binds" now contradicts the stencil's own rules.
   - The brief's "When a recorded operator decision answers the question" steps (`bouncer-template-escalation.md`, "What to do") are not in the override list.
   - Step 1 records with `lym loom decision add {{.slug}} --by parent … never record with --by operator`, while the stencil says to record `--by arbiter`.
   - Step 3 says "Resume the run with `lym loom resume`", while the stencil's "What you never do" says "Move or resume the run."
   - Step 2 is `{{.decide_options}}`, and the brief's "Without a decision record … report back to your parent session" ending is also not covered.
   - An arbiter reading "every other line still binds" has to choose between the brief and the stencil on exactly those steps.
   - Fix direction: state that the brief's recording, resume and report-back steps are replaced by the stencil's own procedure, or scope the "still binds" sentence to the brief's constraints (what never counts, the scope-widening ban, the `budget` ban, the materials list, the refusal text).

2. **LOW** — `contracts/stencils/bouncer/bouncer-fragment-decision.md`, "The arbiter settles it where the board entry and the decision record answer it."
   - The fragment is the `cause_detail` of the brief whenever `escalate_decisions` is on, including when the arbiter is switched off (`arbiter_max_spawns: 0`) or not wired.
   - In that case the parent reads the brief directly, and the sentence promises a settler that never runs.
   - The earlier wording (an operator decision answers it) held in every wiring.
   - A conditional, such as "where an arbiter runs", would hold in both.

**Challenged, no finding**
- **F1**: the notice fires before the yield check, so a consumed attempt that then yields still notices. A busy, errored or refused step sends nothing. The arbiter attempt stays open across a restart, and `arbitrate` re-sets `pendingNotice` from it.
- **F2** (stencil rule changes): the hard-cap and circling-at-cap wording agrees with the brief's design-wrong guard and with the "needs its circling decision recorded first" rule.
- **F5** (stencil rule changes): the refusal sentence agrees with the fragment's refusal paragraph, which says an accept at or past the cap halts if the refusal still holds.

Calls made:
1. `sed -n 25,60p contracts/stencils/bouncer/bouncer-template-escalation.md` — to read the brief's steps against the F6 override list; my context held the review's description of them but not their text.

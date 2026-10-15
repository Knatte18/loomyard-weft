# Card 3 — implementer-deviation-rule

**Edit:**
- `contracts/stencils/webster/webster-body-implementer.md`

**Uses:**
- `contracts/stencils/webster/webster-fragment-guard-fix.md`

**Intent:**
State one deviation rule in `webster-body-implementer.md`, per decision D2, so the rendered fork prompt and the rendered recovery prompt, which both compose this body, each carry it once.

In the per-card loop's step 2, after the sentence "Make exactly the changes the card describes, in exactly the targets its type labels name." and before the `{{.locate}}` line, add the rule as continuation lines indented three spaces like the rest of step 2, one sentence per line:
- A deviation is a change outside the batch's card targets as the scope guard counts them: a member target covers that member, a file target every member of that file, a package target every member of the files directly in its directory, and a rename is inside when either its old or its new name is a target.
- `**Uses:**` entries add no coverage, so a change to a member a card only reads is a deviation unless a target of some card in the batch covers it.
- Three deviations are sanctioned, each within its bound: the guard-step fix in step 3, only in a file the guard step's failure names; the scan-error fix the guard-fix line in the card gate names, only in that file and under the card it names; and a fix to a failure the card caused, under the bounded self-fix section's two-pronged test and its attempt cap.
- No other change outside the targets is made.
- The scope guard records every deviation, sanctioned ones included; you list, compare and report none, and a deviation alone never makes the report `FAILED`.

The rule names `ScopeGuard` by what it does, the scope guard, not by its Go identifier, since the fork never reads its doc comment.
In the guard-step paragraph under step 3, the closing clause "; every other deviation rule stands." becomes a pointer to step 2's rule, such as "Change only a file the guard step's failure names this way, the first sanctioned deviation in step 2's rule."
The prior-batch paragraph's sentence about `scope-guard deviations` stays as it is.
No other sentence of the body is changed, and no test pins this prose, per `PATTERN-test-economy`; the stencil package's existing tests and webster's rendering tests keep passing.

**ImpactSummary:** One stencil body read at call time by both webster's fork and recovery prompts; no marker is added or removed, so the rendering tests' required-marker sets are unchanged.

**Commit:** 3: Webster implementer body states one deviation rule

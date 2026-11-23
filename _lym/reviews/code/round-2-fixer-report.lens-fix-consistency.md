```yaml
lens: fix-consistency
outcome: reported
```

Fork report — lens fix-consistency.

**Findings**

1. **MEDIUM** — `contracts/stencils/bouncer/bouncer-fragment-decision.md`, the "### Refusal" paragraph. This is a twin the fix left as it was.
   - Summary: it still says "An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted". The F13 change in `settleDecisionAccept` (`internal/shedadapters/bouncerdecision.go`) makes that false.
   - Evidence: for a CONTINUE or CIRCLING verdict, an accept now holds over an open non-decision BLOCKING ledger entry. Below the soft-cap window it runs another round, and inside the window it escalates with cause budget. The stencil is the arbiter's and parent's brief for exactly this escalation. It tells them an accept settles over listed BLOCKING findings, while the code runs another round or escalates.
   - The fragment's `{{.blocking_findings}}` list is every BLOCKING finding of the review, not only ledger entries. The sentence should say an accept over a CONTINUE or CIRCLING verdict still holds on an open BLOCKING ledger entry of another class.

2. **LOW** — `internal/loomengine/template.yaml:53`, the `code_auto_accept` comment.
   - It says a round with an open BLOCKING finding "escalates to the run's parent". It does not tell a hub that has not set the key that the flagged meaning change applies to the default.
   - The wording is not false and the F2 fix did not touch it, so this is optional.

**Checked, no finding**
- `_lym/landing/summary.md`: the Door, Blast radius and design-decision lines agree with the new `FlagCodeAutoAccept` behaviour and with `RunsWithoutFile: true`.
- The doc comments on `routeDecisions` and `settleDecisionAccept`, and the test file header, match the new code.
- `FlagCodeAutoAccept` still sits on the one `loom` `Reinterpretation` entry in `internal/configreg/configreg.go`.

**Tool calls**
1. `grep` over md, yaml and go for `FlagCodeAutoAccept` and `code_auto_accept`: my context did not hold the sibling files that mention the setting.
2. `grep` for `escalate_decisions` and decision-escalation text, plus `cat` of the stencil: to find the brief text that describes accept semantics.
3. `grep -rln` for the stencil sentence, and `grep` of `refusalLeavingOut`: to find the stencil's source path in the worktree and confirm which verdict path the dropped-key and BLOCKING checks cover.

```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for what should move out of the decision record or be cut, and checked each candidate against completeness first.

- **MEDIUM — D8.** Most of D8 repeats other sections.
  Its first two bullets give the no-change reasoning for the llm driver and the Webster Master. Scope's "Out" list already excludes both, and the support log's Turn 1 and Rejected alternatives hold the reasoning.
  The third bullet, that `lym shuttle await` takes D4's cap, repeats D4's second paragraph word for word.
  Only the 300000 ms → 3600000 ms text change is a requirement the plan writer needs.
  Fix: keep that bullet, cut the rest or replace it with a single sentence such as "No other agent wait changes", and keep the D8 acceptance line for the stencil.
- **MEDIUM — Notes for the plan writer, bullets 2 and 3.** These two bullets are plan-level detail that the plan writer finds on its own.
  Bullet 2 says where the slot-busy answer is built today (`internal/gatecli/run.go`). Bullet 3 lists which `doc.go` sections change. The rubric's "Do not flag" places that kind of enumeration with Plan-Write.
  The completeness test passes: D1 already says the front names the holders, and D1 to D3 already imply the doc changes.
  Fix: cut both bullets. Keep bullet 1: the fake answering a zero lifetime decides how tests behave under D2.
- **LOW — Constraints.** PATTERN-spawn-observability has two bullets: one shared with run-process-owner and one about tests not re-running `os.Executable()`. Merge them.
  Several bullets also restate their decisions: the runner's registration is in D3, the `CategoryNone` keys are in D4 and D5, and the hidden verb is in D1.
  Fix: cut each to the PATTERN id plus "met by Dn", or keep only the clauses no decision states. Those are the `lymdirs` naming, the once-a-second wake, the optional record field and the test seam.
- **LOW — Scope "In".** Each bullet repeats a decision's title (D1, D6, D4, D5, D7, D9).
  Fix: shorten each to a clause pointing at its decision, so a change to a decision cannot leave a stale copy in Scope.
- **LOW — Auto-mode assumptions.** All six bullets repeat picks that the decisions and the support log's Question ledger already record.
  The section is required, so cut the bullets to "Dn: &lt;pick in a few words&gt;" and keep the reasoning out.
- **LOW — Goal.** Its third and fourth sentences cover the break-even and "other waits", which are what Scope and D5/D8 say.
  Fix: the Goal can stop after the second sentence.
- **Noted beyond leanness (MEDIUM) — Open risks, second bullet.** The 59m10s cap sitting close to the one-hour ceiling is a choice the writer can make, not a question for the parent.
  Settle it in D4/D5 instead, for example with a larger margin for 1-hour roles and the long cap, and drop the risk line.

No tool calls made: my context already held both files, the rubric and the spec.

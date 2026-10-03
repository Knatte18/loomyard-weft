---
round: 1
focus:
  - "D5 (Reflect re-entrancy), against Goal, D9 and the archive rule: settle what counts as a 'finished reflection' whose report licenses archiving its covered notes without spawning, i.e. whether a report left by a reflection that timed out, failed GitHub auth or was killed mid-filing can archive notes that were never filed; and whether D5 states where the covered-notes record lives and what bounds the new archive-without-spawn edge."
  - "D2 versus D5 on concurrency: D2 keeps 'a held reflection lock skips the reflection rather than waiting', D5 says Reflect first probes for a live agent and waits on it. Settle which rule applies on the halt path and the terminal path, and whether either Decision states what happens to a lock or covered-set record left behind by a killed step process."
  - "Attack surface of D1, D3 and D6 together: D1 drops the run-only gate and D6 files every distinct lyx problem to a public repo without approval, and D3 notes carry verb arguments and refusal text. Settle whether the Decisions state what note content can reach a public issue body and what bounds it, beyond the timeout and the hardcoded repo; D3's bound ('files nothing by itself') is the one to check against D6."
  - "D7 and D8 are no-change Decisions that restate Scope Out and Rejected alternatives. Apply the completeness-before-leanness test: decide whether each carries a constraint the plan writer needs (e.g. 'batten recipe and teardown unchanged') that belongs in Constraints, and whether the remaining deliberation (#274 history, why no second filing path) belongs in support-log.md instead."
  - "D4's notification edge: it fires only on a done stop with `friction: failed`. Settle whether the Decision covers the other non-clean `friction` values the envelope can carry (e.g. a skipped reflection on a held lock under D2/D5) or states why they need no notification, since a skipped terminal reflection otherwise leaves notes unfiled with no alert."
---

The highest-risk content is the interaction between D2's lock behaviour and D5's new re-entrancy edges, because both decide whether a note is archived without being filed, which is exactly what the Goal and D9 forbid.
The public-filing attack surface follows from removing D1's gate and D6's file-everything stance; D3 introduces new note content that flows into it.
D7/D8 are a leanness check, run through completeness-before-leanness so nothing the plan writer needs is moved out of the decision record.
Not in focus, per the rubric and the writer's stencil: the optional Notes section, rejected alternatives absent from the decision record, and call-site enumeration.

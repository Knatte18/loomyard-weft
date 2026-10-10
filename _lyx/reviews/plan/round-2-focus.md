---
round: 2
focus:
  - "Cards 7 and 8 (pull-first and seed drop): when both the stencils and specs subtrees have a write due in one pre-run, does each subtree's seed commit survive the pass, given the drop removes every seed commit ahead of upstream and resets to upstream? Settle whether a scenario step covers both subtrees due together."
  - "Card 7 tests: where do the new Bolt pull-first and push-retry tests build their hub geometry, and does it go through `hubforge` as `PATTERN-hubforge-fixtures` requires?"
  - "Card 7 drop gate: what binds the new ownership kind beyond the target's base name, and does the Intent state that bound? Also settle what the drop does on a dirty tracked board file and which skip detail the caller sees."
  - "Card 7 push path: which `Mutations` recorder does `Bolt.Push`'s drop append to, and how does a caller see the dropped commits under `PATTERN-mutation-record`?"
  - "Card 12 `IsNoServer`: does it treat a tmux socket that does not exist (`error connecting to ... (No such file or directory)`) as no server, and does a test row cover that form?"
---

Entries restate their sites so the reviewer can check them against the current plan without outside context.

---
round: 2
focus:
  - "internal/gitrepo/pull.go `Repo.Fetch`: with a records store whose fetch refspecs are `+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*`, does any fetch call it makes delete a local-only `refs/tags/*` tag, and does a branch deleted on the remote still lose its tracking ref?"
  - "internal/gitrepo/pull.go and doc.go: do the `Fetch` doc comment and the package docs state exactly what the code prunes, and does `Fetch` learn no records layout, filtering only on the destination prefix?"
  - "internal/battencli/arm.go `hubRunVerdict` fetch closure: with no pair present, are both the prime's records store and the prime's code store refreshed before the code-tip judgement is repeated, and does the refusal text stay true after a successful fetch?"
  - "internal/battencli/arm_runverdict_test.go `TestJudgeAfterCodeTipFetch` and internal/runfsm/act_test.go `TestAct`: do the rows reach a read error after a successful fetch and the pair-absent refusal with no code tip and no machine, each asserting an independent literal?"
  - "_lym/landing/summary.md `## Merge danger`: does the stated door and blast radius match what the diff deletes on every fetch, by caller and store?"
---

The round-1 BLOCKING finding concerned tag pruning in a one-repository records store; recheck the whole range, not only the fix.

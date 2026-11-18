---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: D3 now says a reach turn's failing result goes to the writer alone, names the wrap's new check on the entry's own and forked closures, bounds it to a result whose findings are the reach note, and keeps every other verify failure going to the fixer first. The re-run mark gives one counted failure per shuttle arrival. The third D3 criterion carries the Darn-Write case with `darn.gate_fixer` set.
  - id: F2
    status: resolved
    answer: The Goal's build order, Scope, D10's opening sentence, D11's printout, board note and rationale, D12's report names and config acceptance, the narrowed `PATTERN-refusal-way-forward` constraint, the D10 criteria marked as checked at the later build, and the reworded Review criterion all agree. The Open risks bullet on the trial's order is gone, and the support log's Round 2 entry records the ruling.
---

## Review

Both focus questions settle in the record's favor; no new finding.

**Focus 1, the reach turn at Darn-Write.**
I read `internal/gatefixer/rowgate.go` against D3.
The premises hold:
`spawning` runs the inner closure, hands a failing non-terminal result to `spawnAndJudgeAgain`, and `waitThenRun` then runs the inner closure a second time over the fixer's tree.
`SkipForked` is consulted only when `forked` is true, so it cannot carry the bypass for Darn-Write, as D3 says.
D3 states the bypass, its bound (only a result whose findings are the reach note; every other verify failure still goes to the fixer first), and the attempt count (a re-run mark, so one counted failure per failing arrival).
The added entry "Reach turn bypasses Darn-Write's cold fixer (round 2 F1)" places the `internal/gatefixer` change inside the task's scope and settles the channel choice.
Not a defect: D3 does not name how the wrap recognizes the reach note; that is a plan-level choice, and the bound on what it lets through is stated.

**Focus 2, the mutation check order.**
The Goal, Scope, D10, D11, D12, the `PATTERN-refusal-way-forward` constraint, the D10 and D12 criteria and the Review criterion all say D10 is built only after the trial pays.
`## Open risks` holds no line asking the parent for that ruling.
The support log's Question ledger entry Q10 still describes the earlier "ships unlisted" pick, but it ends with a pointer to the Round 2 ruling, and `support-log.md` is not read by the plan writer.

```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked the pairs of decisions that share state, files, keys or ordering in the decision record and support log.

**Pairs checked**
- D1 × D2 (dispositions feed the prior-findings block): held.
- D1 × D3 (`withdrawn` only on a disputed finding, `answer` on an open disputed one): held.
- D1 × D6 (an all-disputed fixer round counts as unchanged): held. Two such rounds in a row escalate, which is the intent.
- D2 × D4 (the prior-findings block appears only when the round is incremental): **broken**, finding 2.
- D3 × D4 (`carries` exempts a finding from the new-finding bound; the one-to-one match stops laundering): held.
- D3 × D4 × the existing decision path (an unruled `decision`-class finding): **broken**, finding 3.
- D4 × D5 (the changes file is built from round N-1's snapshot): held. The indexing is inconsistent, finding 5.
- D5 × D6 (the target id after the fix vs the round-start snapshot; restart and relaunch paths): held. A review relaunch rewrites the snapshot before any fix, and a commit seam changes no content.
- D6 × the existing Bouncer order (converged, decision, hard-cap and budget against the stalemate check): **broken**, finding 1.
- D6 × `circling continue` resetting the count: held.
- D4 × amendment rounds × D6: held. An amendment changes the target, so the round counts as changed.
- Acceptance criteria against the decisions:
  - D6's "converged wins" criterion is broken (finding 1).
  - The other D1–D6 criteria hold against their decisions.
  - The D4 criterion "a round whose prior snapshot is missing is full" is ambiguous between the record and the cache (finding 2).

**Findings**

1. **BLOCKING — D6, ordering, and the matching acceptance criterion.** D6 says the converged path keeps its order and wins over a stalemate, but convergence is not a Go check that runs before the judge. `CONVERGED` is the LLM judge's verdict, and Go only guards it afterwards (`judgmentRefusal` and `refusalLeavingOut` in `internal/shedadapters/bouncer.go` around lines 1060–1095). Because D6 puts the stalemate check before the judge call, a convergent round gets escalated as `stalemate` instead of converging.
   - Counterexample: rounds N-1 and N hold only MEDIUM/LOW findings that the fixer disputes, and nothing changes.
   - At round N the Bouncer escalates `stalemate` where the judge would have said CONVERGED.
   - The criterion "the converged, decision and hard-cap paths win over a stalemate on the same round" cannot be met as designed.
   - Fix: gate the stalemate on a Go-readable fact, such as round N's review holding an open BLOCKING finding. Alternatively, run the check only after the judge returns CONTINUE, which costs a judge call.

2. **MEDIUM — D2/D3 × D4/D5, the rebuttal channel depends on the diff cache.** D4 makes a round full whenever round N-1's snapshot is "missing or unreadable", and D5 says a missing cache (the tree object or the `.lym` copy) makes the next round full. A full round renders today's prose text (D2) and forbids `prior` and `carries` (D3). So a lost cache, for example after a gc or an adopt on another machine, drops every rebuttal from its finding, which is the core of D1/D2.
   - The record also blurs "snapshot record missing" with "cache missing".
   - Fix:
     - Make the prior-findings block and `prior` statuses depend only on a complete round N-1 (review and fixer-report).
     - Let only D4's new-finding file bound depend on the changes file. Without it, either refuse every new non-parent finding or fall back to full scope for new findings alone.
     - Say which absence does what.

3. **MEDIUM — D3/D4 × the decision escalation and carry-over, an unruled `decision`-class finding.** D1 leaves out a `decision` finding that no ruling reached, so D2 doesn't list it and D3's `prior` and `carries` can't carry it. D4 then lets the next reviewer re-raise it only at BLOCKING severity on a changed file.
   - Case: round N holds an unruled `decision` finding, the Bouncer escalates, and the parent says `continue` without a ruling reaching the fixer. Round N+1 can neither carry nor re-raise it.
   - The finding drops out of the review files, and the Bouncer's decision path and carry-over read only the latest review.
   - Fix: list unruled `decision` findings in the D2 block with status options of their own (for example `open` carried with `carries`), or exempt `class: decision` carries from D4's bound.

4. **LOW — D6 count reset.** "The latest round whose escalation the parent settled with `continue`" does not say whether a `continue` on another cause (`circling`, `budget`, `hard-cap`) also restarts the stalemate count. Read literally it does, so after a `budget` continue one more unchanged round cannot escalate even if the two before it were unchanged. That is probably the intent, but the record should state it.

5. **LOW — D2/D3 against D4/D6, round indexing.** D2 and D3 index the prior round as N with the current round as N+1. D4 indexes the prior round as N-1 with the current round as N. D6 uses N as the round just fixed. The plan writer could mis-map which snapshot is compared to which. Pick one convention.

**Tool calls**
- `grep` in `internal/shedadapters/bouncer.go` for `APPROVED`/`Converged`, and in `internal` for `Review rounds`: to learn whether convergence is decided in Go before the judge. My context only had the subagent summary that the judge writes the verdict.
- `sed -n 1060,1100p internal/shedadapters/bouncer.go`, plus `grep` for support-log round appenders: to read the CONVERGED guard and confirm it runs after the judge's verdict. My context lacked that code.

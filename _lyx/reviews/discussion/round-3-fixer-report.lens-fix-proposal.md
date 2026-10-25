```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1 accepted bound, F2 halt at the hard cap) for concrete cases where they're wrong.

no findings

- F2: the halt's way forward works. Resuming a budget halt gives one more budget (`internal/shedengine/doc.go:50`, `internal/shedengine/run.go:192`), so the next Call re-judges the round under the recorded accept, which D8 now says. `judgmentRefusal` (`internal/shedadapters/bouncer.go:1023`) refuses CONVERGED only over a BLOCKING item, so a re-judge that rules CONTINUE or CIRCLING lets the accept pass. A dropped-key refusal over an accept at the hard cap also halts and resumes the same way. I found no loop and no unreachable state.
- F1: the accepted bound holds in every case I built. The ring calls the Bouncer first and routes Stuck to Review and then Fix, so round n's fixer has round n's review before the judge sees it. That covers a finding the fixer disputed too. The new criterion "with an empty focus file" holds for the retire-then-`continue` path, because the refusal it describes is the CONVERGED one. A dropped-key refusal over a `continue` halts and starts no round, as D8 already says.

Calls:
- grep in `internal/loomcli` for resume and budget: I needed to know whether `lyx loom resume` gives a halted segment more budget, and the context didn't show it.
- grep in `internal/shedengine/doc.go` for resume with bounce or segment: same reason, and it found nothing.
- grep in `internal/shedengine/*.go` for resume with budget: same reason, and it found the "one more budget" rule.

```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covered every member of each set-premise finding.

Unchecked findings: none. F1, F4, F5, F10, F11 and F12 were searched; F2, F3 and F6 were checked from context.

1. **MEDIUM**: F12, `internal/gatecli/doc.go:353`, a twin left unchanged.
   - The package doc still says "the line says instead that no test of the tier matched it."
   - `filteredNoTestNotice` now says no test of the project matched, so the doc states the behaviour F12 removed.

2. **MEDIUM**: F11, `observeSummary`'s new doc comment in `internal/gatecli/counters.go` ("ends at its `Total time:` line, at a one-line summary or at an invocation boundary").
   - The counter learns of an invocation boundary only through `startInvocation` and `replaceInvocation`.
   - `runInvocations` calls both only when `run.tier.Results == gateslot.ResultsTRX`.
   - On a `vstest` tier without `results: trx`, a summary that never printed `Total time:` therefore stays open into the next invocation, where the old code closed it at the first foreign line. Any later line matching `^\s*(Passed|Failed|Skipped):\s*\d+\s*$` is then counted.
   - `TestVSTestCounterInvocationClosesAnOpenSummary` covers only those two calls, not a boundary on a non-trx tier.

F1, F4, F5 and F10 have no missed member. The remaining `"tiers."` literals in `gateslot/runlog.go`, `config.go` and `tiers.go:187-205` are not lines this diff added.

Tool calls, each a search my context lacked:
1. Grepped the `"tiers.` literals and the diff's added lines for F1; the context held only the earlier, partial grep.
2. Grepped `gatecli` for other timer and timeout bounds for F5; the context had not searched for them.
3. Grepped refusal and notice texts across code and docs for F6, F10, F12 and F13; the context had not searched all twins.
4. Grepped the stencils for another no-gate step for F4; the context had searched only one phrase.
5. Grepped the summary and notice docs and read `doc.go:348-358` for F11 and F12; the context had not shown that part of `doc.go`.

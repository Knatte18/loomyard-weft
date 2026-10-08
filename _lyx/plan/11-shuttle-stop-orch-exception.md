# Card 11 — PATTERN-shuttle-stop names the orch exception

**Prosa:**
- `PATTERN.md`

**Intent:**
Make `PATTERN-shuttle-stop` true on landing by naming its second exception, per the decision added after Discussion, "Orch strand removal is a second PATTERN-shuttle-stop exception".
`orchcli` starts the orch session through `runner.Start` (`internal/orchcli/start.go`), so the orch strand has a shuttle run record, and `orchcli` removes that strand directly through its strand ops' `RemoveStrand` in `stop.go` and `start.go`.
The entry today names only `loomcli`'s removal of the loom driver's own strand, so it is false for orch.

In the `PATTERN-shuttle-stop` line of `PATTERN.md`, replace the clause after the semicolon with exactly:
"no other package removes a shuttle run's strand, except `loomcli`'s removal of the loom driver's own strand and `orchcli`'s removal of the orch strand, since no attach probe or Wait reads either run's record."
Change nothing else in the file; `orchcli`'s removals stay as they are, since moving them onto the stop verb is out of scope.

Bound (this card widens the invariant's exception list): the entry now lets `orchcli` remove the orch strand without settling its run record.
The exception covers only the orch strand, whose run record no attach probe or Wait reads, so an unsettled `running` record there misleads no reader; every other shuttle run's strand still stops only through the stop verb.

**Commit:** 11: PATTERN: name orchcli's orch-strand removal as a PATTERN-shuttle-stop exception

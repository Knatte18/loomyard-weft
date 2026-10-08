# PR-Gate review: shuttle-send (PR #474)

## F1: `PATTERN-shuttle-stop` is false on landing (blocking)

`PATTERN.md:66` says "no other package removes a shuttle run's strand, except `loomcli`'s removal of the loom driver's own strand".
`orchcli` starts the orch session through `runner.Start` (`internal/orchcli/start.go:55`), so the orch strand has a shuttle run record,
and `orchcli` removes that strand directly through `reedStrandOps.RemoveStrand` at `internal/orchcli/stop.go:41` and `internal/orchcli/start.go:225`.
The decision record's parent entry "Orch strand removal is a second PATTERN-shuttle-stop exception" requires the entry to name both exceptions;
the plan restore undid card 9's wording, so the change never reached the file.

Fix: in `PATTERN.md:66`, replace the clause after the semicolon with:
"no other package removes a shuttle run's strand, except `loomcli`'s removal of the loom driver's own strand and `orchcli`'s removal of the orch strand, since no attach probe or Wait reads either run's record."
Change nothing else; verify with the PATTERN scans' own tests only, not the full suite.

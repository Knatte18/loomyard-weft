```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: check the change set against its sibling text and cross-references.

1. **LOW**: `contracts/stencils/darn/darn-template-write.md:109-112` (the "A bug entry" section) is stale against the new unjudged exit.
   - It says the gate fails when every test passes, and when the diff adds or changes no test. It offers the `No test seam` ask only for "when the bug has no seam a test can reach".
   - The fix added a third failure, `RedFirstUnjudgedExit` (5), for a hub whose scoper is not go or a changed non-Go file of a package. The gate's findings tell the writer to send the same creel ask, so the writer is told what to do at the time.
   - A writer reading its prompt first would not expect that case. The twin of `noSeamAsk` and `noSeamWayForward`, the writer prompt, was left as it was.

2. **LOW**: `internal/shedrecipe/doc.go:204-211` ("The red-first gate") is incomplete against the new exit-5 behaviour.
   - It says the gate holds the handoff "until a test the diff adds or changes fails on the merge base". It does not mention the exit-5 hold, which lifts only on the parent's `No test seam` record.
   - The sibling docs the fixer did update (`loomshed/doc.go`, `gateslot/doc.go`, `gatecli/doc.go`, `docs/overview.md`) do mention it. This one is minor.

3. **No finding on `gatecli/doc.go:751`.** "A hub whose scoper is not `go` reads nothing: the verb prints a note naming the scoper, with exit 0" belongs to the reach/mutate sections, not the red-first verb. It is correct there.

4. **No finding on the rest of the changed set.**
   - The three constants (`gateslot.NoTestSeamTitle`, `gateslot.BugLabel`, `gateslot.RedFirstUnjudgedExit`), `gatecli`'s Long help, and the `loomshed`, `gateslot` and `gatecli` docs agree with each other.
   - The summary and `docs/overview.md` use the same exit-code list and the same bounds.
   - My grep found no leftover mention of the old "prints a note, exit 0" red-first behaviour or the retired mixed-report bound.

Tool calls (3):
- A grep across `cmd`, `contracts`, `docs/overview.md`, `pattern` and the shedrecipe files for stale red-first wording. My context only held the files the fixer edited.
- A grep of the stencil, `cli.go`, `shedrecipe/doc.go` and `reach.go` for red-first and "No test seam", for the same reason.
- A read of `darn-template-write.md` lines 104-116 and `shedrecipe/doc.go` lines 204-216, to confirm the two stale-text findings. Neither file was in my context.

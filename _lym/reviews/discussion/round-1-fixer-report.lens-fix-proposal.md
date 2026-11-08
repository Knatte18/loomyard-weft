```yaml
lens: fix-proposal
outcome: reported
```

Task: check, through the fix-proposal lens, each review proposal the fixer carried out as proposed, looking for concrete cases where the proposal is wrong.

Proposals challenged: F1 (reinterpretation route), F3, F5, F6, F7, F8, F9 (skip on a matching report), F10, F11, F12, F13, F14, F16, F17. No case found against F3, F6, F7, F8, F11, F12, F13, F16 or F17.

1. **MEDIUM** (D6, F10): "A dobby stop does not end the child: it ends on its own within D5's time bound…"
   The record says a dobby stop leaves the refresh child running, but in the code a stop ends it.
   Evidence: `internal/beamcli/doc.go:319` says a stop ends the dobby through `runproc.EndRole`. `internal/runproc/end.go:355-385` walks every descendant of a recorded process and ends it, including a descendant that leads its own group. D6 makes the refresh a child the dobby starts and waits on, so it is one of those descendants. The "dobby stop does not end the child" claim and the "one a stopped dobby left behind" bound are both false. The orphan case the record is now written around applies only to a dobby that dies unexpectedly.

2. **MEDIUM** (D7, F9): "It is skipped when the worktree's existing report matches the current files by D8's match…"
   A report that judged nothing for a temporary reason is never redone on the same tree.
   Concrete case: in the first call, a package's coverage build or the `lym` build fails, or the budget runs out, or the package has no map yet because the dobby has not built it. D7 writes each of these as a report line, so the report exists and was not written on a red tier. A later gate key on the same tree, such as `webster` after `code-burler-fix`, matches it by D8's match and skips. The transient failure, the tests the budget cut off and the "no map" verdict then stand until the tree changes, and D8 shows that stale report to Code-Burler-Review. The match should exclude a report that holds a step error, a budget cut or a no-map package, or the skip should be bounded in the record.

3. **MEDIUM** (D7, F1): the reinterpretation that appends `redundancy` to a hub base file's `webster`, `parent-align`, `darn-write` and `code-burler-fix`.
   It appends the step on every hub, including hubs where it cannot work.
   Evidence: `internal/gateslot/doc.go` allows `scoper: dotnet` (and `none`). D7's step only works on Go: `_test.go` files, Go coverage binaries, `spawnscan`. A C# hub that tunes those keys would gain `reports: [redundancy]` on each, and the step would run Go coverage against it on every gate call. D7 says nothing about a hub whose scoper is not `go`. The reinterpretation, or the step itself, needs to be limited to `scoper: go`.

4. **LOW** (D5, F5): "When either differs, as when a Finalize landing moves the prime's checkout mid-package, it discards that package's map".
   Any HEAD move discards the package's map, even when the landing never touched that package.
   Concrete case: on a hub where landings into the prime arrive more often than one package's tests take to run, a slow package is discarded on every refresh and never gets a map. Comparing only the package's own file hashes after its tests, plus HEAD only for the commit the map records, would avoid this. As written, D6's bound ("waits for the next prime HEAD change") does not cover a package that is starved this way.

5. **LOW** (D7 step 6, F14): "the files that differ from the map's hashes and are unchanged since the merge base".
   Drift is hidden in a file that both the run and the parent changed.
   Concrete case: the parent changes `x.go` after the map was built, and the run also edits `x.go`. The file is excluded from the line, so its drift on the parent is never shown. The line is not "drift on the parent alone" but "drift in files the run left alone", and the bound should say so.

Tool calls:
- grep of `runproc` and `dobbyengine` for PGID and group signalling: the context lacked how a dobby stop treats its children.
- `internal/runproc/end.go:355-385` and the `HubOwner` use in `beamcli`: the context lacked the descendant walk and the prime dobby's owner.
- grep of `runproc` End and EndRole and their callers in `beamcli`: the context lacked which end call a dobby stop uses.

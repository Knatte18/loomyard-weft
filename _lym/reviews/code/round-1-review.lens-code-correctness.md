```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. The diff is prose only (21 lines in `contracts/orchrole/orch-role.md`), so I ran each factual claim against the code I had already read.

**List (claim, case run, verdict):**
- `lym batten cancel <slug> --remove`: the flag is registered at `internal/battencli/cancelcmd.go:130` and the help text says "run from the prime or a beam". Correct.
- Create runs from the prime or the parent beam, never from a task worktree: `internal/beamcli/cli.go` create help says "runs from the beam's parent worktree, the prime or the parent beam of a nested beam, and refuses elsewhere". Correct.
- `--anchor <dir>` gives the beam its own anchor, otherwise it inherits its parent's: the flag at `cli.go:524` matches. The role's "a directory of the code checkout" drops two conditions the flag states: the path is relative to the code repo root, and it is tracked in the commit the beam forks from. Correct but incomplete (LOW).
- "dobby starts no project entry, so a release never makes a beam": `cli.go` says "Dobby never starts a beam project, so only this verb, a hand pass or the green end of the adopt check starts or resumes its run", and `internal/beamcli/doc.go` repeats it. Correct. The role's "Only `lym beam create` starts a beam's run" is slightly stronger than the code, which also names a hand pass and the adopt check (NIT).
- Worktree-Create makes the worktree unmarked, Seed-Child seeds the pawl run, and Beam-Up marks the worktree and starts dobby and orch: `internal/battenshed/doc.go:55`, `internal/beamcli/doc.go:28` and `internal/loomcli/peers.go:246` agree. Correct.
- The beam's orch appears only after Beam-Up: the pawl recipe row order is Preflight, Record-Write, Beam-Up. Correct.
- A task run's inner recipe is tapestry, arras, darn or sampler: the four recipe files exist, and the role already lists them in the run loop. Correct.
- `lym beam start <slug>` refuses a task worktree and never makes a beam: `internal/beamcli/target.go:87` returns "is a task pair, not a beam". Correct, and the role's existing `beam start` line agrees.
- `lym batten status <slug>`, filtered to its current producer, tells the two apart (task inner producer is a tapestry, arras, darn or sampler row; beam's is Beam-Up or a later pawl row): **incorrect**, see the finding below.
- Never `lym fabric add`, and never delete a worktree directory by hand: this matches the entry. I found no settings deny for `fabric add`, so the file's header claim "names no denied recovery command" is not contradicted.

**Finding**
- **MEDIUM, design:** The status-based way to tell a task run from a beam does not work as written (`contracts/orchrole/orch-role.md`, the "`lym batten status <slug>`, filtered to its current producer…" sentence in the new section).
  - Both a task run and a beam's create run are batten runs: Worktree-Create, Seed-Child, …, Run-Shed. The tapestry/arras/darn/sampler row or the pawl Beam-Up row is the inner run's producer, not batten's.
  - `lym batten status` reports batten's own `current_producer`, and the activity is composed from batten's own state. See `internal/battencli/arm.go:945` and `internal/shedverbs/status.go:190`. Once the run is past Seed-Child, both kinds of run show Run-Shed.
  - The inner run's producer appears only in `haltedOverDoneChild`, which reads the child status and flags only a done child, so it is no producer read.
  - An orch following this sentence would filter to `current_producer`, see Run-Shed for both, and be unable to tell them apart.
  - The entry only says `lym batten status <slug>` "shows the rows". The row-name contrast (Beam-Up versus a tapestry row) is the writer's addition.
  - A fix would drop the producer contrast and tell the orch to check the entry's `kind` and recipe (`lym board get`), or name a command that does read the inner run's status. I did not verify which command that is.
  - Evidence: `internal/battencli/arm.go:939-960`, `internal/shedverbs/status.go:190`, `internal/battenshed/doc.go:55`.

Tool calls: none; the manifest already held every definition I needed.

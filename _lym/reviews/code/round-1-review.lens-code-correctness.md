```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I traced each branch that the changed Step 4 text and test add, using concrete inputs.

**Branches and the case I ran for each**

1. **No parent, scope still open, no `Added after Discussion` entry (both fragments): correct.**
   - The writer writes the stop report and the stop marker. `darnWrite.Call` then halts Stuck, with the report plus `darnStopWayForward` as the reason (`internal/loomshed/darnwrite.go:189-210`).
   - The operator runs `lym loom decision add` from their own shell. That passes, because an empty strand name passes the caller check (`internal/loomcli/decision.go:772`), the scope is open since no Code-Review segment ran (`scopeClosedAt`, `internal/loomcli/decision.go:399-409`), and `checkDarnRecord` accepts a committed record.
   - `lym loom resume` on that halted go-seeded run spawns a Go driver (`internal/loomcli/resume.go:141-156`). The driver calls Darn-Write again, which removes both markers (`darnwrite.go:166-170`) and spawns the writer again.
   - The writer re-reads the record, finds the entry and runs the allowed calls under its limits. The loop closes.

2. **No parent, Code-Review has run, no entry: incorrect (F1).**
   - `decision add` refuses here (`decision.go:1024-1039`), and its way forward is a board note. The stencil's "on a task a Code-Review round has run on" matches that refusal's trigger, `reviewRan` (`internal/loomrecipe/scopeline.go:42`).
   - After the board note and `lym loom resume`, the writer is spawned again. Step 4 still reads "With no such entry, run none of them and stop", and nothing in it lets a `Decision for &lt;slug&gt;:` note settle the stop.
   - So the resumed writer stops again on the same rule, and the run loops.

3. **Gated fragment, a subagent and a measurement call: correct.**
   - Item 2 tells every subagent to run no `lym gate test`, and the new item-3 sentence keeps the measurement calls in this session.
   - I found no remaining path by which a subagent makes such a call. This now matches the ungated fragment's item 2 line and its item 3 line.

4. **`TestDarnSpec` ungated row: correct.** Both new constants are verbatim substrings of `darn-fragment-make-change.md:30` and `:38`, and that fragment renders into `make_change`, so the row's assertions hold.

5. **`TestDarnSpec` gated row with `GateFixed = true`: correct.** Both constants are substrings of `darn-fragment-make-change-gated.md:46` and `:54`, so the row holds.

**Findings**

- **F1**
  - severity: MEDIUM
  - class: design
  - location: `contracts/stencils/darn/darn-fragment-make-change.md:40` and `contracts/stencils/darn/darn-fragment-make-change-gated.md:56`
  - summary: The closed-scope way forward (a board note, then `lym loom resume`) re-spawns a writer that has no rule letting the note settle the stop. Step 4 still says "With no such entry, run none of them and stop", so the resumed writer stops again in an endless loop. Step 3's closed-scope exception, which lets a `Decision for &lt;slug&gt;:` note settle the point, covers only an outdated entry's stop. Its no-parent bullet even says a note never settles a stop while the scope is open.
  - evidence: `darn-fragment-make-change.md:32-41`; `darn-fragment-make-change-gated.md:48-57`; `contracts/stencils/darn/darn-template-write.md:53-55` and `:60`; `internal/loomshed/darnwrite.go:166-170`
  - fix: Add a Step 4 sentence saying that, once the scope is closed, a `Decision for &lt;slug&gt;:` note filing the measurements for a new task settles the stop. The writer then makes the rest of the entry, leaves the measurements to that task, and names the note in the change description's design decisions, as Step 3's exception does.

Tool calls: none; everything I needed was already in the inherited context.

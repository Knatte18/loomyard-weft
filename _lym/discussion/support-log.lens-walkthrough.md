```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case, using what each step actually receives.

**Findings**

1. **BLOCKING — D1, D2, D3: a sampler run with the perch off has no change description, so Finalize has nothing to land with.**
   - Case: a task naming profile `speed`. Under D2 its sampler block is a copy of the darn block, `loom: {code_max_bounces: 0, ...}`, so the Code-Review budget is 0.
   - Under D1, Describe keeps darn's `entry_gate: budget.Code-Review`, so Describe is skipped.
   - The darn recipe's comment explains the skip: "a budget of 0 turns the perch off: the Bouncer settles approved at once and Describe's entry gate skips it, so the writer's own description stands." It also says Describe's "entry gate budget.Code-Review skips it when the perch is off."
   - The sampler agent writes no description (D8, D5), so none exists. `internal/landingshed/doc.go` says Finalize starts by "parsing the change description".
   - The default case also misses the stated purpose. With `code_review: false` and the hub's `code_max_bounces` above 0, Describe still runs. So "off" is decided by two switches that disagree, and Describe runs only by accident of the loom budget.
   - Fix: Describe in the sampler recipe takes no entry gate and always runs, because the agent writes no description. Pin a test for the speed-profile case.

2. **MEDIUM — D6: a pause can make the row halt itself.**
   - Case: `lym tender pause` covers the run while the agent idles.
   - The first covered wake stops the session, so the strand is dead from the row's own stop.
   - D6 also says "a wake that finds the recorded strand dead is Stuck with Halt". The record does not order the checks, so a later wake (still covered, or the first uncovered one) can read the strand dead and halt before it resumes the session. A hibernate request pauses every session, so this path is the common one.
   - Fix: covered wakes judge nothing, and an uncovered wake resumes the session before any liveness check. Only a strand the row did not stop counts as dead.

3. **MEDIUM — D6, D5: the entry stamp has no stated way to tell a resume from a re-entry.**
   - Case: a dead strand halts the row, then `lym loom resume` runs. Separately, a `goto` sends the run back to the row.
   - Both call the row's Call again from scratch. The record says a resume "keeps" the stamp and a `goto` "is a new entry", but names nothing the row reads to tell them apart, such as the status history length or the goto's history entry.
   - Without that, either a stale flag ends a re-entered row, or a flag written before the halt is dropped on resume.
   - Fix: state what the stamp derives from.

4. **LOW — D6: the verb can end the segment while the run is halted.**
   - Case: the row has halted, the status still names Sampler-Session but is not running, and the agent or operator runs `sampler-done`.
   - The verb checks only "current row is not that row", so it accepts. The next resume reads the flag with the current stamp and ends Done at once.
   - That may be wanted, but the record does not state it as a bound.

**Per decision: case used, purpose met or not**

- **D1**, the row path with the perch off: **not met** (finding 1). The path with the perch on runs Describe and is met.
- **D2**, a task naming `light` (darn block copied, budget 1): met. A task naming `speed`: not met (finding 1).
- **D3**, a `code_review: true` run_settings value read through the `entry_gate`: met. The off default: met only by accident of the budget (finding 1).
- **D4**, the agent calls `decision add --by operator` while the row is current. The caller rule matches the exported strand name against the recorded strand, then commits under `_lym`, which is the weft symlink, so D6's code-tree check stays clean. **Met.**
- **D5**, first entry, then the operator closes the window, then `lym loom resume`. The recorded session id resumes through the unawaited start, with no deadline. **Met**, apart from finding 3.
- **D6**, the agent commits, then runs `sampler-done`. A file event wakes the row, the tree is clean, the row stops the session and ends Done. **Met.** `.dev-bin/` and `.lym/` are in `.gitignore`, so a trial build does not dirty the tree. The pause path is not met (finding 2).
- **D7**, the child at Sampler-Session for 2 hours with the agent idle. Dobby reads the list and treats the wait as live; the loop's activity reading returns activity now; neither the quiet notice nor the watchdog fires. After Done the session is stopped and the row has left the list, so no quiet notice is sent. **Met.**
- **D8**, the stencil rendered with the slug, the record path, the parent directive in its interactive form and the edit directive. **Met.**
- **D9**, the agent runs `deploy-dev`, then `.dev-bin/lym reed up` with its working directory in the sandbox hub. The server name derives from the sandbox hub's path, so the bindings are re-pinned only on that server. **Met**, bounded by the agent following the stencil.

**Tool calls**

- Grepped `entries_gate.go` and `darn-recipe.yaml` for the `budget.` semantics and Describe's gate: my context had the gate line but not darn's comment on why the skip is safe.
- Read `.gitignore` and `deploy-dev`: needed to know whether `.dev-bin` dirties the tree in D6's check.
- Read the head of `entries_recordwrite.go` and the `tree_scope` source in `loomcli`: to confirm Record-Write is not tied to a recipe and where `verify_source: tree_scope` is read.
- Read darn recipe lines 34–60 and grepped `landingshed/doc.go` for Finalize's handling of the change description: to confirm the skipped-Describe rationale and that Finalize parses a description.

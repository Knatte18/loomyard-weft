```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for what to move or cut.

**Findings**

1. MEDIUM. Auto-mode assumptions, `max_runs`. The bullet "`max_runs` defaults to 3, a cap the orch has kept by hand" restates D5's default and adds a claim nothing backs.
   Evidence: D5 already states "default 3". The exploration found no orch rule or stencil line that caps concurrent runs at 3; the board entry only says the orch adds "a cap on concurrent runs" by hand, with no number.
   Fix: cut the bullet, or keep only "3 is the writer's pick" and drop the claim about the orch.

2. LOW. Auto-mode assumptions and D7 Rationale, "dobby included". The reading of the parent's "batten and dobby included" is stated twice: as D7's rationale and as an auto-mode assumption.
   Evidence: D7 Rationale says "dobby drives batten through the Go driver, so the arbiter reaches every stop of a run dobby drives". The auto-mode bullet repeats it, with the clause that dobby's own failures are not shed stops.
   Fix: keep it in one place, the auto-mode assumption, since that is where a pick goes, and cut D7's rationale down to a pointer to it.

3. LOW. Constraints and Open risks, statusless tasks. The no-migration fact appears as a Constraint ("The hub board holds no open task with an empty status…") and again as the first Open risk, which is the same point turned around.
   Evidence: the support log's Rejected alternatives repeats it a third time ("the board holds none today").
   Fix: keep the Constraint, which carries the evidence, and the risk for other hubs. Drop the restatement from the rejected-alternatives line, or shorten it to a pointer.

4. LOW. Constraints, the PATTERN restatement bullets. Several bullets only map a PATTERN entry to a decision ("`PATTERN-daemon-wakeups`: D13.") or restate the entry's text with no design content. Examples: `PATTERN-shed-driver`, `PATTERN-run-process-owner`, `PATTERN-refusal-way-forward`.
   Evidence: the agent's prompt already inlines PATTERN.md. The record only needs entries the design bends or applies non-obviously, such as `.lyx/beam/` under lyxdirs, `DOBBY` as an agentname constant, and dobbyengine's told geometry.
   Fix: cut the bare mappings. Keep the bullets that name a concrete placement the plan writer needs.

5. LOW. Notes for the plan writer, first bullet. It repeats `relevant.md`, which already lists `innerRunProducer`, `childWait` and the batten template as the tackler-override model.
   Evidence: both relevant.md entries say the same thing. The spec says the list of code the writer read belongs in relevant.md, never in the decision record.
   Fix: cut the bullet. `HubOrchs` in the second bullet is the one new hint; add it to relevant.md under `internal/orchcli/doc.go` instead, which already names it, and drop the section.

6. LOW. Scope In, removing note `batten-window-default`. This is an action listed in Scope with no Decision carrying it. The plan writer gets no instruction on how or when the removal happens, or whether a run's agent may run `lyx board remove`.
   Evidence: D4 retires `--window` but says nothing about the note.
   Fix: add one line to D4, "the run removes note `batten-window-default` with `lyx board remove`", or move it to D4 outright.

7. LOW. Support log Q9. The tag "Advisor-free auto-pick" is a nonstandard phrase; the other entries say "Auto-pick".
   Fix: write "Auto-pick".

No content was found that belongs in the support log rather than the record and that the plan writer does not need. The Rationale lines are within the spec's "Decision + Rationale".

Further tool calls: none.

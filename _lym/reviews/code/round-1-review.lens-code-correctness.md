```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff's changed functions, branches and rule paths.

**Per function or branch, the case I ran, and whether it is correct**

- `TestDarnSpec`, row "first spawn renders the prior-work token as none" (ungated, no parent), with `measurementEntryRule` added. The ungated fragment's item 3 carries the substring (`darn-fragment-make-change.md:23`), so the row passes. None of that row's `wantNotInPrompt` values (`gatedNoGateStep`, `reportPath`, `measureCall`) appear in the new lines. Correct.
- `TestDarnSpec`, row "a cold fixer on the verify gate…" (`GateFixed`), with `measurementEntryRule` added. The gated fragment's item 3 carries it (`darn-fragment-make-change-gated.md:41`). The new lines contain none of `ungatedTestStep`, `ungatedSubagentGate` or `--gate darn-write`. Correct.
- `TestDarnSpec`, row "an ungated writer builds with no measuring call" (`MeasureCall=""`). No new assertion on this row, and the rule needs no Go-filled marker, so it renders unchanged. Correct.
- `DarnTable` doc comment (`spec.go:93`), "Either way the writer asks its parent…": it matches both fragments, since both carry the rule whatever `GateFixed` is. Correct.
- `stencils.go` doc comments for `DarnFragmentMakeChange` and `DarnFragmentMakeChangeGated`: each matches its fragment's rule. Correct.
- Rule path in the gated fragment, measurement entry, with a parent: post, end the turn, then run the allowed calls in the foreground. "falls outside these rules" overrides item 3's "Run no `lym gate test`", and item 2's ban on subagent gate runs still holds. The writer gets one answer. Correct.
- Rule path in either fragment, measurement entry, with no parent: run none of the calls and "name them in the change description". Then Step 5 runs and the writer hands off with the entry's deliverable not made. **Defect.** See F1 and F2.
- Rule path in the ungated fragment, an entry that mixes a code change with measurements: does the writer's per-commit gate run (item 3, `darn-fragment-make-change.md:14`) count as "any gate call for it"? The rule's trigger can be read either way. **Ambiguous.** See F3.

**Findings**

- **F1. MEDIUM, design.** Location: `contracts/stencils/darn/darn-fragment-make-change.md:27` and `contracts/stencils/darn/darn-fragment-make-change-gated.md:45`.
  - Summary: on the no-parent path the writer neither does the entry nor stops. It makes no measurement and writes an ordinary change description.
  - Effect: the writer hands off a run whose deliverable is missing, the handoff verifies an empty or partial tree, and the run lands as if done.
  - Every other no-parent path that blocks the entry's work instead writes a stop report and creates the stop marker. That covers an outdated entry (`darn-template-write.md:60-63`) and an own-entry step (`darn-template-write.md:103`).
- **F2. MEDIUM, design.** Location: same lines as F1.
  - Summary: "name them in the change description" tells the writer to list the planned gate calls (`lym gate test --tier …`) there.
  - Conflict: Step 5 forbids naming lym, a gate or a tier in the change description (`darn-template-write.md:78`). On this path the writer gets two contradicting rules.
  - The ungated list's existing no-parent clause names skipped *tests* (`darn-fragment-make-change.md:17`), not gate calls, so it is no precedent for naming the calls.
- **F3. LOW, design.** Location: `contracts/stencils/darn/darn-fragment-make-change.md:23-24`.
  - Summary: "An entry whose deliverable is itself gate measurements … before any gate call for it" does not say whether an entry that mixes a code change with measurements falls under the rule. If it does, the writer must also ask before item 3's per-commit gate runs. If it does not, the measurement part goes unbounded, which is the case the entry was filed over.
  - Evidence: `darn-fragment-make-change.md:14` (per-commit gate) against lines 23-24.

No tool calls made.

---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:25
    summary: Scope Out and Open risks pin the name tools/forkaudit and its marker-row and arm vocabulary, which the glyph-audit-tool entry does not set and which has not landed
    fix: In the Scope Out bullet at line 25, replace the pinned implementation names with the capability the follow-up must add, and keep the depends_on sentence at line 26. Write "The audit tool of `glyph-audit-tool` learns the per-card layout, telling a per-card fork from a file-layout fork by its prompt and by the `pack_layout=card` trace value, and reading spans from the `lyx loom card-pack` tool results with the card pack trace lines as the cross-check." In the Open risks bullet at line 121 write "Until the audit tool learns the per-card layout, every per-card fork classifies as a file-layout fork, with a trace disagreement."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:55
    summary: D3 Bound says card-pack writes only through the durable sink, but its Warn on a quarry failure also reaches stderr at the default threshold
    fix: In D3's Bound paragraph (line 58), add that the Warn on a quarry failure also prints to stderr, because the logger's stderr half admits Warn at its default threshold, so the fork's tool result carries that line beside the markdown on stdout. Keep the Info line durable-only. Name this in the Bound rather than claiming a sole write path.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:93
    summary: D6 says each card-pack call logs one Info line with resolved and unresolved counts, yet a refused call has no counts and the Bound does not say what a refusal leaves behind
    fix: In D6 (line 93) change "each `card-pack` call" to "each `card-pack` call that renders a pack", and in the acceptance criterion at line 137 say a refusal logs no card pack Info line. In D3's Bound (line 58) add that a refusal exits non-zero, which opens the trace file with its header and leaves no Info line in it.
---

### [LOW] Scope Out and Open risks pin tools/forkaudit names the audit entry does not set

**Location:** `_lyx/discussion/decision-record.md:25` and `:121`
**Issue:** Focus entry 1 is settled: no Decision, Scope In bullet, acceptance criterion or Open risk still needs the audit tool before this run's Plan-Write.
The `forkaudit` card arm is a Scope Out follow-up, and its start is bounded by a board row, `depends_on` naming `glyph-audit-tool` and `kickstart-per-card`, which a new entry can carry because only a run-held entry refuses a `depends_on` change.
The remaining Open risk is conditional ("Until ... lands") and needs nothing to arrive first.
What is left is a premise about a tool that has not landed.
`tools/forkaudit` is absent from the tree, and the `glyph-audit-tool` entry (still in Plan-Burler-Fix) describes a shell and jq tool under `tools/` that classifies a fork's layout from the prompt and the `webster: kickstart pack` trace line.
It names neither `forkaudit`, nor a prompt "marker row", nor arms `file` and `card`.
The record states those as fact, so a later plan for `glyph-audit-tool` that names things differently leaves the record false, and the follow-up's author inherits names that may not exist.
The need for a prompt marker that is checked before the file-layout one is a real constraint and stays, as capability wording.

### [LOW] D3 Bound omits the Warn's stderr output

**Location:** `_lyx/discussion/decision-record.md:58`
**Issue:** Focus entry 2 asks whether the Bound names what the verb can do and what limits it.
Checked against `internal/logger`: the durable sink admits Info and above, writes one trace file per process under `<anchor>/.lyx/logs` with a header line first, opens it also on a non-zero exit, and the exit hook sweeps that directory once, only when the sink armed.
D3's list of the trace file, header, Info line, Warn and exit sweep agrees with that.
The word "only" does not hold for the Warn, since the dual handler also sends Warn to stderr at the default Warn threshold.
That line then sits in the tool result next to the pack the fork reads, a visible effect the plan writer should know when it tests the quarry-failure criterion at line 135.
The Info line stays out of stderr at the default threshold, so the Bound is right for it.

### [LOW] D6 and the Bound disagree with the refusal paths on the Info line

**Location:** `_lyx/discussion/decision-record.md:93`
**Issue:** D6 says every call logs a `webster: card pack` Info line with card, resolved and unresolved.
A call refused for a non-numeric argument, an unknown card, a missing plan directory or an unparsable plan has no resolved or unresolved counts to log, so either the line is logged with invented zero counts, which would make the audit count a refused call as an empty fetch, or it is not logged and D6 is wrong.
The acceptance criterion at line 137 reads the same way.
A refusal still exits non-zero, which force-opens the trace file with its header, so the Bound is incomplete for that path as well.
Naming the rule once in D6 and the Bound settles it before the plan writer has to guess.

## Focus checks with no finding

- Focus entry 3, D4 sandbox marker: the marker sits directly after the closing backtick of `lyx loom lint-pattern` and before the word `and` in `contracts/stencils/webster/webster-body-implementer.md:60`.
  Filled with a comma, a space and `lyx loom card-pack` in backticks, the list reads lint-comments, lint-pattern, card-pack and quarry resolve, in the same backtick style and the same no-serial-comma shape as its neighbors.
  Empty, it leaves the sentence byte-identical, so the file, inline, pack-off and recovery prompts keep their bytes.
  The recovery prompt fills its optional markers from the list at `internal/websterengine/render.go:491`, so the new marker joins that list in the plan, which D4's "empty in the recovery prompt" already implies.
- D4 pins the sandbox fill text and does not defer it elsewhere.
  Its fragment bullets describe content and leave the wording to the plan writer, which is not the same text pinned twice.
- Attack surface: `card-pack` is read-only, holds no lock and writes only logs, a bound D3 states.
  The default-on switch is bounded by one `lyx config webster --set`, which D1 names.

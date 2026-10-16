---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 (once-only rewrite keyed on the old template comments) and the matching Acceptance criteria line
    summary: The loom.yaml migration matches each line against the old template's comment, but a reconciled file carries those comments re-encoded with different spacing, so a byte-level match never fires on a real hub and shed.yaml stays masked for loom runs
    fix: In D5, replace "each with the old template's comment on its line" with "each with the old template's comment text on its line, compared as the yaml comment's text after `#` with surrounding whitespace trimmed, never as raw line bytes". In the Acceptance criteria migration line, add that the old-defaults input is the yaml-node-encoded form reconcile writes (a single space before `#`) as well as the template's own bytes (two spaces before `#`), and that both migrate.
---

### [MEDIUM] The migration's comment match is unspecified against the bytes reconcile writes

**Location:** `_lyx/discussion/decision-record.md` D5, and the Acceptance criteria line starting "Migrating a `loom.yaml` holding all four old template defaults".

**Issue:** D5 makes the rewrite fire only when all four keys hold the old defaults "each with the old template's comment on its line".
The old template at HEAD, `internal/loomengine/template.yaml` lines 36–39, spells each line with two spaces before `#` (`driver_kind: llm  # the driver ...`).
The hub's `_board/_lyx/config/loom.yaml` before its latest reconcile commit (`c2dba0788`, read with `git show`) held the same four lines with a single space before `#` (`driver_kind: llm # the driver ...`), because reconcile writes files through yaml node encoding.
So every reconciled `loom.yaml`, which is the file the migration targets, differs from the template bytes.
A plan writer who reads "the old template's comment on its line" as a line or byte comparison writes a rewrite that never fires on a reconciled hub, and an acceptance fixture copied from the old template passes while the live case fails.
The result is the outcome D5's rationale says the migration exists to prevent: `shed.yaml` dead for loom runs.
The decision needs to say what is compared, so the plan writer cannot pick the fragile reading.

## Focus verdicts

- Batten tackler resume in the prime: not a defect.
  D4 states what is let through (a tackler's writes in the prime: files, the index, a local commit, with its verdict still acted on and a `resume` able to re-enter Land or Worktree-Teardown over them), states that the stop-key check covers only run movement, and names the one bound on those writes, the stencil's write rule, as prompt text rather than a mechanical check.
  The write rule exists today in `contracts/stencils/loom/loom-template-tackler.md` ("Write the verdict file and, on an `escalate`, the case brief, and nothing else"), and the Acceptance criteria line "The rendered batten tackler prompt carries the write rule ..." pins it for batten.
  The rubric asks that the bound be stated; it is, and its weakness is stated with it.
- Loom config Migrate: D5 states the scope (every reconcile and every `run_settings` document, confirmed in `internal/configreg/runsettings.go`, which marshals the JSON keys with `yaml.Marshal` and passes them to `module.Migrate`), the once-only bound, the `run_settings` pin case and what the loom, darn and batten templates ship, and the Acceptance criteria cover each.
  One defect remains in how the match is specified: F1.
- Tackler settings source per recipe: not a defect.
  D5 states the arming module passes the run's own recipe's override for every tackler key, the same rule as D6's `driver_kind`, and the Acceptance criteria line with `darn.yaml` `tackler_max_spawns: 0` against `loom.yaml` 4, and the swap, pins it.
- Batten resume goto targets: not a defect.
  The restriction is in D2's seam list ("Resume goto targets"), D4 and D7 refer to it, and the Acceptance criteria line for `lyx batten run` checks that a `resume` naming the routed-from row is refused and hands back.
  It matches today's check in `internal/loomcli/tackler.go` `resumePrecondition`, which allows the current row or the last history entry's producer and hands back otherwise.

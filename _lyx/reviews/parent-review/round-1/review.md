---
verdict: APPROVED
---

## Scope against the board entry

The entry `intake-bugs-1012` bundles four bugs; its brief and title name three, and its body adds `plan-cli-tree-check` (#588).
The decision record covers all four, and Q8 records why.

- External-test gate paths (#571, #577, #580, #587): the entry asks for plan validation of gate-line package paths, rejection of a `_test` path with no directory, and optionally a `lyx gate test` message naming the missing directory.
  D3 adds the plan check, D4 makes `lyx gate test` name the missing directory, and D1 fixes the derived card and batch gates that produced every reported case.
  D2 records D1's mapping as a named change to `PATTERN-glyph-conversion-chokepoint`, which the entry does not rule out.
- Flaky `lyxtemp` sweep (#584): the entry asks to check first whether the flake still reproduces on `78e31aa06`'s bounded retry.
  D6 checks it, finds no reproduction, and closes the issue with no code change.
- Merge-In `status-missing` halt (#581): the entry asks why no status was produced and for a trace file and a more specific reason.
  D5 gives the likely cause, splits the reason into `status-missing` and `status-unreadable: <error>`, and records the loop log as the trace file.
- CLI-tree conflict (#588): the entry asks for a plan-authoring or plan-review check and a doc stating that audience annotations belong on leaf verbs.
  D7 writes the rule into `PATTERN-cli-cobra`'s background, points `docs/overview.md` at it and adds a Plan-Review rubric check.

No goal point is left out, and no decision does work belonging to another entry.
`twill-plan-followups`' wrong-parent-directory miss is a different check and stays there, as support-log turn 1 says.
`enclose-kickstart` keeps glyph-to-span intersection in quarry; D1 maps a unit pseudo-path to its directory, which is a different conversion, and D2 records it.

## Overlap with runs in flight

`lyx loom peers` lists two peers and no parent commits.

- `convergence-plan` (Webster): it changes `PATTERN.md` lines for `PATTERN-sole-parsers`, `PATTERN-pane-text-injection` and `PATTERN-test-economy`, none of which this run touches; this run changes the `PATTERN-glyph-conversion-chokepoint` line.
  Its `docs/overview.md` change is a module-table line and a module paragraph for `report`; this run changes the audience paragraph.
  Its D7 classes halt reasons by their leading clause, and this run's D5 keeps the `<class>: <detail>` form for that reason, so `status-unreadable` forms its own class.
  It shares no Go package with this run's scope (`planparser`, `gatecli`, `shedverbs`, `batcher`, `websterengine`).
- `operator-followups` (Plan-Write): its D5 adds `internal/shortcutcli` to the package-naming list in `pattern/PATTERN-cli-cobra.md`, and this run's D7 rewrites the audience bullet in the same file.
  The two edits sit in different sections and neither changes the `PATTERN-cli-cobra` index line.
  The rules agree: shortcuts are leaf root commands carrying an audience, and D7's rule is about commands with subcommands.
  I count this as a shared background file, not a substantive overlap.
  Its `docs/overview.md` change is a line naming the shortcuts, separate from the audience paragraph.

Shared bookkeeping, named and not escalated: `PATTERN.md` (different index lines), `docs/overview.md` (different paragraphs), `pattern/PATTERN-cli-cobra.md` (different sections) and possibly `cmd/lyx/testdata/test-budget.yaml`.

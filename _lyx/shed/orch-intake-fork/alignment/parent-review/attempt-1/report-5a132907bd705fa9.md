---
verdict: APPROVED
---

## Scope check

The design covers every point of the `orch-intake-fork` entry's operator decision and Idea list:

- The orch checks the inbox by number and title only: D1 makes `intake list` compact by default, and D3 has the orch run only that bare command and never `--full` or `import`.
- A non-empty inbox goes to a one-shot fork, not a fresh agent: D3, with the operator's reason kept as its rationale.
- The fork takes each issue to a new note, a fold or a close as noise, gives it its module labels and returns one line per issue: D4.
- The stencil's Where a finding goes and After landing sections carry the rule: D3.
- `intake import` prints a short result with slug, issue and labels: D2, which also adds `dropped` and keeps the close-failure envelope short, both serving the same goal of keeping bodies out.

The `ly:board` Intake section (D5) and `internal/boardcli/doc.go` go beyond the Idea list, but D1 and D2 would make both wrong, and CLAUDE.md requires docs in the same commit, so neither is extra scope.

## Other board entries

- `orch-split`: its Go run driver would take over intake at the landing routine; this design keeps the check in the orch and lists moving it to Go as out of scope, so the two neither duplicate nor contradict each other.
  `orch-split` is a note and would pick up whatever check the orch then runs.
- `labels-from-diff`: deriving labels from a diff is listed as out of scope; the fork labels by hand from the issue, as today.
- `creel-notices`, `stencil-intent-audit`, `board-viewer`: touch the orch or the board elsewhere; no conflict.

## Peers and parent commits

`lyx loom peers` lists one peer and no parent commits.

- `shuttle-open-permission` (row 7, own row 3): adds `--permission-mode` to `lyx shuttle open` in `shuttlecli`, `shuttleengine` and `claudeengine`.
  It shares no file, behaviour or PATTERN entry with this run, so there is no overlap and no landing order to give.

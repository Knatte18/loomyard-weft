# Decision record — checklist-skipped-row-box

## Goal

The loom checklist frame marks a recipe row the run passed because its entry gate did not hold with its own box, `[-]`, so a skipped row no longer reads as not yet reached.

## Scope

- `internal/loomcli/checklist.go`: the row box and the review segment heading box learn a skipped state, read from the status history's `OutcomeSkipped` entries.
- The halted box, today also `[-]`, moves to a glyph of its own.
- `internal/loomcli/checklist_test.go`: rows covering the skipped box and the moved halted box.
- `internal/loomcli/doc.go`: the checklist paragraph names the skipped box.
- Out of scope: shed's skip mechanics, the history format, and every other status view.

## Decisions

### A skipped row is boxed `[-]`

A row with at least one `OutcomeSkipped` history entry and no visit gets `checklistBoxSkipped`, `[-]`.
Rationale: the board entry asks for `[-]`, for every row an entry gate skipped.

### Precedence: current producer, then visited, then skipped, then not reached

The current producer's state box wins, a row with any visit is `[x]`, and only a row never visited but skipped gets `[-]`.
Rationale: a row that ran at least once has settled work to show; the skip box marks rows that were settled without running.

### The halted box becomes `[=]`

`checklistBoxHalted`, the box of a paused, blocked or awaiting current producer, changes from `[-]` to `[=]`.
Rationale: one glyph for both a halted row and a skipped row would read the same in the frame, and the board entry fixes `[-]` for the skipped row; `[=]` echoes a pause sign.

### A review segment never visited but with a skipped row is boxed `[-]`

The heading of a review segment with no visit and a skipped row reads `[-] <number>. <segment>` instead of `[ ]`.
Rationale: a review segment whose rows an entry gate skipped, such as a zero review budget, is settled exactly as a skipped plain row is.

## Constraints

- `PATTERN-sole-parsers` and `PATTERN-tolerant-records`: the status is read through `shedengine.Status` as today; no new parsing.
- `PATTERN-test-speed`: tests are rows in the existing table-driven checklist tests, tagged as that file already is.

## Auto-mode assumptions

- The operator accepts the halted box moving to `[=]` as the cost of giving skipped rows `[-]`.

## Open risks

- An operator used to `[-]` for a halted row may misread the new `[=]` until it is familiar; the state line still names the halt.

## Acceptance criteria

- A row skipped by its entry gate and never visited is drawn `[-]`, with no time and no visit count.
- A row both skipped and visited is drawn `[x]`.
- A paused, blocked or awaiting current producer is drawn `[=]`.
- A review segment whose rows were only skipped is drawn `[-]` as a single heading line.
- An unreached row is still drawn `[ ]`.

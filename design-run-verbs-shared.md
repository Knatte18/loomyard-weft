# loom names the general run, and Discussion becomes a block every variant can turn on

`run-verbs-shared` · Task · enhancement, shed, loom, darn, landing, drift · draft

## Seen (operator, 2026-10-14)

Since `0850f76e` a darn run has a decision record, and `lym loom decision add` works on it.
The operator finds it absurd that a `loom` command acts on something that is not a loom run.
The same holds for the other run verbs the orch uses on darn runs: `lym loom status`, `goto`, `resume`, `approve`, `reject`, `start`, `review approve`.
A third recipe is planned, `Hardener` (the note `hardener`), which would make three recipes steered with `lym loom`.

## Direction (operator, 2026-10-29)

`loom` becomes the name of the general run in Shed, the generic driver of a run, and `lym loom ...` stays as it is today.
What is called loom today is one variant of it, `tapestry`, darn is another, and `Hardener` becomes a third.
The mechanical rename of today's variant to `tapestry` is task `tapestry-rename`; this note is the design work after it: moving the generic parts under `loom`, built once that rename has landed.

## Discussion as a shared block (operator, 2026-10-31)

Today the two variants keep the same decision record, in the same format and checked by the same parser, but make it differently:
- loom's Discussion-Write writes it alone, and Discussion-Review's perch reviews it before Plan starts;
- darn's one writer turns the board entry into the record after a coarse reading of the code, asks the orch only when the entry is outdated, and then writes the code in the same session; no perch reviews the record itself.

The design:
- Discussion, a writer and its perch, becomes one block of the general run that a variant turns on or off: always on in `tapestry`, off by default in darn.
- With Discussion on, the writer starts from the board entry and makes the record thorough, and the perch reviews it, as `tapestry` does today.
- With Discussion off, the run writes the record mechanically, with no LLM: the board entry's body verbatim under `## Goal` and every other section empty.
  darn treats the entry as already discussed and plans and writes the code from that record in one session.
  This fits `loom-beam-darns`, whose darns are pieces a loom Discussion already cut.
- Either way `lym loom decision add` appends later decisions to the record as `Added after Discussion` entries, as today.
- The record is the decided scope: Code-Review checks the change against the record, never against the board entry's text, so darn's separate snapshot of the entry is no longer what the review compares with.

## What the general run holds (study of the code, 2026-10-31)

Both recipes already share one CLI (`lym loom`), one wiring path (`internal/loomcli/wiring.go`, with `wireDarn` overriding fields), one recipe builder (`internal/loomrecipe`) and the same row engines; darn is loom with another writer and a set of recipe branches.

The general run holds:
- The frame both run with the same engine today: Preflight, the Drift rows, the Code-Review perch, Describe, PR-Review, Finalize and Friction-Reflect.
- The Discussion block above, one gate and one validator in place of `NewDiscussionGate` and `NewDarnDiscussionGate` (`internal/loomshed/gates.go`).
- A writer slot each variant fills.
- One copy of each mechanism written twice today: Describe (`DescribeTable` and `DarnDescribeTable` in `internal/landingshed/describe.go`, with two stencils), the description commit, the writer wrappers (`loomshed/darnwrite.go` and `loomshed/discussionwrite.go`), the verify source, and routing, shed and retired-row lists (`Routing` and `DarnRouting`, `New` and `NewDarn`), each driven by the variant.

The variants keep:
- `tapestry`: Loom-Preflight, Plan and its perch, Webster, Parent-Align, plan amendments, the PATTERN record and PR rework.
- darn: its writer, tree-scoped verify and the prior-work respawn.

The `lym loom` verbs tell the variants apart, which three gaps the study found show they do not today; the task checks each and fixes what is real:
- `lym loom peers` reports a darn run's row position against loom's rows (`internal/loomcli/peers.go`, `hubPeersDeps`).
- `lym loom decision add` seems to accept `--pattern-*` on a darn run, which has no PATTERN record (not verified).
- `validate-plan`, `insert-card`, `remove-card` and `restore-renumber` seem to carry no recipe check, so on a run without a plan they fail on a missing plan instead of saying the verb does not apply (not verified).

The row order is one for every variant: Drift-Verify runs before Code-Review, as darn does today, since Drift-Verify can change the code a good deal, in `tapestry` especially, and the review must see those changes (operator, 2026-10-31).

## Open

- Whether `lym loom validate-discussion` accepts a record whose sections bar `## Goal` are empty, or the skeleton needs a fixed line in each.
- Where the check that the entry is not outdated (landed work or another entry contradicting it) lives once the record is written mechanically: in darn's writer, as today, or a step of its own.
- How `loom` as the general driver relates to `shed`: `lym shed` already carries `run`, `step`, `status`, `pause` and `goto`, which the general `lym loom` verbs would overlap.

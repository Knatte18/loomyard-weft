## Goal

Raddle's perch mode: recorded from the recipe's rows, with its outcome checks and refusals

Takes: raddle's auto and interactive modes in the raddle state, the raddle gate, raddle-propose, raddle-approve, Raddle-Write and Raddle-File, internal/raddleparser's checks and `lym loom decision add`
Delivers: the raddle mode `perch`, recorded by Raddle-Write on its first call when the recipe's rows hold a split-perch Bouncer over the proposal, tested with a fixture recipe; in perch mode the gate stores the cold proposal, raddle-approve refuses naming the perch, Raddle-File files with no approval-hash match and a fresh entry moves the last proposal aside; the D5 outcome checks in internal/raddleparser behind a perch option, with the slug-sketch and slug-taken exemption, the spec text and the raddle stencil's perch mode text; `freshRaddleEntry` and its table test corrected for a goto entry; `lym loom decision add` refusing in perch mode from the Raddle-File row on, with its ways forward. A perch-mode project proposal passes the checks here; its filing as the converted entry comes with arras-family-filing
Exports: the `perch` RaddleMode, the perch option of raddleparser's checks, the rows test that decides perch mode

## Scope

The part changes these packages:

- contracts/specs (about 30 changed lines)
- internal/loomshed (about 120 changed lines)
- internal/loomcli (about 180 changed lines)
- internal/raddleparser (about 110 changed lines)
- internal/loomengine (about 60 changed lines)
- internal/shedrecipe (about 30 changed lines)

## Decisions

### D4 — An arras run files on the perch's approval, with no operator approval

Decision: Raddle-Write records a third raddle mode, `perch`, in the raddle state on its first call when the recipe's rows hold the split perch, and then does not read `raddle_auto_approve`.
The mode is the one signal of an arras run: the `raddle` gate, `lym loom raddle-propose`, `lym loom raddle-approve` and Raddle-File read it from the state.
`lym loom raddle-approve` refuses in `perch` mode, naming the perch as the approval.
The perch's approved settle routes to Raddle-File, which files with no approval-hash match, as in auto mode.
In `perch` mode Raddle-Write ends on the finished proposal and the raddle gate's pass stores the cold proposal, as in auto mode.
Every fresh entry into Raddle-Write, a return from Raddle-File or a goto into the row, moves the last proposal aside, as in auto mode.
`freshRaddleEntry` in `internal/loomcli/raddlewiring.go` is corrected to match a goto entry as `shedengine.Goto` and `shedengine.Relaunch` write it:
the target row as its producer, outcome `goto` and no `RoutedTo`.
Its goto branch tests that the last entry's outcome is `goto` and its producer is `Raddle-Write`;
the Raddle-File Stuck branch keeps its `RoutedTo` test.
The goto rows of its table test in `internal/loomcli/raddlewiring_test.go` are rebuilt the same way,
since they carry producer `Raddle-File` and a `RoutedTo`, an entry no verb writes.
Without the correction a goto into the row is never fresh, so the seat's new session starts over the old proposal,
in a `raddle` run as in an arras run.

Bound: this removes the operator's approval only in a run whose recipe rows hold the split perch.
Raddle-File still runs every proposal check and the board re-check before and inside its write, and every filed task stays draft until the operator releases it.
Two routes file a proposal whose last round still had open findings.
With the split perch's auto-accept key on, a spent budget accepts such a round.
That route is bounded by the key's default of false and its guard category, which marks it as a key that hollows an approval.
With the key off, the Bouncer's soft-cap rule (`softCapRule` in `internal/shedadapters/bouncer.go`) settles a spent-budget round:
it accepts automatically a round in which no open BLOCKING or MEDIUM ledger entry carries a fix tag other than `mechanical`, a missing tag included,
as `designWrongKeys` in `internal/shedadapters/bouncerfiles.go` counts them,
and a round with such an entry continues automatically or escalates at the hard cap or on circling.
The accepted round may still hold open LOW and NIT entries of any tag and open BLOCKING or MEDIUM entries the judge tagged `mechanical`,
and the proposal is filed with them;
the judge's tag is the only thing between a BLOCKING `mechanical` entry and the filing.
That route is bounded by the judge's fix tags, the soft-cap window and the automatic flag on the round's decision file.
Since D2 sets no `carry_over`, the open findings of an automatically accepted round are dropped from the record;
they stay only in the round's ledger in the perch's run directory.
Raddle-File's proposal checks and the filed tasks' draft status hold on both routes.
D2's goto from a perch that has not settled is a third route, which can file a proposal no round reviewed;
the same checks and draft status hold on it.

Rationale: the perch replaces the manual approval the operator kept only while raddle was being tuned; release stays the operator's.

### D5 — An arras run ends in one drafted task or one project of drafted tasks

Decision: in `perch` mode the proposal checks allow two outcomes:

- One task: shape `tapestry`, or shape `tasks` with one part.
  The task is filed draft with its record, and the arras entry ends done through `boardengine.Board.EndRaddleEntry`, so the landing's board prune removes it.
- A family: shape `project`, whose `project.slug` equals the arras entry's slug.
  Each part is filed draft as a member of that project, with its `depends_on` and its part of the record.

Shape `beam`, and shape `tasks` with more than one part, fail the checks in `perch` mode with a finding naming the allowed shapes.
The `slug-sketch` and `slug-taken` checks exempt a `perch`-mode project slug, which must equal the run's entry.
The raddle stencil's mode text for `perch` mode tells the agent these outcomes.

Rationale: the operator set these two ends; a beam would start the family's runs, which would take the release from the operator.

### D7 — Decisions made after the Discussion enter the record through `lym loom decision add`

Decision: during an arras run past the Discussion approval and before Raddle-File, an operator decision is recorded with `lym loom decision add --by operator`.
It appends to the run's decision record and stores it on the board, as for a run of any recipe past that approval.
Raddle-Write reads the record at its start, the perch reads it every round, and every filed part's record carries every Added after Discussion entry whole through `discussionparser.SplicePart`.
The orch text says an operator decision about a running arras entry goes in this way and never into the entry body, which the run lock holds.

Bound: from the Raddle-File row on, `lym loom decision add` on a run whose raddle state records `perch` mode refuses.
D4 records that mode from the recipe's rows, so the refusal is decided by the rows and a `raddle` run keeps today's behaviour.
Before the filing its way forward is `lym loom goto --to Raddle-Write`, which re-splits with the decision recorded first.
After the filing the refusal lists the filed slugs, read from the run's filing.
Its way forward is `lym loom decision add` on the part the decision concerns, once the operator has released that part and its run has passed its record row.

Rationale: the `slack-v2` miss came from decisions left in the entry body; the record is the one place the split and its perch read.

## Constraints

- `PATTERN-shed-recipe-registry`: arras declares `contract: loom`, and a verb that serves some loom recipes refuses on others by the recipe's rows, never its name (D4's mode, D7's refusal).
- `PATTERN-stencil-ownership`: the new rubric and the raddle stencil's mode text are read at call time through `stencilstore`.
- `PATTERN-sole-parsers`: D5's outcome checks live in `internal/raddleparser`, the proposal's sole parser, which takes the mode as an option.
- `PATTERN-refusal-way-forward`: the refusals of D4, D5, D7 and D8 name their way forward, and a test reaches each.
- `PATTERN-run-records`: post-Discussion decisions are written only by `lym loom decision add`.
- `PATTERN-dobby-runs`: released members start only through their parent's dobby, which D8 relies on and does not change.

## Auto-mode assumptions

- The perch's fixer is cold, not Raddle-Write re-entered (D2).
- A project release releases its draft members at once, and the project keeps the whole record (D8).
- Shape `beam` is refused in an arras run (D5).
- The family's project is the arras entry turned into a project (D6).

## Open risks

## Acceptance criteria

- D4: a `raddle` run keeps its interactive approval when `raddle_auto_approve` is off.
- D5: in `perch` mode a proposal of shape `beam`, of shape `tasks` with two parts, or of shape `project` whose project slug differs from the entry's slug fails its checks, and the fixer's gate run applies the same checks.
- D7: `lym loom decision add` on an arras run at Raddle-File or later refuses with its way forward: before the filing the goto to Raddle-Write, after it the filed slugs and the record step on a released part.
- D7: `lym loom decision add` on a `raddle` run at Raddle-File behaves as it does today.

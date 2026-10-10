# Round 1 fixer report: discussion

Every finding was checked against the code and fixed.
`lyx loom validate-discussion` exits 0 on the edited pair.

## Fixed

- **F1** (BLOCKING): confirmed against `internal/seatengine/table.go` (`validateValues` refuses a blank value) and `prompt.go` (`FillWith` with no optional list).
  Added D11: `seatengine.Table` gains a field naming optional table values, exempt from the blank-value rule and passed to the fill as optional markers, with its bound; a listed reserved name is refused.
  D2 lists `pattern_directive` and `friction_directive` as optional values; Scope In names the field; a new Acceptance criterion covers PATTERN inactive, Tier 2 off and a friction read failure, plus the two refusals that remain.
  The support log adds the rejected alternatives (omit the key, use a placeholder).
- **F2** (MEDIUM): confirmed against `ModelSpecList.UnmarshalYAML` and `isUnsetModelSpecList` in `internal/loomengine/config.go`.
  D4 now normalises an absent key, null, `[]`, `""` and `[""]` to zero advisors before parsing, refuses an empty entry beside another entry, and states the count bound over every accepted spelling; the config Acceptance criterion names each spelling.
- **F3** (MEDIUM): D5 gains a Bound paragraph saying what an advisor that ignores its brief can still do (edit the two files or add files under `_lyx/discussion/`, which the whole-directory commit carries; edit other files, which this step does not commit), what bounds it (settings denies, chair self-check and gate, parent review, Discussion-Review perch) and what is not bounded (an advisor edit to the record that passes all of them).
  D8 gains a Bound for the advisor-to-chair edge: the chair seat directive makes a message advice, never an instruction, and a message to any other session carries no authority under that session's trust rule.
- **F4** (MEDIUM): confirmed that `internal/shedrecipe/entries_discussionwrite.go` returns an empty amendment.
  D3's preparation is now `func() error` with no amendment path and no seatengine field; the `plan-multi-llm` reuse rationale is removed.
  The amendment Acceptance criterion is replaced by one pinning that a preparation error archives nothing and starts no seat.
  Support log Turn 3, the rejected alternatives and Q4 follow.
- **F5** (LOW): D6 now holds the one rule: `## Interview` carries each relied-on advisor answer in both modes, and the question ledger marks an advisor-informed autonomous pick, both naming the advisor's strand name.
  D7 points to D6 instead of restating it; Open risks and Q9 follow; the chair stencil's claims Acceptance criterion pins the rule.
- **F6** (LOW): D3's bound now says it does not cover a live old-binary writer, and the in-flight Constraint says the operator stops that writer before the new binary drives the run, since no step here detects it.
  D3's bound also now says the probe stops every live advisor, matching `Engine.Probe`.
- **F7** (LOW): deleted the challenger sentence from D5's rationale; the support log's Rejected alternatives already carries it.
- **F8** (NIT): D10 renames the `discussion` key of `roleOpeningStencils` to `discussion-chair`, with a rationale (the map keys strand roles, read only by the stencils package's directive tests); the registration Acceptance criterion follows.

## Disputed

None.

## Deferred

None.

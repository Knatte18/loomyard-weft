# Card 4 — plan gate PATTERN checks

**Edit:**
- `internal/loomshed#PlanContext`
- `internal/loomshed#planContextFindings` -> `func planContextFindings(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string, pc PlanContext) ([]planindex.Finding, error)`
- `internal/loomshed#ValidatePlan`
- `internal/loomshed#ValidateReworkPlan`
- `internal/loomshed#TestNewPlanGate`
- `internal/loomshed#seedPlanContextFixture` -> `func seedPlanContextFixture(t *testing.T, anchorPath, renamePair, planSection, recordSection, cardFence string) PlanContext`
- `internal/loomshed#TestNewReworkPlanGate`
- `internal/loomshed/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`

**Create:**
- `internal/loomshed#patternHeadroomCheck`
- `internal/loomshed#patternRemovesPrescribedCheck`
- `internal/loomshed#patternLineUnprescribedCheck`
- `internal/loomshed#patternPlanFindings`

**Uses:**
- `internal/pattern#Headroom`
- `internal/pattern#ProjectOverview`
- `internal/pattern#EntryBullet`
- `internal/planparser#CardPatternEntries`
- `internal/testkit/plankit#Write`

**Intent:**
D1 and D2 as plan-gate context checks, beside `pattern-changes-mismatch`.

`PlanContext` gains `PatternBase func() ([]byte, error)`: the overview's bytes at the task branch's merge base with its parent, or at `HEAD` with no parent recorded, nil for an absent file.
A nil `PatternBase`, which the zero `PlanContext` has, runs neither check, so every existing caller passing `PlanContext{}` is unchanged.
Card 5 wires it.

`planContextFindings` takes the worktree root and appends `patternPlanFindings` after the mismatch findings, passing its `cards` argument, the cards not recorded done; its two callers in `gates.go`, `ValidatePlan` and `ValidateReworkPlan`, pass their `worktreeRoot`.
`pattern-removes-prescribed` reads only those cards: a done card's fenced line is already built, and the projection applies `removes` after the prescribed lines, so the entry is still dropped.
The projection and `pattern-line-unprescribed` read `plan.Cards`, every live card, never the pending subset, since a done card's prescribed line is already in the tree and the projection replaces by ID.

`patternPlanFindings` reads `PATTERN.md` in `worktreeRoot`; with no such file, or a nil `PatternBase`, it returns nil and reads no base, so a repository without a PATTERN, such as every plan fixture, never reaches git.
Otherwise it reads the base through `PatternBase`, whose error is returned, and then, in this order:

1. `pattern-removes-prescribed`: one blocking finding per `removes` entry of `plan.PatternChanges` whose ID a card not recorded done fences a line for (`planparser.CardPatternEntries`), naming the bullet and the card's number, with the way forward: drop the fenced line from the card, since the plan both drops and rewrites the entry; no verb withdraws a `removes` bullet from the record, so the finding never asks for that.
   Reaching input: a plan whose `## PATTERN changes` declares `removes` for `PATTERN-x` and whose card fences ``- `PATTERN-x` — …``.
2. `pattern-headroom`: the projection is `pattern.ProjectOverview` over the worktree overview, every card's fenced lines in plan order and card order, and the `removes` IDs; `pattern.Headroom(base, projection)` decides.
   A finding is blocking, carries `Headroom`'s message (projected size, base size, the line and the bytes to cut), and the way forward: shorten the prescribed lines or move detail into the entries' background files, and when that is not enough ask the parent to record a `changes` decision that shortens other entries with `lyx loom decision add --pattern-entry … --pattern-action changes --pattern-text …` and prescribe those lines on a card; once the plan is approved, the parent passes `--amends-plan` as well.
   Reaching input: a fenced line that grows a base just under `HeadroomBytes` past it.
3. `pattern-line-unprescribed`, only when `pc.DecisionRecordPath` is set (a loom run, where `pattern-changes-mismatch` runs): one blocking finding per `adds` or `changes` entry of `plan.PatternChanges` that no card fences a line for and whose `pattern.EntryBullet` in the worktree overview equals the one at the base (both absent counts as equal).
   An entry an earlier generation already built differs from the base and passes.
   Its way forward: prescribe the entry's whole overview line in a fenced block on a pending card that edits `PATTERN.md`, or, when that card is done, on a card inserted with `lyx loom insert-card` after the last begun batch.

What this lets through, the bound D2 states: an undeclared prose edit of `PATTERN.md`, and a further prose change to a declared entry already differing from the base that no card fences; the card gate's `lyx loom lint-pattern` and the verify gate's headroom lint stay the guard for both.
On a run with no parent recorded the base is `HEAD`, so an entry an earlier generation built and committed reads unchanged and raises `pattern-line-unprescribed` again; fencing the line on a card, the finding's way forward, still clears it.

`doc.go` in `internal/loomshed` names the three checks, their order, the `PatternBase` read and the no-PATTERN skip beside the two context checks it describes.
`loom-plan-spec.md`'s validation-checks section gains the three IDs in the paragraph naming `rename-tool-missing` and `pattern-changes-mismatch`, and row 39's sentence that a prose instruction to append to an entry passes is replaced by D2's rule.
`loom-template-plan.md`'s `PATTERN.md` paragraph says a declared `adds` or `changes` entry is prescribed verbatim as its whole overview line in a fenced block, that a card fences only the new line, never the old one or an example, and that the plan gate measures the projected overview against the headroom line.

Tests extend the plan-context table of `TestNewPlanGate`, which already drives `ValidatePlan` through `seedPlanContextFixture` and counts each context check.
`seedPlanContextFixture` takes `cardFence`, fenced text written into the card's `Intent` (empty writes none); its four callers pass `""`.
The new rows set three more row fields: the worktree's `PATTERN.md` bytes, written into the row's worktree root (absent when empty), and the base bytes a fixed `pc.PatternBase` closure returns, or an error it returns; each row counts the three new checks and asserts each finding's detail holds its way forward:
lines that grow the projection past the line from a base under it (one `pattern-headroom`, whose detail names the bytes to cut and the `lyx loom decision add` way forward);
lines replacing existing bullets at equal size (none);
a `removes` entry and no fenced line (none, and a smaller projection passes where the same overview with the bullet would not);
a `removes` entry with a card fencing that ID (one `pattern-removes-prescribed` naming card and bullet, its detail saying to drop the fenced line), and the same with the fencing card recorded done in a Webster state the row saves, with the card's hash, as the done-card subtests of the same file do (no `pattern-removes-prescribed`);
two cards fencing one ID where only the later card's line fits (none) and the reverse (one `pattern-headroom`), and two lines of one card likewise;
a rework overview already holding the lines projecting the base's size (none);
a declared `adds` entry with no fenced line and an unchanged bullet (one `pattern-line-unprescribed`, its detail saying to fence the whole overview line on a pending card that edits `PATTERN.md` or on a card inserted with `lyx loom insert-card`), the same with the line fenced (none), the same with the worktree bullet already changed (none), and the same with no decision record path (none);
no `PATTERN.md` in the worktree with a `PatternBase` that would fail (none, no error);
and a `PatternBase` error (a returned error).
The existing rows set none of the new fields and keep their expectations.
`TestNewReworkPlanGate` gains a subtest: a rework plan whose card fences a line that grows a worktree `PATTERN.md` past the line from a fixed base fails the rework gate naming `pattern-headroom`, so the checks run through `ValidateReworkPlan` too.

**ImpactSummary:** `planContextFindings` has two callers, both in `gates.go` and re-signed here; `PlanContext` gains one optional field whose nil value keeps every existing caller's behavior; the spec and plan-writer stencil change in prose only.

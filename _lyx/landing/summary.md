# Discussion scope check into perch lenses; Merge-In → Parent-Align

Loom's Discussion scope check moves from the alignment agent into the Discussion-Review perch's lenses, and a question only the run's parent can settle now reaches the parent instead of the fixer.
The darn writer checks its board entry before it writes its record.
Loom's `Merge-In` row becomes `Parent-Align`.

## What changes

**Alignment agent and parent review removed.**
`internal/alignment`, `internal/parentreview`, the `parent-review` gate kind, `Env.ParentReview`, the `lyx loom review` verbs (`notify`, `delivered`, `approve`, `reject`), batten's `ReviewWait` seam, `lyx loom status`'s waiting note and their stencils are deleted.
Discussion-Write's only gate is `discussion`, and a passing discussion goes straight to Discussion-Bouncer with no agent spawn and no parent message.
No landing-time approval replaces the agent: the Parent-Align fixer's check of the work against the decision record and what the parent brought in is the landing-time alignment.
`DecisionNoteTitle` (`Decision for <slug>:`) survives in `internal/orchengine`.

**Scope lenses in the perch.**
`goal-scope` and `board-fit` in `internal/lenses/template.yaml` are rewritten as procedure lenses ending in their own list, and the `review-discussion` fan gains both.
`goal-scope` reads the board and the task's decision notes; `board-fit` also reads `lyx loom peers` and the parent's landed commits.
Each stays within six tool calls.
`twill-discussion` and `discussion` keep their members and run the rewritten texts.

**`decision` escalates to the parent.**
In the Discussion-Review segment, `decision` now means a question only the run's parent can settle: a plainly outdated entry, a design conflict with another run, a landed commit contradicting a decision, or doubt about what the task is for (`loom-rubric-discussion-review.md`).
A Discussion-Bouncer with the new row key `escalate_decisions` escalates a judged round holding a `decision` finding to the parent with cause `decision`, at any severity, before the soft cap, circling escalation and `discussion_auto_accept`.
The Discussion-Burler-Fix row (key `decisions_escalated`) still withholds such findings from the fixer but posts no notice of its own.
Plan-Review and Code-Review keep today's behaviour.
`internal/arbiterstop` hands a `decision` escalation back to the parent.
On `accept`, the round's `decision` findings are carried into `## Open risks`, each naming the `Added after Discussion` entries recorded since the escalation.
`loom-template-discussion.md` tells the writer to leave such a question open under `## Open risks` rather than settle it.

**Darn writer.**
`darn-template-write.md` has the writer read the board, the entry's decision notes and `lyx loom peers` before writing its record.
If the entry is plainly outdated, it asks its parent by `SendMessage` and waits; with no parent it stops before any change and writes a stop report as its change description.

**Rename.**
Row `Merge-In` → `Parent-Align`, with engine key, `landingshed` producer, fixer mode, `landing.yaml`'s `merge_fix_attempts` → `parent_align_fix_attempts`, `gate.yaml`'s `merge-in` → `parent-align`, stencils and docs.
`lyx fabric merge-in` and `fabricengine`'s merge-in names are unchanged.

## Compatibility

- Reconcile drops the retired `loom.yaml` keys `alignment`, `alignment_timeout_min`, `skip_alignment` and `parent_review_wait_min`, and renames `merge_fix_attempts` and the `merge-in` gate key, reporting each.
  A run profile or board `run_settings` holding an old key is migrated at read time with a Warn.
- A run parked on `Merge-In` resumes on `Parent-Align` through `shedengine.RenameProducers`, keeping its spent attempts, `Baseline`, `AlignFrom` and halt.
  The Code-Perch still finds the old row-named fixer report.
- A run waiting in a parent-review round passes Discussion-Write on its next gate evaluation under the new binary; a run halted at the parent-review reject cap needs `lyx loom resume`.
- `internal/reportengine` keeps counting archived parent-review rounds and alignment attempts and filing archived `Merge-In` rows, through read-only decoders kept in the package.
- A hub `lenses.yaml` that defines `goal-scope` or `board-fit` keeps its own text, so that hub runs its old lens in `review-discussion` until it drops the entry; reconcile does not report this.

## Design decisions to check

- A `decision` finding stops the run until the parent answers, so a writer's choice mis-classed `decision` costs a stop; the rubric is the only guard.
- A decision recorded on an escalation applies the judgment refusal before it acts: an `accept` is re-judged first, and a refusal that holds at the budget re-escalates below the hard cap or halts with a budget-exempt Stuck at or past it, naming `lyx loom resume`.
- The scope check now runs after the writer finishes, so a round may polish a record whose scope the parent then changes.
- `umbrella-daemon` plans to move the arbiter into shed and must keep the refusal of a `decision` escalation; `loom-beam-darns` may run darn pieces where the darn writer's check would see sibling pieces as peers.
- Board notes and hub orch rules that still name `Merge-In`, parent review or the alignment agent are not rewritten here.

## To check by hand

- `d8-next-fixer-carrier-does-not-exist` and `d8-hard-cap-refusal-has-no-exit` (design, MEDIUM, `internal/shedadapters/bouncerdecision.go`): a `continue` hands the brief's BLOCKING findings to no fixer, and the hard-cap refusal's way out is `lyx loom resume`; confirm both bounds are acceptable.
- `docs-overview-keeps-retired-names` (design, MEDIUM, `docs/overview.md`): the final fixer pass may leave retired names, so grep it for `Merge-In`, parent review and the alignment agent; the remaining `merge-in` hits should be the fabric verb and git-level merges only.
- `loom-help-removed-review-verb` (scope, MEDIUM, `internal/loomcli`): confirm no help text or refusal still names `lyx loom review`.
- The plan-level verify failed on `TestDiscussionTable/lens_review` (four subtests, `internal/loomengine`) and `TestSmokeBootstrapLifecycle/malformed_status_proceeds_to_the_success_envelope_and_logs_why` (`internal/loomcli`); one fix commit (`19147b1df`) addressed them.
  Rerun both with `lyx gate test`.

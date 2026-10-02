# Decision record — rework-generations

## Goal

A PR-Rework round starts a new plan generation instead of appending to the existing plan.
After a round, `_lyx/plan/` holds only that round's cards and a fresh `00-overview.md`, so Webster's prompt and context stay flat however many rounds a task takes.
Everything scoped to the retired generation (its plan, Webster's run record, and the Plan-Review and Webster-Review round files) moves into the round's own directory, which git keeps.
A rework generation whose cards change code goes through Plan-Review;
a generation of prose-only cards keeps today's exemption.

Two review-stencil fixes ride along:
the discussion and plan rubrics ask what a new verb or downgraded guard can now skip or let through,
and a Burler focus directive no longer forces the reviewer to read prior-round files before its own review is saved.

## Scope

In:

- PR-Rework (`internal/loomshed/prrework.go`, its wiring in `internal/loomcli/wiring.go`, `contracts/stencils/loom/loom-template-rework.md`): archive the live generation before the session, have the session write a whole new plan, classify the generation for Plan-Review, commit the archive moves and the new plan together.
- Plan format: an optional `first_card` scalar in `00-overview.md` frontmatter so card numbers stay unique across generations (`contracts/specs/loom-plan-spec.md`, `internal/planparser`).
- The rework gate (`rework-plan`, `lyx loom validate-plan --rework`, `loomshed.ValidateReworkPlan`): new meaning, see Decisions.
- Retiring the append-only machinery: `planparser.CheckAppendOnly` and the new-cards-only scoping, plus PR-Rework's call to `websterengine.RebaselinePlanFingerprint`.
- `websterengine`: a primitive that moves the durable run record into a told directory, so the next `lyx webster run` starts fresh.
- Recipe routing (`contracts/recipes/loom-recipe.yaml`): PR-Rework's `on_done` becomes `Plan-Bouncer`; Plan-Bouncer gains a skip seam for an exempt rework generation.
- Plan-Review's answer key for a rework generation (Plan-Burler `fasit` instructions, `loom-rubric-plan-review.md`).
- Webster-Review's review range for a rework generation (`loom-rubric-webster-review.md`).
- Describe: told every generation's run record, not only the live one.
- Attack-surface question: `loom-rubric-discussion-review.md`, `loom-rubric-plan-review.md`, and the writer-side symmetry lines in `loom-template-discussion.md` and `loom-template-plan.md`.
- Burler focus/prior-round conflict: `contracts/stencils/bouncer/bouncer-template-judge.md` and `contracts/stencils/burler/burler-focus-directive.md`.
- Docs in the same commit: `docs/overview.md`, `README.md`, the recipe header comment, `lyx loom reject`'s help text, `CONSTRAINTS.md`.

Out:

- Any change to PR-Gate, the rework budget, `lyx loom reject`'s state checks, or the rejection record format.
- The Master prompt's "Still to run" line (`3613d5076`) and the prompt-as-file launch (`e39909bbc`): both stay; they still serve crash-resume within one generation.
- `lyx webster rebaseline` and `lyx webster restore-plan` as operator verbs: unchanged.
- Migrating an in-flight run parked at PR-Rework under the old append-only rule: it is restarted, not migrated.
- Conditional routing in `internal/shedengine` (a new outcome or edge kind).

## Decisions

### D1 — A generation is everything the plan scopes

Decision: when PR-Rework takes a new rejection, Go moves the live generation into `_lyx/loom/rework/round-<N>/prior-generation/` before the rework session starts.
The moved set is: every entry of `_lyx/plan/` (card files, `00-overview.md`, `amendments.md`, any `archive-*/` rotation directory), Webster's durable run record, and the Plan-Review and Webster-Review run directories under `_lyx/reviews/`.
`_lyx/plan/` then holds exactly one generation: the live one.
Rationale: Webster, Plan-Review and Webster-Review each read a whole directory; leaving any of the retired generation's files there brings the dead cards back (Master's `ls`, a judge's ledger carry-forward of findings against cards that no longer exist, a Burler's prior-round list).
Keeping the archive outside `_lyx/plan/` keeps the plan directory's contents equal to the plan.

### D2 — The round directory owns its archive

Decision: round `N` is the round that retired generation `N-1` and started generation `N` (generation 0 is the Plan-Write plan).
Its directory holds `findings.md`, `record.json` (written per D14), `coverage.md` and `prior-generation/`.
Rationale: one place answers "what did round N see, decide and replace", and the round directory is already committed and already crash-keyed by rejection identity.

### D3 — Card numbers continue across generations

Decision: `00-overview.md` frontmatter gains an optional scalar `first_card` (default `1`).
A generation's cards are numbered `first_card .. first_card+n-1` with no gap; `index-file-mismatch` and `card-numbering` check against `first_card` instead of `1`.
A rework generation's `first_card` is one past the highest card of the generation it retires, which is today's `NextReworkCardNumber` value.
Rationale: card numbers appear in commit subjects (`N: <name>`), coverage maps, review findings and friction notes; a number that means one card for the whole task keeps all of those unambiguous across rounds.

### D4 — The rework session writes a whole plan

Decision: the rework session writes a complete plan into the empty `_lyx/plan/`: a fresh `00-overview.md` (`approved: false`, `first_card` set to the told number, a Card Index listing only the new cards) and the new card files.
It reads the archived overview and cards for context and carries forward whatever task framing, `## Shared Decisions`, `## Rename mechanic` and `## verify:` content still holds; it never copies a retired card.
Rationale: an overview listing only the live cards is what makes Webster see "this round's cards plus `00-overview.md`"; the `## verify:` command is task-wide and must survive into every generation.

### D5 — The rework gate validates the whole new plan

Decision: the `rework-plan` gate (`lyx loom validate-plan --rework`, `loomshed.ValidateReworkPlan`) runs the same format-only check set as the `plan` gate over the whole live plan, plus one rework check: `first_card` equals the number Go told the session.
The append-only comparison (`planparser.CheckAppendOnly`) and the new-cards-only scoping are deleted.
Rationale: every card in the live plan is new and unbuilt, so resolving each against the current tree is correct; the scoping existed only because old cards lived beside new ones.

### D6 — Plan-Review routing is decided mechanically by card type

Decision: after the session and gate pass, PR-Rework classifies the generation: `exempt` when every card's every type label is `Prosa` and no `Prosa` target is a source file, `required` otherwise.
A `Prosa` target is a source file when it is a whole-package self glyph, or a file self glyph whose extension is a source extension of the plan's `language:`.
The classification is one function, shared by PR-Rework and the Plan-Bouncer skip seam (D7).
PR-Rework adds the class to the round's `record.json` (D14).
Rationale: the operator wants cards that create or change a classifier, verb or guard reviewed, and text-only findings exempt; a label the rework agent chooses for itself cannot be trusted to flag its own risky card, while card type and target paths are already in the plan and mechanical.
`prosa-symbol-target` admits any file or whole-package self glyph under `Prosa`, so the type label alone would exempt a `Prosa` card that rewrites Go code.
Any code-touching card (`Create`, `Edit`, `Delete`, `Rename`, `Move`, `Custom`), and any `Prosa` card on a source file, routes to review; this over-covers rather than under-covers.

### D7 — Routing goes through Plan-Review's own segment, with a skip seam

Decision: `PR-Rework.on_done` becomes `Plan-Bouncer`, so a rework generation walks the same main line as the original plan: Plan-Review → Batchifier → Webster → Webster-Review → Describe → Publish → PR-Gate.
The Plan-Bouncer row gains a recipe config key naming a skip seam.
The live generation's round is the highest committed rework round whose `record.json` `first_card` equals the live overview's `first_card`; with no such round, the live plan is treated as generation 0.
The seam reports skip only when the live generation's round exists, its `record.json` says `exempt`, and D6's classification run over the live committed cards also says `exempt`.
On skip, Plan-Bouncer settles as approved without spawning a judge: approve seam (sets `approved: true`), commit seam, Done.
Bound: the seam's inputs are not Go-only (the rework session writes the overview's `first_card`, and every status commit sweeps `_lyx/loom/rework/`, so an agent-written `record.json` gets committed); re-classifying the live cards means a stale or forged record can at most skip review of a generation whose live cards are themselves exempt.
Rationale: shed has one Done edge per row and no conditional routing; the skip seam keeps approval on the one row that owns the approve seam, and a generation that never passed review or exemption can never reach Webster unapproved.
Tying the skip to `first_card` means generation 0 (no round record) and any later generation without a matching record always get a real review.

### D8 — Webster starts a fresh run per generation

Decision: `websterengine` exposes one primitive that moves the durable run record (state, reports, outcome, summary, integration report, plan baseline) into a directory the caller tells it; the wiring tells it `round-<N>/prior-generation/webster/`.
PR-Rework calls it during the archive step and no longer calls `RebaselinePlanFingerprint`.
The next `lyx webster run` finds no state and starts a new run over the live plan only.
Rationale: a resumed run carries every earlier batch's record into Master's prompt; a fresh run is what removes them.
The archive layout is the caller's told path, per the Told-Geometry Invariant.

### D9 — Plan-Review's answer key in a rework generation

Decision: when the live generation's round (D7) exists, its `findings.md` is part of Plan-Review's answer key beside the decision record: every finding is covered by a card, and every card's scope is licensed by the decision record or a finding.
With no such round (generation 0, or a fresh Plan-Write plan after an operator `goto Plan-Write`), the answer key is the decision record alone.
This is stated in the Plan-Burler `fasit.instructions` and in `loom-rubric-plan-review.md`.
Rationale: the decision record alone would flag a rework card as unlicensed scope, and nothing would check finding coverage; the live generation's round is derivable state, so no new marker is needed.

### D10 — Webster-Review reviews only the live generation's commits

Decision: `loom-rubric-webster-review.md`'s "Determining the review range" section gains a rework branch: when the live generation's round (D7) exists, the range starts at that round's recorded `head_sha` (the rejected PR head) instead of the merge base, restricted to the branch's own commits.
With no such round, the range stays the merge base.
Earlier generations' code is context, never the subject.
Rationale: the fasit is the live plan; reviewing code no live card describes produces findings the segment cannot resolve against its answer key, and that code already passed Webster-Review and the operator's own PR review.

### D11 — Describe sees every generation

Decision: Describe is told the live run record and every archived generation's `summary.md`, oldest first.
Rationale: the PR description covers the whole branch; manual-check items from earlier generations stay relevant after a rework round.

### D12 — Attack-surface question in the discussion and plan rubrics

Decision: both rubrics gain an "Also flag" item: for every new verb, flag, escape hatch or routing edge, and every guard that is removed, downgraded or bypassable, the artifact states what it can now skip or let through and what bounds it; an unbounded one is a finding.
The discussion and plan writer stencils gain the matching instruction, keeping the writer/reviewer symmetry each rubric already invokes.
Rationale: Plan-Review checks fidelity to the decision record, so the question has to be asked where the decision is made and where the card states it.

### D13 — A focus directive is self-contained

Decision: the Bouncer judge template's "What a `focus` entry may say" gains a rule: an entry restates every site, commit and claim it depends on and never refers the reviewer to a prior round's review, fixer report, or finding ID.
`burler-focus-directive.md` states that the directive never licenses reading prior-round files before the review is saved; a directive that points into one is followed only as far as its own text goes, and the gap goes under `## Focus departures`.
Rationale: the clean-room rule is what makes each round an independent judgment; restating sites costs the judge a few lines and keeps the rule unconditional.

### D14 — Crash safety of the archive step

Decision: the archive step is idempotent and keyed by rejection identity.
Its completion marker is `record.json`: the archive step writes it (rejection identity and the told `first_card`) last, after every move.
A round directory whose `record.json` records the pending rejection's identity means the archive is done and the session may have begun; the archive step never re-runs there.
The highest round directory with no `record.json` is the pending rejection's interrupted archive: PR-Rework resumes into it and completes any unfinished move, never allocating a new round for it.
So a crash at any point resumes into the same round directory and never moves a file the rework session wrote into the new `_lyx/plan/`.
After the session and gate pass, PR-Rework adds the class (D6) to the same `record.json`.
The commit after the session includes the moved-from paths, so the deletions and the archive land in one commit.
Rationale: PR-Rework already promises crash convergence by rejection identity (`roundFor`, `committedRejections`); the archive step moves files the session then replaces, so it must not re-run once the session has begun.

### D15 — New cross-cutting invariant

Decision: `CONSTRAINTS.md` gains a Plan Generation Invariant: `_lyx/plan/`'s top-level files hold exactly one generation; a retired generation lives only under the rework round that retired it, except where Plan-Write's own `archive-*/` rotation moves the live plan aside on a re-run; only PR-Rework archives a generation into a round; Webster's run record and the plan and webster review run directories are generation-scoped.
The Gate Self-Check Parity line for PR-Rework is updated to the new gate.
Rationale: several modules (loomshed, websterengine, the review segments, Describe) now depend on the same rule, and the repo records cross-cutting rules there.

## Constraints

- Told-Geometry Invariant: `websterengine` receives the archive destination as a told path and derives none.
- Lyxdirs Single-Declarer Invariant: every new path is built from the existing accessors (`planparser.PlanDir`, `loomengine.LoomReworkDir`, `loomengine.LoomReviewsDir`, `websterengine.Dir`), never from an `_lyx` literal.
- Fabric Git Invariant: agents write into `_lyx`, Go commits; the archive is read-only to the rework session, which never edits, commits, moves or deletes it.
- Gate Self-Check Parity Invariant: `lyx loom validate-plan --rework` and the PR-Rework gate run one shared implementation.
- Planparser Sole-Parser Invariant: `first_card` is parsed and checked only in `internal/planparser`.
- Shed Recipe Registry and Recipe-Format Sole-Parser invariants: the skip seam is a registered config key, parsed where the other row config keys are.
- Refusal Way-Forward Invariant: every new or reworded Stuck reason and refusal names a way forward.
- Stencil Ownership Invariant: stencils are read at call time; marker changes keep the stencil tests passing.
- Row names are durable resume identities: no row is renamed or removed.
- Markdown semantic line breaks in every edited `.md`.

## Auto-mode assumptions

- D6 reads the board's "text-only findings" as `Prosa` cards on non-source files.
- D3 reads "a new generation" as continuing card numbers, not restarting them.
- D1 reads "archived (git keeps them)" as a move into the rework round directory.
- D1 extends the board's generation from the plan to the review round files and Webster's run record.
- D10 narrows Webster-Review to the live generation's commits; the board does not name the range.
- D11 extends Describe to every generation; the board does not name Describe.
- D12 adds the writer stencils, beyond the two rubrics the board names.
- D13 fixes the focus-directive conflict on the judge side.
- Scope Out restarts an in-flight run parked at PR-Rework rather than migrating it.

## Open risks

- Webster-Review range with a parent merge: Webster may merge the parent branch mid-run, so a plain `<head_sha>..HEAD` diff includes parent changes; the rubric must restrict the range to the branch's own commits, and the exact git form is the plan's to settle and test.
- Plan-Review bounce budget on re-entry: the segment is re-entered once per required generation; the plan must confirm the episode budget resets on entry (it is episode-scoped per `internal/shedengine/doc.go`) rather than accumulating across generations.
- An operator `goto` to Plan-Bouncer on an exempt generation skips review again rather than forcing one; accepted, since a forced review of prose-only cards is not a case anyone has asked for.
- An exempt generation can still change behaviour through a `Prosa` card on a non-source file that agents follow as rules (a rubric, a template, a stencil); D6 exempts it.
  Accepted: Webster-Review still reviews the resulting diff against the card, and the operator authored the findings and reviews the PR.
- `gate-list` runs in parallel and replaces every row's `gate`/`gate_attempts` keys in `contracts/recipes/loom-recipe.yaml` with a `gates:` list, PR-Rework's `gate: rework-plan` included.
  Whichever task lands second resolves the overlap at Publish: this task's routing and Plan-Bouncer skip-seam changes kept, written beside the `gates:` form; the `rework-plan` gate's new meaning (D5) is unaffected by the key's shape.
- Removing the append-only check removes the guard that stopped a rework session from silently rewriting built cards; D1 makes that impossible in `_lyx/plan/`, since the built cards are no longer there when the session runs.
  The residual is a session that edits the archived generation it reads despite the read-only rule; the round commit records it as a modified rename against the moved-from paths, visible in the PR.

## Acceptance criteria

- After a PR-Rework round, `_lyx/plan/` holds only the new generation: a `00-overview.md` whose Card Index lists only the new cards and whose `first_card` is one past the retired generation's highest card.
- `_lyx/loom/rework/round-<N>/prior-generation/` holds the retired plan (every entry, `archive-*/` included), Webster's run record, and the Plan-Review and Webster-Review run directories; the origin paths no longer exist; one commit records both sides.
- Two consecutive rework rounds leave `round-1/prior-generation/` holding generation 0 and `round-2/prior-generation/` holding generation 1, and `_lyx/plan/` holding generation 2 only.
- Crash-resume tests: a crash mid-archive (before `record.json` is written), a crash after the archive and before the session, after the session and before the commit, and after the commit and before the rejection is cleared each converge to one round directory and one commit, and the archive step never moves a file the session wrote.
- `planparser`: absent `first_card` behaves as `1`; a plan numbered from `first_card` with no gap validates; a gap, a card below `first_card`, and a non-positive or non-integer `first_card` each produce a finding.
- The rework gate: a whole new plan passes; a `first_card` that differs from the told number fails; glyphs in every card resolve against the current tree.
- Classification: all-`Prosa` cards on non-source files → `exempt`; any card carrying a non-`Prosa` label, including a multi-label `Prosa`+`Edit` card → `required`; a `Prosa` card targeting a `.go` file or a whole package → `required`; the class and `first_card` are in `record.json`.
- Plan-Bouncer skip seam: an exempt generation with matching `first_card` approves, commits and returns Done without spawning a judge; a `required` generation runs a normal review; generation 0 and a mismatched `first_card` never skip; a record saying `exempt` over live cards that classify `required` never skips.
- Recipe shape test: `PR-Rework.on_done` is `Plan-Bouncer`; the coverage guard and the recipe header comment agree.
- After a rework round, `lyx webster run` starts a fresh run whose state and Master prompt name only the live generation's batches; PR-Rework no longer calls `RebaselinePlanFingerprint`.
- Describe's rendered prompt names every generation's `summary.md`, oldest first.
- Stencil tests: both rubrics carry the attack-surface item and both writer stencils the matching instruction; the judge template carries the self-contained focus rule; `burler-focus-directive.md` carries the prior-round rule; the webster rubric's range section carries the rework branch; the Plan-Review rubric carries the rework answer key; both name the live generation's round (D7) and fall back to the generation-0 form without one.
- The rework integration test (`internal/loomcli/wiring_rework_integration_test.go`) drives a rework round end to end through the new route.
- `docs/overview.md`, `README.md`, `lyx loom reject` help text, the recipe header comment and `CONSTRAINTS.md` describe generations, not appended cards.

## Notes for the plan writer

- `prRework.roundFor` and `committedRejections` already key a round by rejection identity; `roundFor` today returns highest-plus-one for a round directory without `record.json`, which D14's interrupted-archive rule changes.
- `NextReworkCardNumber` reads the plan committed at HEAD, which is still the retired generation when PR-Rework starts; read it before the move.
- `NewPlanDirRotator` and `loomengine.PriorPlanBlock` are prior art for moving a plan aside and telling a session where it went.
- `websterengine/archive.go` holds the archive helpers the `--fresh` escape uses; the new primitive fits beside them.
- Plan-Bouncer's skip seam reads the round record through the same committed-file reader PR-Rework uses (`committedAnchoredReader`).

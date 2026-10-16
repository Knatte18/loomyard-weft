# Decision record — webster-fork-stencil-fixes

## Goal

Fix the five Webster fork and pattern-directive findings of the 2026-10-15 stencil intent audit, as the board entry's five sections and their operator decisions set them:
a symbol `Rename` uses the hub's structure-aware rename tool,
a fork's PATTERN departure reaches Webster-Review and the Master through its commit,
the fork spawn and completion wording matches how Webster runs,
the inline kick-start pack layout is gone,
and a task that changes a PATTERN entry declares it so no later role judges the change against the old text.

## Scope

In:

- `language.yaml` gains a `rename_tools` key, rendered into the Webster implementer body; the edit directive names the one exception it licenses.
- The plan spec's `Rename` row, its `## Quarry integration` Rename bullet and the canonical `## Rename mechanic` text, with `loom-template-plan.md`'s copy; a plan-gate check and a `begin-batch` refusal for a symbol `Rename` in a hub with no rename tool.
- The Webster-Review rubric's `Rename` check, its reading of commit messages and its check that declared PATTERN changes land.
- A `Pattern-Departure:` commit trailer, extracted by Go into a new digest field and the verify gate result the Master reads.
- Wording in `webster-body-implementer.md`, `webster-body-verify-fix.md` (its departure pointer and its "Your final action" paragraph), `webster-prefix-fork.md`, `webster-template-master.md` and the five `pattern-directive-*.md` stencils.
- Removal of the inline pack layout and retirement of `kickstart_pack_file`.
- An optional `## PATTERN changes` section in the decision record and the plan overview, checked by the discussion and plan gates, appendable by `lyx loom decision add`, and rendered into every pattern directive.
- The docs these change: `internal/languageconfig/doc.go`, `internal/websterengine/doc.go`, `internal/pattern/doc.go`, `loom-discussion-spec.md`, `loom-plan-spec.md`, `refusal-spec.md`, the config reference docs.

Out:

- `CLAUDE.md` edits get no declaration; see D5.6.
- `tools/forkaudit`'s `inline` arm stays, for reading old sessions.
- `review-inline-code`'s design; it answers the D4 decision on its own entry.
- Landing a PATTERN edit during Discussion (`discussion-lands-invariant`).
- Webster's other hard-coded Go text, such as the verify-fix fork's `lyx gate test --tags integration` line.

## Decisions

### D1 — Rename uses the hub's structure-aware tool

D1.1 `internal/languageconfig` gains a third key, `rename_tools`: an ordered list of strings, each a rule naming the hub's structure-aware rename tool for one language and how to run it, Guard-classified and outside `LocalKeys`, validated exactly as `verify_tiers` is (no empty entry, no edge whitespace, no line break), absent resolving to the empty list.
Rationale: the concrete tool comes from the hub's language setup (operator decision); `verify_tiers` already set this shape, so loader, refusal and way forward (`lyx config language` from the prime) are reused.

D1.2 This hub's `language.yaml` is left to the operator; `internal/languageconfig/doc.go` carries a Go hub's example built on `gopls rename -w`, and states that a Python-based tool needs shuttle's `deny_python` off.
Rationale: lyx hard-codes no language; the shell guard's python row refuses a python tool otherwise.

D1.3 `webster-body-implementer.md` gains a required `rename_tools` marker, filled like `verify_tiers` (bullets, or `none`), in both the fork's and the recovery strand's value maps.
Its sentence: for each symbol pair of a `Rename` card, run the hub's rename tool listed there, then the grep pass for zero old-name references and the updated doc-comment opening; a file pair is `git mv` plus surgical Edits.
Rationale: the fork prompt never named a tool, so forks renamed by hand.

D1.4 `edit-directive.md` states the exception once: a Webster implementer working a `Rename` card's symbol pair runs the rename tool its prompt names; no other role and no other card runs a script or tool that rewrites file content beyond what this directive already allows.
Bound: the exception covers only commands the `rename_tools` list names, only on a `Rename` card's symbol pairs, only in a Webster fork or recovery strand.
The bound is prose only: no audit checks which command a fork ran, and the shell guard's rows still apply to the tool.
Rationale: the tool is the edit directive's one exception and only for this part of Webster (operator decision); `PATTERN-edit-directive` keeps the directive as the rule's only statement.

D1.5 The plan spec's `Rename` row reads "the hub's structure-aware rename tool (`language.yaml` `rename_tools`) + grep verify, never text or regex replace"; the `## Quarry integration` Rename bullet drops its per-language library list for the same pointer, and keeps the quarry sentence.
Rationale: no Go or Python tool is written into the stencil or spec (operator decision).

D1.6 The canonical `## Rename mechanic` text in `loom-plan-spec.md` and `loom-template-plan.md` separates the two pair kinds: a file pair is `git mv` first, then surgical edits (the current four steps); a symbol pair is the hub's rename tool, then the grep pass.
`rename-mechanic-missing` keeps its condition.
Rationale: the current text prescribes a file move for pairs that are mostly symbol renames.

D1.7 A new plan-gate check, `rename-tool-missing`: a plan holding a symbol `Rename` pair (a file pair is exempt) in a hub whose `rename_tools` is empty is a finding, naming the way forward: set `rename_tools` with `lyx config language` from the prime, or recast the card.
It runs outside `internal/planparser`, in the shared plan-validation function the Plan-Write and Plan-Burler-Fix gate closures and `lyx loom validate-plan` all call (`loomshed`), in the default, lens-review and rework modes alike, with the language config passed in.
Its row goes in the plan spec beside the gate's other non-parser checks, not in the `internal/planparser` list.
Rationale: the tool is required (operator decision), and a plan that cannot run is caught before a batch is spent; `PATTERN-gate-self-check-parity` keeps gate and CLI on one function.

D1.8 `begin-batch` refuses a batch holding a symbol `Rename` pair when `rename_tools` is empty, naming the same way forward; `recover-batch` refuses the same way.
The refusal carries `{"rename_tool_missing": true}`, and the master stencil gains its rung: write `outcome: stuck` quoting the message and stop.
It gets its `refusal-spec.md` row and a test.
Rationale: a standalone run has no plan gate, and a list emptied after the plan gate would otherwise fail the fork and both recoveries; `PATTERN-refusal-way-forward`.

D1.9 The Webster-Review rubric's per-card mechanical check for a `Rename` group checks the outcome a diff shows: no reference to the old name left, and the symbol's doc-comment opening updated; never that a tool ran.
Rationale: a reviewer sees the diff, not the fork's transcript.

### D2 — A fork's PATTERN departure travels in its commit

D2.1 The `Pattern-Departure:` trailer is owned by a new subsection of `loom-plan-spec.md`: a fork, the recovery strand or the verify-fix fork that departs from its card because a PATTERN entry forbids the card's way records each departure as one trailer line in the departing commit's body, `Pattern-Departure: <PATTERN-entry-id> — <what differs from the card and why>`.
A departure from an entry the task declares changed (D5) cites the declared new text, never the old.
Rationale: a commit is the one output Webster-Review and Go both read, and no new report field is added (operator decision).

D2.2 The stencils point at that subsection instead of restating the shape: `webster-body-implementer.md` (every fork and the recovery strand), `webster-body-verify-fix.md` (whose "Your final action" paragraph also changes, per D2.8), `pattern-directive-orchestrator.md`'s bullet on the forks it spawns, and `pattern-directive-implementer.md`, whose "say so in your report" becomes: state the conflict and the entry where your output records its reasons, a `Pattern-Departure:` trailer for a commit you make and `## Shared Decisions` for a plan.
Rationale: an in-session fork never receives the implementer directive; `PATTERN-producer-pointer-rule`.

D2.3 `record-batch` and `recover-batch` read the trailers of the batch's own non-merge commits on the first-parent line from the batch's start head to its recorded head, and fill a new digest field `departures`: one string per trailer line, `<commit subject>: <trailer value>`, carried verbatim whatever its content; Webster-Review judges validity.
The field is informational and never changes a batch's status, like `deviations`.
`digestSummaryLine` adds `pattern departures=` when the list is non-empty, so the next fork's prior-batch line carries them, and the master's `{{.progress}}` trail renders each persisted digest's departures, so a resumed Master holds them too.
Rationale: a departure reaches the Master through Go, so every later fork inherits it (operator decision); a parent merge-in's commits are not the batch's, and a resumed session has only the trail.

D2.4 `webster-template-master.md`'s quoted digest field list gains `departures`, and the summary names each batch's departures beside its deviations.
Rationale: the Master reads only the listed fields.

D2.5 The Webster-Review rubric reads the commit messages of its range as well as the diff.
A difference from the plan that a `Pattern-Departure:` trailer explains is judged on that reason: the departure holds when the named entry does forbid the card's way, and it is a finding when the entry is unknown or does not forbid it.
Bound: a trailer excuses only the difference it names, and only when the cited entry, read in its declared new text where D5 lists it, supports it.
Rationale: without the reason the fixer reverts a correct departure into the violation.

D2.6 `webster-body-implementer.md`'s "name it in your final reply to Master" sentence goes: a failure the card did not cause is left to the plan-level verify gate.
When the plan declares a `## verify:` section, a batch gate step that fails only on failures the batch's cards did not cause does not make the report `FAILED`; the `status` paragraph says so, so the self-fix section and the status rule agree.
When the plan declares none, a batch gate failure stays `FAILED` as today.
The fork learns which case holds from a required `plan_verify` marker in `webster-body-implementer.md`, rendered from the plan's parsed `## verify:` command in both the fork's and the recovery strand's value maps, or `none`.
Bound: causation is decided by the self-fix section's two-pronged test, unchanged; a misjudged failure passes the batch and is caught by the plan-level verify gate, which runs the plan's `## verify:` command and spawns a fixer when it fails, so the catch covers what that command runs.
Rationale: operator decision.

D2.7 The report paragraph says `record-batch` reads the two report fields, not the Master.
Rationale: Go reads the report; the Master reads only the digest.

D2.8 The verify gate closure, which already walks every commit from the recorded pre-fix head to HEAD on the first-parent chain (`fixCommitRejection`), also records, at each evaluation, each `fix:` commit's SHA and subject and each `Pattern-Departure:` trailer, as `<commit subject>: <trailer value>`, into its `VerifyGateNotes`.
The notes keep each commit once, keyed by SHA, the way `addFlaky` keeps each identity once, so a commit seen in several evaluations is listed once.
The trailer read covers the range's non-merge commits only, as D2.3 does, so a clean parent merge in the range is neither listed as a fix commit nor read for trailers.
Run appends them after the wait as a `## Verify gate fix commits` section of `summary.md`, the way `VerifyGateNotes.Apply` already writes the flaky identities into `summary.md`.
Go appends the section after the Master's last rewrite of `summary.md`, and appends nothing when the notes hold no fix commit.
The master stencil's step 4 renames its own section to `## Verify gate findings`, which names only the findings from the findings file, never what the fixer's reply says; the master stencil's "never read raw fork output" rule keeps no exception.
`webster-body-verify-fix.md`'s "Your final action" no longer asks the fixer to name each commit it made: the reply names only a failure the fixer could not fix, which the Master's outcome rewrite needs.
Rationale: the Master still never reads raw fork output and a departure reaches the summary through Go (operator decision); the Master runs no git, the fixer's commits fall outside every batch range, and the Master writes the summary before the gate walks the fixer's commits, while a passing evaluation sends it nothing, so the gate's existing walk and the notes Run applies after the wait are the route.

### D3 — Spawn and completion wording

D3.1 `webster-prefix-fork.md`: once the report is written the fork's turn ends and the Master is woken by the fork's completion notification; the `await-batch` takeover sentence goes, the ban on running any `lyx webster` command stays; the banner states the spawn instruction the Master sends.
Rationale: the Master no longer runs `await-batch`.

D3.2 `webster-template-master.md`'s self-identification rule keys on the opening the Master's two spawn prompts share, "You are an implementer fork", instead of an instruction it never sends.
Rationale: one key that both the batch fork and the verify-fix fork prompts open with.

D3.3 `webster-template-master.md`'s plan-drift paragraph says the run resumes once the plan is fixed: under loom the stuck run is routed to plan review, which edits the plan; standalone, the operator edits the plan and runs `rebaseline` or `restore-plan`.
Rationale: under loom the orch never edits `_lyx/plan`.

### D4 — The inline pack layout is removed

D4.1 The inline layout goes: its three fragments, their embeds and registry rows, its render branch and the `kickstart_pack_inline` marker in both implementer value maps and the body.
Rationale: code text is never pasted into a prompt without a written reason (operator decision); the inline layout pasted code that goes stale.

D4.2 `kickstart_pack_file` is retired through the webster spec's `RetiredKeys` with `CategoryNone`, so a hub file holding it still loads and reconcile prunes it.
`kickstart_pack_per_card: true` in a hub geometry selects the per-card layout; every other built pack uses the pack-file layout.
Rationale: with one layout left besides per-card, a key that picks between file and inline has nothing to pick.

D4.3 `internal/websterengine/doc.go` and `config.go` drop the inline layout and the frozen-baseline note.
Rationale: built design lives in the package doc.

### D5 — A declared PATTERN change

D5.1 The decision record gains an optional section, `## PATTERN changes`, placed after `## Acceptance criteria` and before `## Notes for the plan writer`, its bullet shape defined in `loom-discussion-spec.md`: `` - `PATTERN-<id>` — adds: <new entry text> ``, `changes: <new entry text>` or `removes`.
`discussionparser.Validate` checks the shape, so the Discussion gates and `validate-discussion` share it: one bullet per entry, a known action, new text for `adds` and `changes`, none for `removes`.
`pattern-directive-designer.md` tells the designer to list there every entry its design adds, changes or removes, pointing at the spec for the shape.
The PATTERN edit itself lands in a Webster card, never during Discussion or Plan (operator decision).
Rationale: operator decision; optional keeps every record without a PATTERN change, and every in-flight run, valid.

D5.2 `lyx loom decision add` gains an optional PATTERN-change form that appends one bullet to the record's `## PATTERN changes`, creating the section when absent, beside its `### Added after Discussion` entry.
A writer never edits a bullet the verb appended, and the format check refuses a record that drops one, as for `Added after Discussion` entries.
Bound: the form appends only a bullet of the D5.1 shape, through the verb's existing checks and its existing `--by parent|operator|arbiter` caller guards, so only the run's parent, the operator's own shell or the run's own arbiter declares a change after Discussion; it removes and rewrites nothing.
Rationale: a PATTERN change decided at PR-Rework or by a parent after Discussion otherwise has no way into the declaration.

D5.3 The plan overview gains the same optional section, `## PATTERN changes`, carrying the record's entries.
A plan-gate check, `pattern-changes-mismatch`, flags a plan whose section is missing or lists different entries or actions when the decision record has one, and a plan section the record lacks.
It runs in the same shared `loomshed` plan-validation function as D1.7, in every mode, with the record's parsed section passed in; a darn plan, with no decision record, is not checked.
A bullet `decision add` appends after the plan gate reaches the plan's section only at the next plan gate, where `pattern-changes-mismatch` asks for it.
Rationale: the plan carries the statement (operator decision), so Webster-Review, which measures against the plan, sees it.

D5.4 Each pattern directive stencil gains an optional `pattern_changes` marker, rendered by `internal/pattern` as the task's declared changes and the rule that an entry listed there is followed and judged by its listed new text, never the old; empty when the task declares none.
The directive banners, `internal/pattern/doc.go`'s one-marker statement and the marker-set test change with it.
`internal/pattern` stays a leaf: each `pattern.Directive` caller reads the record's section through `discussionparser` at a path its geometry is told, and passes the entries in; a standalone webster run and a darn run pass none.
A record present at a told path that fails to parse refuses the spawn, naming the way forward: the operator fixes and commits the named section of `_lyx/discussion/decision-record.md` (the orch never edits the record and escalates the refusal to the operator), since no later role's write surface holds the record, confirms it with `lyx loom validate-discussion`, then re-steps the row.
Each caller's refusal gets a `refusal-spec.md` row and a test: websterengine's in `## webster`; the Bouncer's in `## shed` beside its other review-segment rows; loomengine's in `## loom`; burlerengine returns the error to the review segment's Burler-round row that spawned it, whose row goes in `## shed` beside the Bouncer's.
Bound: a declaration lets every reviewer, judge and implementer follow and judge a listed entry by the declared text, never `PATTERN.md`'s current text; it covers only the entries the record's `## PATTERN changes` lists, only through bullets of the D5.1 shape, appended after Discussion only through D5.2's caller guards, and D5.7 checks the declared text lands.
The sentence of `webster-template-master.md` that says the Master's reads stay current for the whole run because the run refuses a drifted plan is narrowed to the plan the drift check covers, and the declared changes rendered in its directive cover PATTERN.
Rationale: the judge never opens artifacts and the in-session forks read only the master's context, so rendering into the directive reaches every role; `PATTERN-leaf-packages` keeps `internal/pattern`'s import set.

D5.5 This task changes no PATTERN entry.
Rationale: `PATTERN-edit-directive` keeps the edit directive as the rule's only statement, the exception included, and `PATTERN-leaf-packages` holds by passing entries in.

D5.6 `CLAUDE.md` gets no declaration.
Rationale: no directive makes a `CLAUDE.md` rule gating, so a task that changes one meets no BLOCKING-never-downgrade conflict.

D5.7 The Webster-Review rubric checks that the branch's `PATTERN.md` and `pattern/` diff carries each declared entry's add, change or remove with the declared text; a declared entry the diff does not land is a BLOCKING finding.
The check reads the declared entries from the record's `## PATTERN changes` as rendered into the Webster-Review directive by D5.4, never from the plan's section, so a bullet appended after the plan gate is checked too.
Rationale: without it a task that declares a change and never makes it lands code every review judged against text that never reached `PATTERN.md`.

## Constraints

- `PATTERN-edit-directive`: the rename exception is stated only in `edit-directive.md`; the fork body names the tool, never restates the rule.
- `PATTERN-leaf-packages`: `internal/pattern` imports only the standard library, `lyxdirs`, `stencilstore` and `stencil`.
- `PATTERN-sole-parsers`: the record's section is parsed only by `internal/discussionparser`, the plan's only by `internal/planparser`.
- `PATTERN-told-geometry`: each `pattern.Directive` caller is told the decision record's path; no engine derives it.
- `PATTERN-gate-self-check-parity`: the new plan checks live in the one function gate and CLI share.
- `PATTERN-producer-pointer-rule`: stencils point at the spec that owns the trailer and section shapes.
- `PATTERN-refusal-way-forward`: D1.8's and D5.4's refusals name their way forward, with spec rows and tests.
- `PATTERN-stencil-ownership`: new markers and text live in the stencils, read through `stencilstore`.
- `PATTERN-tolerant-records`: the digest's new `departures` key is added under the same tolerant JSON handling as `deviations`.
- `PATTERN-fabric-git`, `PATTERN-gitrepo-client-boundary` and `PATTERN-gitexec-checked-call`: the trailer read is a read-only git read through `internal/gitrepo`, behind webster's `Git` seam.
- `PATTERN-config-layers` and `PATTERN-hub-wide-config`: `rename_tools` loads through `configengine` as a `language.yaml` key.

## Auto-mode assumptions

D1.7 and D1.8 (gate check and refusal), D2.1 (trailer shape), D2.3 (first-parent non-merge range), D2.6 (`OK` on an uncaused batch-gate failure), D2.8 (verify-fix commits and departures written by Go from the gate's notes), D4.2 (key retired), D5.1 (optional section), D5.2 (`decision add` form), D5.4 (rendered into every directive; refusal on a malformed record), D5.6 (`CLAUDE.md` out).

## Open risks

- Until the operator sets `rename_tools` in this hub's `language.yaml` (`lyx config language` from the prime), every plan here with a symbol `Rename` fails `rename-tool-missing`; the task cannot write the board's hub config.
- A hub's rename tool that the shell guard refuses, or that rewrites files outside the card's targets, fails the fork or leaves scope-guard deviations; the fix is the hub's tool choice or its shuttle switches, not a guard exception.
- A rename tool that needs a building package fails on one that does not build, and the fork falls to its self-fix bound.
- A fork may omit the trailer; Webster-Review then flags the departure as a plan mismatch, as today.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - verify-gate-fixes-heading-two-writers (class consistency, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-1-review.md; fixer report: _lyx/reviews/plan/round-1-fixer-report.md.
  - bouncer-refusal-row-misstates-outcome (class consistency, severity MEDIUM, open in rounds 1)
  - d5-4-bound-stated-by-no-card (class scope, severity MEDIUM, open in rounds 1)
  - test-helper-resign-arrows-cards-9-13 (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- `language.yaml` with `rename_tools` loads; a local override holding it, an empty entry or one with a line break is refused with the `lyx config language` way forward; an absent key loads empty.
- The implementer body renders `rename_tools` as bullets, or `none`, for the fork and the recovery strand.
- The plan gate flags `rename-tool-missing` for a symbol `Rename` pair under an empty list, in Plan-Write, Plan-Burler-Fix and `validate-plan` alike, and not for a file pair or a non-empty list.
- `begin-batch` and `recover-batch` refuse a batch with a symbol pair under an empty list, with the way forward; a batch with only file pairs proceeds.
- A batch whose own commits carry `Pattern-Departure:` trailers records them in `departures`, in `record-batch` and `recover-batch` alike; a batch with none records no field; the next fork's prior-batch line and a resumed Master's progress trail carry them.
- After a fix the verify gate passes on, `summary.md`'s `## Verify gate fix commits` section names each `fix:` commit's SHA and subject and each `Pattern-Departure:` trailer, written by Go after the wait; a fix with no trailer names no departure.
- A plan with a `## verify:` section renders it into the implementer body's `plan_verify` marker, and a batch gate failure the cards did not cause reports `OK`; a plan with none renders `none`, and the same failure reports `FAILED`.
- A trailer in a commit before the batch's start head, or in a parent merge-in's commits within the range, is not attributed to the batch; a malformed trailer line is carried verbatim.
- A webster run with `kickstart_pack_per_card: false` renders the pack-file layout; a hub `webster.yaml` still holding `kickstart_pack_file` loads, and reconcile prunes it.
- A decision record with a well-formed `## PATTERN changes` passes `validate-discussion`; an unknown action, a missing new text for `adds` or `changes`, or text after `removes` fails; a record without the section passes; a record that drops a bullet `decision add` appended fails.
- `lyx loom decision add` with the PATTERN-change form appends the bullet, creating the section when absent.
- The plan gate flags `pattern-changes-mismatch` for a missing, extra or differing plan section, and is clean when both match or neither has one.
- Each pattern directive renders the declared changes when the record has them, renders as before when it has none or no record exists, and refuses the spawn when the record fails to parse, naming the operator's record fix, `validate-discussion` and the re-step.
- The Webster-Review rubric flags as BLOCKING a declared entry whose add, change or remove with the declared text the branch's `PATTERN.md` and `pattern/` diff does not carry, and raises nothing when the diff carries every declared entry.
- A `## PATTERN changes` bullet `decision add` appends after the plan gate is rendered into every later directive and checked by D5.7's landing check.
- `lyx loom lint-comments`, `lyx gate test ./...` and `lyx gate test --tags integration ./...` pass.

## Notes for the plan writer

- `rename_tools` follows `verify_tiers` end to end: `languageconfig` load and validation, `VerifyTiersValue`'s rendering, and the two webster value maps that already carry `verify_tiers`.
- `Digest.Deviations` and `digestSummaryLine` are the model for `departures`; the batch state's `StartSHA` and `RecoveryStartSHA` bound the range, and webster's `Git` interface has test fakes the trailer read should extend.
- `internal/shedadapters/bouncer.go` already imports `discussionparser`; `burlerengine`, `websterengine` and `loomengine` callers of `pattern.Directive` gain it.

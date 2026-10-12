# Twill for Plan-Write at opus[medium] — decision record

## Goal

Build the twill producer for Plan-Write that the benches recommend: the plan writer forks itself once per lens of a tuned four-lens fan before it writes, writes and self-checks the plan, marks it ready, takes the forks' findings in, and rejects a BLOCKING or MEDIUM finding only with a cited code read.
Add a deterministic `lyx loom validate-plan` check that refuses a name a plan declares which the fabric-vocabulary scan would refuse, covering the declared test functions behind the bench's shared vocabulary miss.
Select twill at `opus[medium]` for this repository's own runs.

## Scope

In:

- A `loom.yaml` key `plan_producer`, `single` (default) or `twill`, read by Plan-Write only.
- Under `twill`, the Plan-Write prompt carries the twill directive between Step 2 and Step 3, and the spec authorizes fork subagents.
- A `twill-plan` fan in the embedded `burler.yaml` template with four new lenses, texts in D8.
- The shared twill directive stencil gains the host values Plan needs; the Discussion host fills them with today's text.
- A plan ready marker, removed before every fresh Plan-Write spawn.
- A tracked lens-review file in the plan directory.
- A new plan check, `fabric-vocabulary-name`, backed by the fabric-vocabulary predicates moved out of `internal/lyxcwd/enforcement_test.go` into a new production leaf package that the enforcement test also calls.
- This repository's tracked `_lyx/config/loom.yaml`: `plan_producer: twill`, `plan: opus[medium]`.
- Docs and PATTERN background updates the above force.

Out:

- PR-Rework's plan writer and Plan-Burler-Fix: neither forks.
- `plan_max_bounces` tuning against twill (the board's first Open item): it is a measurement after landing.
- The shared "wrong parent directory" miss (the board's second Open item): no lens or check is designed for it here.
- Checks for PATTERN scans other than fabric vocabulary, such as the `_lyx`/`.lyx` literal scan or the retired hub suffix: no bench miss traces to them, and a plan's declared names do not carry path literals.
- Test row and subtest names, which a plan writes only in prose (D9's bound).
- `card-structure` and `pattern-fit` in the Plan fan; the existing `plan` fan used by `plan_fan` is unchanged.
- Any change to Discussion twill's procedure or lenses; D4's reviewed-files rendering fix is the one change to its rendered prompt.
- Folding `plan_producer` into a seats axis: `seats-replaces-singlellm` owns that.

## Decisions

### D1 — `plan_producer` key, `single` default

`loom.yaml` gains `plan_producer`, accepting `single` and `twill`; an empty or absent value reads as `single`.
Any other value fails config load with an error naming both values, and the refusal spec gains its row.
The embedded template defaults to `single`, so a hub that does not opt in renders today's plan prompt byte for byte.
Rationale: a selection key like `discussion_producer`, which the board asks for.
An absent key takes the template's `single` through `configengine`'s in-memory fill of a present file's missing template keys;
an explicit empty value reads as `single`, unlike `discussion_producer`, which refuses it.

### D2 — Twill is a spec variant of the single writer

Under `twill`, `PlanSpec` renders the directive into a new optional marker of `loom-template-plan` placed between Step 2 and Step 3, and sets `ForkSubagents`.
The row, gates, output file (`00-overview.md`), timeout, resume identity and plan-directory rotation stay `single`'s.
Fork authorization follows Discussion's: with `claude_deny_agent_tool` true the Agent hook admits only `subagent_type: "fork"`; with it false no Agent hook is installed, as for `single`.
Bound: forks run on the writer's model, read-only and without the Agent tool by stencil; the plan gates bound what lands in the plan.
A fork giving up after 15 empty waits of up to five minutes each, against this repository's 120-minute plan timeout, leaves the plan without its lens review, recorded in `lens-review.md`; the Plan-Review segment and the plan gates still run on it.
Rationale: Discussion twill showed the variant needs no new engine; Plan-Write is gated, so its run already waits on outstanding forks before counting an arrival.

### D3 — Fork spawn before any plan file

The directive sits between Step 2 and Step 3.
The Plan host's spawn point says:

- this lens review is the one review step the writer runs on its own plan, overriding the template's "no review logic of your own" for this procedure;
- once Step 2's reading is done, read the rubric, state the cards to be written in a text reply, one line each (number, slug, the decisions it carries), and spawn the forks in the same turn;
- write no file under the plan directory before the Agent call;
- add to every fork's prompt, after the boilerplate and before its lens text, the line "Report defects only, never a passed check. Give each finding's evidence as the `path:line` or glyph read."

Rationale: round 1's writers spawned late; round 2's splice and guard made every writer spawn on time.
The template's opening sentence stays, so `single`'s prompt keeps its bytes.

### D4 — Shared directive stencil, three new host values, one rendering fix

`contracts/stencils/twill/twill-directive.md` stays the one procedure for both hosts.
It gains three required host values:

- the spawn point, the text opening the procedure (Discussion: today's "When the design is settled and before you write any output file, run this procedure once:");
- the rejection rule in step 6 (Discussion: today's "Fix the record, or reject the finding with a one-line reason.");
- the severity rule in the fork boilerplate's last line (Discussion: today's "a severity (`BLOCKING`, `MEDIUM` or `LOW`)").
  The marker replaces only that parenthesised phrase, so Discussion keeps its bytes, and every host's value reads as a noun phrase in that slot: "each with <value>, the section or decision it concerns, …".

The reviewed files become one host-rendered phrase instead of a bullet list rendered inside backticks, which today breaks the boilerplate's "read the files under …" line.
Discussion's rendered prompt changes only on that line.
Rationale: the procedure, bounds and boilerplate are host-neutral; the three values are where the plan bench's text differs, and none is host-neutral (Discussion's `board-fit` and `leanness` findings have no code to cite).
One stencil keeps the two hosts' procedures from drifting.

### D5 — Plan host values

- Reviewed: the plan under the plan directory, `00-overview.md` and every card file its Card Index names.
- Rubric: `loom-rubric-plan-review`.
- Rejection rule: fix the plan, or reject the finding; a rejection of a BLOCKING or MEDIUM finding cites the code read to reject it, a `path:line` or glyph opened in this session and what it shows against the finding; such a finding the writer cannot refute from code it has read is fixed; keeping a waiver, a deferral or an unplanned test against such a finding is a rejection and needs the same citation; a LOW finding is rejected with a one-line reason.
- Severity rule: a severity (`BLOCKING` when the plan as written fails its verify or ships a wrong behavior, `MEDIUM` when a decision, criterion or branch goes unreached or untested, `LOW` otherwise).
- Format step: Step 3 and Step 4; self-check step: Step 5; self-check command: `lyx loom validate-plan`.
- Disposition place: the lens-review file (D7).

The rejection and severity rules, and the line D3 adds to every fork's prompt, are rendered as written here, since the bench file they come from is unreadable to the plan writer.

The Plan template renders no write-scope paragraph: it has no write-scope section to widen, and the directive's own step 4 tells the writer to create the marker and its parent directory.
Rationale: the round-2 bench text, which the build decision adopts; the board scopes the citation rule to BLOCKING and MEDIUM.

### D6 — Plan ready marker

The marker is `00-overview.md.ready` under `.lyx` at the subpath mirroring the plan directory, built in `internal/loomengine` beside `DiscussionReadyMarker` from `lyxdirs` and `planparser`'s plan-directory segment, never a second spelling of it.
Before every fresh Plan-Write spawn, Go removes it as its own step, unconditionally, ahead of the plan-directory rotation, whose no-stale-files path returns early; a missing marker is success.
Attaching to a live writer or relaunching a dead one keeps it.
It is not an output file, so the completion check and gate list are unchanged.
Rationale: same lifecycle as Discussion's marker; a stale marker would wake a later spawn's forks on an old plan.

### D7 — Tracked lens-review file in the plan directory

The writer records the lens review in `lens-review.md` in the plan directory: one line per finding with its lens and `fixed` with what changed or `rejected` with the citation or reason, naming any fork that returned no report or a cut-short review.
Its file name is declared once in `planparser`, and the plan-directory enumerations treat it as follows:

- `planparser`'s index check admits it beside `00-overview.md` and the amendments file, so `index-file-mismatch` does not count it as a card;
- webster's plan fingerprint excludes it, as it excludes the amendments file, so it is no part of plan identity;
- the Plan-Write prior-plan block omits it from the files it names to a re-run writer;
- Plan-Write's `archive-*` rotation and PR-Rework's round archive move it with the generation, and the existing plan commit commits it.

Bound: the name is admitted under every producer and its content is never parsed or validated; no reviewer or fixer prompt points at it.
Rationale: the board tunes the lenses later from findings a lens should have caught, which needs the lens record to survive pair teardown; the plan directory is the generation's durable home (`PATTERN-plan-generation`).

### D8 — `twill-plan` fan in the burler template

The embedded `burler.yaml` template gains four lenses with these texts and a fan `twill-plan` listing them in this order:

- `criteria-coverage`: Focus on the plan against the decision record: list every decision and every acceptance criterion, and for each find the card whose Intent carries it and the planned test row that would fail if it were broken. A criterion the plan argues holds without a test, waives or defers is a finding, and so is a decision, failure exit or branch the record names that no row reaches. Apply the Plan-Review rubric's "Fidelity to the answer key". Report anything else you notice too — emphasis, never exclusion.
- `row-reachability`: Focus on whether each planned test row can do what the card says: open every test function a card extends or names, and the fakes, fixtures and helpers it runs on, and check that the row's input reaches the branch it claims (the fake can produce that state at that point of the sequence), that its assertion would fail if the code were wrong, and that each new fake field or helper the row needs is listed on the card. Apply the Plan-Review rubric's "Reachability of a prescribed refusal" and "Verify coverage". Report anything else you notice too — emphasis, never exclusion.
- `forced-edits`: Focus on what each change forces outside the members a card names: for every member a card changes, find its callers, the tests that assert its output or wording, and the comments, doc files and stated reasons (such as a `//testtiming:keep` reason) that describe it, and report each one the change makes wrong that no card lists. Check that each `Create` handle's package is the package clause of the file the Intent puts it in, an external `_test` package included, and that each listed glyph is the one `lyx quarry` gives. Apply the Plan-Review rubric's "Member glyphs on an `Edit` card" and "The re-sign arrow". Report anything else you notice too — emphasis, never exclusion.
- `test-economy`: Focus on test economy: for every new top-level test and every new row, open the test file it lands in and the existing tests of the same function, and report a new top-level test that should be a row of an existing test, a row that repeats a behavior an existing test already reaches, and a new top-level test that its package's row in `cmd/lyx/testdata/test-budget.yaml` does not budget. Apply the Plan-Review rubric's "Test economy". Report anything else you notice too — emphasis, never exclusion.

The lens texts are rendered as written here, since the bench file they come from is unreadable to the plan writer.
The fan name is fixed in `internal/loomengine`; redefining it in a worktree's `burler.yaml` changes the lenses.
Bound: a redefined fan changes only the lens texts the forks review through; the fan resolver refuses an empty fan, one past its maximum size and one naming an undefined lens, and the forks stay read-only per D2.
An unreadable `burler.yaml` or an unresolvable `twill-plan` fails the Plan-Write spawn with an error naming the fan, `burler.yaml` and the way forward, each with a refusal-spec row.
Discussion's fan-resolution code is shared, parameterised by fan and row name.
Rationale: same seam as `twill-discussion`; the board fixes the lens set.

### D9 — `fabric-vocabulary-name` plan check

A `planparser` function, appended by the plan index to its format validation beside `card-fabric-reference`, so both plan gates, `lyx loom validate-plan` and dispatch report it.
Names checked: the member of every `Create` handle and every `Rename` pair's `New` handle; a handle that fails to parse is skipped, since `handle-malformed` or `handle-name-failed` already reports it.
For a `Type.Method` member, the bare-token rule reads the whole member and the test-prefix and host-identifier rules read its last segment.
A name fails when:

- it contains a bare fabric side token (case-insensitive substring, as the scan matches) and its package is outside the vocabulary owner set, and either the package is under `internal/` or a doc root of the scan, or the member is a test function (`Test`, `Benchmark`, `Fuzz` or `Example` prefix); `internal/configsync`'s identifier rule applies, so a configsync handle fails too;
- or it is one of the scan's host-geometry identifiers, case-insensitive, as a member without a test prefix under `internal/` or `cmd/`.

A name whose landing file the plan text cannot determine (a member without a test prefix outside `internal/` and the doc roots) is not checked for the bare token.
The scan's package-name carve-out for files importing the vocabulary owner's name package does not apply: a declared member is never that package name.
One blocking finding per name, attributed to the card, naming the member and `PATTERN-fabric-vocabulary`; its way forward is to rename the member.
It runs under any `language:`.
Bound: it only refuses.
It misses a name spelled only in a card's prose, such as a test row or subtest name, or built at implementation time, which the verify gate still catches.
It may refuse a host-geometry helper declared in a `_test.go` file under `internal/`, which the scan admits; the rename it asks for is harmless.
Rationale: declared names are the plan's only fixed spellings, and the bench's vocabulary miss was a declared test name.

### D10 — Vocabulary predicates move to a production leaf package

The bare-token predicate with its package-name stripping (`bareVocabularyTokenIn` and its wrapper), the host-geometry identifiers, the owner set with configsync's narrower rule, and the scope roots move from `internal/lyxcwd/enforcement_test.go` into a new stdlib-only package, which both `TestEnforcement_FabricVocabulary` and the D9 check call.
The allowlist, skipped roots, the walk and the import rule stay in the test.
The package spells the tokens it bans, so it joins the owner set.
Bound: the owner-set row exempts the whole package from the bare-token rule, so the package holds only the vocabulary predicates and their data, imports only the standard library, and its leaf row says so.
Rationale: production code cannot import a test file or a `testkit` kit, and one source keeps the plan check and the scan from drifting.

### D11 — This repository selects twill at `opus[medium]`

The tracked `_lyx/config/loom.yaml` sets `plan_producer: twill` and `plan: opus[medium]`.
Rationale: the operator's build decision.

### Added after Discussion (parent, 2026-10-12): Plan severity rule fits D4's slot

Decision: Accept the round-2 fix: D5's severity rule ends at the closing parenthesis, the evidence definition lives in D3's per-fork line, and the acceptance criterion pins the exact rendered boilerplate line.

Rationale: Verified in the record (D3 line 62, D5, criterion): the value now reads as a noun phrase in D4's slot and a test can pin the line byte for byte; the round-2 LOW findings F2 and F3 are fixed too, so the open ledger keys are closed in the record.

### Added after Discussion (parent, 2026-10-12): D11 lands after deploy, outside the cards

Decision: D11 (plan_producer=twill, plan=opus[medium] in the hub's loom config) is not a plan card. After this task lands and deploys, the orch runs lyx config loom --set plan_producer=twill --set 'plan=opus[medium]' in the hub's parent worktree, which commits through fabric sync.

Rationale: _lyx/config/loom.yaml sits on the records side; no card commit covers it, Webster treats writes under _lyx as outside the tracked tree, and the prod binary refuses the key until the new binary is deployed.

## Constraints

- `PATTERN-stencil-ownership`: the directive and the plan template are read at call time from the told stencils directory.
- `PATTERN-told-geometry`: `loomengine` builds the marker and plan paths from the told location; the new package derives no path.
- `PATTERN-lyxdirs-single-declarer`, `PATTERN-durable-vs-ephemeral-state`: the marker is built through `lyxdirs` under `.lyx`; the lens-review file is tracked under `_lyx`.
- `PATTERN-sole-parsers`, `PATTERN-plan-generation`: the lens-review name is declared in `planparser` only, and the file lives in the generation.
- `PATTERN-refusal-way-forward`: each new refusal (D1, D8) names its way forward, lands its `refusal-spec.md` row and a test.
- `PATTERN-fabric-vocabulary`: the owner set gains the new package; no other new production file spells a fabric side token, including `loom-plan-spec.md`'s new row and the lens texts.
- `PATTERN-leaf-packages`: the new package gets its row, standard library only.
- `PATTERN-gate-self-check-parity`: D9 joins the plan index's format validation, which both plan gates and `validate-plan` already share; no new gate or verb.
- `PATTERN-no-denied-recovery`: the directive names `lyx shuttle await` and `lyx loom validate-plan`, both allowed to agents.
- Docs in the same commit: `internal/twilldirective`, `internal/loomengine` and the new package's `doc.go`, `docs/overview.md`'s loom config and Plan-Write lines, `contracts/specs/loom-plan-spec.md`'s numbered check list, `planglyph`'s doc where it lists the appended checks, and the two PATTERN background files.

## Auto-mode assumptions

- A1: D1 and D11, only this repository opts in.
- A2: D3, the bench's "state the cards" line is kept.
- A3: D7, the lens review is tracked, not untracked as in the bench.
- A4: D4, Discussion twill is not strengthened.
- A5: D9, declared member names only.
- A6: D8, the fan name is fixed.

## Open risks

- `config-layers-run-settings` is in flight and changes how `_lyx/config/*.yaml` layers resolve; D1's new key and D11's edit to the tracked `loom.yaml` may need rebasing onto its layer model.
- `seats-replaces-singlellm` and `plan-multi-llm` both move Plan-Write off the single producer; each then carries `plan_producer: twill`, the ready marker and the lens-review file over to its own producer.
- Landing warns when a task branch changes a per-worktree config file; D11 triggers that warning once, by intent.
- D9's test-prefix heuristic misses a helper without a test prefix declared in a `_test.go` file outside `internal/`.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - plan-severity-rule-marker-slot-fit (class design, severity BLOCKING, open in rounds 1, 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- `plan_producer: twill` loads; an empty or absent key loads as `single`; an unknown value fails with an error naming `single` and `twill`.
- With `single`, the Plan-Write prompt is byte-identical to today's and the spec has `ForkSubagents` false.
- With `twill`, the prompt carries the directive after Step 2 and before Step 3 with the four `twill-plan` lenses in order, the plan rubric path, the ready marker, the plan host's spawn point, rejection rule and severity rule, and the spec has `ForkSubagents` true.
- The twill Plan prompt's fork boilerplate last line reads exactly "Return your findings only as your final message, each with a severity (`BLOCKING` when the plan as written fails its verify or ships a wrong behavior, `MEDIUM` when a decision, criterion or branch goes unreached or untested, `LOW` otherwise), the section or decision it concerns, a one-line summary and the evidence."
- The Discussion twill prompt differs from today's only on the reviewed-files line, which now names both discussion files as readable paths.
- A missing host value or an empty lens list fails the directive render with an error naming the marker.
- An unreadable `burler.yaml` and an unresolvable `twill-plan` each fail the Plan-Write spawn with the refusal-spec text.
- A stale plan ready marker is removed before a fresh Plan-Write spawn, including one whose plan directory holds no stale files; it is kept when attaching to a live writer and when relaunching a dead one.
- `lens-review.md` in the plan directory raises no `index-file-mismatch`, changes no webster plan fingerprint, is not named in the prior-plan block, and is committed and rotated with the plan.
- `fabric-vocabulary-name` refuses: a `Create` handle `Test…` with a side token in a package outside `internal/`; a member with a side token and no test prefix under `internal/`; a member with a side token and no test prefix in a package under a doc root such as `cmd/`; a configsync member with a side token; a `Rename` `New` handle with a side token; a host-geometry identifier under `internal/`, including as a method's last segment; a host-geometry identifier under `cmd/`; a `Type.Method` member under `internal/` whose type segment alone carries a side token.
- It passes: a side token in an owner-set package other than configsync; a member without a test prefix outside `internal/` and the doc roots; a host-geometry identifier in a package outside `internal/` and `cmd/`, such as one under `tools/`; a name with no token; a handle that fails to parse, with no second finding.
- `TestEnforcement_FabricVocabulary` and its fixtures pass unchanged in behavior, calling the moved predicates.
- The embedded burler template resolves `twill-plan` to the four lenses.

## Notes for the plan writer

- Discussion twill landed in `50cf6a425`: `internal/loomengine/twill.go`, `discussion.go`, `config.go` (marker), `internal/loomcli/wiring.go` (`PrepareDiscussionSpawn`), `internal/shedrecipe/entries_planwrite.go` (rotator as fresh-spawn preparation).
- Plan-directory enumerations D7 touches: `knownNonCardFiles` in `internal/planparser/validate.go`, `planFileNames` in `internal/websterengine/fingerprint.go`, `stalePlanRotation` and `NewPlanDirRotator` in `internal/loomshed/planwrite.go`.
- The scan's predicates: `bareVocabularyTokenIn`, `hostGeometryIdentifiers`, `fabricVocabularyOwners`, `failsBareVocabularyCheck`, `vocabScanDocRoots` in `internal/lyxcwd/enforcement_test.go`.
- `card-fabric-reference` is the precedent for a `planparser` function the plan index appends.

# Discussion-Write on MultiLLMProducer: a chair writes, advisors answer

## Goal

Discussion-Write runs as a table of seats: the writer is the chair, and one or more advisors study the task, the board, the code and PATTERN in parallel and answer the chair's design questions live over the session message channel.
The chair alone writes the decision record and the support log, ends the step and takes the gates, exactly as the single agent does today.
The operator sets the chair's model and the advisors' models in `loom.yaml`, and an empty advisor list runs the chair alone.

## Scope

In:

- The Discussion-Write row runs on `MultiLLMProducer`; the single-agent spec path for the discussion role is retired, with zero advisors as the single-agent case.
- A Go seat table for the discussion step in `internal/loomengine`, told to the registry entry through a per-call closure, replacing the discussion spec closure.
- Two seams on `MultiLLMProducer`: a per-call table source and a fresh-spawn preparation.
- One `seatengine.Table` field naming the table values that may be empty, for the two optional directives.
- A chair stencil (the existing discussion stencil, adapted) and a new advisor stencil, registered as opening stencils of their roles.
- One `loom.yaml` key, `discussion_advisors`, a list of model-specs, one advisor per entry.
- An advisor-aware autonomous `mode_rules` text.
- Docs: `docs/overview.md`'s Discussion producer paragraph and `loom.yaml` key list, the package docs of `shedadapters` and `shedrecipe`, the recipe spec's `MultiLLM` paragraph and the refusal spec's loom section.

Out:

- Burler on `MultiLLMProducer` (task `burler-multi-llm`) and Plan-Write on it (note `plan-multi-llm`).
- `creel`, the even reed layout (`reed-layout-shares`) and any change to how seats' strands are laid out.
- Advisors messaging each other, a parent-review of advisors, or any advisor text in the decision record.
- A migration of a run parked on Discussion-Write under the old binary: it is restarted, following the recipe header's precedent for in-flight runs.
- Any change to the Discussion-Review perch, the parent-review gate's verdict verbs, `lyx loom decision add`, or the discussion gate's checks.

## Decisions

### D1. One producer path: the Discussion-Write row runs on `MultiLLMProducer`, and zero advisors is the single-agent case

The `DiscussionWrite` registry entry builds a `MultiLLMProducer` behind the existing commit decorator, with the row's gates on the chair.
`loomengine.DiscussionSpec`, its prompt composer and the single-agent prompt path are removed; the chair's prompt is composed by `seatengine` from the chair stencil like every seat's.
The row name `Discussion-Write` and the engine name `DiscussionWrite` stay.

Rationale: two prompt paths for one role drift, and the operator's "step down to opus with a sonnet advisor" is a config change, not a code path.
Keeping the engine name keeps the row-to-engine table, the cross-consumer coverage guard and the gate-quiescence writer-engine list as they are; the engine name names the row's contract, not the adapter behind it.

### D2. The seat table is built in Go in `internal/loomengine` and told to the entry through a per-call closure

`internal/loomengine` gains a table builder that returns a `seatengine.Table` from the worktree location, the stencils directory, the parent name, the loom config, the model registry and the slug:

- Role prefix `discussion` (the existing role constant), segment discussion, timeout `discussion_timeout_min`, interactivity `discussion_interactive`.
- Table values: `slug`, `decision_record_path`, `support_log_path`, `mode_rules`, `pattern_directive` (the designer variant) and `friction_directive` (the interview variant, swallowed to empty on a read failure as today).
- Optional table values: `pattern_directive` and `friction_directive`, which are empty when PATTERN is inactive, when Tier 2 is off and on a friction read failure.
- The chair seat: the chair stencil, the `discussion` model, outputs the decision record and the support log, inputs every advisor's notes file, skills `scribe:prose` and `scribe:conversation`.
- Advisor seat `n`: the advisor stencil, the `n`-th entry of `discussion_advisors`, output `_lyx/discussion/advisor-<n>.md`, skill `scribe:prose`.

`shedrecipe.Env.DiscussionSpec` is replaced by `Env.DiscussionTable`, a closure the entry evaluates once per `Call`, so stencils and directives are read at call time as PATTERN-stencil-ownership requires.
The entry keeps requiring the commit seam and the seat runner, and keeps the parent-review preparation wired exactly as today when the gate list holds an enabled `parent-review` entry.

Rationale: the seats carry per-run values (slug, paths, mode rules, directives) that a static recipe `tokens` map cannot hold, the same reason the entry never used the generic `SingleLLM` row; a Go table is one of the two seat declarations the producer task allowed.
The entry's dedicated closure follows the precedent of every writer row.

### D3. `MultiLLMProducer` gains a per-call table source and a fresh-spawn preparation that returns only an error

A second constructor takes a table source closure and a `prepareFreshSpawn func() error`; the existing constructor keeps its signature and wraps a constant table with no preparation.
`Call` order on the fresh path is probe, then prepare, then archive, then run, as in `SingleLLMProducer`; the preparation never runs on the resume path.
The preparation carries no prompt amendment, so `seatengine` gains no field for one; the Discussion-Write entry wraps `PrepareRound` and drops its round.

Bound: the preparation is the only hook before the archive, it runs only after the probe found no live chair and stopped every live advisor, and it changes no seat's prompt.
A preparation error returns an error without archiving or starting any seat.
The probe sees only `discussion-chair` and `discussion-advisor-<n>` strands, so the bound does not cover a live writer of the old binary; the Constraints bullet on in-flight runs closes that case.

Rationale: the parent-review store's `PrepareRound` must run on each fresh spawn as it does today, and the table must be rebuilt per call so stencils are read at call time.
This task's only preparation returns an empty amendment, so an amendment path would be built for no caller.

### D4. One config key: `discussion_advisors`, a list of model-specs, one advisor per entry, empty legal

`loom.yaml` gains `discussion_advisors`, read as a model-spec list with the same scalar-or-sequence shape as `review`.
Before any entry is parsed, the load normalises an absent key, null, the empty sequence, the empty scalar and a sequence holding one empty string to the empty list, as the per-segment keys' unset rule does; the scalar `""` and the sequence `[""]` load identically, so they cannot be told apart.
Every entry of what remains is parsed and resolved at load like `discussion`; an entry that is empty, fails to parse or fails to resolve is a config refusal naming the key, the entry index and the way forward, so an empty entry beside another entry is refused.
The empty list means no advisors, unlike the review lists, whose empty value is refused.
The `discussion` key stays the chair's model; the template default adds `discussion_advisors: [sonnet[high]]` and leaves `discussion` unchanged.

Bound: the normalised list's length is the advisor count, for every spelling the load accepts; there is no separate count key.

Rationale: the operator named the two models per seat, and one list gives both the count and the models without a second key; the template default turns advisors on because the operator judged the protocol worth the tokens.

### D5. One advisor brief, repeated per seat: a scout that studies first and answers with evidence

The advisor stencil, `loom-template-discussion-advisor`, carries the parent and edit directives, the advisor seat directive, the pattern directive and the markers `slug`, `decision_record_path`, `support_log_path` and `output_files`.
Its instructions:

1. Read the board entry for `slug` and every entry that touches it, PATTERN with the background files that touch the task, and the code at module-boundary altitude.
2. Write the notes file as soon as the study is done, before any question arrives: the module boundaries the task falls under, the PATTERN entries and board entries it touches, and any conflict seen.
3. Answer every question from the chair over the session message channel, naming the evidence (a path, a board entry, a PATTERN entry) and a recommendation, and say so when the code does not answer.
4. Send the chair an unprompted message when a design message from the chair, or the draft decision record it may read, conflicts with a PATTERN entry or a board entry.
5. Write only the notes file, message only the chair, never the operator or the parent; everything else in the worktree is read-only, and git is never mutated.
6. Keep answering until the chair ends the step; the strand is stopped by lyx.

With several advisors the chair's first message to each names the area it owns; the brief is the same for every seat.

Bound: item 5 is stencil text, so an advisor that ignores it can do what the single writer could when ignoring its own fence, and no more.
The agents' settings denies hold for it as for every spawned session, so a denied git or deletion command stays denied.
It can still edit the two discussion files or add files under `_lyx/discussion/`, and the whole-directory discussion commit carries them; edits to the two files still pass through the chair's self-check, the chair's gate, the parent review and the Discussion-Review perch before Plan reads the record.
It can still edit other worktree files; the discussion commit stages only `_lyx/discussion/`, so nothing in this step commits them.
Not bounded: an advisor edit to the decision record that passes the gate and both reviews reaches Plan; no check here compares the record's author to the chair, and the plan decides whether one is needed.

Rationale: the seat engine needs every advisor to write an output, and a study written early is useful to the chair as an input whether or not questions follow.

### D6. The chair stencil is the existing discussion stencil, with the chair seat directive and an advisors step

`loom-template-discussion` stays the chair's stencil and keeps every fenced claim it carries today.
It gains the chair seat directive as an include, and its interview step says how advisors take part:

- After Step 2, the chair sends each advisor its first batch of design questions, and assigns each advisor an area when there is more than one.
- In autonomous mode, every interview batch goes to the advisors before the chair settles it; a pick made with no usable answer stays an auto-pick.
- Advisor reliance has two records with one identifier, the advisor's strand name as the chair's prompt lists it: in both modes, `## Interview` carries each advisor answer the chair relied on, under its turn; in autonomous mode, the question ledger marks a pick that rests on such an answer as advisor-informed, naming the same strand.
- In interactive mode, batches go to the operator in the pane as today and advisors supply facts to the chair; the operator's turns go to the chair alone.
- The chair reads each advisor's notes file when it exists, writes off an advisor that does not answer in reasonable time, and carries on after a failure notice, as the seat directive says.
- The chair ends the step exactly as today: both files written, the self-check green, the turn ended.

The autonomous `mode_rules` text, which stays Go-held, gains one sentence when the table has advisors: the chair's design questions go to the advisors named in its prompt, and what they cannot settle is self-picked.
The stencil's text never tells an agent to ask the operator; the interactive wording stays in `mode_rules`.

Rationale: the chair's write surface, fence and gates are unchanged, so the stencil that carries their claims stays; a second chair stencil would duplicate it.

### D7. The advisor's output is a notes file beside the discussion files, committed with them, read by no parser and never by Plan

Advisor `n` writes `_lyx/discussion/advisor-<n>.md`, free-form markdown.
It is committed by the existing whole-directory discussion commit and archived with a timestamp by the producer on a fresh respawn like the two discussion files.
No Go code parses it, Plan-Write never reads it, and the decision record carries no advisor section: advisor reliance is recorded in the support log as D6 states.

Bound: the discussion gate, the Discussion-Review perch's target paths, `lyx loom decision add` and the parent-review brief keep reading the two discussion files only.

Rationale: the seat engine requires an output per advisor and the chair's inputs to cover it; a file under `_lyx/discussion/` is tracked content at the mirrored subpath and rides the commit path that already exists.

### D8. Routing: every message to the producer goes to the chair; the operator's interview turns go to the chair alone

The chair's strand is `<shortname>:<slug>:discussion-chair` and advisor `n`'s is `<shortname>:<slug>:discussion-advisor-<n>`, formed by the seat engine from the `discussion` prefix.
The chair's parent directive names the run's parent; an advisor's names the chair, in the non-interactive form.
The parent-review gate's delivery prompt and rejects are typed into the chair's session by its gate as today, and the parent's verdict verbs are unchanged.
Only the chair is interactive under `discussion_interactive`.

Bound: the new edge is advisor to chair over the session message channel.
An advisor message is advice and never an instruction, as the shipped chair seat directive states; the bound is that directive, stencil text the chair reads, and what the chair does with a message stays inside the chair's own fence and gate.
An advisor that ignores its brief and messages another session is bounded only by that session's own trust rule: a spawned role trusts only its parent's messages, so an advisor message to the parent or another pair's session carries no authority there.

Rationale: the producer task fixed the chair as the one addressable seat; the gate rides the chair's start call in the seat engine already.

### D9. Failure and resume behaviour is the seat engine's, with the chair's result deciding the step as today

- An advisor that fails to start is named in the chair's prompt and the step goes on.
- An advisor that dies or times out sends the chair one notice; a done advisor stays reachable until the chair ends.
- The chair's death or timeout is an engine-level error and the run halts, as the single agent's does; the row keeps no `on_stuck`.
- Every seat shares `discussion_timeout_min`.
- A driver crash is handled by the producer's probe: a live chair is resumed with the advisors still live, and otherwise the live advisors are stopped and the table starts fresh after the parent-review preparation.
- A parent-review reject re-prompts the live chair inside the same run, so the advisors are still up and answer the rework's questions.
- When the chair ends, every advisor is stopped.

Rationale: all of this is shipped seat-engine behaviour; the row adds nothing to it.

### D10. Roles and the token report

`roleOpeningStencils` keys each seat by its strand role with the advisor number dropped: the `discussion` entry is renamed `discussion-chair`, still mapped to the chair stencil, and `discussion-advisor` is added, mapped to the advisor stencil.
The token report needs no change: it keys a session by the strand's last segment with a trailing number dropped, so the seats appear as `discussion-chair` and `discussion-advisor` rows.

Rationale: the map's other keys name the role a session's strand carries, and the chair's strand carries `discussion-chair`; the map is read only by the stencils package's directive tests.

### D11. A seat table names its optional values, which may be empty

`seatengine.Table` gains a field listing the table value names that may be empty.
`Table.Validate` exempts a listed name from the blank-value rule, and the seat engine passes the list to the stencil fill as its optional markers, so an empty listed value renders as nothing.
A listed name that is also a reserved marker is a validation error.
The discussion table lists `pattern_directive` and `friction_directive`; the field is empty for every other table.

Bound: only a listed name may be empty; every unlisted table value, every seat value and every reserved marker keeps the blank-value refusal and the fill's missing-key refusal.

Rationale: the two directives are optional markers on today's single-agent path, empty when PATTERN is inactive, when Tier 2 is off and on a friction read failure, and the seat engine refuses a blank value; a new field is additive, as the Constraints require of changes to `seatengine`.

### Added after Discussion (parent, 2026-10-10): PATTERN overview stays under its cap

Decision: Main's PATTERN.md overview is at 16375 of its 16384-byte cap. Every card that adds or edits a PATTERN.md overview line keeps the overview at most 16200 bytes on the task branch: added text is offset by shortening wording in the same or another overview line, with detail moved into the pattern/ background file, and no invariant is dropped.

Rationale: Three runs (gate-test-scope, daemon-wakeups, bolt-and-store) hit TestOverview_RealFilePassesCheck at Publish after merging main; fixing it in the cards avoids another Publish block and a goto.

## Constraints

- PATTERN-stencil-ownership: the table closure is evaluated per call and the seat engine reads every stencil and include from the told directory at run time; no embedded bytes.
- PATTERN-parent-directive and PATTERN-edit-directive: both seat stencils carry the two markers, filled by the seat engine; the parent-directive test's role map covers the advisor stencil; no stencil text asks the operator.
- PATTERN-role-skills-typed and PATTERN-spawn-color: skills and segment ride the seat and table; no stencil asks for a skill or a color.
- PATTERN-agent-name: the `discussion` prefix stays a constant in `loomengine`, and seat roles are formed only by the seat engine.
- PATTERN-producer-pointer-rule: the advisor stencil points at the chair's format contract and never restates the decision record's section set.
- PATTERN-told-geometry: `loomengine` builds the table from the told location; `seatengine` and `shedadapters` derive no path.
- PATTERN-sole-parsers and PATTERN-durable-vs-ephemeral-state: the advisor notes file is tracked under `_lyx/discussion/` and no Go code parses it.
- PATTERN-config-strictness: the new key rides the existing loom config load, template and reconcile paths.
- PATTERN-refusal-way-forward: the new config refusals (an entry of `discussion_advisors` that fails to parse or resolve) land with their rows in the refusal spec's loom section and a test that reaches them.
- PATTERN-test-economy and PATTERN-test-speed: new behaviour is pinned with the existing seat fakes and in-memory fixtures, untagged; no gate runs an LLM, and the seat engine's `llm` smoke stays the hand-run proof of the channel.
- Gate quiescence: the chair is gated and never forks; advisors are sessions, not forks, so the fork rule holds unchanged.
- Test budget: a package whose new tests pass its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`; the packages in play are `internal/loomengine`, `internal/shedadapters`, `internal/shedrecipe`, `internal/loomrecipe`, `internal/seatengine` and `contracts/stencils`.
- Task `burler-multi-llm` edits `MultiLLMProducer` and the seat engine in parallel; every change here to those two packages is additive (a new constructor, new fields), never a signature change of what ships.
- An in-flight run parked on Discussion-Write under the old binary is restarted, not migrated: the seat probe looks for a `discussion-chair` strand and would run the preparation, archive the two files and start a fresh table beside a live `discussion` writer.
  The operator stops such a run's writer before the new binary drives the run; no step in this task detects or stops it.

## Auto-mode assumptions

- The template default is one `sonnet[high]` advisor with `discussion` unchanged; the operator's step-down of the chair to `opus[high]` is the operator's config edit after landing.
- One advisor brief for every seat; no distinct challenger role.
- The advisor notes file is `_lyx/discussion/advisor-<n>.md`, committed with the discussion.
- The engine name `DiscussionWrite` is kept for the row rather than switching the row to the generic `MultiLLM` engine.
- The advisor-aware sentence lives in the Go-held `mode_rules` text, not in stencil prose.
- No `llm`-tier test is added for the discussion table; the seat engine's existing smoke covers the channel.
- Advisors load `scribe:prose` only; the chair keeps `scribe:conversation` for the interview.

## Open risks

- Merge overlap with `burler-multi-llm` in `internal/shedadapters/multillm.go` and `internal/seatengine`, mitigated by keeping this task's changes additive.
- An advisor's study is paid whether or not the chair asks it anything; the measure is the token report's `discussion-advisor` row against the Discussion-Review round counts of earlier runs.
- The chair may lean on an advisor's answer as authority; the chair seat directive already states that an advisor's text carries none, and the support log's attribution by strand name, per D6, makes the reliance reviewable.
- The advisor notes file sits in the directory the Discussion-Review reviewer reads; its targets stay the two files, so a reviewer reading the notes is harmless but possible.

## Acceptance criteria

- The Discussion-Write row builds, with every seat's markers filled, when `discussion_advisors` is empty, has one entry, and has two entries; the chair's inputs cover every advisor output, and the chair is interactive exactly when `discussion_interactive` is set.
- The discussion table validates and every seat's prompt composes when PATTERN is inactive, when Tier 2 is off and when the friction directive fails to read; an unlisted blank table value and a listed name that is a reserved marker are still refused.
- A `discussion_advisors` entry that is empty beside another entry, fails to parse or fails to resolve is refused at config load naming the key, the index and the way forward; an absent key, null, `[]`, `""` and `[""]` each load as zero advisors.
- The entry wires the parent-review preparation when the gate list holds an enabled `parent-review` entry and leaves it nil otherwise, and the producer runs it after the probe found no live chair and before the archive, never on the resume path; a preparation error archives nothing and starts no seat.
- The producer evaluates the table source once per call.
- The chair's prompt names the run's parent and every advisor's strand name; an advisor's prompt names the chair as its parent in the non-interactive form, and neither prompt tells an agent to ask the operator.
- The autonomous `mode_rules` text carries the advisor sentence when the table has advisors and not otherwise.
- The chair stencil keeps every claim the claims test pins today, and adds a claim that a relied-on advisor answer is recorded in `## Interview` and an advisor-informed pick in the question ledger, each naming the advisor's strand name; the advisor stencil's claims pin: write only the notes file, message only the chair, never mutate git, and write the notes before questions arrive.
- The advisor stencil is registered and listed under the `discussion-advisor` role, the chair stencil under `discussion-chair`, and both carry both directive markers.
- The loom recipe's built row list still resolves every row and the Discussion-Write row's producer type is the commit decorator.
- The discussion gate and the parent-review brief read the two discussion files only, with an advisor notes file present beside them.
- The reconciled template carries `discussion_advisors`, and `docs/overview.md` names the key and describes the Discussion producer as a chair with advisors.

## Notes for the plan writer

- The single-agent path to remove: `internal/loomengine/discussion.go`, its prompt composer, `shedrecipe.Env.DiscussionSpec` and the `DiscussionSpec` closure in `internal/loomcli/wiring.go`; their tests fold into the table builder's.
- The seat engine fills `parent_directive`, `edit_directive`, `seat_name`, `chair_name`, `advisor_names`, `failed_advisors`, `inputs`, `outputs`, `output_files` and the three geometry markers itself and refuses a table value that collides with one; `pattern_directive`, `friction_directive`, `mode_rules`, `slug` and the two paths are table values.
- `seatengine.Table.Validate` already requires every advisor output among the chair's inputs and reads every stencil and include from the stencils directory, so the table builder's tests can run it against the embedded stencils seeded to a temporary directory.
- `loomrecipe`'s test environment fills `Env.DiscussionSpec` today and must fill `Env.DiscussionTable` and `Env.Seats` instead; the shape test's row-to-engine table and the cross-consumer coverage guard need no new entry because the engine name stays.
- The parent-review store's `PrepareRound` and the gate's reject re-prompt are unchanged; the seat engine's `StartGated` carries the chair's gate.

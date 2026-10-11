# Decision record: small-fixes-oct11

## Goal

Land five small, independent fixes from the board entry `small-fixes-oct11`:

1. `tokencount-crash-gap`: a crashed or abandoned turn's gap to the session's next line stops counting as working time.
2. `tokencount-slug-reuse`: a reused slug counts only its own run's transcripts, for tokens, time, forks and the orch charge.
3. `loom-step-help-seed`: `lyx loom step --help` says only what step does, and the help and the missing-seed refusal name the driverless seed command beside `lyx loom start`.
4. `plan-rubric-file-target` (issue #542): the plan review rubric's "Proposed fixes" section names the target-redundancy checks a target-pairing fix must pass.
5. `notice-custom-state`: the batten notices with their own text (quiet, api-error, driver-dead) name the child's state and row again.

## Scope

In:

- `tools/tokencount`: the working-interval rule, a lower bound on which transcripts a run counts, the package doc in `main.go`, and the launcher descriptions in `tokencount.sh` and `tokencount.cmd` where they describe how time or transcripts are measured.
- `internal/loomcli`: the `step` and `run` help text and the missing-seed refusal's way forward; `contracts/specs/refusal-spec.md`'s loom "seed missing" row.
- `contracts/stencils/loom/loom-rubric-plan-review.md` and its claims in `contracts/stencils/claims_test.go`.
- `internal/battenshed`: the condition text of the quiet, api-error and driver-dead notices, and the notice wording in its `doc.go`.
- `docs/overview.md`'s batten `step` sentence, only where it reads as if `lyx loom step` seeds a fresh run.

Out:

- Any change to what `lyx loom step` or `lyx loom run` do: no auto-seeding, no new flag.
- An upper bound on a run's transcripts: lines after the span's end stay counted for tokens; time stays clipped to the span as today.
- Notice episode keys, delivery, timing and the set of notice conditions.
- `lyx loom validate-plan` itself and the `redundant-file-target` check's behaviour.
- `docs/research/discussion-bench.md`: a dated research record of the bench run, left as written.

## Decisions

### D1 — Tokencount: an interval that ends at a turn-start line is idle

A working interval runs from a user or assistant line to the next, unless the first line ends the turn (unchanged) or the second line starts a turn.
A turn-start line is a user line whose content holds text (a string, or a `text` item) and no `tool_result` item, other than an interrupt-marker line or a `<local-command-stdout>` line, which keep their current turn-ending role.

Rationale: in a normal transcript the line before a prompt already ends the turn, so the rule changes nothing there.
After a crash, the transcript holds a line that does not end the turn (an assistant `tool_use` or null-`stop_reason` line, or a `tool_result`) followed, possibly hours later, by the resume's prompt; that gap is exactly the interval the new rule drops.
`stop_reason` null cannot itself end a turn: real transcripts write it on intermediate content blocks mid-turn.

Bound: the rule removes only an interval whose later line is a turn-start line.
Mid-turn user text lines (a skill body after a `Skill` result, a queued operator message) follow their predecessor within moments, so the working time they drop is that short gap.
A crash with no later line in the same transcript already contributes nothing and still does not.
No time cap is applied, so a long foreground tool call or a long thought keeps counting in full.

### D2 — Tokencount: a run counts only transcripts that start after the slug's previous run

Each counted run gets a lower bound: the committer date of the newest archive tag that predates the run.
For a run whose worktree `<hub>/<slug>` still exists (in flight), that is the slug's newest tag; for a finished run, the next-older tag, the one `ShedRecord.Older` already reads.
A slug with no such tag has no bound and counts every transcript, as today.

A main-session transcript whose earliest user or assistant timestamp is before the bound is left out whole, with its sub-agent transcripts: its tokens, cost, working intervals, session count, forks, Merriam starts and begin-batch base, and its lines in the run's First/Last window, which the orch charge reads.
The bound is computed before `CountRun` reads any transcript, so `main.go` reads the archive tags first and passes the bound in.
A run whose bound left transcripts out says so in its per-run section with one line: how many transcripts were left out and the bound's date.

Rationale: the project directory is keyed by the worktree path, which the slug's earlier runs shared, and the worktree did not exist between the previous run's teardown and this run's creation.
Judging a transcript by its first line, not its last, keeps an older run's session out even when its last lines postdate that run's final tagged commit.
Whole-transcript granularity fits: lyx starts every run with fresh sessions, so no transcript spans two runs.

Bound: the filter can drop only transcripts that started before the previous run's final tagged commit; a run's own sessions all start after its worktree exists, which is after that commit.

### D3 — `lyx loom step` and `lyx loom run` help say they need a seed and name both seeding routes

`step`'s help drops the claim that it bootstraps "exactly as `lyx loom start` does".
It says: step needs a seed at the addressed run-id and refuses without one, since it never writes a seed; with one, it runs start's seed-agreement, status-seeding and commit block, then drives exactly one producer.
It names the two ways to seed: `lyx loom start`, which seeds and spawns the LLM driver, and `lyx shed seed self --recipe loom --driver go --param parent=<parent-branch>`, which seeds a driverless run that `step` or `run` then drives, `<parent-branch>` being the pair's parent branch.
`run`'s help paragraph on seeding names the same two routes.

Rationale: the bench agent followed the help and the refusal to `lyx loom start` and got a driver it did not want; the driverless route existed but no text on the path named it.

### D4 — The missing-seed refusal names the driverless seed for `run` and `step`

For `run` and `step`, the missing-seed refusal's remedy names both routes from D3.
For `status`, `pause` and `goto` it keeps `run "lyx loom start" first to bootstrap this task`, since a driverless seed gives those verbs nothing to act on.
The message still renders through `shedrun.MissingSeedMessage`, and the refusal stays a `KindlessRefusal`.
The `seed missing` row of the loom section in `contracts/specs/refusal-spec.md` changes its way forward in the same commit (`PATTERN-refusal-way-forward`), and a test reaches both remedies.

Rationale: `lyx shed seed` is not in the denied set, so naming it complies with `PATTERN-no-denied-recovery`; the parent branch stays a placeholder because the refusal path reads only the seed and the run list, and adding an origin-record read there would widen a deliberately I/O-light check.

### D5 — The plan rubric points at the redundancy checks by name

The "Proposed fixes" section of `loom-rubric-plan-review.md` gains one sentence naming `redundant-package-target` and `redundant-file-target` in `{{.specs_dir}}/loom/loom-plan-spec.md` as the checks a fix that adds or pairs targets must pass: a file or package listed beside a member of it on one card is refused, and the spec says what counts and the way forward.
`claims_test.go` pins the sentence for this stencil, as it already does for the plan template.

Rationale: the section already requires a fix every validation check admits, yet a reviewer still proposed a refused shape; naming the two checks most target-pairing fixes trip puts them in front of the reviewer.
The sentence mirrors the plan template's existing pointer and does not restate the check, per `PATTERN-producer-pointer-rule`.
Pointing reviewers at `lyx loom validate-plan` as a pre-check is not adopted: it needs a trial edit of the plan, which a reviewer does not make.

### D6 — Quiet, api-error and driver-dead notices lead with the child's state

`conditionText` renders these three conditions as `haltText(status)`, then `; `, then the condition's own text, the shape the driver-busy notice already uses, for example `running at Plan-Write; agents idle for 25m0s`.
Their own texts drop `while the child is running`, which the state now says.
Episode keys are unchanged: they hold the condition name, not its text, so a batten restart's marker still matches.
`internal/battenshed/doc.go` says every notice names the child's state and row.

Rationale: the rephrase in commit `17b09e97f` moved state into `haltText` for state-changed notices only, so the notices with their own text lost it.

## Constraints

- `PATTERN-refusal-way-forward`: D4's changed way forward lands with its spec row and a test that reaches it.
- `PATTERN-no-denied-recovery`: D3 and D4 name `lyx shed seed` and `lyx loom start`, neither denied; `internal/loom*` string literals are scanned.
- `PATTERN-producer-pointer-rule`: D5 points at the plan spec's checks and does not paraphrase their rules beyond the plan template's existing one-clause gist.
- `PATTERN-comment-line-breaks` and semantic line breaks: every Go comment and `.md` edit is one sentence per line.
- `PATTERN-test-economy` and `PATTERN-test-speed`: extend existing tests and tables where one covers the behaviour; tokencount and battenshed scenarios stay in the untagged tier with injected clocks.
- `PATTERN-gitexec-checked-call` and `PATTERN-gogit-read-helper`: D2 reads the archive tags through the existing `finishedRuns` path, adding no new git call site.
- `PATTERN-lyxdirs-single-declarer`: no new `_lyx` or `.lyx` literal.
- CLAUDE.md "Docs land in the same commit": the tokencount package doc, battenshed's `doc.go` and the `docs/overview.md` sentence change with the code.
- CLAUDE.md "Test budget": a card whose new top-level tests push a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- No operator answered; every choice below is a self-pick recorded in the support log's question ledger.
- D1's turn-start rule is preferred over a time cap; no cap constant is introduced.
- D2 bounds below only, by the previous run's archive tag date, not by the seed's `started_at`, and adds no upper bound.
- D2 adds one per-run report line for transcripts left out; no new flag disables the filter.
- D3 and D4 use the placeholder `<parent-branch>` rather than reading the recorded parent into the refusal.
- D4 limits the driverless remedy to `run` and `step`.
- D6 drops `while the child is running` from the three texts rather than keeping it beside the state.

## Open risks

- D1 assumes a resumed session's first line after a crash is the resume prompt.
  If Claude Code instead writes a synthetic `[Request interrupted by user for tool use]` result stamped at resume time, the crash gap ends at an interrupt line and still counts as working.
  No crash transcript was available to confirm which happens.
- D2 trusts the archive tag's committer date as the end of the previous run's sessions' starts.
  A session the previous run started after its final tagged commit would still be counted.
- D2 changes figures in reports regenerated for reused slugs; earlier reports are not comparable for those runs.

## Acceptance criteria

- A transcript whose non-turn-ending line (an assistant `tool_use` line, or a user `tool_result` line) is followed an hour later by a user prompt line counts that hour as idle, not working, for the role and the run.
- A normal turn (prompt, `tool_use`, `tool_result`, `end_turn`) keeps its full working time, and an interval ending at an interrupt-marker line still counts as working.
- A mid-turn user text line right after a `tool_result` does not cut the working time of the lines around it beyond the gap into it.
- For a slug with two archive tags whose worktree is gone, a transcript starting before the older tag's date is left out of tokens, time, forks, Merriam starts, base and the run window, a transcript starting after it is counted, and the per-run section states the left-out count and the bound's date.
- For an in-flight slug (worktree present) with one archive tag, transcripts starting before that tag's date are left out.
- A slug with a single run (one tag, worktree gone, or no tag) counts every transcript, as today, and prints no left-out line.
- `lyx loom step --help` no longer says step bootstraps exactly as `lyx loom start` does, says it refuses without a seed, and names both `lyx loom start` and `lyx shed seed self --recipe loom --driver go --param parent=<parent-branch>`; `lyx loom run --help` names the same two routes.
- `lyx loom step` and `lyx loom run` on an unseeded pair refuse with a message naming both routes; `status`, `pause` and `goto` refuse naming `lyx loom start` only.
- The loom `seed missing` row in `contracts/specs/refusal-spec.md` matches both remedies.
- The rendered plan review rubric names `redundant-package-target` and `redundant-file-target` in its "Proposed fixes" section, and `claims_test.go` pins it.
- A quiet, an api-error and a driver-dead notice each start their condition with the child's state and current row, for example `running at Plan-Write; `, and none says `while the child is running`.
- A batten restart after a notice was sent under the old wording does not resend it: the episode key is unchanged.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` are green.

## Notes for the plan writer

- Tokencount: `endsTurn` and `workingIntervals` in `tools/tokencount/count.go` hold the interval rule; `turnPoint` gains whether a line starts a turn; `userTexts` already parses string and item content but ignores `tool_result` items, so the turn-start test needs the item types.
- Tokencount: `run` in `main.go` calls `CountRun` before `finishedRuns`' tags are consulted per slug; `readShedRecord` and `useShedRecord` in `shed.go` already use the next-older tag as the span's floor.
- Loom: `step`'s and `run`'s `Long` text sit in `internal/loomcli/cli.go`; the refusal is built in `resolveRunID` in `internal/loomcli/arm.go`; `arm_seed_test.go` drives it at Tier 1.
- Rubric: the plan template's pointer sentence and its claim in `claims_test.go` are the model for D5.
- Batten: `conditionText`, `haltText` and the driver-busy notice's composition are in `internal/battenshed/notice.go`.
- `docs/overview.md`'s batten `step` sentence ends "the same single-producer primitive `lyx loom step` is" right after "seeding a fresh run first"; reword so it does not read as loom step seeding.

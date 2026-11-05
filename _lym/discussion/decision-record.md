# Decision record — drift-rework

## Goal

Drift stops depending on an agent's reading of an opaque compare.
lym's code diffs the baseline capture against the current capture into changed values.
Code checks those values against an expectation an agent writes and seals before the writer starts.
A separate judge rules on each expected changed value against its exact old and new value.
An acceptance covers one exact value, never a whole test.
Drift stays a general lym module: nothing in it assumes one hub's harness or output format.

## Scope

In:

- A Go diff of two capture directories into changed values, keyed by line key or line number.
- The `lines` and `command` diff modes, chosen in `drift.yaml`.
- Scope chosen by config, never by an agent.
- A new Drift-Expect row and agent writing a structured, sealed `expected.yaml`.
- Containment by code in Drift-Verify, in Drift-Recheck (the same engine in a second row), in the writer's gate entry and in a `lym drift check` verb.
- A judge agent with structured output, and judged values stored as exact tuples.
- A parent amendment verb for the expectation.
- A verdict rewrite.
- Config key changes with their migration and reinterpretation.
- New and rewritten stencils for the expect agent, the judge and the writers' drift paragraph.
- Removal of the machinery the board entry lists.
- Both the tapestry and darn recipes.

Out:

- Hub-level exclusion of nondeterministic cases or noisy lines: the capture owes deterministic output.
- Drift verbs in a sampler (`sampler-drift-trial` is revisited after this lands).
- Correcting a baseline for parent work merged after the branch point (see Open risks).

## Decisions

### D1 — The capture contract and the profile diff

Decision: A hub's `capture` command writes deterministic text files under `{baseline_dir}`, one file per case.
Every regular file under the capture directory is a case, named by its path relative to that directory; a harness that writes a log or other non-case file there breaks the contract.
Code diffs the baseline capture directory against a current capture directory file by file:

- A file only in the current capture is an `added case`; a file only in the baseline is a `removed case`.
- Inside a case, each line is classified on its own side.
  A line that matches one of the configured key patterns is identified by its key, taken from the first pattern that matches; its value is the rest the pattern's `value` group holds.
  A key that occurs more than once in a file is identified as `key#n`, its occurrence counted from 1 in file order.
- A line that matches no key pattern is identified as `#n`, its position among the file's unkeyed lines, counted from 1.
- A keyed identity present on both sides with a different value is a `changed` cell (case, identity, old value, new value).
  An identity on one side only is an `added line` or `removed line` cell.
- Unkeyed identities pair by position among unkeyed lines, so inserting or removing a keyed line moves no unkeyed identity.
- The `drift.key_patterns` list in `drift.yaml` holds the patterns, regular expressions with named groups `key` and `value`.
  The template's default holds two, for `name = value` and `name: value`; an empty list makes every line unkeyed.
- Line endings are normalised before comparing, and a trailing newline at end of file is no line.

Rationale: Keys keep a cell stable when a change inserts or removes lines; numbering unkeyed lines among themselves keeps a keyed insert from shifting them, and open ranges cover unkeyed inserts (D5).
The patterns come from config, so no hub's format is hard-coded (`PATTERN-no-hard-coded-choice`).
The `cs` hub's profile files carry lines such as `fluid = water` and `Re = 127069, dP = 7925.267 Pa`; the default patterns key them as `fluid` and `Re`, the second with the value `127069, dP = 7925.267 Pa`, so one line yields one key.
Source: `docs/research/drift-csharp-trial-2026-11-03.md` and `docs/research/drift-csharp-trial-2-2026-11-03.md`.
A repeated key's occurrence number shifts when an earlier occurrence is inserted; the shifted cells are judged again (D10), which costs judge work and lets no regression through.

### D2 — Two diff modes: `lines` and `command`

Decision: `drift.diff` is `lines`, the default, or `command`.

- `lines` uses D1's Go diff; the `compare` command is ignored, and `lym drift status` says so.
- `command` keeps today's opaque compare for a hub whose capture cannot meet D1's contract; a `command` mode with no `compare` set is refused, naming the key to set.
  The compare command runs against the baseline; exit 0 passes.
  A nonzero exit spawns the judge (D9) with the compare's log and the expectation; the judge rules per drifted case it reads in the log.
  Any ruling other than `matches` is a Stuck.
  No ruling is stored, so each failing compare is judged again.
  The writer's gate entry and `lym drift check` (D8) pass in `command` mode and report the compare's exit and log path, since only the judge can tell expected drift from a regression there.

Rationale: The board entry keeps an opaque compare with whole-test judgment for a hub whose capture cannot be line-diffed.
Bound: `command` mode gives up containment by code and stored rulings, and the judge sees only what the compare's log shows; a log that shows the first deviating line of a case, as the `cs` compare's does, hides a later regression in a case that already drifts.
Only the hub's `drift.yaml` sets `diff` (D15).

### D3 — Config chooses the scope

Decision: Drift-Baseline records the scope once, from config: `off` when `drift.scope` is `off`, `full` otherwise.
Every later row reads the recorded scope, never config, so a run's scope does not change mid-run.
No agent chooses the scope.
`partial`, `skip` and the `{skip_file}` placeholder are removed.
A `drift.scope` of `partial` in a run's config is recorded as `full`, with one Warn naming the key.
A run whose scope record already says `off` or `full` when this lands keeps it.
The `docs` profile's `darn` block in `internal/runprofile/template.yaml` sets `drift: {scope: off}`.
The scope record names where `off` came from, the run's `drift.scope`, and the verdict states it (D13).

Rationale: The trial faults `drift-skip-all-capture` and `drift-baseline-skips-intended` came from an agent choosing the scope.
A run sees only config keys, never its profile's name (`internal/battencli/runsettings.go` resolves the profile at Seed-Child), so the `docs` profile turns Drift off through a config key.
Bound: a task's `run_settings` may still set `drift.scope: off`; that is the parent's or operator's call, recorded with its source and stated in the verdict.

### D4 — Rows and engines

Decision: Both recipes run, in this order:

1. `Drift-Baseline` (engine `DriftBaseline`): code only. It records the scope (D3) and, unless the scope is `off`, starts the detached capture as today. It spawns no agent.
2. `Drift-Await` (engine `DriftAwait`): unchanged.
3. `Drift-Expect` (new engine `DriftExpect` (new)): spawns the expect agent (D5) and seals its file (D6). Done at once when the scope is `off`, the capture was deferred (D12) or a seal commit exists.
4. The writer (`Darn-Write` or `Webster`), with the drift gate entry (D8).
5. `Drift-Verify` (engine `DriftVerify`): capture, seal check, diff, containment, judge (D6, D7, D9, D10). It spawns no agent that edits code.
6. The Code-Review perch.
7. `Drift-Recheck`: the same `DriftVerify` engine in a second row.

Drift-Verify records each check's result in `scope.json`: the warp head it checked, whether it passed (every cell expected and ruled `matches`), and the counts and lists D13 renders.
Either row is Done at once only when the warp head equals the head of the last passing check and no amendment (D11) was recorded since; otherwise it runs the full pass, and only cells whose tuple has no stored ruling are judged.
In tapestry, `Drift-Expect` sits after `Drift-Await` and before `Webster`; `Drift-Baseline` keeps its place before the parts and plan writers.
In darn, `Drift-Expect` sits after `Drift-Await` and before `Darn-Write`.
`Drift-Recheck` loses its `on_stuck` route to `Drift-Verify`: a Stuck at either row goes through the stop chain (`PATTERN-shed-stop-chain`).
The `DriftRecheck` constructor and engine are removed from `internal/shedrecipe`; the row name `Drift-Recheck` stays, so no rename or retirement is needed.
Every drift row keeps the entry gate `drift.enabled`.

Rationale: Drift-Expect runs after the capture so its agent writes against real case names and identities.
Keeping the existing row names keeps resume of in-flight runs simple: a run parked before `Drift-Expect` reaches it, and a run already past it has no seal and is checked against an empty expectation (D6).
Drift-Verify writes no code, so one engine serves both compare rows; skipping only after a passing check means a resumed Stuck row checks again.

### D5 — The expectation file

Decision: Drift-Expect hands the expect agent, for every baseline case, a listing of the case's identities as D1 reads them, each with its text.
The agent writes `_lym/drift/expected.yaml`.
Per case it lists:

- `keys`: identities it expects to change, be added or be removed, each with one `why`; `key` covers every occurrence of the key, and `key#n` only that occurrence.
- `lines`: unkeyed positions or ranges, as `12`, `12-20` or the open range `12-`, each with one `why`.
- `added` and `removed`: the cases it expects added or removed, each with one `why`.

The file states what is expected and why, never what may be accepted, and carries no tolerance.
An empty expectation is valid.
One sole parser reads `expected.yaml` (`PATTERN-sole-parsers`) and validates it: a named case must exist in the baseline unless listed under `added`; a `removed` case must exist in the baseline; a `keys` identity of an existing case must exist in its baseline unless the agent marks it `added: true`; a range must be well formed; every entry has a non-empty `why`.
The expect agent's session carries a gate entry, `expectation`, that runs that validation; findings re-prompt the agent, and an exhausted entry is a Stuck through the stop chain, whose resume spawns the agent again.
The agent's model is `drift.expect_model`.

Rationale: A structured file lets code decide containment, and the listing makes the agent write against code's own identities, such as `Re` for the `cs` line above.
Open ranges are allowed without a halt, per the operator's decision 2; D9 and D13 list every one.

### D6 — The seal

Decision: When the expect agent's file validates, Drift-Expect commits it on the weft in one commit carrying a `Drift-Seal: <sha256 of the file>` trailer beside fabric's `Warp-SHA` trailer.
The seal is the oldest weft commit carrying `Drift-Seal`; no seal fields are kept in `scope.json`.
On re-entry, Drift-Expect is Done when a seal commit exists, and seals an existing `expected.yaml` that validates without spawning the agent.
Drift-Verify, in either diff mode, and the drift gate entry and `lym drift check` (D8) check the seal first:

- with no seal commit, no `expected.yaml` and a run past Drift-Expect, the expectation is empty, and the check goes on;
- otherwise the seal commit must exist, its `Drift-Seal` hash must equal the hash of its own copy of `expected.yaml`, the current file must have that hash, and its `Warp-SHA` must equal the capture's baseline head, so the seal precedes every warp commit the writer makes.

A failed seal check is a finding naming which check failed; at Drift-Verify it is a Stuck whose way forward is the parent's `lym drift amend --restore` (D11), which writes the sealed copy back.
The git reads and the trailer commit go through `internal/fabricengine` behind general seams that name no drift concept: commit given paths with an extra trailer, find the oldest weft commit carrying a trailer key, and read a file at a weft commit (`PATTERN-fabric-git`, `PATTERN-fabric-isolation`).

Rationale: The operator's decision 4: the hash catches an edited file, and the `Warp-SHA` shows the expectation existed before any code.
Every weft commit fabric makes carries `Warp-SHA` (`internal/fabricengine/trailer.go`).
Keeping the seal in the commit, not in a record, means editing both the file and a record cannot fake it, and one commit leaves no half-sealed state to restart from.
Bound: forging a seal needs a hand-made weft commit carrying the trailer before the writer's first warp commit, which no agent's verbs make; a rewrite of the weft history is outside the check.

### D7 — Containment by code

Decision: In `lines` mode Drift-Verify captures the current code, checks the seal (D6), diffs (D1) and classifies every cell:

- expected: covered by an entry of the sealed expectation or an amendment (D11);
- unexpected: covered by none.

Any unexpected cell is a regression: the row is Stuck, the reason naming each unexpected cell with case, identity, old and new value.
No agent stands between the finding and the stop chain.
An expectation entry that covers no cell is reported in the verdict, never failed.
Every current capture, here and in D8, runs in a scratch directory of its own under `.lym/drift/`, under the capture lock and in a gate slot, inheriting the caller's held slot when it runs inside a gate; it writes no `capture.json` and never touches the baseline directory or its backups.

Rationale: Whether a drift occurred and whether it was expected are facts code can establish; trial 2's `oil-steel-6in-turbulent` showed an expected entry with no drift is normal.
The lock serialises concurrent checks, such as a Webster fork's `lym drift check` beside Merriam's gate, and inheriting a held slot keeps a check run from a gate key from waiting on its own slot (`PATTERN-gate-slot`).

### D8 — The writer's drift gate entry and `lym drift check`

Decision: A `drift` gate entry runs D7's capture, seal check, diff and containment, and returns the unexpected cells and seal failures as findings; it runs no judge and writes no tracked record.
It passes when the scope is `off`, when the capture was deferred, or in `command` mode (D2).
The entry is added to Darn-Write's `gates` list in `contracts/recipes/darn-recipe.yaml` and resolved by `internal/shedrecipe/entries_gate.go` like the existing plain entries.
For tapestry the entry goes on the Webster row's `gates` list in `contracts/recipes/tapestry-recipe.yaml`, which reaches Merriam's run-level gate; per the operator's decision 3 the plan reads Webster's gate seam and settles whether a card or batch gate key runs it too.
The entry carries no cold fixer: the writer, who holds the context, fixes its own regressions.
A new `lym drift check` verb runs the same check and prints the findings, so any session in the run's worktree can run it mid-work, such as a Webster fork after a card.
Bound: `check` and the entry write only their scratch capture directory, and decide nothing: Drift-Verify repeats the check as the guarantee.

Rationale: The writer meets "key `dP` of case X moved, outside the expectation" while it can still fix it, up to the entry's attempts.
Webster's run-level gate is fed by the Webster row's `gates` list (`internal/websterengine/runlevel.go`), so the recipe-level entry reaches tapestry's writer.

### D9 — The judge

Decision: In `lines` mode, when every changed cell is expected and some cell's exact tuple has no stored ruling (D10), Drift-Verify spawns one judge agent.
It sees each such cell with an id, its old and new value, the covering entry's `why`, whether that entry is an open range, and the task's decision record or darn record; for an added case it sees the new file, for a removed case the old one.
It writes `_lym/drift/judgement.yaml`, in one of two forms:

- `lines` mode: one ruling per cell id, `matches` or `does not match`, each with a reason, and an `open_ranges` list naming every open range that covers a judged cell, each with a sentence on what the judge checked under it;
- `command` mode (D2): one ruling per drifted case read from the log, `matches`, `does not match` or `unexpected`, each with a reason.

One sole parser reads it; the judge's session carries a gate entry, `judgement`, that checks the form for the mode: every cell id has exactly one valid ruling and every covering open range is listed, or every case has a valid ruling.
The judge edits no code and no expectation: Drift-Verify snapshots the tracked tree before the spawn, and a tracked change the session made outside `judgement.yaml` is a Stuck.
Its model is `drift.judge_model`; with `drift.judge_interactive` set the judge presents each cell to the operator and records the operator's ruling.
Any `does not match` or `unexpected` is a Stuck naming the cells or cases and reasons.

Rationale: The judge wrote neither the expectation nor the code, and rules against facts code produced.
Listing open ranges in the judge's own output meets the operator's decision 2.

### D10 — Judged tuples

Decision: Each `lines`-mode ruling is stored in a tracked record, `_lym/drift/judged.json`, as its exact tuple (case, identity, kind, old value, new value) with ruling, reason, the judge's strand and time.
A later check that reproduces a stored tuple reuses its ruling; a cell whose tuple changed is judged again.
A stored `does not match` for a reproduced tuple is a Stuck without a new judge.
The record is written through `internal/state` and tolerates unknown keys (`PATTERN-tolerant-records`).

Rationale: An acceptance covers exact values, so a later move of an accepted value is judged again while unchanged values cost nothing.

### D11 — Parent amendment

Decision: A new verb, `lym drift amend <slug>`, takes the run's slug and checks its caller as `lym loom decision add` does; it is admitted for the run's parent orch and the operator, at a Stuck writer, Drift-Verify or Drift-Recheck row.
It does one of two things:

- adds expectation entries in the D5 shape, each with a `why`, validated by D5's rules;
- `--restore` writes the seal commit's copy of `expected.yaml` back over the current file.

An amendment is recorded in `scope.json` with who, when and its entries or the restore, and committed with the drift records; it never edits the seal commit.
Containment reads the sealed expectation and every amendment.
An amendment that adds entries drops the stored rulings of the cells they cover, so those cells are judged again against the amended `why`.
Every amendment is listed in the verdict (D13).
Bound: an amendment only adds coverage or restores the sealed file; it cannot remove an entry, cannot change the seal, and reaches no agent of the run.

Rationale: The parent rules only on a changed expectation; the seal stays intact and the amendment is visible on the pull request.

### D12 — A deferred or missing capture

Decision: `lym drift defer` stays for a capture that cannot run, and is also admitted, for the run's parent and the operator, at a Stuck writer, Drift-Verify or Drift-Recheck row whose baseline directory is missing.
With a deferred capture, Drift-Expect is Done at once, the drift gate entry and `lym drift check` pass, and Drift-Verify and Drift-Recheck are Done with the verdict stating that no compare ran because the capture was deferred.
A run whose capture was counted but whose baseline directory is gone, such as after `.lym` was cleaned or on another machine, is Stuck at Drift-Verify naming `lym drift defer` as the way forward.
`lym drift waive` is removed.

Rationale: `defer` is the parent's decision to go on without a baseline; with no acceptances to rule there is nothing left for `waive` to settle.
The baseline lives under `.lym`, so its loss must have a way forward; recapturing at the branch point is not offered, because the worktree has moved past it.
Bound: `defer` after the writer skips the regression check for the rest of the run; only the parent or operator runs it, and the verdict states it.

### D13 — The verdict

Decision: `driftengine.Verdict` renders the pull request's and landing commit's `## Regression check` section from `scope.json` and `judged.json`, without running a capture:

- the scope, and for `off` its source (D3); a deferred capture (D12);
- the diff mode, and in `command` mode that containment was the judge's;
- from the last check: changed cells, expected cells, cells judged as matching, cells reused from stored rulings, unexpected cells;
- every expectation entry that covered no cell;
- every open range with its `why`;
- every amendment with who and its entries or restore.

Rationale: The pull request shows what Drift checked and what a human ruled, without reading run records.

### D14 — Removals

Decision: Removed with their tests and their stencil and recipe-comment text: `partial`, `skip` and `{skip_file}`; the `scope`, `accept`, `withdraw`, `waive` and `compare` verbs and `capture --side`; ruled and unruled acceptances and their halt marks; the run budget and `runs.json`; side capture; the accepted profile, `RunAcceptanceCapture` and the `CompareAccepted` seam; `.lym/drift/`'s `accepting`, `accepted-tests` and `accept.lock`; the verify agent's dirty, verify and compare gate entries; the language skills and PATTERN directive on the Drift-Verify seat; `baseline.md` and `verify-report.md`; the `drift-template-baseline` and `drift-template-verify` stencils; the `DriftRecheck` engine.

Rationale: Each exists for an agent choosing scope, accepting whole tests or fixing code at Drift-Verify, which D3, D9 and D10 replace.

### D15 — Config keys

Decision: `drift.yaml`'s template after the change:

- kept: `enabled`, `capture`, `scope` (`""` or `full`, or `off`);
- kept, used only under `diff: command`: `compare`;
- added: `diff` (`lines`), `key_patterns` (D1's two defaults);
- renamed, value carried over: `baseline_model` to `expect_model`, `verify_model` to `judge_model`, `verify_interactive` to `judge_interactive`;
- removed: `skip`, `verify_max_runs`.

The renames are the drift module's migration, as darn, gate, landing and loom migrate their keys; `lym config reconcile` prunes the removed keys (`PATTERN-config-layers`).
`skip` and `verify_max_runs` are declared retired keys with a way forward, so a board entry whose `run_settings` still carries one is refused at seeding, naming the edit.
`diff` and `key_patterns` are refused in a task's `run_settings`, naming the hub's `drift.yaml` as their place.
A `configreg.Reinterpretation` on the drift module (`PATTERN-config-reinterpretation`):

- flags a hub base whose `compare` is set while `diff` is unset: `compare` now runs only under `diff: command`, and the hub's capture must meet D1's contract to keep the default;
- flags a `capture` or `compare` command holding `{skip_file}`, naming the edit.

A capture or compare command still holding `{skip_file}` is refused at run time, naming the edit to the hub's `drift.yaml` as the way forward (`PATTERN-refusal-way-forward`).

Rationale: `compare` and the capture command change meaning for a hub configured today, such as `cs`, whose capture command passes `--skip-file {skip_file}`; a rewrite of a hub's command line would guess at its harness's flags, so the reinterpretation flags them instead.
Keeping `diff` and `key_patterns` out of `run_settings` keeps containment by code from being switched off per task.

### D16 — Stencils

Decision: Two new stencils, read at call time through `internal/stencilstore` (`PATTERN-stencil-ownership`):

- `drift-template-expect`: reads the identity listing, the capture files and the task's decision record and plan or darn record, writes `expected.yaml` per D5, states what is expected and why, never what may be accepted; a stated output invariant yields an empty expectation.
- `drift-template-judge`: rules each cell or case per D9 against its `why`, edits nothing else, and lists every open range with what it checked under it.

The darn writer's and Webster's stencils gain a drift paragraph: the path of the sealed `expected.yaml`, that the writer must not edit it, that the `drift` gate entry checks containment, and that `lym drift check` runs it on demand.

Rationale: The writer never saw the expectation; now it reads it and meets the check in its own session.

## Constraints

- `PATTERN-sole-parsers`: `expected.yaml` and `judgement.yaml` each have one parser; consumers read its model.
- `PATTERN-tolerant-records`: `scope.json` and `judged.json` tolerate unknown keys; `expected.yaml` and `judgement.yaml` are agent-written contract files and stay strict.
- `PATTERN-durable-vs-ephemeral-state`: tracked drift records under `_lym/drift/`, capture directories and locks under `.lym/drift/`, both named only through `internal/lymdirs`.
- `PATTERN-shed-recipe-registry`: `DriftExpect` is a `Constructor` entry; the `drift`, `expectation` and `judgement` gate entries resolve through `internal/shedrecipe`; rows stay off through `drift.enabled`.
- `PATTERN-fabric-isolation`: D6's fabric seams name no drift concept.
- `PATTERN-refusal-way-forward`: every new refusal names its way forward, and a test reaches it.
- `PATTERN-parent-directive`: the judge and expect seats' skills and colour go on the launch spec.
- `PATTERN-cli-cobra`: `check` and `amend` are subcommands of the existing `lym drift` tree.

## Auto-mode assumptions

- D1: unkeyed lines pair by position among unkeyed lines; a repeated key is told apart by occurrence.
- D3: the `docs` profile turns Drift off through `drift.scope: off`; `partial` is recorded as `full`.
- D6: the seal is a trailer on one weft commit, not a record.
- D8: the writer's `drift` entry has no cold fixer; a `lym drift check` verb is added.
- D11: amendments live in `scope.json`, and `--restore` is the way forward for a failed seal.
- D10: judged tuples live in a new tracked `judged.json`.
- D12: `defer` is admitted after the writer when the baseline directory is missing.
- D15: the model and interactive keys are renamed; `diff` and `key_patterns` are hub-only.
- Hub-level exclusions are not built.

## Open risks

- Parent work merged before a compare can move outputs relative to the branch-point baseline: tapestry's Parent-Align runs before Drift-Verify, so those cells show as unexpected and need a parent amendment; darn's Finalize merges the parent after Drift-Recheck, so no compare covers that merge. Drift today has the same exposure. Raised for the parent: whether to re-capture the baseline at the merged parent is a follow-up note.
- The `cs` hub's capture command passes `--skip-file {skip_file}`; until the operator edits it after D15's notice, its captures are refused.
- Webster's gate seam: the plan confirms the run-level entry and settles a card or batch gate key (operator's decision 3).
- `sampler-drift-trial` (board note) assumes the verbs this task removes; it is revisited after landing, per the board entry.
- A run past Drift-Expect when this lands keeps acceptances in `scope.json` that no code reads; with no seal it is checked against an empty expectation (D6), so any drift stops it at Drift-Verify, and the parent's way forward is `lym drift amend` (D11).

## Acceptance criteria

- D1: a diff of two capture directories yields `added case`, `removed case`, `changed`, `added line` and `removed line` cells; inserting a keyed line moves no other keyed or unkeyed identity; a repeated key is told apart as `key#n`; a line keyed on one side and unkeyed on the other yields a keyed `added line` or `removed line` and an unkeyed cell; an empty `key_patterns` makes every line unkeyed; CRLF and LF captures of the same text yield no cell; a non-case file under the capture directory is read as a case.
- D2: under `diff: command` a passing compare is Done with no judge; a failing compare spawns the judge with the log; an `unexpected` or `does not match` ruling is a Stuck; a missing `compare` is refused naming the key; the drift gate entry passes and reports the log path; under `lines` the compare command is never run.
- D3: `drift.scope: off` records scope `off` with its source and takes no capture; any other value, `partial` included, records `full`; a later config change does not change the recorded scope; the `docs` profile's darn block sets `drift.scope: off`.
- D4: both recipes list `Drift-Baseline`, `Drift-Expect` after `Drift-Await`, the writer, `Drift-Verify`, the perch and `Drift-Recheck` in that order; `Drift-Recheck` uses the `DriftVerify` engine and has no `on_stuck` route; a row is Done at once only at the head of the last passing check with no amendment since, and a resumed Stuck row at an unchanged head checks again; the registry has no `DriftRecheck` entry; Drift-Baseline spawns no agent.
- D5: the parser accepts `key`, `key#n`, closed and open ranges, `added` and `removed`; it refuses an unknown case, a `removed` case missing from the baseline, an unknown key not marked `added`, a malformed range and an empty `why`; `key` covers every occurrence and `key#n` one; a line entry covers no keyed cell; the expect agent is handed every baseline case's identity listing.
- D6: Drift-Expect makes one weft commit carrying `Drift-Seal`; a re-entered Drift-Expect with a seal commit spawns no agent, and one with a valid unsealed file seals it without spawning; an edited `expected.yaml`, an edited file with a hand-edited record, and a seal commit whose `Warp-SHA` differs from the baseline head each fail the seal check naming the failed check; a run with no seal commit and no file is checked against an empty expectation; the seal is checked in `command` mode too.
- D7: a changed cell outside the expectation stops Drift-Verify with a reason naming case, identity, old and new value; an expected entry with no cell is listed and does not fail; two concurrent checks each diff their own capture directory; a check writes no `capture.json` and leaves the baseline directory untouched.
- D8: the `drift` gate entry fails with the unexpected cells and passes when every cell is expected; it passes under scope `off`, a deferred capture and `command` mode; `lym drift check` prints the same findings; neither writes a tracked record.
- D9: the judge is spawned only when tuples with no stored ruling exist; the `judgement` entry refuses a missing, duplicate or unknown cell id and a missing open range in `lines` mode, and a missing case ruling in `command` mode; a `does not match` ruling stops Drift-Verify naming the cell; a tracked change the judge session made outside `judgement.yaml` is a Stuck, and a change present before the spawn is not.
- D10: a reproduced tuple reuses its stored ruling without a judge; a changed new value for the same identity is judged again; a stored `does not match` for a reproduced tuple stops the row without a judge.
- D11: `lym drift amend <slug>` from the parent at a Stuck drift row adds coverage that makes a formerly unexpected cell expected and drops the stored rulings of the cells it covers; it refuses an entry D5 refuses; `--restore` makes a failed seal check pass; it is refused from a non-parent session and at a running row, naming the way forward; the seal check passes after an amendment that adds entries.
- D12: after `lym drift defer`, Drift-Expect, the drift gate entry, Drift-Verify and Drift-Recheck pass without a capture, and the verdict states the deferral; a counted capture whose baseline directory is gone stops Drift-Verify naming `lym drift defer`; `lym drift waive` does not exist.
- D13: the verdict, rendered with no capture, lists scope and its source, diff mode, the last check's cell counts, uncovered entries, open ranges and amendments.
- D14: the removed verbs, records, `.lym/drift/` files, seams and engine no longer exist, and no code reads `baseline.md` or `verify-report.md`.
- D15: the template carries the kept, added and renamed keys; a base layer with the old model and interactive keys migrates their values to the new names; `run_settings` carrying `drift.skip` or `drift.verify_max_runs` is refused at seeding, and carrying `drift.diff` or `drift.key_patterns` is refused, each naming the way forward; the reinterpretation flags a set `compare` with no `diff` and a command holding `{skip_file}`; a capture command holding `{skip_file}` is refused naming the edit.
- Review: D16's `drift-template-expect` and `drift-template-judge` stencils say what D16 names, the darn writer's and Webster's stencils carry the drift paragraph, and no recipe comment names a removed verb.

## PATTERN changes

- `PATTERN-sole-parsers` — changes: Parsing plan, parts file, discussion, summary, recipe, halt note, raddle proposal, drift expectation or judgement, or classifying a plan ref: only its sole parser.

## Notes for the plan writer

- The `PATTERN-sole-parsers` line above assumes `raddle-recipe` lands its "raddle proposal" first; the card adds the two drift items to whatever list the line holds when it is edited.
- Slices that depend on nothing else: the diff with the expectation and judgement parsers; the fabric seams; the stencils; the verdict.
  Containment in Drift-Verify, the judge, the gate entries and the verbs wait for the diff; the deletions go last, once nothing reads the removed records.
- `shuttleengine.GateEntry` is the shape for the `drift`, `expectation` and `judgement` entries; `internal/loomcli/gatefixer.go` names the entries that carry fixers, and the `drift` entry is not one.

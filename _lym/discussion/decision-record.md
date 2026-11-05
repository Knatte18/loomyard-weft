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
  A key that occurs more than once in a file is identified by the key and its occurrence `n`, counted from 1 in file order.
  The key and the occurrence are separate fields wherever an identity is written, in the listing, `expected.yaml` and a Stuck reason, so a key holding `#` or digits, such as `C#` or `Test #3`, reads unambiguously.
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
  A nonzero exit spawns the judge (D9) with the compare's log, the sealed expectation and every amendment (D11); the judge rules per drifted case it reads in the log.
  Any ruling other than `matches` is a Stuck naming the ways forward D7 names.
  A `matches` ruling is not stored, so a failing compare at a head no passing check covers (D4) is judged again.
  A `does not match` or `unexpected` ruling is stored in `judged.json` keyed by (warp head, case) with its reason;
  a resumed row at that head with no amendment (D11) since is a Stuck naming the stored rulings, without a new judge.
  The expectation is per case, since code cannot read a case's identities here: Drift-Expect hands the expect agent the baseline's case list only, and `expected.yaml` holds, per case it expects to drift, one `why` and no `keys` or `lines` entries, beside `added` and `removed` cases each with a `why`.
  Its parser refuses a `keys` or `lines` entry and an empty `why`, and checks no case name against the baseline, since the capture's files need not be cases; an amendment (D11) is validated by the same rules.
  The writer's gate entry and `lym drift check` (D8) run the baseline check (D7), the records and seal checks (D6) and the compare, report any baseline, records or seal failure, and pass, reporting the compare's exit and log path, since only the judge can tell expected drift from a regression there.

Rationale: The board entry keeps an opaque compare with whole-test judgment for a hub whose capture cannot be line-diffed.
Bound: `command` mode gives up containment by code, stored `matches` rulings and any expectation finer than a whole case, and the judge sees only what the compare's log shows; a log that shows the first deviating line of a case, as the `cs` compare's does, hides a later regression in a case that already drifts.
Code cannot tell which cases drifted, so the judge's ruling covers only the cases it reads from the log, and a judge that omits a drifted case passes; the `judgement` entry refuses one that rules no case after a failing compare (D9).
Only the hub's `drift.yaml` sets `diff` (D15), and Drift-Baseline records it per run (D3).

### D3 — Config chooses the scope

Decision: Drift-Baseline records the scope once, from config: `off` when `drift.scope` is `off`, `full` otherwise.
Every later row reads the recorded scope, never config, so a run's scope does not change mid-run.
Drift-Baseline records `drift.diff` and `drift.key_patterns` in `scope.json` beside the scope;
every later row, the drift gate entry and `lym drift check` read the recorded values, never config, so a config change mid-run changes neither the identities nor the mode of the run.
A scope record with no recorded `diff`, from a run seeded before this lands, records config's values at its next drift row and keeps them; until then the drift gate entry and `lym drift check`, which write no tracked record (D8), read config.
No agent chooses the scope.
`partial`, `skip` and the `{skip_file}` placeholder are removed.
The drift config loader admits a stored `partial` and reads it as `full`, so a base not yet edited still loads; D15's reinterpretation flags it in a hub base.
A `drift.scope` of `partial` in a run's config is recorded as `full`, with one Warn naming the key.
A run whose scope record already says `off` or `full` when this lands keeps it.
A run whose scope record already says `partial` is read as `full`: the diff leaves out the cases its recorded skip list names, because the baseline holds none of them, and the verdict (D13) states the old `partial` scope and the cases left out.
The `docs` profile's `darn` block in `internal/runprofile/template.yaml` sets `drift: {scope: off}`.
The scope record names where `off` came from, the run's `drift.scope`, and the verdict states it (D13).

Rationale: The trial faults `drift-skip-all-capture` and `drift-baseline-skips-intended` came from an agent choosing the scope.
A run sees only config keys, never its profile's name (`internal/battencli/runsettings.go` resolves the profile at Seed-Child), so the `docs` profile turns Drift off through a config key.
Bound: a task's `run_settings` may still set `drift.scope: off`; that is the parent's or operator's call, recorded with its source and stated in the verdict.

### D4 — Rows and engines

Decision: Both recipes run, in this order:

1. `Drift-Baseline` (engine `DriftBaseline`): code only. It records the scope (D3) and, unless the scope is `off`, starts the detached capture as today. It spawns no agent.
2. `Drift-Await` (engine `DriftAwait`): unchanged, except that it records the baseline digest (D7) when it counts the capture.
3. `Drift-Expect` (new engine `DriftExpect` (new)): records a missing baseline digest (D7), spawns the expect agent (D5) and seals its file (D6). Done at once when the scope is `off`, the capture was deferred (D12) or a seal commit exists.
4. The writer (`Darn-Write` or `Webster`), with the drift gate entry (D8).
5. `Drift-Verify` (engine `DriftVerify`): capture, seal check, diff, containment, judge (D6, D7, D9, D10). It spawns no agent that edits code.
6. The Code-Review perch.
7. `Drift-Recheck`: the same `DriftVerify` engine in a second row.

Drift-Verify records each check's result in `scope.json`, committed as D6's records rule says: the warp head it checked, whether it passed (in `lines` mode every cell expected and ruled `matches`; in `command` mode the compare exited 0 or the judge ruled every case it read `matches`), and the counts and lists D13 renders.
After the records check (D6), either row is Done at once when the recorded scope is `off`, when the capture was deferred or the scope record holds an old waiver (D12), or when the warp head equals the head of the last passing check and no amendment (D11) was recorded since;
otherwise it runs the full pass, and only cells whose tuple has no stored ruling are judged.
Drift-Baseline and Drift-Verify, on entry and before their own work, end through `runproc.EndRole` any chair still registered for the run under the removed agents' roles, `drift-baseline` and `drift-verify`, so no pane keeps running the old stencil.
The new seats take roles of their own, `drift-expect` and `drift-judge`, so the sweep never ends an expect agent or a judge.
In tapestry, `Drift-Expect` sits after `Drift-Await` and before `Webster`; `Drift-Baseline` keeps its place before the parts and plan writers.
In darn, `Drift-Expect` sits after `Drift-Await` and before `Darn-Write`.
`Drift-Recheck` loses its `on_stuck` route to `Drift-Verify`: a Stuck at either row goes through the stop chain (`PATTERN-shed-stop-chain`).
The `DriftRecheck` constructor and engine are removed from `internal/shedrecipe`; the row name `Drift-Recheck` stays, so no rename or retirement is needed.
Every drift row keeps the entry gate `drift.enabled`.

Rationale: Drift-Expect runs after the capture so its agent writes against real case names and identities.
Keeping the existing row names keeps resume of in-flight runs simple: a run parked before `Drift-Expect` reaches it, and a run already past it has no seal and is checked against an empty expectation (D6).
Drift-Verify writes no code, so one engine serves both compare rows; skipping only after a passing check means a resumed Stuck row checks again.

### D5 — The expectation file

Decision: In `lines` mode Drift-Expect hands the expect agent, for every baseline case, a listing of the case's identities as D1 reads them, each with its text; `command` mode's per-case expectation is D2's.
The agent writes `_lym/drift/expected.yaml`.
Per case it lists:

- `keys`: identities it expects to change, be added or be removed, each with one `why`; an entry names a key and, optionally, an occurrence `n`: without `n` it covers every occurrence of the key, with `n` only that occurrence.
- `lines`: unkeyed positions or ranges, as `12`, `12-20` or the open range `12-`, each with one `why`.
- `added` and `removed`: the cases it expects added or removed, each with one `why`.

The file states what is expected and why, never what may be accepted, and carries no tolerance.
An empty expectation is valid.
One sole parser reads `expected.yaml` (`PATTERN-sole-parsers`) and validates it: a named case must exist in the baseline unless listed under `added`; a `removed` case must exist in the baseline; a `keys` identity of an existing case must exist in its baseline unless the agent marks it `added: true`; a range must be well formed; every entry has a non-empty `why`.
The expect agent's session carries a gate entry, `expectation`, that runs that validation; findings re-prompt the agent, and an exhausted entry is a Stuck through the stop chain, whose resume spawns the agent again.
The agent's model is `drift.expect_model`.

Rationale: A structured file lets code decide containment, and the listing makes the agent write against code's own identities, such as `Re` for the `cs` line above.
Open ranges are allowed without a halt, per the operator's decision 2.
Bound: an open range moves every unkeyed cell from its start to the end of the case out of containment by code, to the judge.
It is bounded by the judge ruling each covered cell, the `judgement` entry requiring the range to be listed (D9), and the verdict listing it (D13).

### D6 — The seal

Decision: When the expect agent's file validates, Drift-Expect commits it on the weft in one commit carrying a `Drift-Seal: <sha256 of the file>` trailer beside fabric's `Warp-SHA` trailer.
The seal is the oldest weft commit carrying `Drift-Seal`; no seal fields are kept in `scope.json`.
On re-entry, Drift-Expect is Done when a seal commit exists.
It seals an existing `expected.yaml` that validates without spawning the agent only when the expect session ended normally, as its session mark records: an `expect_session` field in `scope.json`, opened at the spawn and closed when the session returns.
An interrupted session, its mark still open, is probed first, resumed when live and respawned otherwise.
Drift-Expect refuses to seal when the warp head differs from the baseline head: a Stuck naming the commits made since the baseline head and two ways forward, the parent's `lym drift defer` (D12), or the operator resetting the run's warp branch to the baseline head in the run's worktree followed by `lym loom resume`.
Drift-Expect, when it is Done without sealing, and the first Drift-Verify or Drift-Recheck that finds a run past Drift-Expect with no seal commit record `unsealed` in `scope.json`; on a run so marked any later seal commit is ignored and the expectation stays empty.
Drift-Verify, in either diff mode, and the drift gate entry and `lym drift check` (D8) check the seal first:

- with a `--discard` amendment recorded (D11), the seal check is skipped and the expectation is empty;
- with no seal commit and no `expected.yaml` on a run past Drift-Expect, the expectation is empty, and the check goes on.
  A run is past Drift-Expect when its current row, read from the run status the drift deps already read, comes after Drift-Expect in the recipe's row order.
  Before that point `lym drift check` reports that no expectation is sealed yet and checks nothing;
- otherwise the seal commit must exist, its `Drift-Seal` hash must equal the hash of its own copy of `expected.yaml`, the current file must have that hash, and its `Warp-SHA` must equal the capture's baseline head, so the seal precedes every warp commit the writer makes.

A failed seal check is reported naming which check failed and its way forward; at Drift-Verify it is a Stuck naming the same way forward:

- an edited `expected.yaml` names the parent's `lym drift amend --restore` (D11), which writes the sealed copy back;
- a seal commit whose copy fails its own `Drift-Seal` hash, a seal commit whose `Warp-SHA` differs from the baseline head, and an `expected.yaml` with no seal commit on a run past Drift-Expect each name the parent's `lym drift amend --discard` (D11), after which the run is checked against an empty expectation and its amendments.

No session at the writer's row can run either verb, so the drift gate entry reports a seal failure without failing: it checks containment against the seal commit's copy when one exists, otherwise against an empty expectation, and leaves the stop to Drift-Verify; `lym drift check` reports it the same way.

The drift records, `scope.json` and `judged.json`, get the same guard.
Every write of either, by a drift row or by `lym drift amend` or `lym drift defer`, starts from the copies below and is committed at once in one weft commit carrying a `Drift-Records: <sha256 over scope.json and judged.json>` trailer, the hash covering each file's presence as well as its content.
That covers the recorded scope, `diff` and `key_patterns` (D3), the check results (D4), the session marks, the amendments (D11), a deferral (D12) and the stored rulings (D10).
The newest weft commit carrying `Drift-Records` holds the copies: every drift row, the drift gate entry, `lym drift check` and the verdict (D13) read the records from that commit's copies, never from the current files.
Drift-Verify, Drift-Recheck, the drift gate entry and `lym drift check` first run the records check: the current files must hash to that commit's trailer.
A mismatch is reported like a seal failure: at Drift-Verify and Drift-Recheck it is a Stuck naming the parent's `lym drift amend --restore` (D11), which writes both files back from that commit; the drift gate entry and `lym drift check` report it without failing and check against the commit's copies.
A records write interrupted before its commit shows as a mismatch, a Stuck only the parent or operator clears: `--restore` drops the write, and the row does that work again.
A run with no commit carrying `Drift-Records`, seeded before this lands, reads the files as they are until its first records write makes one, so a hand edit made before that write is committed as genuine.

The git reads and the trailer commits go through `internal/fabricengine` behind general seams that name no drift concept: commit given paths with an extra trailer, find the oldest or the newest weft commit carrying a trailer key, and read a file at a weft commit (`PATTERN-fabric-git`, `PATTERN-fabric-isolation`).

Rationale: The operator's decision 4: the hash catches an edited file, and the `Warp-SHA` shows the expectation existed before any code.
Every weft commit fabric makes carries `Warp-SHA` (`internal/fabricengine/trailer.go`).
The records decide containment as much as the seal does: a hand-added amendment or stored `matches` ruling, a hand-set last passing check head, deferral or scope `off` would pass a check that consulted neither the expectation nor the judge.
Bound: forging a seal or a drift record needs a hand-made weft commit carrying its trailer, for a seal one made before the writer's first warp commit; the checks cannot tell such a commit from one lym's verbs made;
on a run in flight with no `Drift-Records` commit when this lands, a records edit made before its first records write is committed as genuine;
on a run in flight past Drift-Expect when this lands, a seal forged before its first Drift-Verify marked the run `unsealed` is still read as the seal.
A rewrite of the weft history is outside the check.

### D7 — Containment by code

Decision: In `lines` mode Drift-Verify captures the current code, checks the baseline digest, the records and the seal (D6), diffs (D1) and classifies every cell:

- expected: covered by an entry of the sealed expectation or an amendment (D11), the sealed expectation counting as empty after a `--discard`;
- unexpected: covered by none.

Any unexpected cell is a regression: the row is Stuck, the reason naming each unexpected cell with case, identity, old and new value.
The reason names two ways forward: the parent's `lym drift amend` (D11) when the drift is intended, and a route that fixes code when it is a regression, whether the writer, Parent-Align's merge fixer or Code-Burler-Fix introduced it.
In darn that route is the parent's `lym loom goto --to Darn-Write` then `lym loom resume`, which re-enters the writer once per goto, its `drift` entry meeting the cells (D8).
Bound: `lym loom goto` keeps the settled verdicts of the review segments, so the writer's fix is checked again by Drift-Verify and Drift-Recheck but not reviewed again; the parent chooses the goto.
In tapestry the plan settles that route under the operator's decision 3, and the reason names it; the route must be reachable from a Stuck Drift-Verify and a Stuck Drift-Recheck, since Webster re-entered after its last batch has no card to run and `lym loom goto` refuses a fix row whose review verdict has settled.
No agent stands between the finding and the stop chain.
An expectation entry that covers no cell is reported in the verdict, never failed.
Every current capture, here and in D8, runs in a scratch directory of its own under `.lym/drift/`.
Every capture, the main one included, takes its gate slot before the capture lock: the slot first, or the caller's held slot inherited when it runs inside a gate, then the lock.
A check that finds the lock held waits for it rather than refusing.
A current capture writes no `capture.json` and never touches the baseline directory or its backups.
A current capture that fails or times out is a Stuck at Drift-Verify naming the capture's log and three ways forward: `lym loom resume` for a passing fault, the route that fixes code when the change broke the capture, and the parent's `lym drift defer` (D12) when the capture cannot run; at the drift gate entry and `lym drift check` it is a failure naming its log.

When Drift-Await counts the capture, it records a digest of the baseline directory's tree, every file's path and content, as a `Drift-Baseline: <digest>` trailer on the weft commit of its drift records; the digest is the oldest weft commit's carrying that trailer, read through D6's fabric seams, so no edit in the worktree can change it.
Every check, in either diff mode, at Drift-Verify, Drift-Recheck, the drift gate entry and `lym drift check`, first recomputes it.
A mismatch, or a missing baseline directory, stops the check and names the parent's `lym drift defer` (D12) as the way forward, since the branch-point code cannot be captured again from the moved worktree: at a row it is a Stuck, while the drift gate entry and `lym drift check` report it without failing and check nothing further, since no session at the writer's row can run `defer`, and leave the stop to Drift-Verify.
A capture counted before this lands, with no digest, gets one on Drift-Expect's seal commit; a run already past Drift-Expect with none is checked without this guard.

Rationale: Whether a drift occurred and whether it was expected are facts code can establish; trial 2's `oil-steel-6in-turbulent` showed an expected entry with no drift is normal.
The baseline lives outside git under `.lym`, where a writer handed an unexpected cell's old and new value could edit it to match; the digest, held in a weft commit made before the writer, turns such an edit into a stop instead of a pass.
Today's capture lock refuses after a bounded retry (`internal/driftengine/capture.go`), and the main capture takes the lock before its slot; under this order a check waits instead, and since the lock's holder already has its slot, a check waiting for the lock never blocks the holder, so concurrent checks, such as a Webster fork's `lym drift check` beside Merriam's gate, serialise without deadlock.
Bound: a check waiting for the lock holds its slot idle for the length of the other check's capture.
Inheriting a held slot keeps a check run inside Merriam's run-level gate from waiting on its own slot (`PATTERN-gate-slot`).

### D8 — The writer's drift gate entry and `lym drift check`

Decision: A `drift` gate entry runs D7's capture, records and seal checks, diff and containment, and returns the unexpected cells as findings and reports any records or seal failure (D6); it runs no judge and writes no tracked record.
It passes when the scope is `off` or the capture was deferred.
When the counted baseline directory is missing or its digest mismatches (D7), it and `lym drift check` report that without failing, naming the parent's `lym drift defer` (D12), and check nothing further.
In `command` mode it runs the baseline check (D7), the records and seal checks (D6) and the compare, reports any baseline, records or seal failure, and passes, reporting the compare's exit and log path (D2).
The entry is added to Darn-Write's `gates` list in `contracts/recipes/darn-recipe.yaml` and resolved by `internal/shedrecipe/entries_gate.go` like the existing plain entries.
At Darn-Write the entry carries no cold fixer: the writer edits code and, holding the context, fixes its own regressions.
For tapestry the entry goes on the Webster row's `gates` list in `contracts/recipes/tapestry-recipe.yaml`, which reaches Merriam's run-level gate.
Master edits no code and answers a run-level gate failure only through a fixer fork or `lym webster fix-verify`, driven by the verify gate's prompt file; a failed `drift` entry at Merriam's gate is cleared through those same paths, so its findings reach the fixer's prompt, and the plan wires that under the operator's decision 3.
The card and batch gate keys run only test tiers and the closed lints `comments` and `pattern`, so running the drift check from one needs a new mechanism; per decision 3 the plan reads Webster's gate seam and settles whether to build it.
A new `lym drift check` verb runs the same check and prints the findings, so any session in the run's worktree can run it mid-work, such as a Webster fork after a card.
Bound: `check` and the entry write only their scratch capture directory, and decide nothing: Drift-Verify repeats the check as the guarantee, against a baseline whose digest (D7) still matches the one recorded when it counted.

Rationale: At Darn-Write the writer meets "key `Re` of case X changed from `127069, dP = 7925.267 Pa` to its new value, outside the expectation" while it can still fix it, up to the entry's attempts.
Webster's run-level gate is fed by the Webster row's `gates` list (`internal/websterengine/runlevel.go`), so the recipe-level entry reaches Merriam's run-level gate, whose fixer acts on its findings.

### D9 — The judge

Decision: In `lines` mode, when every changed cell is expected and some cell's exact tuple has no stored ruling (D10), Drift-Verify spawns one judge agent.
It sees each such cell with an id, its old and new value, the covering entry's `why`, whether that entry is an open range, and the task's decision record or darn record; for an added case it sees the new file, for a removed case the old one.
It writes `_lym/drift/judgement.yaml`, in one of two forms:

- `lines` mode: one ruling per cell id, `matches` or `does not match`, each with a reason, and an `open_ranges` list naming every open range that covers a judged cell, each with a sentence on what the judge checked under it;
- `command` mode (D2): one ruling per drifted case read from the log, `matches`, `does not match` or `unexpected`, each with a reason.

One sole parser reads it; the judge's session carries a gate entry, `judgement`, that checks the form for the mode: every cell id has exactly one valid ruling and every covering open range is listed, or every case has a valid ruling and, after a failing compare, at least one case is ruled.
An exhausted `judgement` entry is a Stuck through the stop chain whose resume respawns the judge.
Before each spawn Drift-Verify removes any `judgement.yaml` and hands the judge a spawn id; the file carries that id, and the `judgement` entry refuses a file carrying any other.
The judge's session mark is a `judge_session` field in `scope.json`, opened at the spawn and closed when the session returns;
an interrupted judge, its mark still open, is probed, resumed with the same spawn id when live, and respawned otherwise.
The judge edits no code, no expectation and no record: Drift-Verify writes and commits its own records (D6) and opens the judge's mark first, then snapshots, in both checkouts, warp and weft alike, HEAD, the tracked tree and every untracked path outside ignore rules, keeping each such path's content as a copy beside the snapshot, and a list of path and content hash of the current capture's scratch directory (D7), from which the judge reads an added or removed case's file.
The snapshot and its copies live in a directory under the hub's board-level `.lym`, beside the run process record, under a name `internal/lymdirs` declares: the worktree's `.lym` is a link into the weft side's `.lym`, which the judge session writes through.
After the judge session returns and before closing its mark, Drift-Verify compares both checkouts and the scratch directory against the snapshot and recomputes the baseline digest (D7).
A moved HEAD in either checkout, or a change the session made in either tree or the scratch directory, other than `judgement.yaml`, is a Stuck naming the changed paths, the moved HEADs and the snapshot's directory, whose way forward is the parent or operator resetting a moved branch to its recorded HEAD, restoring a changed tracked file from that HEAD, restoring a modified or deleted untracked path from the snapshot's copy, deleting an added untracked path and deleting a changed scratch capture directory, then `lym loom resume`.
A baseline digest mismatch is a Stuck naming the parent's `lym drift defer` (D12), the way forward D7 names.
A resumed row compares against the recorded snapshot, never a new one, until the trees match it; a scratch capture directory that is gone is left out of that compare, since the resumed row captures into a new one.
When the snapshot is gone while the judge's mark is open, such as after `.lym` was cleaned, on another machine or after the judge deleted it, Drift-Verify records the loss in `scope.json` and is Stuck naming it, whose way forward is the parent or operator checking both checkouts for edits the judge made, then `lym loom resume`; a resume with the loss recorded takes a new snapshot before it resumes or respawns the judge.
Bound: ignored paths other than the scratch capture directory, and files outside both checkouts, are outside the guard; a baseline change is caught by the digest on a run that has one (D7), not by the snapshot.
Its model is `drift.judge_model`; with `drift.judge_interactive` set the judge presents each cell to the operator and records the operator's ruling.
Any `does not match` or `unexpected` is a Stuck naming the cells or cases, the reasons and the two ways forward D7 names: the parent's `lym drift amend`, which drops the covering rulings (D11), and the route that fixes code.

Rationale: The judge wrote neither the expectation nor the code, and rules against facts code produced.
Listing open ranges in the judge's own output meets the operator's decision 2.

### D10 — Judged tuples

Decision: Each `lines`-mode ruling is stored in a tracked record, `_lym/drift/judged.json`, committed and read as D6's records rule says, as its exact tuple (case, identity, kind, old value, new value) with ruling, reason, the judge's strand and time.
A later check that reproduces a stored tuple reuses its ruling; a cell whose tuple changed is judged again.
A stored `does not match` for a reproduced tuple is a Stuck without a new judge, naming the ways forward D9 names.
A `command`-mode ruling is stored as D2 says.
Beside the tuples, the record stores each open range the judge lists under `open_ranges` as (case, range, the judge's sentence on what it checked, strand, time), replaced when a later judge lists the same range.
The record is written through `internal/state` and tolerates unknown keys (`PATTERN-tolerant-records`).

Rationale: An acceptance covers exact values, so a later move of an accepted value is judged again while unchanged values cost nothing.

### D11 — Parent amendment

Decision: A new verb, `lym drift amend <slug>`, takes the run's slug and checks its caller as `lym loom decision add` does; it is admitted for the run's parent orch and the operator, at a Stuck writer, Drift-Verify or Drift-Recheck row.
It does one of three things:

- adds expectation entries in the D5 shape, each with a `why`, validated by D5's rules, or by D2's under `command` mode;
- `--restore` writes back whatever failed its check (D6): the seal commit's copy of `expected.yaml` over the current file after a failed seal check, and `scope.json` and `judged.json` from the newest `Drift-Records` commit after a failed records check;
- `--discard` sets the expectation aside: the seal check is skipped and containment reads an empty expectation and every amendment, the way forward for a seal check `--restore` cannot clear (D6).

An amendment is recorded in `scope.json` with who, when and its entries, restore or discard, and committed in a `Drift-Records` commit (D6); it never edits the seal commit.
Containment reads the sealed expectation, or an empty one after a `--discard`, and every amendment.
An amendment that adds entries drops the stored rulings of the cells they cover, so those cells are judged again against the amended `why`.
Every amendment is listed in the verdict (D13).
Bound: an amendment only adds coverage, restores the sealed file or sets the whole expectation aside, which only removes coverage; it cannot remove one entry or change the seal.
An amendment entry written into `scope.json` by hand is no amendment: D6's records check stops the row naming `--restore`.
An amendment reaches the judge as a covering entry, in `lines` mode and in `command` mode alike, where the judge reads the sealed expectation and every amendment beside the compare's log (D2).
It reaches no writer or expect agent.
Its caller check is bounded by the strand-name check and the shell guard that refuses setting lym's strand variables.

Rationale: The parent rules only on a changed expectation; the seal stays intact and the amendment is visible on the pull request.

### D12 — A deferred or missing capture

Decision: `lym drift defer` stays for a capture that cannot run, and is also admitted, for the run's parent and the operator, at a Stuck Drift-Expect, writer, Drift-Verify or Drift-Recheck row whose baseline directory is missing or whose baseline digest mismatches (D7), at a Stuck Drift-Expect whose warp head moved off the baseline head (D6), at a Stuck Drift-Verify whose capture never counted, and at a Stuck Drift-Verify or Drift-Recheck whose current capture failed (D7).
With a deferred capture, Drift-Expect is Done at once, the drift gate entry and `lym drift check` pass, and Drift-Verify and Drift-Recheck are Done with the verdict stating that no compare ran because the capture was deferred.
A run whose capture was counted but whose baseline directory is gone, such as after `.lym` was cleaned or on another machine, is Stuck naming `lym drift defer` as the way forward: at Drift-Expect, which needs the baseline for its identity listing (D5), or at Drift-Verify.
A run whose capture never counted, such as one that passed Drift-Baseline before Drift-Await joined the recipe, is Stuck at Drift-Verify naming `lym drift defer`.
`lym drift waive` is removed.
A scope record whose `WaivedAt` is set, from a run waived under the old Drift, is Done at Drift-Verify and Drift-Recheck, the verdict rendering the waiver (D13).

Rationale: `defer` is the parent's decision to go on without a baseline; with no acceptances to rule there is nothing left for `waive` to settle.
The baseline lives under `.lym`, so its loss must have a way forward.
Bound: `defer` after the writer skips the regression check for the rest of the run; only the parent or operator runs it, and the verdict states it.

### D13 — The verdict

Decision: `driftengine.Verdict` renders the pull request's and landing commit's `## Regression check` section from the copies of `scope.json` and `judged.json` in the newest `Drift-Records` commit (D6), or, on a run with no such commit, from the files as they are, without running a capture:

- the scope, and for `off` its source (D3); a deferred capture (D12);
- for a scope record that said `partial` (D3), the old scope and the cases left out;
- the diff mode, and in `command` mode that containment was the judge's;
- from the last check: changed cells, expected cells, cells judged as matching, cells reused from stored rulings, unexpected cells;
- every expectation entry that covered no cell;
- every open range with its `why` and the judge's sentence on what it checked under it (D10);
- every amendment with who and its entries, restore or discard.

A scope record with no new check result but with the old compare, acceptance or waiver fields, from a run that checked under the old Drift, renders those fields under a line saying the check ran under the old Drift.

Rationale: The pull request shows what Drift checked and what a human ruled, without reading run records.

### D14 — Removals

Decision: Removed with their tests and their stencil and recipe-comment text: `partial`, `skip` and `{skip_file}`; the `scope`, `accept`, `withdraw`, `waive` and `compare` verbs and `capture --side`; ruled and unruled acceptances and their halt marks; the run budget and `runs.json`, whose agent-session mark moves to `scope.json`'s `expect_session` and `judge_session` fields (D6, D9); side capture; the accepted profile, `RunAcceptanceCapture` and the `CompareAccepted` seam; `.lym/drift/`'s `accepting`, `accepted-tests` and `accept.lock`; the verify agent's dirty, verify and compare gate entries; the language skills and PATTERN directive on the Drift-Verify seat; `baseline.md` and `verify-report.md`; the `drift-template-baseline` and `drift-template-verify` stencils; the `DriftRecheck` engine; the `drift-verify` gate key, from the gateslot template and `goReadGates`, with the Drift-Verify rows' `gate_key`.
A hub's `gate.yaml` entry for `drift-verify` is left in place as one nothing reads, since `gates` is a per-name map whose names the operator owns.
D7's capture takes its gate slot from the pool without a gate key.
The `scope.json` model keeps read-only the old compare, acceptance and waiver fields and the old scope value `partial` with its `skip` field, for D3's in-flight rule and the verdict of a run that checked under the old Drift (D13).

Rationale: Each exists for an agent choosing scope, accepting whole tests or fixing code at Drift-Verify, which D3, D9 and D10 replace.

### D15 — Config keys

Decision: `drift.yaml`'s template after the change:

- kept: `enabled`, `capture`, `scope` (`""` or `full`, or `off`; a stored `partial` is admitted and read as `full`, D3);
- kept, used only under `diff: command`: `compare`;
- added: `diff` (`lines`), `key_patterns` (D1's two defaults);
- renamed, value carried over: `baseline_model` to `expect_model`, `verify_model` to `judge_model`, `verify_interactive` to `judge_interactive`;
- removed: `skip`, `verify_max_runs`.

The renames are the drift module's migration, as darn, gate, landing and loom migrate their keys; `lym config reconcile` prunes the removed keys (`PATTERN-config-layers`).
`skip` and `verify_max_runs` are declared retired keys under the category they held, `CategoryNone`, so a board entry whose `run_settings` still carries one seeds, and the load drops the key.
`diff` and `key_patterns` are refused in a task's `run_settings`, naming the hub's `drift.yaml` as their place.
A `configreg.Reinterpretation` on the drift module (`PATTERN-config-reinterpretation`):

- flags a hub base whose `compare` is set while `diff` is unset: `compare` now runs only under `diff: command`, and the hub's capture must meet D1's contract to keep the default;
- flags a `capture` or `compare` command holding `{skip_file}`, naming the edit;
- flags a hub base whose `drift.scope` is `partial`: `partial` is gone and the base now runs as `full`; the way forward is to set `scope` to `full` or `off` with `lym config drift`, and `lym config reconcile` prunes `skip`.

A capture or compare command still holding `{skip_file}` is refused at run time, naming the edit to the hub's `drift.yaml` as the way forward (`PATTERN-refusal-way-forward`).

Rationale: `compare` and the capture command change meaning for a hub configured today, such as `cs`, whose capture command passes `--skip-file {skip_file}`; a rewrite of a hub's command line would guess at its harness's flags, so the reinterpretation flags them instead.
Keeping `diff` and `key_patterns` out of `run_settings` keeps containment by code from being switched off per task.

### D16 — Stencils

Decision: Two new stencils, read at call time through `internal/stencilstore` (`PATTERN-stencil-ownership`):

- `drift-template-expect`: reads the identity listing, or under `command` mode the case list (D2), the capture files and the task's decision record and plan or darn record, writes `expected.yaml` per D5, states what is expected and why, never what may be accepted; a stated output invariant yields an empty expectation.
- `drift-template-judge`: rules each cell or case per D9 against its `why`, writes the spawn id it was handed into `judgement.yaml`, edits nothing else, and lists every open range with what it checked under it.

The darn writer's and Webster's stencils gain a drift paragraph: the path of the sealed `expected.yaml`, that the writer must not edit it, that the `drift` gate entry checks containment, and that `lym drift check` runs it on demand.
In the darn writer's, the writer acts on the entry's findings itself; in Webster's, Master hands a failed `drift` entry's findings to the fixer fork or `lym webster fix-verify` as for any run-level gate failure (D8), and a card fork may run `lym drift check` after its card.

Rationale: The writer never saw the expectation; now it reads it and meets the check in its own session.

### Added after Discussion (parent, 2026-11-05): Old drift chairs end through the strand table

Decision: Live chairs of the removed roles drift-baseline and drift-verify are ended through a Deps.EndRoles seam over reed's strand table, as conflictSessionStopper does, stopping only this worktree's strands whose role is exactly drift-baseline or drift-verify; this replaces D4's runproc.EndRole.

Rationale: No seat chair is registered in internal/runproc under its seat role, so runproc.EndRole would end nothing; the strand table reaches the live panes, with the same effect D4 wanted: no pane keeps the old stencil.

### Added after Discussion (parent, 2026-11-05): Short PATTERN-sole-parsers text

Decision: PATTERN-sole-parsers reads 'Parsing an on-disk format its background lists, or classifying a plan ref: only its sole parser.'; the list of formats lives in pattern/PATTERN-sole-parsers.md, which gains the drift expectation and judgement section.

Rationale: The declared text grew PATTERN.md 42 bytes past its headroom line; the list belongs in the background file.

### Added after Discussion (parent, 2026-11-05): Drift-Verify removes judgement.yaml after storing the rulings

Decision: Once the changes check passes, Drift-Verify stores the judge's rulings in judged.json and removes _lym/drift/judgement.yaml in the same step, on a Done and on a does-not-match Stuck alike; judgement.yaml is described as a file the judge writes, and TestDriftVerify asserts it is gone after the Done row and the does-not-match row.

Rationale: The stored tuples in judged.json are the record; an untracked judgement.yaml left in the tracked _lym/drift directory is stray state, and every spawn removes a stale one anyway.

## Constraints

- `PATTERN-sole-parsers`: `expected.yaml` and `judgement.yaml` each have one parser; consumers read its model.
- `PATTERN-tolerant-records`: `scope.json` and `judged.json` tolerate unknown keys; `expected.yaml` and `judgement.yaml` are agent-written contract files and stay strict.
- `PATTERN-durable-vs-ephemeral-state`: tracked drift records under `_lym/drift/`, capture directories and locks under `.lym/drift/`, the judge snapshot under the hub's board-level `.lym` (D9), all named only through `internal/lymdirs`.
- `PATTERN-shed-recipe-registry`: `DriftExpect` is a `Constructor` entry; the `drift`, `expectation` and `judgement` gate entries resolve through `internal/shedrecipe`; rows stay off through `drift.enabled`.
- `PATTERN-fabric-isolation`: D6's fabric seams name no drift concept.
- `PATTERN-refusal-way-forward`: every new refusal names its way forward, and a test reaches it.
- `PATTERN-parent-directive`: the judge and expect seats' skills and colour go on the launch spec.
- `PATTERN-cli-cobra`: `check` and `amend` are subcommands of the existing `lym drift` tree.

## Auto-mode assumptions

- D1: unkeyed lines pair by position among unkeyed lines; a repeated key is told apart by occurrence.
- D3: the `docs` profile turns Drift off through `drift.scope: off`; `partial` is recorded as `full`.
- D6: the seal is a trailer on one weft commit, not a record; the drift records are guarded by a `Drift-Records` trailer on the commit of each write.
- D8: Darn-Write's `drift` entry has no cold fixer; a `lym drift check` verb is added.
- D11: amendments live in `scope.json`, and `--restore` or `--discard` is the way forward for a failed seal.
- D10: judged tuples live in a new tracked `judged.json`.
- D12: `defer` is admitted at a Stuck row from Drift-Expect on when the baseline directory is missing or changed, and at a Stuck Drift-Verify whose capture never counted.
- D15: the model and interactive keys are renamed; `diff` and `key_patterns` are hub-only.
- Hub-level exclusions are not built.

## Open risks

- Parent work merged before a compare can move outputs relative to the branch-point baseline: tapestry's Parent-Align runs before Drift-Verify, so those cells show as unexpected and need a parent amendment; darn's Finalize merges the parent after Drift-Recheck, so no compare covers that merge. Drift today has the same exposure. Raised for the parent: whether to re-capture the baseline at the merged parent is a follow-up note.
- The `cs` hub's capture command passes `--skip-file {skip_file}`; until the operator edits it after D15's notice, its captures are refused.
- `sampler-drift-trial` (board note) assumes the verbs this task removes; it is revisited after landing, per the board entry.
- A run past Drift-Expect when this lands keeps acceptances in `scope.json` that only the verdict reads (D13); with no seal it is checked against an empty expectation (D6), so any drift stops it at Drift-Verify, and the parent's way forward is `lym drift amend` (D11).
  One whose scope record says `partial` is checked as `full` with its skipped cases left out of the diff (D3).
  One already past Drift-Recheck lands with a verdict rendered from its old compare, acceptance and waiver fields (D13).
  One parked with a live old Drift-Baseline or Drift-Verify agent has that agent ended at the row's entry (D4).
  One waived under the old Drift is Done at both compare rows (D12); one whose capture never counted is Stuck at Drift-Verify naming `lym drift defer` (D12).
  One whose capture counted before this lands and that is already past Drift-Expect has no baseline digest and is checked without that guard (D7).
  One with no `Drift-Records` commit reads its records unguarded until its first records write (D6).

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - d9-baseline-change-names-conflict (class consistency, severity MEDIUM, open in rounds 3)
  - judge-snapshot-misses-ignored-and-weft (class design, severity MEDIUM, open in rounds 2, 3)
  - mutable-records-bypass-seal (class design, severity BLOCKING, open in rounds 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: a diff of two capture directories yields `added case`, `removed case`, `changed`, `added line` and `removed line` cells; inserting a keyed line moves no other keyed or unkeyed identity; a repeated key is told apart by its occurrence; a line keyed on one side and unkeyed on the other yields a keyed `added line` or `removed line` and an unkeyed cell; an empty `key_patterns` makes every line unkeyed; CRLF and LF captures of the same text yield no cell; a non-case file under the capture directory is read as a case; a repeated key holding `#` and digits, such as `Test #3`, is written with its occurrence as a separate field and reads unambiguously.
- D2: under `diff: command` a passing compare is Done with no judge; a failing compare spawns the judge with the log, the sealed expectation and the amendments; an `unexpected` or `does not match` ruling is a Stuck; a missing `compare` is refused naming the key; the drift gate entry passes, reporting the compare's exit, its log path and any seal failure; under `lines` the compare command is never run; under `command` Drift-Expect hands the case list only, and the parser refuses a `keys` or `lines` entry and checks no case name against the baseline; a stored `does not match` or `unexpected` ruling at the same warp head, with no amendment since, stops a resumed row without a new judge.
- D3: `drift.scope: off` records scope `off` with its source and takes no capture; any other value, `partial` included, records `full`; the loader admits a stored `partial`; a later config change does not change the recorded scope, and a later config change of `diff` or `key_patterns` changes neither the identities nor the mode of the run; a scope record holding `partial` with a skip list is checked as `full` with the listed cases left out of the diff, and the verdict names them; the `docs` profile's darn block sets `drift.scope: off`.
- D4: both recipes list `Drift-Baseline`, `Drift-Expect` after `Drift-Await`, the writer, `Drift-Verify`, the perch and `Drift-Recheck` in that order; `Drift-Recheck` uses the `DriftVerify` engine and has no `on_stuck` route; a row is Done at once under a recorded scope `off`, after a deferral, and at the head of the last passing check with no amendment since, in `command` mode a check whose judge ruled every case `matches` counting as passing, and otherwise a resumed Stuck row at an unchanged head checks again, reusing stored rulings; the registry has no `DriftRecheck` entry; Drift-Baseline spawns no agent; Drift-Baseline and Drift-Verify end a live chair of the removed agents on entry.
- D5: the parser accepts a key with and without an occurrence, closed and open ranges, `added` and `removed`; it refuses an unknown case, a `removed` case missing from the baseline, an unknown key not marked `added`, a malformed range and an empty `why`; a key entry without an occurrence covers every occurrence and one with an occurrence covers that one; a line entry covers no keyed cell; the expect agent is handed every baseline case's identity listing.
- D6: Drift-Expect makes one weft commit carrying `Drift-Seal`; a re-entered Drift-Expect with a seal commit spawns no agent, one with a valid unsealed file and a normally ended expect session seals it without spawning, and one whose expect session was interrupted probes it before sealing; an edited `expected.yaml` fails the seal check naming `--restore`, and a seal commit whose `Warp-SHA` differs from the baseline head fails it naming `--discard`; a run whose current row comes after Drift-Expect in row order, with no seal commit and no file, is checked against an empty expectation, and one with a file and no seal commit fails naming `--discard`; before Drift-Expect `lym drift check` checks nothing and says so; the drift gate entry reports a seal failure without failing and checks containment against the seal commit's copy; the seal is checked in `command` mode too; a re-entered Drift-Expect reads its own `expect_session` mark; Drift-Expect refuses to seal when the warp head differs from the baseline head, naming the moved commits and `lym drift defer`; on a run marked `unsealed` a later seal commit is ignored; every write of `scope.json` or `judged.json` makes one weft commit carrying `Drift-Records`; a hand-edited amendment, last passing check head, deferral or scope in `scope.json`, or a hand-added stored ruling in `judged.json`, stops Drift-Verify naming `--restore`, the drift gate entry reports it without failing and checks against the commit's copies, and `--restore` writes both files back from the newest `Drift-Records` commit.
- D7: a changed cell outside the expectation stops Drift-Verify with a reason naming case, identity, old and new value, `lym drift amend` and the route that fixes code, in darn `lym loom goto --to Darn-Write` then `lym loom resume`; an expected entry with no cell is listed and does not fail; two concurrent checks each diff their own capture directory; a check writes no `capture.json` and leaves the baseline directory untouched; an edited baseline file stops Drift-Verify naming `lym drift defer`, and the drift entry reports it without failing; editing the worktree cannot change the recorded digest; a failed or timed-out current capture stops Drift-Verify naming its log, `lym loom resume`, the route that fixes code and `lym drift defer`; a check inside a gate and a fork's check with one free slot both finish.
- D8: the `drift` gate entry fails with the unexpected cells and passes when every cell is expected; it passes under scope `off` and a deferred capture; in `command` mode it passes, reporting the compare's exit and any seal failure; `lym drift check` prints the same findings; neither writes a tracked record; when the counted baseline directory is missing, the entry and `lym drift check` report it without failing, naming `lym drift defer`.
- D9: the judge is spawned only when tuples with no stored ruling exist; the `judgement` entry refuses a missing, duplicate or unknown cell id and a missing open range in `lines` mode, and a missing case ruling or, after a failing compare, an empty ruling list in `command` mode; a `does not match` ruling stops Drift-Verify naming the cell, `lym drift amend` and the route that fixes code; an exhausted `judgement` entry is a Stuck whose resume respawns the judge; a `judgement.yaml` left from an earlier spawn is removed before the next, and one carrying another spawn id is refused; an interrupted judge is probed, resumed when live and respawned otherwise; a change in either checkout's tracked tree or untracked paths, a modified or deleted untracked file, a commit on the warp or the weft, or a change in the current capture's scratch directory that the judge session made is a Stuck naming the changed paths and the restore, an edit of `scope.json` or `judged.json` by the judge included, a change present before the spawn is not, and a resumed row compares against the recorded snapshot; a baseline change made by the judge session stops Drift-Verify naming `lym drift defer`; the snapshot lives under the hub's board-level `.lym`; a snapshot gone while the judge's mark is open is a Stuck whose resume takes a new snapshot; a deleted scratch capture directory is left out of a resumed row's compare.
- D10: a reproduced tuple reuses its stored ruling without a judge; a changed new value for the same identity is judged again; a stored `does not match` for a reproduced tuple stops the row without a judge, naming `lym drift amend` and the route that fixes code; a stored ruling added to `judged.json` by hand stops Drift-Verify naming `--restore` and is never reused.
- D11: `lym drift amend <slug>` from the parent at a Stuck drift row adds coverage that makes a formerly unexpected cell expected and drops the stored rulings of the cells it covers; it refuses an entry D5 refuses; `--restore` makes a seal check failed by an edited file pass; `--discard` clears a seal check failed by a `Warp-SHA` mismatch or an unsealed file, the run then checked against its amendments alone; it is refused from a non-parent session and at a running row, naming the way forward; the seal check passes after an amendment that adds entries; each amendment is committed with a `Drift-Records` trailer, and an amendment entry added to `scope.json` by hand covers no cell and stops Drift-Verify naming `--restore`.
- D12: after `lym drift defer`, Drift-Expect, the drift gate entry, Drift-Verify and Drift-Recheck pass without a capture, and the verdict states the deferral; a counted capture whose baseline directory is gone stops Drift-Expect or Drift-Verify naming `lym drift defer`, and a Stuck Drift-Expect is Done after it; `lym drift waive` does not exist; a scope record with `WaivedAt` set is Done at Drift-Verify and Drift-Recheck with the verdict rendering the waiver; `lym drift defer` is admitted at a Stuck Drift-Verify whose capture never counted.
- D13: the verdict, rendered with no capture, lists scope and its source, diff mode, the last check's cell counts, uncovered entries, each open range with its `why` and the judge's sentence, and amendments; a scope record from `partial` names the cases left out; a scope record with only the old compare and acceptance fields renders them under a line saying the check ran under the old Drift.
- D14: the removed verbs, records, `.lym/drift/` files, seams and engine no longer exist, and no code reads `baseline.md` or `verify-report.md`; the session marks live in `scope.json`; nothing reads a hub's `gate.yaml` entry for `drift-verify`.
- D15: the template carries the kept, added and renamed keys; a base layer with the old model and interactive keys migrates their values to the new names; `run_settings` carrying `drift.skip` or `drift.verify_max_runs` seeds with the key dropped, and carrying `drift.diff` or `drift.key_patterns` is refused naming the way forward; the reinterpretation flags a set `compare` with no `diff`, a command holding `{skip_file}` and a base holding scope `partial`; a capture command holding `{skip_file}` is refused naming the edit.
- Review: D16's `drift-template-expect` and `drift-template-judge` stencils say what D16 names, the darn writer's and Webster's stencils carry the drift paragraph, and no recipe comment names a removed verb.

## PATTERN changes

- `PATTERN-sole-parsers` — changes: Parsing an on-disk format its background lists, or classifying a plan ref: only its sole parser. — added after Discussion (parent, 2026-11-05)

## Notes for the plan writer

- The `PATTERN-sole-parsers` line above assumes `raddle-recipe` lands its "raddle proposal" first; the card adds the two drift items to whatever list the line holds when it is edited.
- Slices that depend on nothing else: the diff with the expectation and judgement parsers; the fabric seams; the stencils; the verdict.
  Containment in Drift-Verify, the judge, the gate entries and the verbs wait for the diff; the deletions go last, once nothing reads the removed records.
- `shuttleengine.GateEntry` is the shape for the `drift`, `expectation` and `judgement` entries; `internal/loomcli/gatefixer.go` names the entries that carry fixers; the `drift` entry is not one at Darn-Write, and how its findings reach Merriam's fixer is the plan's (D8).

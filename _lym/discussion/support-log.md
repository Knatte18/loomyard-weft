# Support log — drift-rework

## Interview

The run is autonomous: no operator turns.
The board entry carries the operator's four decisions of 2026-11-05, which bind this record.

### Turn 1 — board and research

Read the board entry, `sampler-drift-trial`, and `docs/research/drift-whole-review-2026-11-04.md`.
No `Decision for drift-rework:` note exists.
The entry's design and the operator's decisions 1, 2 and 4 are taken as given; decision 3 leaves the tapestry fix loop to the plan.

### Turn 2 — exploration by five subagents

- Drift module: four producers, `scope.json` tracked and capture records ephemeral, nine verbs, Drift-Verify's gate table of `dirty`, `verify` and `compare` entries, `driftengine.Verdict`, two stencils.
- Recipes: row order in both recipes; `Drift-Recheck` routes `on_stuck` to `Drift-Verify`; row names are durable, with `RenameProducers` and retired rows for renames and drops; a run sees config keys, never its profile name.
- Writer gates: Darn-Write's `gates` list in the recipe resolves through `internal/shedrecipe/entries_gate.go`; the Webster row's `gates` list feeds Merriam's run-level gate; card and batch gates are hub `gate.yaml` keys a fork runs itself.
- Config: the template's keys; an existing drift reinterpretation; reconcile prunes only on `--apply`; `lym loom decision add` as a model for an amendment verb; the `cs` capture writes `key = value` lines and passes `--skip-file {skip_file}`.
- Seal: drift records sit on the weft and are committed at each drift producer's exit; every fabric weft commit carries `Warp-SHA`; no guard keeps an agent from editing `_lym/drift/`.

### Turn 3 — design picks

All picks are auto-picks; see the question ledger.

### Turn 4 — lens review

Six forks returned reports: interactions, existing-paths, bound-breaker, walkthrough, board-scope and leanness. None was missing.
Duplicates are merged; each line names the lenses that raised it.

- interactions 2, walkthrough 1, bound-breaker 1 (no seal on a run past Drift-Expect stops before the empty-expectation path): fixed; D6 reads no seal commit and no file past Drift-Expect as an empty expectation, and D8 no longer refuses before a seal.
- interactions 3, walkthrough 1, bound-breaker 5 (an amendment cannot clear a failed seal): fixed; `lym drift amend --restore` writes the sealed copy back, and the seal lives in a trailer commit, so no record field can be mismatched.
- interactions 8, existing-paths 2, walkthrough 6, bound-breaker 6 (two commits, restart between them): fixed; the seal is one weft commit with a `Drift-Seal` trailer, and re-entry seals a valid file or is Done.
- bound-breaker 15 (forge by editing file and record before the first warp commit): fixed by the trailer seal; the bound now names a hand-made weft commit.
- interactions 1, existing-paths 1 (Drift-Recheck skips a resumed Stuck at an unchanged head): fixed; D4 skips only at the head of the last passing check with no amendment since.
- walkthrough 4 (verdict has no stored source): fixed; D4 records each check's result in `scope.json`, and D13 renders from it without a capture.
- interactions 4, walkthrough 5, bound-breaker 14 (amend at a Stuck Drift-Expect; amendment validation): fixed; Drift-Expect dropped from amend's rows, and amendment entries go through D5's rules.
- interactions 5 (no seal check in `command` mode): fixed; D6 checks the seal in either mode.
- interactions 6 (judge output undefined for `command` mode): fixed; D9 gives both forms and their gate checks.
- interactions 7, bound-breaker 7, existing-paths 7 (concurrent checks share one scratch directory and may touch `capture.json`): fixed; D7 gives each check its own directory under the capture lock and forbids `capture.json` and baseline writes.
- bound-breaker 8 (gate slot deadlock when a gate key runs the check): fixed; D7 inherits a held slot.
- existing-paths 3 (parent cannot reach `amend` from its own worktree): fixed; `amend` takes the slug and checks its caller as `lym loom decision add` does.
- existing-paths 4 (`run_settings` can switch `diff` and `key_patterns`): fixed; D15 refuses both there, and D3 states the `scope: off` bound.
- existing-paths 5, bound-breaker 13 (`run_settings` carrying removed keys): fixed; D15 declares `skip` and `verify_max_runs` retired under `CategoryNone`, so the load drops them (corrected in review round 1).
- existing-paths 6 (recipe comments name removed verbs): fixed; D14 and the review criterion cover recipe comments.
- bound-breaker 2 (counted baseline lost under `.lym`, `waive` gone): fixed; D12 admits `defer` at a Stuck row with a missing baseline.
- bound-breaker 2, second half (old run whose agent chose `off`): fixed; D3 makes later rows read the recorded scope, which an old run keeps.
- walkthrough 2, bound-breaker 12 (`Re …, dP …` keys as `Re`; misspelled key): fixed; D5 hands the agent the identity listing and refuses an unknown key not marked `added`.
- walkthrough 3, bound-breaker 3 (unkeyed lines shift after a keyed insert; mixed lines undefined): fixed; D1 numbers unkeyed lines among themselves and classifies each side's line on its own.
- bound-breaker 4 (repeated key coverage too wide): fixed; D5 adds `key#n`; occurrence shifts are re-judged (interactions 9, noted in D1's rationale).
- bound-breaker 9 (judge blamed for a tree dirty before it spawned): fixed; D9 snapshots before the spawn.
- leanness 1, bound-breaker 10 (garbled `compare` rule in D2): fixed.
- walkthrough 8, bound-breaker 11 (`command` mode judge sees only the log's first deviating line): fixed; D2's bound says so.
- walkthrough 7 (`check` under `off` passes and refuses): fixed by removing the refusal.
- board-scope 1 (PATTERN line drops raddle's addition): fixed; the text includes "raddle proposal" and a plan note says to add to the line as landed.
- board-scope 2 (open ranges in the judge's own verdict): fixed; `judgement.yaml` lists every covering open range, checked by its entry.
- board-scope 3 (does the capture directory hold only case files): fixed; D1 says every file is a case.
- bound-breaker 16 (darn Finalize merge uncovered): fixed; the Open risks line covers darn.
- leanness 2 to 6 (repeated notes, D14's sequencing sentence, D9's rationale repeat, Auto-mode restating, pointer-only Constraints): fixed; cut or shortened to D-id pointers.
- leanness 7 (Scope "In" lists decisions): rejected; the section is required and short.
- leanness 8 (in-flight risk line): fixed; it now names the parent's way forward.

## Rejected alternatives

- Pairing unkeyed lines by a sequence diff (LCS): heuristic pairings change with small edits; pairing by number plus open ranges is deterministic.
- Keeping `compare` as an extra check under `lines` mode: the `cs` compare exits 1 on any drift, expected drift included, so it would fail every task that expects drift.
- Rewriting a hub's capture command to drop `{skip_file}`: the flag before the placeholder is the harness's own and cannot be removed safely by code.
- A cold fixer on Darn-Write's `drift` entry: the writer edits code and holds the context; a fixer adds a session for no gain.
- An automatic `on_stuck` route from Drift-Verify back to the writer row: Webster re-entered after its last batch has no card to run; the stop chain already routes a Stuck, and in darn the parent's `lym loom goto` re-enters the writer (D7).
- Keeping amendments inside `expected.yaml`: it would break the seal on every amendment.
- Hub-level exclusions of cases or lines: no hub needs one, and the capture owes deterministic output.
- Keeping `baseline_model` and `verify_model` names: they would name agents that no longer exist.
- Seal fields (hash, commit, warp head) in `scope.json`: two commits with a restart gap, and an edit of file and record together could fake it; a trailer on the sealing commit has neither fault.
- Numbering unkeyed lines by absolute line number: a keyed insert shifts every unkeyed line below it.
- Admitting `amend` at a Stuck Drift-Expect: an amendment cannot repair or seal an invalid file; the stop chain's resume respawns the agent.
- Recapturing a lost baseline at the branch point: the worktree has moved past it; `defer` is the honest way forward.

## Review rounds

### Round 1

Every finding was fixed; the tapestry halves of two, the wiring of Master's fixer for the `drift` entry and the code-fixing route from a Stuck drift row, are handed to the plan under the operator's decision 3. See `_lym/reviews/discussion/round-1-fixer-report.md`.
The main changes: a reinterpretation and a run rule for `partial` (D3, D15); `--discard` as the way forward for seal failures `--restore` cannot clear, and a gate entry that reports a seal failure without failing (D6, D11); the command-mode seal check and pass rule aligned (D2, D4, D8); the tapestry `drift` entry's findings go to Master's fixer paths (D8, D16); a code-fixing way forward for an unexpected-cell Stuck (D7); the judge's snapshot covers both checkouts' HEADs (D9); an identity's occurrence is a separate field (D1, D5).

### Round 2

Every finding was fixed; see `_lym/reviews/discussion/round-2-fixer-report.md`.
The main changes: a baseline digest held in a weft trailer from Drift-Await and checked before every diff, with `lym drift defer` as the way forward (D7, D8, D12); per-seat session marks in `scope.json` replace `runs.json`'s (D6, D9, D14); `diff` and `key_patterns` recorded per run (D3); a per-case expectation and stored failing rulings under `command` mode (D2); a wider judge snapshot recorded for resume, a spawn id on `judgement.yaml` and an interrupted-judge rule (D9); a way forward at every Stuck; the capture lock taken after the slot and waited on (D7); in-flight runs with a live old agent, a waiver or an uncounted capture (D4, D12).

### Round 3

Every finding was fixed; see `_lym/reviews/discussion/round-3-fixer-report.md`.
The main changes: `scope.json` and `judged.json` committed with a `Drift-Records` trailer at every write and read from that commit's copies, with `lym drift amend --restore` as the way forward for a mismatch (D4, D6, D10, D11, D13); the judge snapshot moved under the hub's board-level `.lym`, extended to untracked copies and the scratch capture directory, with a rule for a lost snapshot (D9); the baseline digest recomputed after the judge returns, naming `lym drift defer` (D9).

## Question ledger

- Q1 — How are unkeyed lines paired? Auto-pick: by position among unkeyed lines; open ranges cover inserted lines. Resolved (D1), revised after the lens review.
- Q2 — How is a repeated key told apart? Auto-pick: by occurrence number in file order. Resolved (D1).
- Q3 — How does a `docs` run turn Drift off when code never sees the profile name? Auto-pick: the `docs` profile's darn block sets `drift.scope: off`. Resolved (D3).
- Q4 — What happens to `partial` in a seeded run? Auto-pick: read as `full` with a Warn; a recorded `partial` leaves its skipped cases out of the diff, and the reinterpretation flags a hub base holding it. Resolved (D3, D15), revised in review round 1.
- Q5 — What does `command` mode keep? Auto-pick: the compare command, with the judge reading its log per case, a per-case expectation, and only failing rulings stored, keyed by warp head. Resolved (D2), revised in review round 2.
- Q6 — Does the writer's drift entry carry a cold fixer? Auto-pick: no. Resolved (D8).
- Q7 — Is a `lym drift check` verb worth adding? Auto-pick: yes, so Webster forks and writers can check mid-work. Resolved (D8).
- Q8 — Where do amendments and rulings live? Auto-pick: amendments in `scope.json`, rulings in a new `judged.json`. Resolved (D10, D11).
- Q9 — How strong is the seal? Auto-pick: one weft commit carrying a `Drift-Seal` hash trailer, whose copy and the current file must match it and whose `Warp-SHA` must equal the baseline head. Resolved (D6), revised after the lens review.
- Q10 — What replaces `waive` after a deferred capture? Auto-pick: nothing; a deferred capture skips the compare and the verdict says so. Resolved (D12).
- Q11 — Rename the model keys? Auto-pick: yes, through the module's migration. Resolved (D15).
- Q12 — Should the baseline be re-captured after parent work is merged? Open: recorded under Open risks for the parent.
- Q13 — What clears a failed seal check? Auto-pick: the parent's `lym drift amend --restore` for an edited file, `--discard` for a mis-sealed or unsealed file. Resolved (D6, D11), revised in review round 1.
- Q14 — What happens when a counted baseline directory is lost? Auto-pick: Stuck naming `lym drift defer`, admitted there for the parent. Resolved (D12).
- Q15 — May `run_settings` set `diff` or `key_patterns`? Auto-pick: no; hub-only. Resolved (D15).

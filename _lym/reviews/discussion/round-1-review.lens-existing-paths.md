```yaml
lens: existing-paths
outcome: reported
```

**Existing-paths lens — paths traced**

D1 (kept directory):
- `statusCommitPathspec` (stages dir when it holds a file) — survives.
- `PathsOutsideRunRecords` (batten re-step dirty check) — survives once the dir joins the records set.
- teardown `commitPendingRecords` (stages structural `_lym` whole) — survives.
- Finalize's `pushRecords` via `CommitStatus` — survives only if the index row is written before it; the record does not order them (see F5).
- atomic writes into the dir via `fsx.AtomicWriteBytes` (`.tmp-*` sibling) vs a concurrent whole-dir `git add` — breaks (F6).

D2 (`lym loom pr-findings`):
- send-back path, re-run of a send-back (early return on `rec.SendBack`) — survives.
- re-run of a fix-here answer after a failed `deps.commit`, or of an already counted set — breaks (F7).
- `pullRequestRecordCommitter` (commits only the PR record) — survives once its pathspec is extended as D2 states.

D3 (parent-review ask record):
- every writer goes through `state.UpdateJSON` in `burlerengine` (`RecordParentAsk`, `MarkParentAskPosted`, `EndParentAsk`, `ClearParentAsk`, `SettleParentAsk`, `AnswerParentAskOrUnreached`, `ExpireParentAsk`), callers in parent seat, PR-Review ask, drift judge, drift verify's clear, and `lym loom parent-review` from the orch with the target's location — survives.
- no reset of `.lym/parent-review` exists — survives.
- a run moved to another machine (takeover/adopt): `.lym` starts empty while `_lym` comes from origin — breaks (F3).

D4 (landing fixer report):
- `fixerRun.run` single session spanning many turns writing one `reportPath` — breaks (F1).
- attempt record resets: new row entry, `RemoveFixerAttempts`, escalate decrement, start-failure restore — breaks (F2).
- `archiveStaleReport` already keeps each earlier session's report under a timestamped name — contradicts D4 rationale (F8).

D5 (index row):
- `recordLanding` reads `HeadSHA` at call time; writes only to prime `.lym` scratch — breaks on resume (F4).
- `dropBeamRecord` after Done — breaks PR URL (F5).
- Drift-Recheck tracked records read before Finalize — survives.

D6 (`lym report landed`):
- `selectRuns`/`readLoomRecords` already split partial runs (`noFinalizeReason`); D6 silent on how such a tag is shown — F9.
- `TagRefs("archive/")` skips `archive-owner/` — survives.

D7 (archive tags):
- `Repo.Fetch --prune` prunes remote-tracking refspecs only, `--no-tags` — survives.
- `PlainArchiveTagsOnOrigin` only lists and prints an operator command; no code deletes a tag — D7's exception misdescribed (F10).

**Findings**

- **F1 — BLOCKING, design, D4.** One landing fixer *session* spans every turn left: `fixerRun.run` gives a single seat run `Gate{Attempts: turnsLeft}` and re-prompts on a failing gate in the same session, while each turn overwrites the same `&lt;row&gt;-fixer-report.md`. A copy "after each landing fixer session ends" keeps only the last turn's report; D4's own criterion ("two attempts in one row, the first failing, leaves two tracked reports") fails. Evidence: `internal/landingshed/fixerrun.go:105,142-145,317-334`.

- **F2 — MEDIUM, design, D4.** The attempt number in the copy's name is not monotonic, so one copy overwrites another: `ReadFixerAttempts` returns a zero record for a new row entry (`Spent` restarts at 0), `RemoveFixerAttempts` deletes the record (and its halt's way forward tells the operator to remove it), an escalate does `Spent--`, and a failed start writes back the original count. Evidence: `internal/landingshed/fixerattempts.go:64-83,95`, `fixerrun.go:150,296`.

- **F3 — MEDIUM, design, D3.** The live ask record is machine-local (`.lym/parent-review/asks.json`), and no path seeds it from the tracked copy. After a takeover/adopt on another machine, `RecordParentAsk` numbers from 1 again (`Latest()` on an empty record), so D3's whole-file copy overwrites the tracked history that came from origin, and `ask-1`'s report copy overwrites the earlier `ask-1`. Evidence: `internal/burlerengine/parentreview.go:146-149,204-211`.

- **F4 — BLOCKING, design, D5.** "Every field comes from durable records" is false for the landing commit and its time: `recordLanding` reads the parent pair's `HeadSHA()` when called, and `RecordLanding` writes only the prime's `.lym` scratch (`shedrun.RecordLandedCommit`), neither tracked nor durable. A Finalize resumed after the parent moved on (other runs landed) reads another commit, so the resumed row is not byte-identical — D5's acceptance criterion breaks. Evidence: `internal/landingshed/finalize.go:738-753,853-854`, `internal/loomcli/landingdeps.go:249-256`.

- **F5 — MEDIUM, design, D5.** "The last step of Finalize" falls after `pushRecords`, `markTaskDone` and `dropBeamRecord`. A beam landing's pull request record is dropped once Done is answered, so a row written last has lost the PR URL; and a row written after `pushRecords` misses Finalize's own records push, depending on the Friction-Reflect transition or teardown's commit to reach origin. The record names neither ordering. Evidence: `internal/landingshed/finalize.go:419-424,859-865`.

- **F6 — MEDIUM, design, D1/D3.** Atomic writes do not keep a concurrent status commit from staging partial content: `fsx.AtomicWriteBytes` creates `.tmp-*` in the target dir, and the status commit `git add`s the whole dir. No `.gitignore` covers `.tmp-*`, so a commit landing between create and rename stages a half-written temp file (then a deletion on the next commit), contradicting D3's stated guarantee. Evidence: `internal/fsx/fsx.go:57-72`, `internal/loomcli/wiring.go:96-101`.

- **F7 — LOW, design, D2.** "Each [copy] takes the next free number" makes every re-run of the verb — the re-run its own failed-commit refusal asks for, or an agent re-running over an already counted set (`counted` == true) — write a new duplicate copy; no bound on duplicates is stated. Evidence: `internal/loomcli/prfindings.go:204-208,224-228`.

- **F8 — LOW, consistency, D4 rationale.** "each attempt overwrites" the one scratch report — but `archiveStaleReport` already renames an earlier session's report to a timestamped sibling, so overwriting happens only between turns of one session. Evidence: `internal/landingshed/fixerrun.go:129,196-203`.

- **F9 — MEDIUM, design, D6.** `reportengine`'s existing tag walk already classifies tags that are not landed runs (cancelled, no Finalize) into `Partial` with a reason, but D6 lists every archive tag and fills a missing index row with a derived landing line, without saying how a cancelled run's or orphan pair's tag shows; such tags read as landed. Evidence: `internal/reportengine/runs.go:153-167`, `internal/battencli/cancel.go:152,822`.

- **F10 — LOW, consistency, D7.** D7 exempts "the existing cleanup in `cleanupremotewarp.go`" as a deleter of archive tags, but `PlainArchiveTagsOnOrigin` only lists the plain tags and returns a way-forward command for the operator; no code path deletes an archive tag at all. Evidence: `internal/fabricengine/cleanupremotewarp.go:336-379`, `internal/fabriccli/fabric.go:906-911`.

Tool calls:
- Read `internal/state/state.go` `WriteJSON`/`WriteFile`, `fsx.AtomicWriteBytes`, and `.gitignore` entries for temp files — context lacked where atomic writes place their temp file.

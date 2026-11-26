```yaml
lens: bound-breaker
outcome: reported
```

**Claims attacked (bound-breaker lens)**

- D1 "outside the reviews root, so the segment walk never reads it" — held (`readSegments` walks under the reviews root, `internal/reportengine/rounds.go:155-157`).
- D1 "joins `statusCommitPathspec` and `PathsOutsideRunRecords` so a file there never blocks re-step" — held as design; but a leftover atomic-write temp file under it is committed as junk — F1.
- D2 "every findings file passes through this one verb" — broken (F6).
- D2 "no copy overwrites another; one shared sequence" — held for overwrite, broken for duplication on retry (F6).
- D2 "the verb already commits on both paths" — held (`answer()` calls `deps.commit` on both, `internal/loomcli/prfindings.go:204-209`, `:268`).
- D3 "one locked update path every write goes through, from any process" — held (all writers are `state.UpdateJSON` closures in `internal/burlerengine/parentreview.go:204-444`; orch-side answer resolves target anchor, `internal/loomcli/parentreview.go:400`).
- D3 "written atomically through `internal/state`, so a concurrent status commit never stages a half-written file" — broken (F1).
- D3 "copy keeps the record's history whatever process writes" — broken by owner-machine move (F2).
- D3 "last write under the lock leaves the final state" — weakened (F7).
- D4 "copy at each session end keeps every attempt" — broken (F3).
- D4 "named by row and attempt number" (no overwrite) — broken (F3).
- D4 rationale "each attempt overwrites the one scratch path" — partly false (F3).
- D5 "resumed Finalize rewrites the row byte-identically, every field from durable records" — broken (F4).
- D5 "the landing commit's message is the Describe text as merged" — partly broken (F8).
- D5 "last step, after the landing and any later fixer" — broken (F8).
- D5/D6 "the archive tag is already the per-run list of landed work" — broken (F5).
- D6 "`reportengine.Open` already enumerates the tags" — held (`internal/reportengine/source.go:77`).
- D6 bound "writes only under `&lt;dir&gt;`, never fetches, never writes a ref" — unbounded for symlinks / empty-symlink `&lt;dir&gt;` (F9).
- D6 `--tag` picks between two tags of one run — unbounded for another slug's tag / non-archive ref (F9).
- D7 "no code deletes an `archive/` tag" — held (no tag deletion; pruning fetch uses remote-tracking refspecs only, `internal/gitrepo/pull.go:70-74`).
- D7 carve-out "the existing cleanup in `cleanupremotewarp.go`" deletes plain-namespace tags — broken (F10).
- Constraint "teardown's commit of uncommitted records runs before the tag on every teardown path" — held for `Remove` (`commitPendingRecords` stages structural `_lym` before `tagWeftTipForArchive`, `internal/fabricengine/remove.go:307-335`, `internal/fabricengine/junctionnames.go:26`); local mode pushes no `archive/` tag by design.

**Findings**

F1 — BLOCKING, design, D3 (also D1/D2/D4/D5). Atomic writes do not keep a temp file out of a concurrent status commit. `fsx.AtomicWriteBytes` (used by `state.WriteJSON`/`WriteFile`) creates `.tmp-XXXX` *in the target directory itself*, and that file is not git-ignored in the records worktree (`git check-ignore` on `_lym/x/.tmp-123` exited 1). The status commit does `git add -- &lt;kept dir&gt;` while D3's writer (the orch's answer process) is mid-write, staging the half-written temp file under its temp name; the rename then leaves a deleted/orphaned path in the commit. A writer killed between create and rename leaves a `.tmp-*` file the defer never removes, and the next transition commits it. The record states the guarantee as fact with no pin (unpinned tool fact too). Evidence: `internal/fsx/fsx.go:57-74`, `internal/state/state.go:30-58`, `internal/loomcli/wiring.go:99-101` (directories staged whole).

F2 — MEDIUM, design, D3. The live ask record sits under machine-local `.lym` (`internal/burlerengine/parentreview.go:146-149`). After an adopt/takeover on another machine (PATTERN-owner-machine), the new owner starts an empty record and `RecordParentAsk` numbers from 1 again (`parentreview.go:206-209`). D3's next locked write then copies the whole file over the tracked copy — dropping every earlier machine's asks — and writes `ask-1`'s report copy over the earlier `ask-1` copy. Neither the copy names nor the merge rule are bounded against a restarting record.

F3 — MEDIUM, design, D4. One fixer *session* runs several turns: a failed gate with turns left charges a turn and re-prompts inside the same session, and every turn writes the same `reportPath` (`internal/landingshed/fixerrun.go:327-334`, `:105`, `:143`). A copy at session end keeps only the last turn's report, so D4's own criterion ("two attempts in one row, the first failing, leaves two tracked reports") fails for the ordinary in-session retry. Attempt numbers also repeat: `ReadFixerAttempts` returns a zero record for a new row entry, restarting `Spent` at 1 (`internal/landingshed/fixerattempts.go:79-81`); an escalate takes its turn back (`fixerrun.go:296`); a start failure restores the original record (`fixerrun.go:150`) — a later copy then reuses `row`+attempt and overwrites an earlier one. The rationale "each attempt overwrites" is partly false: each new session already renames the earlier report to a timestamped name (`fixerrun.go:129`, `:196-203`).

F4 — BLOCKING, design, D5. The landing commit is not a durable record. `recordLanding` reads the parent pair's current `HeadSHA()` (`internal/landingshed/finalize.go:738-746`) and stores it only in the prime's scratch via `RecordLandedCommit` (`.lym`, machine-local; `internal/loomcli/landingdeps.go:249-256`); it even re-reads after a push-failure archive (`finalize.go:853-854`). A Finalize resumed after the parent push, once another run has landed on the parent, reads that run's commit: the row's landing commit and committer time change, so "a resumed Finalize leaves a byte-identical row" cannot be met as specified.

F5 — MEDIUM, design, D5/D6. "The archive tag is already the per-run list of landed work" is false: archive tags are also made for cancelled runs' pairs (cancel path's `ArchiveTags`, `internal/battencli/cancel.go:152`, `:822`), orphan pairs (PATTERN-pair-remote background), and a second origin tag on a diverged teardown (`internal/fabricengine/archive.go:177-187`). `lym report landed` would list each as a "derived" run with a landing line; the record does not say how a non-landed tag is shown, though reportengine already classifies such tags as partial (`internal/reportengine/runs.go:163-166`).

F6 — LOW, design, D2. Not every findings set reaches the verb: the PR-Review stencil sends a single requested change "straight to the fix" (`contracts/stencils/landing/landing-template-pr-review.md:59`). And a retry duplicates copies: after a failed commit `answer()` returns an error and the agent re-runs (`internal/loomcli/prfindings.go:204-207`), each re-run taking the next free number; a fix-here re-run over the same already-counted file copies again while the record counts it once (`prfindings.go:222-228`). The decision never says whether the copy is keyed to the content hash or placed before the early send-back return (`prfindings.go:218-220`).

F7 — LOW, consistency, D3. "inside the same locked update and from the value written": `state.UpdateJSON` persists only after the closure returns, so a copy written inside the closure lands before the record; a failed persist leaves a tracked copy showing a state the live record never reached. Evidence: `internal/burlerengine/parentreview.go:369-397`, `internal/state/state.go:157`.

F8 — LOW, consistency, D5. No fixer runs after the landing — fixer and fix-review passes run inside `landUnderLock` before the merge (`internal/landingshed/finalize.go:379-384`); "last step of Finalize" is ambiguous against `pushRecords` and `markTaskDone` (`finalize.go:859-865`), and a row written after `pushRecords` misses Finalize's own records push. And the landing commit's message is not the Describe text alone: Finalize adds the flaky section and credit block (`finalize.go:154-156`), a GitHub merge uses a title with the PR number (`internal/landingshed/finalize_pr.go:496`), and the already-landed path carries the operator's own message.

F9 — MEDIUM, design, D6 (attack surface). The "writes only under `&lt;dir&gt;`" bound states no rule for: a symlink entry in the tag tree (would create a link pointing outside `&lt;dir&gt;`); `&lt;dir&gt;` that exists, is empty and is itself a symlink to elsewhere (passes "empty" and writes outside); a `--tag` naming another slug's tag or a ref outside `archive/&lt;slug&gt;/`; or a partial tree left after a mid-extract failure.

F10 — LOW, consistency, D7. The carve-out misstates the code: `PlainArchiveTagsOnOrigin` only lists plain-namespace tags and returns a way-forward command for the operator; no code deletes them, so the rule can be stated without an exception. Evidence: `internal/fabricengine/cleanupremotewarp.go:336-379`, `internal/fabriccli/fabric.go:904-911`.

Tool calls:
1. `internal/fsx/fsx.go`, `internal/state/state.go` head, records worktree `.gitignore`/toplevel — atomic write temp-file location was not in context.
2. `git check-ignore` for a `.tmp-` file in the records worktree, plus grep for `.tmp-` handling — to confirm the temp file is not ignored.
3. `taskbranchdiscard.go` functions and `commitPendingRecords` callers — to test the teardown-commit constraint on the cancel discard path.
4. `ownerarchive.go` lines 285-345 and grep for tag-delete git args — to test D7's "no code deletes an archive tag" claim.

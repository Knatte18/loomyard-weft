# Support log — warp-visibility

## Interview

Autonomous run: no operator answered; every pick below is the writer's own, and the question ledger marks each as an auto-pick.

### Turn 1 — reading the task

The board entry carries the operator's design of 2026-10-31 and 2026-11-01 in detail; no `Decision for warp-visibility:` note exists.
Touching entries read: `bolt-dir-rename` (lands after this task and changes only the constant's value), `codex-engine` (heddle must leave room for a Codex engine's file and `developer_instructions`), `run-pins-its-version` (the operator places "a worktree keeps its version" there).
`lym loom peers`: `seed-starts-at-start` and `finalize-fix-in-place` change no path this design touches; no parent commit is listed.
The repository's `CLAUDE.md` was already deleted (`c15d6390`); lym writes no `CLAUDE.md` anywhere today, so the task builds heddle rather than removing anything.

### Turn 2 — facts settled from the code

- `fabricengine` imports `runproc`, and `lymcwd` may import only the standard library, `dotgit` and `weftname`; so the one `_board` literal cannot live in `fabricengine` if `lymcwd` and `runproc` are to drop their copies. Settled in D9: the literal lives in `lymcwd`, re-exported as `fabricengine.BoltDirName`.
- The board replay seam reaches past `replayBolt`: the ended set, the local-entry stamps and the origin owner read all read `board.json` and the `done` status inside fabric. Settled in D7 with a run-ledger role.
- `seedGitExclude` anchors lines at the worktree's anchor and unseeds them with a sibling census; heddle's files go at the warp root, so D5 anchors at the root and drops the unseed.
- `claudeengine` writes the append-system-prompt file at each Prepare, fresh and resume; D4 keeps a resume's passages fixed.
- Claude Code loads `CLAUDE.local.md` from the working directory upward; Codex loads one instruction file per directory from the git root down. Both make a warp-root file reach a session at any anchor.

### Turn 3 — design picks

Picks made without an operator, each with its rationale in the decision it belongs to: the entry shape (D1), the verb set (D2), root placement and the marker (D3), the shuttle seam (D4), the exclude key and its category (D5), creation on the warp seam and warnings over failures (D6), the run ledger (D7), the shared merge package (D8), the constant's home (D9).

### Turn 4 — lens review

All six forks returned; reports are in the `support-log.lens-<lens>.md` files beside this log.

- board-scope 1 (entries name engines and delivery, not targets; `PATTERN.local.md` not one row away): fixed — D1 entries name `targets` from D3's target table, each row carrying its delivery, file and re-render paths.
- board-scope 2 (`decision`: the literal's home in `lymcwd` against the entry's wording): kept under Open risks for the parent; the import-graph evidence stands in D9.
- board-scope 3 (weft not seeded, against the entry's "warp and weft"): fixed — recorded as a reading under Auto-mode assumptions.
- interactions 1, walkthrough 2, bound-breaker 2 (a commit carrying a heddle change and an ended run's board change is set aside whole): fixed — D7 sets aside only the run ledger's part; acceptance case added.
- interactions 2 (`board.lock` is a board file name left in fabric, against D7's criterion): fixed — criterion narrowed to the replay, ended-set and owner-read code; Scope says the lock name stays.
- interactions 3 and bound-breaker 3 (unknown entry keys lost on an updated or shifted entry, and on a replay write): fixed — D1's types keep unknown keys themselves, every rewrite goes through them; D1 and D8 criteria extended.
- interactions 4, existing-paths 4, walkthrough 5 ("made" ordering against the exclude seeding and `CloneHub`'s Bolt checkout): fixed — D6 fires "made" last.
- interactions 5 (a heddle refusal tagged under a run-slug namespace): fixed — D7 tags a non-ledger replayer's set-aside under the replayer's name.
- walkthrough 3 and bound-breaker 1 (D2 described a pull-first write path the board does not use): fixed — D2 now saves under the lock and starts the board's detached sync.
- existing-paths 1, walkthrough 1, bound-breaker 4 (an adopted resume runs `Prepare` in a new run directory with no earlier passages): fixed — D4 keeps passages only for a resume through the launch's own line; an adopted resume reads heddle afresh.
- existing-paths 2 (outside-fabric packages use the removed seam): rejected — the matches in `battencli`, `driftengine`, `loomcli` and `shedrecipe` are unrelated names (`claimBoardEntry`, `SubjectBoardEntry`), checked by grep.
- existing-paths 3 (Merriam's base counts `CLAUDE.local.md` but not prompt passages): rejected — the base's existing rule counts instruction files only; prompt passages are the same class as lym's appended sentences, which it does not count either.
- walkthrough 4 and bound-breaker 6 (`ffWorktree` also moves records and Bolt branches): fixed — D6 fires at code-branch call sites only; acceptance case added.
- bound-breaker 5 (the temporary file of an atomic write shows as untracked): fixed — D3 writes in place; the text is read only at session start and concurrent renders write the same text.
- leanness 1 (Constraints repeating decisions): fixed — cut to the constraints no decision states.
- leanness 2 (D1's "when" condition is a rejected alternative): fixed — cut from D1.
- leanness 3 (D9's import argument stated three times): fixed — kept in D9's first bullet only.
- leanness 4 (D5's last sentence repeats D3): fixed — shortened.
- leanness 5 (third plan-writer note repeats D5): fixed — cut.
- leanness 6 (D2 flag spellings): fixed — flag names dropped, the field rule kept.
- leanness 7 (operator-attributed lines in D6): rejected — the lens itself keeps them.
- leanness 8 (multi-line append file uncited): fixed — D4 states the file is passed to Claude Code and echoed nowhere; bound-breaker's check of the same claim held.

## Rejected alternatives

- An entry naming `engines` and one `delivery`: first picked, then rejected in the lens review, since a file belonging to no engine, such as `PATTERN.local.md`, would need a new axis rather than one target-table row.
- An atomic write by a temporary file renamed into place: rejected, since the temporary file shows as untracked in warp while it exists.
- Keeping a resumed session's passages across an adopted resume: rejected, since an adopted resume prepares a new run directory that holds no earlier passages.
- A heddle write that pulls and commits in the foreground: rejected, since the board's writes leave commit and push to the detached sync.
- Entry order by position in the JSON array: rejected, since a replay merges by field and a position has no field to merge.
- A `when` condition on an entry: rejected, no use for one was named.
- Rendering at the anchor: rejected, since an anchor change would strand the files and unexclude them, and both engines read a root file from any anchor.
- Asking git whether a target file is tracked: rejected, since only `fabricengine` runs git on warp; the render marker answers the same question.
- Unseeding a listed path when a worktree goes: rejected, since `info/exclude` is shared by every worktree of the warp repository and every lym worktree carries the files.
- Seeding the exclude list into weft: rejected, since nothing on it is written there.
- Callers of `Topology.Add` calling heddle themselves: rejected, since fabric is the one place every worktree is made, and a caller-side call is forgotten at the next caller.
- A heddle write lock of its own: rejected, since Bolt's pull and replay serialize against the board's write lock, and a second lock would let a pull interleave with a heddle write.
- `claudeengine` reading `heddle.json` itself: rejected under PATTERN-shuttle-provider-seam.
- Re-reading heddle on resume: rejected, since a resumed session's system prompt would change under it.
- Leaving the ended set and owner read on `board.json` inside fabric: rejected, since the operator wants no board path left in the replay seam and both read through it.
- Declaring the literal in `fabricengine` with `lymcwd` keeping a private copy: rejected, since the operator wants one declaration; declaring it in a new leaf package: rejected, since `lymcwd` already declares it and adding an import to `lymcwd` changes PATTERN-cwd-resolution.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): entry shape — slug, text, integer order, targets from a target table (D1, D3), revised in the lens review.
- Q2 (resolved, auto-pick): verb set — list, get, set, remove, show, render (D2).
- Q3 (resolved, auto-pick): where rendered files sit — warp root (D3).
- Q4 (resolved, auto-pick): how heddle tells its own file from the team's without git — a marker line (D3).
- Q5 (resolved, auto-pick): how prompt passages reach Claude — a shuttle seam, appended after lym's sentences; kept across a resume through the launch's own line, read afresh on an adopted resume (D4).
- Q6 (resolved, auto-pick): exclude key name and config category — `warp_exclude`, Cross-pair (D5).
- Q7 (resolved, auto-pick): does the warp seam also carry creation — yes (D6).
- Q8 (resolved, auto-pick): does a seam failure fail the fabric verb — no, a warning (D6).
- Q9 (resolved, auto-pick): how the ended set and owner read lose the board file name — a run-ledger role on one replayer (D7).
- Q10 (resolved, auto-pick): where the shared merge lives — `internal/entrymerge` (D8).
- Q11 (open, for the parent): the literal's one home is `lymcwd`, not `fabricengine`, against the entry's wording — recorded under Open risks (D9).

# Fixer report — discussion, round 1

Every finding F1 to F14 is fixed in `_lym/discussion/decision-record.md` and `_lym/discussion/support-log.md`; `lym loom validate-discussion` passes.

## Changes

- F1 (fixed as proposed, first option): D3's `claude-file` row now opens with an `@AGENTS.md` import line when the warp root holds an `AGENTS.md` and no `CLAUDE.md` there or in its `.claude` directory, and re-renders on a change to `CLAUDE.local.md`, `AGENTS.md` or such a `CLAUDE.md`.
  D3 records the Claude Code fact with its evidence (code.claude.com/docs/en/memory, sections AGENTS.md and Choose which instruction files load; from v2.1.277, checked on the installed v2.1.316), and its rationale says why an import rather than shuttle's settings file.
  D3 gained the acceptance case; Open risks line 3 is deleted.
  An Auto-mode assumption, a support-log rejected alternative (the `instructionFiles` setting) and Q12 record the pick.
- F2 (fixed as proposed, first option): D9's first bullet extends the import argument to the `runproc` → `lymcwd` edge; a Constraints line states that `runproc`'s closed import set gains `internal/lymcwd`; `## PATTERN changes` gains a `PATTERN-leaf-packages` changes bullet, its overview line unchanged since the admission sits in the entry's background file.
  The D9 criterion still derives `runproc.HubOwner` from the one declaration.
- F3 (fixed as proposed, first option): before any render writes, the lines are seeded: fabric seeds them before firing either D6 event, and `lym heddle render` first has fabric seed them through `fabricengine.SeedWarpExclude` (new), under the exclude lock; D5 lists both seeding points; Open risks line 4 names the seeding step; D3 gained the acceptance case.
- F4 (fixed as proposed): D3 replaces "two renders of one worktree write the same text" with a per-worktree `internal/lock` file under the pair's `.lym` tree, its directory named by `internal/lymdirs`; D3 gained the acceptance case.
- F5 (fixed as proposed, second option): D6 states a seam failure in a detached process is log-only, the warning naming the worktree and `lym heddle render`, and the stale file stays until the next take-in of a re-render path or a hand render; D6 gained the acceptance case.
- F6 (fixed as proposed): D9's third bullet adds `fabricengine.BoltDir` (new) from `BoltDirName`, moves fabric's Bolt-checkout uses of `BoardDir`, `BoardWriteLockPath` included, to it, and builds `BoardDir` from `BoardDirName`; Scope's Out bullet now names only un-renamed board names such as `board.lock`.
- F7 (fixed as proposed): the `PATTERN-heddle-instructions` text binds only heddle's content and its writers, `lym heddle` verbs and heddle's Bolt replayer.
  The reviewer's exact wording ran to 290 characters, over the 280 cap `lym loom validate-discussion` applies, so the second naming of `internal/heddleengine` was dropped.
- F8 (fixed as proposed): D4 states the adopted resume's re-read as the one bounded exception to the no-mid-run-shift rule; the support log narrows the rejected re-read to a resume through the launch's own line.
- F9 (fixed as proposed): D7 tags a non-ledger replayer's set-aside `set-aside/replayer/<name>/<short sha>` and states `setAsideTag`'s empty-slug fallback no longer names the board.
- F10 (fixed as proposed): D7's last bullet adds the origin owner read to the reads that fail with no run ledger registered; the D7 criterion covers it.
- F11 (fixed as proposed): D3 states removing a stale marked render is never refused by the exclude-list check; the D3 criterion covers it.
- F12 (fixed as proposed): Open risks line 1 is deleted, its reading moved to Auto-mode assumptions; Q11 is resolved as an auto-pick citing D9's import argument.
- F13 (fixed as proposed): D9's fourth bullet and criterion exempt `TestEnforcement_GeometryLiterals`'s policed-token list.
- F14 (fixed as proposed): the D7 criterion says board status `done`.

The support log's `## Review rounds` gained a Round 1 entry.

## Self-check

- fix-claims, LOW — D7 criterion names a `heddle` replayer no decision names: fixed as proposed; D7 names the replayers `board` and `heddle`.
- fix-claims, LOW — the import judges only a root `CLAUDE.md`, so an anchor-level `CLAUDE.md` does not suppress it: merged with fix-proposal's finding below.
- fix-consistency, LOW — the `PATTERN-leaf-packages` bullet records no admission while D9 points at it: fixed with own change; D9 now says the background admits it, as Constraints states, and the overview line stays. The bullet itself is kept, since the review's F2 fix asked for it and the change is to the entry's background file.
- fix-consistency, LOW — Turn 4 lines (board-scope 2, bound-breaker 5) contradict the record: fixed; the Round 1 entry states both are superseded, leaving the Turn 4 history as it was recorded.
- fix-consistency, LOW — the plan-writer note does not name `BoltDir`: fixed as proposed.
- fix-way-forward, LOW — the exclude-list refusal names no way forward: fixed as proposed; it names the path's return to `warp_exclude`.
- fix-way-forward, LOW (noted outside its lens) — unnamed `heddle` replayer: duplicate of fix-claims' first finding, fixed there.
- fix-proposal, LOW — the root-only import condition can load `AGENTS.md` into a session whose anchor holds the repository's only `CLAUDE.md`: fixed with own change, not by checking every directory, since a render serves every anchor of a worktree at once; D3's sentence now says "root `CLAUDE.md`", and the Open risk names both root-only gaps.

## Disputed

None.

## Deferred

None.

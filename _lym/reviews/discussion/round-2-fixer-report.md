# Round 2 fixer report — discussion

Every finding's premise held against the code and the Claude Code memory docs (code.claude.com/docs/en/memory, fetched 2026-11-01).
All seven are fixed in `_lym/discussion/decision-record.md` and `_lym/discussion/support-log.md`.

## Changes

- F1 (MEDIUM), fixed as proposed: D3's in-place bullet now writes each file atomically, through a temporary file in a `.heddle` directory at the warp root renamed onto the target, and the emptied directory is removed.
  Of the reviewer's two placements, `.heddle` on fabric's exclude list was taken over the `.lym` tree, since `.lym` is a junction into weft and a rename onto the warp root may cross a filesystem.
  D3 states the compaction re-read, citing the docs sections "Instructions seem lost after `/compact`" and "Import additional files".
  D5's template default gains `.heddle`, D3's exclude refusal covers it, and D4's "one bounded exception" now names this second one.
  The PATTERN-durable-vs-ephemeral-state change names the directory.
  In the support log, the Turn 4 bound-breaker 5 line is corrected and the rejected alternative is reversed to "A render written in place", with the `.lym` placement added as a rejected alternative.
  A Round 2 entry is added.
  New acceptance cases in D3 and D5.
- F2 (MEDIUM), fixed as proposed: D3 has a render read `heddle.json` under the Bolt write lock, only around the read, and states the lock order (render lock, then Bolt write lock) and that fabric fires D6's events with no Bolt write lock held.
  Checked: `Bolt.Pull` (`internal/fabricengine/bolt.go:251`), `retryBoltPush` (`pushclassified.go:678`) and `reconcileRecordsBranch` (`originreconcilemoves.go:171`) hold the lock; code-branch moves run under the prime lock (`originreconcilemoves.go:336-392`).
  D3 acceptance case added.
- F3 (MEDIUM), fixed differently after the self-check: the Bolt sync example and its acceptance case are gone, as proposed.
  The proposed replacement, `ReconcileOrigin` from `battencli` or `hibernatecli` as a log-only case, was not kept: those callers and `orchcli` print `OriginReport.Summary()` in the foreground (`internal/orchcli/start.go:346`).
  D6 now has the move's result carry the warning, with `OriginReport` gaining a per-branch warning that `Summary` prints, and keeps log-only for a process that prints no result.
  The acceptance case is a failing seam in a `ReconcileOrigin` code-branch fast-forward that logs the warning, puts it in `Summary` and leaves the move intact.
- F4 (LOW), fixed as proposed: the `claude-file` row imports each `AGENTS.md` the root holds there or in its `.claude` directory, and re-renders on a change to either.
  The rationale cites "When Claude Code reads AGENTS.md", and the D3 acceptance case covers a repository whose only `AGENTS.md` sits in its `.claude` directory.
  After the gate, every bare citation of the `.claude` directory's `AGENTS.md` path is reworded as "an `AGENTS.md` in its `.claude` directory", as the record already words `CLAUDE.md` there.
  That path is a file in the repository heddle renders into, not in this one, so the gate's `citation-path-missing` check read it as a missing path; ` (new)` would be false, since no task creates it.
  The `@.claude/AGENTS.md` import line stays as the literal line written.
- F5 (LOW), fixed as proposed: D9's first bullet and Constraints name the PATTERN-leaf-packages background edit.
- F6 (LOW), fixed as proposed: D9's fourth bullet and acceptance widen the exception to `TestEnforcement_GeometryLiterals`'s policed-token list, owner-row key and planted fixture sources.
- F7 (LOW), fixed as proposed: D2's `render` takes a hub warp worktree's name, resolved through `lymcwd.NamedWorktree`.
  A name matching no worktree `lym fabric list` lists, or only a prunable one, is refused with `lym fabric list` as the way forward.
  The refusal is added to D2's refusal list and acceptance.

## Self-check

- fix-consistency MEDIUM (D6 log-only rule contradicts "a foreground verb also reports it"), merged with fix-proposal LOW 2: fixed with my own change, as recorded under F3 above.
- fix-consistency LOW (D6's detached-sync sentence left stale): fixed as proposed. The sentence now says every lym process runs the same pre-run, so every such move reaches the seam.
- fix-consistency LOW (D5 acceptance "neither" counts two): fixed as proposed, now "none of them".
- fix-consistency LOW (D5's "every lym worktree carries the files" fails for `.heddle`): fixed as proposed, now "may write the paths".
- fix-consistency LOW (the Constraints durable-vs-ephemeral line misses `.heddle`): fixed as proposed.
- fix-way-forward LOW 1 (the way forward lists paths, `render` takes a name), merged with fix-proposal LOW 3: fixed as proposed. D2 says the name is the last element of a path `lym fabric list` prints.
- fix-way-forward LOW 2 (a prunable entry passes the bound): fixed as proposed. A name matching only a prunable entry is refused, in D2 and its acceptance.
- fix-proposal MEDIUM 1 (an unbounded Bolt-lock wait can hang a fabric verb behind a push's fetch): fixed as proposed. D3 bounds the wait with `lock.AcquireWriteLockWithin`, and a timeout fails the render as a seam failure or a hand-render refusal. D3 acceptance case added.
- fix-proposal LOW 4 (an existing `.heddle` path in the repository): fixed as proposed. D3 names the temporary files `<target>.tmp`, overwrites a crash's leftover, and writes nothing with a warning when `.heddle` is a file or holds another file. D3 acceptance case added.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.

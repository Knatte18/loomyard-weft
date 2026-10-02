## Goal

Every process lyx spawns gets one hierarchical name, used as its strand name, its Claude session name and its pane title, and every worktree records its parent so questions go to the session that spawned it.
Delivery is Claude Code's own SendMessage; creel is deferred.

## Decided with the operator (2026-10-01/02)

- Full name `<repo>:<slug>:<role>`, e.g. `ly:landing:driver`; a process in the prime sits at the repo level and has no slug segment, so the hub is `ly:orch`.
  Two segments mean prime, three mean task.
  This replaces `process-naming`'s `<slug>:<role>[.<repo>]` grammar below.
- Several of one role: `-N` (`ly:orch-2`) or a free suffix (`ly:orch-design`); Discussion settles which.
- lyx writes the full name wherever something is addressed (Claude session name, `LYX_STRAND_NAME`, `LYX_PARENT`), so delivery is exact.
  People and agents write the short form (`orch`, `landing:driver`), read relative to their own repo.
- Every repo has a short code (`ly`, `mh`, `qu`).
  It is given at `lyx fabric clone` the first time a weft is bound, beside the warp URL, and recorded on weft:main next to the `.lyx-warp` binding and the lyx-anchor subpath, so later clones derive it.
  Existing hubs get it once through a fabric verb.
  Build the short code first: no name can be formed without it.
- No machine-wide slug check: fabric keeps slugs unique within a repo.
  Discussion settles whether lyx checks that two hubs on one machine do not share a short code.

## From reed-mailbox.md (naming part only)

From `manifest/designs/reed-mailbox.md`, which is otherwise deferred as the tier-3 note `creel`:

- Inject `LYX_STRAND_NAME` into every process reed spawns.
- The reed watchdog owns pane titles and Claude session names, restores them when they drift (Claude resets the session name after compact) and logs the drift.
- `lyx reed list` is the directory of names.

## From `process-naming`: Process naming and parentage: one name per process, the same in Claude messaging and creel

Operator 2026-09-30/10-01: a proper naming system so processes are easy to find, out of creel and general; one name per session and its strand (the hub, today ad hoc named lyxhub:orch, becomes loomyard:orch), usable both with Claude Code's built-in messaging, which already addresses sessions with a leading @, and with creel later, with the same syntax: two different addresses would be a mess. No @ inside names.
Grammar: `<slug>:<role>`, the shape the existing orchestrators already use ((quarry:orch, MH:orch)). The slug is always the worktree the process runs in; the hub orchestrator runs in the prime, whose slug is the repo name, so it is loomyard:orch (no special case; quarry:orch already follows this); role from a fixed vocabulary (orch, driver, worker, reviewer, operator) with an optional `-N` for several of one role. Slugs are [a-z][a-z0-9-]* and cannot contain `:`, so the split is unambiguous. Owned by one package. This supersedes the `name@slug[.hub]` grammar in manifest/designs/reed-mailbox.md (creel), which must use these names as its addresses; across repos creel uses the same name with an optional repo suffix, `<slug>:<role>[.<repo>]`, the suffix skippable when the repo is obvious (own repo); slugs and roles contain no `.`, so the suffix splits unambiguously. The Claude session name stays the short form: hub orchestrators are unique across repos anyway (the prime slug is the repo name), and a rare clash between two repos' worktree slugs is resolved by Claude's own [ref] disambiguation.
Mirrors: the strand name is a birth attribute in reed state, mirrored to the Claude session name (`claude --name`), the pane title (allow-set-title off) and `LYX_STRAND_NAME`; the reed watchdog restores drifted mirrors (Claude resets the session name after compact); `lyx reed list` is the directory.
Parentage: the spawner records `parent` (its own name) in the run seed at create time (batten run / loom start); every strand reed starts in that worktree gets `LYX_PARENT`; stencils tell agents that questions and notices go to `$LYX_PARENT`; no parent recorded means the operator (awaiting state). If the hub spawns a worktree, the hub is the parent of every process in it.
Delivery v1: Claude Code peer messaging (SendMessage to the parent's session name, used successfully 2026-09-30); creel replaces it later with the same names.
The hub sets its own name at session start or via lyx orch.
Prerequisite for orch-discussion-review's live question mode.
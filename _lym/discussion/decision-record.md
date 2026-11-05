## Goal

```markdown
## Seen (worktree-create-serial's Finalize fixer and hub orch, 2026-11-04)

An empty directory `/tmp/.git` appeared at 15:27:32 local, during a gate's tier1 run; no test or process was found to have made it.
With it in place, go's VCS stamping ran git in `/tmp`, and `TestPrebuildLym` failed on `worktree-create-serial`'s Finalize; the fixer hardened that test on the branch (`311bb513`).
The hub orch removed the empty `/tmp/.git` by hand.

## Idea

- Find the test or code that creates `.git` under the shared `/tmp` instead of under its own `t.TempDir()`; a likely shape is a path joined from an empty or unset directory, which resolves to `/tmp/.git` through a temp-dir parent.
- A test never writes outside its own `t.TempDir()`; a guard (testkit or a gate check) fails a run that leaves a new entry directly under `/tmp` or the gate's temp root.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-05): Accept the .git-only guard and the open leaker hunt

Decision: Both departures from the board entry's Idea are accepted. The guard in tmuxkit.Main watches only a new .git directly in the lym temp root and the system temp directory, not any new entry. The leaker stays unfound in this task; the guard names the failing package the next time the leak happens. Review finding F1 is settled by this decision; no code change follows from it.

Rationale: /tmp and the lym temp root gain legitimate entries from concurrent binaries and other programs, so a check on any new entry would fail innocent packages; a .git there is never legitimate and is what breaks VCS stamping. The writer searched every .git-creating, TempDir-reading, chdir and subprocess-env site and found no path to /tmp; the guard attributes the next leak to its package, which is the cheapest way to find it.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria

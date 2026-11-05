# A test leaves an empty /tmp/.git, which breaks go VCS stamping

`tmp-git-leak` · Task · bug, testkit, gate · running · Drift-Baseline

## Seen (worktree-create-serial's Finalize fixer and hub orch, 2026-11-04)

An empty directory `/tmp/.git` appeared at 15:27:32 local, during a gate's tier1 run; no test or process was found to have made it.
With it in place, go's VCS stamping ran git in `/tmp`, and `TestPrebuildLym` failed on `worktree-create-serial`'s Finalize; the fixer hardened that test on the branch (`311bb513`).
The hub orch removed the empty `/tmp/.git` by hand.

## Idea

- Find the test or code that creates `.git` under the shared `/tmp` instead of under its own `t.TempDir()`; a likely shape is a path joined from an empty or unset directory, which resolves to `/tmp/.git` through a temp-dir parent.
- A test never writes outside its own `t.TempDir()`; a guard (testkit or a gate check) fails a run that leaves a new entry directly under `/tmp` or the gate's temp root.

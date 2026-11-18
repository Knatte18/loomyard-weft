# Case brief: scratch-symlink-two-paths, Seed-Child producer stop

## 1. The question the next judge must answer

The Seed-Child producer stopped with `fabricengine: push records side at loomyard-weft: the entry of branch main-weft is ended on origin, so the branch was not pushed and is left to its teardown`. Is this a real refusal that a re-launch of Seed-Child can clear, or a false reading of the origin state by fabric's "ended entry" check? The judge must read the fabric code that decides "entry ended" and say which of the two it is, and whether a teardown marker written by this run at prime time is what fabric reads as "ended".

## 2. The facts, each with a pointer

- The stop envelope: `_lym/shed/scratch-symlink-two-paths/drive-reports/drive-envelope-20261118-095658-1c09.json` (kind `producer`, `transient` empty, `history_length` 1, `interrupt_policy` handback, state `failed`).
- The step envelope: `.lym/shed/scratch-symlink-two-paths/steps/fda79c4adc275dac.json` (same error text).
- The status file `_lym/shed/scratch-symlink-two-paths/status.json`: `current_producer` Seed-Child, history only `Worktree-Create` done at 08:56:40Z (machine neve).
- Trace `.lym/shed/scratch-symlink-two-paths/steps/fda79c4adc275dac.trace.log`:
  - line 33: `_bolt` board commit `55b2548e` created at 09:56:33.
  - line 34 (09:56:35.304): `_bolt` push to `main` refused: `cannot lock ref 'refs/heads/main': is at 55b2548e... but expected ca60c705...`.
  - line 35 (09:56:37.575): `fabric: mutation file_written _bolt/.lym/teardown/scratch-symlink-two-paths.lock`, written under this run's trace id. The lock file is present and empty.
  - line 36 (09:56:37.576): `battenshed: prime lock contended, waiting` on `.lym/shed/run.lock`, producer Worktree-Create.
  - lines 37-48: worktree, branch and link creation; `loomyard-weft` commit `540a6c49` (`batten: Seed-Child -> running`) created at 09:56:40.081 (line 53).
  - line 49: detached push spawned (pid 2146078) with `--teardown-slug scratch-symlink-two-paths --pause-slug=scratch-symlink-two-paths`.
  - line 54 (09:56:58.729): `shed: step refused`, kind producer, with the error quoted in section 1.
- Stderr log `.lym/shed/scratch-symlink-two-paths/steps/fda79c4adc275dac.stderr.log`: the same two lines as trace lines 34 and 54.
- Shed driver trace `.lym/logs/trace-20261118T085623Z-80dfbc13a3f5a518-2142259.log`: tackler spawn at 09:56:58.893, deadline 10:06:58.
- Read-only remote checks (`git ls-remote` and `merge-base --is-ancestor`, run 09:57):
  - `loomyard-weft` origin `refs/heads/main-weft` = `540a6c49a69e45a6650f53d980e99ab3e0c7515a`. That is the commit this run created at trace line 53, so the branch is present on origin at the step's commit. The "not pushed" claim is contradicted by origin.
  - `loomyard-weft` local `main-weft` has later commits by the same batten flow: `1ac8836b` (Open-Window, 09:56:58) and `b0264696` (Seed-IDE, 09:57:23). `540a6c49` is an ancestor of the tip.
  - `loomyard-weft` origin has `scratch-symlink-two-paths-weft` = `03f0644a`, the run's own commit (trace line 46). That commit is not an ancestor of `main-weft`, which is expected for an unmerged task branch.
  - `_bolt` origin `refs/heads/main` = `b11a490c`, a later board sync. `55b2548e` is an ancestor of it, so the board commit did land. The rejection at line 34 was a lock race on that ref, not a missing commit.
- `main-weft` commits `d357fa4c` and `2bb7fa72` (`batten: Seed-Child -> running`, 09:56:38 and 09:56:39) predate this run's commit `540a6c49` and do not come from this run's trace. Their origin is not established from what I read.

## 3. What you ruled out, and how

- A transient or network error: `transient` is empty on the stop envelope and in the trace, and the only remote rejection (line 34) was a ref-lock race that resolved (`_bolt` main later holds the commit).
- A missing commit on origin: `ls-remote` shows `main-weft` at this run's commit `540a6c49`, which is an ancestor of the later tip. The commit the stop concerns is present (rule in the reading order: present when it equals the tip or is an ancestor of the tip).
- A `resume` on a plain re-launch: not named. Re-launching Seed-Child will re-run the same fabric check. The check reads an "ended" entry for `main-weft`, and this run wrote a teardown lock for its own slug at line 35. Whether that lock marks the entry ended is a code question I did not settle.
- A `hand-back`: the question is not a scope or design choice for the run's parent. It is whether fabric's check is wrong.
- I did not change any file except the two permitted outputs. I ran only read-only git and file reads.

## 4. Your hypothesis, marked as a hypothesis

HYPOTHESIS (not verified): fabric's "entry ended on origin" check reads this run's own teardown marker `_bolt/.lym/teardown/scratch-symlink-two-paths.lock`, written at trace line 35 during prime, as the entry being ended. The push therefore refused a branch that was in fact on origin. The fix would be in the check or in when the marker is written, which needs code read and judged by a stronger agent or the run's parent.

```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the fix's new refusals, next steps and criteria through the fix-way-forward lens.

Ways forward the fix introduced:
1. D1: a front that cannot take the key lock while 30 s of its cap remain answers as at a passed cap (75, `done: false`, run the same command again).
2. D1: at a start failure the front makes its own exclusive create of the process file and exits 1. A runner whose claim comes second refuses.
3. D1: the next front finds the front-written process file's pid dead or the lease expired, then supersedes and starts afresh.
4. D4: a cap below 60 s is refused, naming `lym config claude`.
5. D6, Linux: a front that supersedes a dead card-verify runner kills the recorded line group through `proc.KillTree`.
6. D6, Windows: the runner's kill-on-close job ends the line's tree when the runner dies.
7. New acceptance criteria: the D1 late claim, the D1 lock-wait answer, the D1 Windows join and supersede, the D4 60 s floor, and the D6 supersede kill and `4`/`4-&lt;slug&gt;` join.

Each of these can be reached from the state it names and leads to progress:
- **Way forward 2:** the front's process file names its own pid, which dies when the front exits 1, and the lease it expired already blocks a join. The next call supersedes and starts a fresh run.
- **Way forward 5:** the recorded group outlives the shell's parent-death kill while any member lives, so `proc.KillTree`'s live-member check reaches the orphaned `--wait` gate child.
- **Way forward 6:** Windows inheritance holds. Neither `verifyrun` nor `procgroup_windows.go` starts a child with `CREATE_BREAKAWAY_FROM_JOB`, so the line stays in the runner's job.
- **Way forward 4:** the 250 s template passes the floor, and the refusal names a verb the operator can run.

Findings:

- **LOW.** Changed text: D1, "A front waits for the key directory's lock only while at least 30 s of its cap remain … and otherwise answers as at a passed cap, below."
  Summary: the passed-cap answer prints "the phase file's phase", but a front that gave up on the lock may find no phase file. That happens while another front is still starting the run and its runner has not claimed.
  Evidence: D1's passed-cap bullet requires "the phase file's phase". The lock holder in this case is a starting front whose runner has not yet claimed its subdirectory, and the runner is the only writer of the phase file. The answer's "run the same command again" still leads to progress. Only the envelope's phase field is undefined for this state.

Tool calls:
- One grep of `internal/proc/procgroup_windows.go`, `internal/verifyrun`, `tied_windows.go` and `namedjob_windows.go` for breakaway flags. My context did not show how a line is spawned on Windows, and I needed to know whether it can leave the runner's job.

```yaml
lens: way-forward
outcome: reported
```

**List: each refusal or stop, the next step it names, and whether that step progresses**

- Card 2, `BuildShed` refusal when the machine name is unknown → "give the machine a hostname, then re-run lym batten run <slug>". Progresses: no shed is built and the status is untouched, so the re-run can build one.
- Card 3, `machineLandings` name read failing → "give the machine a hostname". Progresses: the name is read again on every tick, and the tick-failure episode recovers.
- Card 7, step 1, the re-exec test's state read fails → logs `dobby: stopped on error: <error>`, posts nothing, names no next step. Finding 1.
- Card 7, step 3, refused re-exec whose second read still holds its own pid → "run lym beam status, and once it shows the dobby not alive, run lym beam start". Progresses: the holder releases, and `EnsureLive` then spawns.
- Card 7, step 3, refused fresh start → nothing needed. A dobby is live, so this is the idempotent case.
- Card 7, step 5, fatal error after a re-exec → "fix the cause, then run lym beam start". Progresses: the process exits and the lock is free.
- Card 7, step 5, fatal error on a fresh start → line only, naming no next step. Covered by Finding 1.
- Card 7, failed exec → "fix the binary, then run lym beam stop and lym beam start". Progresses. `Locked` registered the dobby, so the stop's `EndRole` reaches it. A fixed binary is also a new identity, so `selfexec` would retry the exec on its own.
- Card 7, owner exit → "lym adopt <subject> takes it over here". Progresses.
- Card 8, worktree-gone exit → nothing needed, since the beam is gone.
- Card 8, a failed tidy-strand stop → "lym reed remove <guid>". Progresses: the verb exists and takes a positional guid (`internal/reedcli/remove.go:118`). The line names no directory to run it from, a NIT left out.
- Card 10, `placeDobby` refusal after a re-exec → "fix the cause, then run lym beam start". Progresses: the process has exited and `EnsureLive` respawns.
- Card 10, a refusal with no resolved location → its envelope still names `lym beam start`. Progresses.
- Card 10, wiring failure after a re-exec → "fix the cause, then run lym beam start". Progresses.
- Card 10, failed stdio reopen → Warn, and the dobby goes on. No step needed.
- Card 6, failed Windows rotation → Warn, and the file is left for the next spawn. Progresses.
- Card 13, reconcile failure → "fix the cause, then run lym fabric reconcile in the prime and lym gate test --gate adopt". Progresses. That hand route writes no `check.json`, so `lym beam status` keeps the failed record, which card 15's view text admits.
- Card 13, gate failed or ended with no tier → rerun `lym gate test --gate adopt` and `suite.log`. Progresses: `suite.log` holds the gate's own refusal with its way forward. With `--wait` set, the "busy slots" case cannot occur.
- Card 13, check lock held → exits 0 with `already_running`. No step needed.
- Card 14, check not started → "run lym fabric reconcile in the prime, then lym gate test --gate adopt there". Progresses.
- Card 15, "ended without a verdict" → names `suite.log`, `check.out`, the rerun, and that the next `lym adopt` with an owner write clears it. Progresses, as stated.

**Findings**

1. LOW, consistency. Card 7 (`07-dobby-start-and-exits.md`, `Run` steps 1 and 5), and the fresh-start branch of step 5 in particular.
   - Summary: the terminal line `dobby: stopped on error: <error>` names no next step. A fresh-start fatal error, and a re-exec whose state read failed, post nothing, so this line in `dobby.log` is the operator's only trace and it gives no way forward. Every sibling notice names one, for example "fix the cause, then run lym beam start".
   - Fix: append "; way forward: fix the cause, then run lym beam start" to the line.
   - Evidence: card 7 Intent steps 1 and 5 (`_lym/plan/07-dobby-start-and-exits.md:320`, `:324`); Decision 3 (`_lym/discussion/decision-record.md:76-77`).

Tool calls:
- Ran a grep for the reed `remove` verb and read `shuttleengine` `Runner.StopStrand`, to check that card 8's named next step `lym reed remove <guid>` exists. My context had no reedcli code.
- Read `internal/reedcli/remove.go:100-150`, to check that `lym reed remove` accepts a positional guid. My context lacked the verb's argument contract.

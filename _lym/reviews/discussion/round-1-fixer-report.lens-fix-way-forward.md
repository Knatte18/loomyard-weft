```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every refusal, error, next step, row and test the fix introduced can be carried out and leads to progress.

**MEDIUM: a caller whose worker lost the lock to `--discard` or to a fast winner waits forever.**
- Changed text: D1's "The caller follows another pid's record only when it is unfinished and its start is later than the caller's own start; until such a record appears, it keeps waiting under the same wake rules." This interacts with D2's new "`lym adopt --discard` takes `adopt.lock` non-blocking".
- Evidence: the lock now has a holder that is not a worker. If a discard holds `adopt.lock` when the caller's worker starts, the worker "exits at once and writes nothing". No newer unfinished record ever appears. The same happens when the winning worker reaches `done` between two follow wakes, since a `done` record is never followed.
- Result: in a plain shell "the follow waits until the worker ends", but its worker has already ended, so the caller hangs. Only the agent-session keep-warm cap ends it.
- What is missing: a way out once the caller's worker has exited and no qualifying record appears. One option is to end non-zero naming the re-run, as an `interrupted` record does.

**MEDIUM: `lym s` on a subject whose adopt finished is blocked by another subject's held-back stop.**
- Changed text: D10's "When an unfinished record lists the subject in its resolved subject set, `--here` takes the record's request as its own, whoever the owner reads as".
- Evidence: D3 says a record with a project held back by a local-commit stop ends `stopped`, while the other subjects "go on through every step". Take a record over `{root, b1}` where b1 is held back. In the prime, `lym s` reads `root` as owned here, but the record still lists `root`, so `--here` resumes the whole request.
- The resume restarts at the first step not done, and the held-back project stops it again. The caller exits non-zero, so the shortcut never reaches `beam start` for `root`, whose owner write stood.
- The way out is a hand `lym adopt root b1 --keep-local` or `--drop-local`, or `--discard`. The shortcut cannot add either flag, and `--here` refuses a subject.
- Before the fix, this case ended `owned_here` and the beam started.
- What is missing: `--here` should take up the record only for a subject that is not among those whose owner write stood. Otherwise it should resume only the steps after the owner write for that subject, such as `deploy`, `beam` and `check`, and still end ok when that subject is through.

**LOW: the caller's non-zero exit on an `interrupted` record names no way forward.**
- Changed text: D1's "The caller exits 0 when the record ends `done`, and non-zero when it ends `stopped` or reads `interrupted`."
- Evidence: the `stopped` case carries the stop's own way forward, and the keep-warm cap names `lym beam status` and the re-run. The `interrupted` exit, the one D1's anchor bound now produces, names neither. Only D9's status view carries the re-run.
- What is missing: the `interrupted` exit should name the re-run of the same `lym adopt` and `lym adopt --discard`.

I made no further tool calls.

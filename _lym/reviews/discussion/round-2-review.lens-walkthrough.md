```yaml
lens: walkthrough
outcome: reported
```

Task: walk each Decision's mechanism end to end through the walkthrough lens and report where a step's input or the end state misses the Decision's purpose.

**Walkthrough list**

- D1: a loom Implement transition commits through `commitStatusDepsGuardedBy`, with the guard removed, and a pusher starts. Purpose met.
- D1: `commit-records` commits, starts the pusher and reports `committed: true`. Purpose met.
- D2: another machine adopts at origin's last transition and the original machine's pusher meets the entry. Purpose met.
- D3: a pair pusher pushes the records side, then the code side, then reads the tip again. Purpose met while it runs. The restarts after a code-side gap are broken; see W2.
- D5 (dobby tick): a pair pusher is killed after the records side landed and before the code push. **Purpose missed (W2).**
- D5 (Worktree-Teardown): shutdown under the prime lock, release, push, take the lock again, compare the tips, remove. Purpose met, given the producer change D5 names (`internal/battenshed/teardown.go:96-121` holds the lock across both calls today).
- D6 (pair): a loom pair's records push is rejected and `takeoverStop` returns a `*TakeoverError`. `PushPairAnchored` releases the shared teardown lock on return (`pushanchored.go:185`), and then `PushRun` calls `HandOver` with `InsidePair` true. `HandOverTips` takes the lock exclusive, archives, and writes the mark, so no restart follows. Purpose met.
- D6 (prime): batten run A is taken over while the prime holds A's and B's unpushed commits. The pass reads the trailers, hands A over through `HandOver(..., InsidePair false)` and archives the prime tip. On the retried pass, `takeoverStop` for A now returns `ErrHandedOver` (`takeover.go:124-125`). The replay can therefore go on only if that gate treats a handed-over run as one to leave out rather than as a stop. D6 names the gate and the replay as the sites that change, so purpose met.
- D7: the pusher's reconcile finds the entry stopped and pauses "with the pause record `lym loom pause` writes". The next commit's start gate then reads the pause. **Purpose missed (W1).**
- D7 (owned elsewhere): the pusher's reconcile reads `EntryOwnedElsewhere`. **Purpose unclear (W3).**
- D8: origin is unreachable. `checkOrigin`'s `awaitOrigin` transport failure (`pushclassified.go:626-627`) is one of the "owed-class failures `pushClassified` records" that D5 suppresses, so `PushOwed` stays unset and the run walks on. Purpose met.

**Findings**

W1 — BLOCKING, design, `_lym/discussion/decision-record.md#D7`
- Summary: D7 pauses a stopped entry "with the pause record `lym loom pause` writes, … as the pre-push check does today". That premise is false, and the pause D7 writes is not the pause D3's start gate reads, so the gate never stops the pusher.
- The pre-push check pauses through `tendercli.PausePair` (`internal/loomcli/prepush.go:77`). That is `tenderengine.Pause`, which writes a scope entry into the hub's pause record (`internal/tenderengine/pause.go:119-149`).
- `lym loom pause` does something different: it records a stop condition, `--before`/`--after &lt;producer&gt;`, in the run's status (`internal/loomcli/cli.go:430-445`).
- D3's "no pause covers its run" and D7's "while the pause covers the run" can only be read through `pauserecord.ReadCoverage`, as `pauseCovered` does (`internal/loomcli/landingdeps.go:692-705`). That read never sees a `lym loom pause` stop condition.
- Walked through: the reconcile stops the entry and a `lym loom pause`-style stop condition is written. The next commit checks coverage, finds none, and starts a pusher. That pusher reconciles again, stops again and posts again, on every commit and every dobby tick, against D7's "no commit, exit re-take or dobby tick starts it".
- Fix: name the pause the pusher writes as the tender pause the pre-push check writes today (`tendercli.PausePair`, read through `pauserecord.ReadCoverage`). Make D3's start condition and D7's "pushes nothing more" name that same record, and drop "the driver honours that pause where it honours `lym loom pause`", or restate it for the tender pause.

W2 — BLOCKING, design, `_lym/discussion/decision-record.md#D6`
- Summary: D6 sets `CodeTipMissing` unset on the owner machine whenever the local code branch holds the tip. That removes the one existing restart for a held code tip, and D5's dobby tick restarts a pusher only for a records branch that is ahead. A code tip left unpushed after its records landed therefore has no restart, against D5's "Nothing stays unpushed".
- Walked through: a pair pusher pushes the records side, and the machine sleeps or the pusher is killed before the code push. On the next dobby tick the records branch reads level, so D5's tick starts nothing.
- `ReadRun` now leaves `CodeTipMissing` unset (D6), so the verdict is never `ReasonCodeTip`.
- `settleCodeTip` pushes a held tip only for a `ReasonCodeTip` verdict (`internal/battencli/codetipsettle.go:117-158`), so it pushes nothing either.
- The code tip reaches origin only at the run's next commit, a resume or Worktree-Teardown. For a run halted or stuck at that transition, there is no bound.
- On another machine the run waits out `runfsm.CodeTipGrace`, then returns to its previous transition.
- D5's rationale, "every path that can leave a commit behind has a restart", does not hold for the code side.
- Fix: in D5's dobby-tick bullet, count a pair as ahead when its records branch or its code branch holds a commit its tracking ref lacks. Alternatively, keep the owner's code-tip settle firing on a held missing tip, and state which of the two carries the restart.

W3 — MEDIUM, consistency, `_lym/discussion/decision-record.md#D7`
- Summary: D7 says "an entry owned elsewhere is handed over, as the catch-up does today" without saying who hands it over. Today the pre-push check only stops the push and leaves the hand-over to the dobby's catch-up (`internal/loomcli/prepush.go:116`, `156-158`).
- If the pusher only stops, as today, it writes no hand-over mark and no pause. D3's start conditions then still hold, and every later commit restarts a pusher that reconciles and posts the same "stopped before its push" notice again until the catch-up runs.
- Fix: state whether the pusher hands the run over itself, through `pairteardown.HandOverOwnedElsewhere`, which writes the mark D3's gate reads, or stops and exits. If it stops and exits, add the owned-elsewhere verdict to D3's start conditions.

Tool calls: none. The manifest's reads held everything this lens needed.

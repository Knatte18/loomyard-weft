---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md#D7
    origin: lens:walkthrough
    carries: F3
    summary: "D7 names the wrong pause, so D3's pause gate never sees the pause D7 writes. D7 says the pusher pauses a stopped or ended entry 'with the pause record `lym loom pause` writes ... as the pre-push check does today'. But `lym loom pause` records a stop condition in the run's own status. The pre-push check pauses through `tendercli.PausePair`, which writes a slug-scoped entry in the hub pause record. D3's 'no pause covers its run' can only read that record's coverage. A pusher built as D7 words it is restarted by the next commit, exit re-take or dobby tick, and pushes the stopped run's commits."
    fix: "In D7, replace 'with the pause record `lym loom pause` writes' with the tender pause the pre-push check writes today: `tendercli.PausePair` (`tenderengine.Pause`), a slug-scoped entry in the hub pause record (internal/loomcli/prepush.go:77, internal/tenderengine/pause.go). State that D3's start condition 'no pause covers its run', D5's dobby-tick condition and D7's 'while the pause covers the run' read that record through `pauserecord.ReadCoverage`, as `pauseCovered` does (internal/loomcli/landingdeps.go:692). Replace 'The driver honours that pause where it honours `lym loom pause`' with the tender pause's existing effect on the driver, or drop the sentence."
  - id: F2
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md#D5
    origin: lens:bound-breaker
    summary: "After its records side lands, a pair's code branch has no restart. D5's dobby tick starts a pusher only for a records branch ahead of origin. D6 now leaves `CodeTipMissing` unset on the owner machine while its local code branch holds the tip, and the owner dobby's `settleCodeTip`, today's push of a held tip, acts only on a `ReasonCodeTip` verdict (internal/battencli/codetipsettle.go:117,144-150). A pusher killed between its records push and its code push therefore leaves the code tip local until the next commit or resume. A halted run has no bound. Another machine waits out `runfsm.CodeTipGrace` and returns the run to its previous transition, against D5's 'every path that can leave a commit behind has a restart' and D1's 'with none left out'."
    fix: "In D5's dobby-tick bullet, count a pair as ahead when its records branch or its local code branch holds a commit its origin tracking ref lacks, a code branch with no upstream and at least one commit included. Add that case to the D5 acceptance criterion on the dobby tick: a pair whose records branch is level and whose code branch is ahead gets a pusher. If the restart should instead stay with the owner's code-tip settle, say so in D6's code-tip bullet: name how `settleCodeTip` still pushes a held missing tip on the owner machine while `CodeTipMissing` stays unset for the act rule. State which of the two carries the restart."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: open
    answer: "D7 now stops the push on a pause, and D3 and D5 gate every start on one. But the pause D7 names, `lym loom pause`'s status stop condition, is not the coverage D3's gate reads, so the stopped run's commits are still pushed on the next pass; carried as F1."
  - id: F4
    status: resolved
  - id: F5
    status: resolved
    answer: "The fixer's variant is accepted: a pair's pusher passes InsidePair true and exits, and the prime's pusher passes false and goes on with the other batten runs it serves."
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
---

### [BLOCKING] D7's pause is not the pause D3's gate reads

**Location:** `_lym/discussion/decision-record.md#D7`

**Issue:** D7 says the pusher's reconcile pauses a stopped or ended entry "with the pause record `lym loom pause` writes", "as the pre-push check does today".
Both halves are false.
- `lym loom pause` writes a stop condition (`--before`/`--after <producer>`) into the run's status, fired at a producer boundary (`internal/loomcli/cli.go:430-436`).
- Today's pre-push check pauses through `tendercli.PausePair` (`internal/loomcli/prepush.go:77`). That is `tenderengine.Pause`, which writes a slug-scoped entry into the hub pause record (`internal/tenderengine/pause.go:119-149`).
- D3's start condition "no pause covers its run" and D7's "while the pause covers the run" can only be read through `pauserecord.ReadCoverage`, as `pauseCovered` (`internal/loomcli/landingdeps.go:692-705`) and `ReadRun`'s `pauseCovers` (`internal/pairteardown/runread.go:135-142`) read it.
  A status stop condition never shows there.

A pusher built as D7 words it pauses the run in a place D3's gate does not look.
Every commit, exit re-take and dobby tick then starts a pusher that reconciles, stops and posts again, or pushes past the stop.
That breaks the D3 and D7 acceptance criteria that a pause stops the pusher.
Round 1's F3 fix put the gate in place but rests on this premise, so F3 is carried here.
All four lenses reported it independently.

### [BLOCKING] A pair's code branch has no restart once its records side landed

**Location:** `_lym/discussion/decision-record.md#D5`

**Issue:** Round 1's F7 fix changed D6 so that the code-tip read leaves `CodeTipMissing` unset on the owner machine while its local code branch holds the tip.
That switches off the owner dobby's existing push of a held tip:
`settleCodeTip` runs from the dobby read (`internal/battencli/dobbyreads.go:260`) and pushes a held tip only on a `ReasonCodeTip` verdict (`internal/battencli/codetipsettle.go:117`, `144-150`).
D5's dobby tick restarts a pusher only for a pair "whose local records branch is ahead of origin", so nothing replaces it.

Counterexample: a pair's pusher lands the records side and is killed, by a crash, a sleep or a reboot, before its code push.
The run is halted or stuck, so no commit follows.
On the next tick the records branch reads level, so the tick starts nothing, and the settle never fires.
The code commits stay local until a commit, a resume or Worktree-Teardown.
Meanwhile another machine reads `CodeTipMissing` and, past `runfsm.CodeTipGrace`, returns the run to its previous transition.
This contradicts D5's rationale that "every path that can leave a commit behind has a restart", and the operator's D1 that every commit reaches origin "with none left out".
The defect sits on D5's and D6's changed hunks.

## Rejected

Each item below is a real gap reported by the lenses, but an incremental round raises new findings only at BLOCKING, and none of these reaches BLOCKING on the evidence.
They are listed so the parent can weigh them.
- Every pause, not only the reconcile's, stops a pair's pusher (interactions F-b, existing-paths 4, bound-breaker F-D). This includes an operator's or a hub-scope tender pause, and a Bolt hold's pause entry. Meanwhile the prime's pusher ignores pauses. MEDIUM: the commits reach origin at resume, and D2 accepts the redone work an adopt during the pause costs.
- A refused code-side push leaves its record on the code branch, which D5's gate does not read, so each commit restarts a pusher that is refused again (interactions F-d, existing-paths 5, bound-breaker F-C). MEDIUM: the cost is repeated pushes and notices, and no commit is lost.
- D7's "an entry owned elsewhere is handed over" does not say whether the pusher hands over itself or stops and leaves it to the catch-up, as the check does today (interactions F-e, walkthrough W3). MEDIUM: the catch-up ends the loop either way.
- Pushes outside the pusher still write owed-class records naming the run, so `PushOwed` can still hold the run during an outage, against D8 (existing-paths 3, bound-breaker F-E, interactions F-f). Those pushes are the detached `SpawnStatusPush` children, `lym fabric push`, the code-tip settle, `HandOverOwnedElsewhere`, and Worktree-Teardown's own push, whose mode D5 leaves unstated. MEDIUM: these are occasional pushes, and the pusher's landing clears their records. Round 1's F1 named only the pusher's path, so F1 stays resolved.

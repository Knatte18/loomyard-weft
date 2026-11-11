```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked each bound, guarantee and claim about existing code in the record.

**Claims attacked**
- D1, the assist keys are category `none`, empty by default and per round: held (`internal/loomengine/config.go:657-659`, `template.yaml:24-26`).
- D1, `fable[high]` is a valid model-spec: held (`contracts/specs/llm-model-spec.md:79`).
- D2, "a fresh `lym loom start`, a goto back to Discussion-Write and an amendment each start a new generation, while round numbers carry on": broken.
- D2, `discussion_max_bounces: 0` runs no round: held (`template.yaml:45`).
- D2, "with the key off the round asks nothing": held.
- D3, a restarted review row of the same round sends no second ask and keeps the first deadline: broken, because a Bouncer clear re-seeds round 1 under the same generation number.
- D3, the order of the ask post and the ask-record write across a crash: broken, since the record does not state it.
- D3, "no live orch resolves → `no-parent`": broken, because `NotifyParent` falls back to the prime's orch.
- D3, `NotifyParent` is the path PR-Review's parent ask uses: held as far as read (`internal/loomcli/landingdeps.go:425`).
- D4, the caller guard admits the parent orch by strand name and the unnamed shell: held, but it is only a mistake guard (`internal/loomcli/decision.go:11`).
- D4, "the verb writes only … the slug's current round": broken, since an ask nobody closed leaves more than one open ask, or one stale ask.
- D5, "the capped deadline … the chair always outlives the wait": broken, because the chair outlives the wait with no time left to consolidate.
- D5, the per-run lock makes `open` → `done` or `timeout` happen once: held between the two verbs, but nothing times out an open ask when no await runs.
- D6, "`lym loom` verbs carry no fork check; a fork shares the strand name": held. Webster spots forks by auditing who wrote its files (`internal/webstercli/acceptaudit.go:36`), and `lym loom` has no such check.
- D6, "no code enforces it beyond D4's caller guard": broken in substance, because D4's guard does not tell the fork from the orch at all.
- D7, the chair keeps every parent finding: held as instruction-only. It rests on the chair running await (see the next-to-last finding).

**Findings**

1. BLOCKING · design · D2 "Which rounds ask" and D3 ask-record key
   - The record says round numbers carry on across generations. They restart at 1:
     - `startNextGeneration` archives the run directory and seeds round 1 again (`internal/shedadapters/amendment.go:42-59`).
     - `archiveRunDir` recreates the directory empty (`internal/shedadapters/archive.go:67-76`).
     - `seedCall` seeds `focusPath(runDir, 1)` (`internal/shedadapters/bouncer.go:1622`).
   - The Bouncer's clear path also archives and re-seeds round 1. It writes no new generation number: `CarryRecords` copies the old generation record into the fresh directory (`internal/shedadapters/bouncer.go:458-470`, `internal/shedadapters/settlerecord.go:261-282`), and a commit-seam failure followed by a resume reaches this path (`internal/shedadapters/bouncer.go:465`).
   - Counterexample: generation 1, round 1 asks and the parent answers `done`. The settle's commit fails and the resume clears. The new round 1 has key (1, 1) again: D3 sends no ask, and await returns the old report at once. Round 1 then reviews a rewritten record against stale parent findings, or gets no parent review.
   - The record never says where the generation number is read from (`generation-record.json` in the run directory) or how a clear is told apart.

2. MEDIUM · design · D2 parenthetical on which events start a generation
   - "A goto back to Discussion-Write … starts a new generation" is false for an unsettled perch. A goto over an undecided escalated round halts Awaiting again, and a `continue` sends the rewritten record to a fresh round of the same generation (`internal/shedadapters/bouncer.go:367-369`).
   - "A goto or a resume restarts nothing" (`internal/shedadapters/bouncer.go:238`). So a fresh `lym loom start` on an existing run also starts no generation.
   - Counterexample: round 1 asks and the parent is satisfied. The operator runs a goto to Discussion-Write, which rewrites the record, and the next round has the same generation and round 2. It does not ask, because the round before was satisfied. That breaks D2's stated purpose: "every rewritten or amended record goes to the parent".

3. MEDIUM · design · D3, unreachable parent → `no-parent`; acceptance criterion D3 "with no parent orch to reach"
   - `NotifyParent` falls back to the prime's orch when no orch above the pair is live (`internal/orchcli/notify.go:44`, `orchcli.nearestLiveParent` → `hubgeom.NearestLiveOrch`).
   - With no strand recorded, it returns `posted=false, err=nil` and does not fail (`internal/orchcli/notify.go:193-195`).
   - So "no live orch resolves" never reaches the failure the record names. The ask goes to the prime's orch, possibly dead and with its mail queued, and stays `open` until the deadline.
   - A not-posted `(false, nil)` return is not mapped to `no-parent` in the record, so that path is unbounded too.

4. MEDIUM · design · D5 deadline cap
   - "Capped so it falls before the chair's own `review_timeout_min` runs out … so the chair always outlives the wait" names no margin.
   - The assist cap is `room = chair − startAllowance − awaitCap` (`internal/loomengine/review.go:64-83`). For the parent, the deadline runs from the ask, which is sent before the table starts. After the wait the chair must still consolidate the lens forks, the assistant and the parent.
   - Counterexample: `review_timeout_min: 20` with the deadline capped just under it. Await returns `ready` with seconds left, and the chair times out before writing the review. The round fails, though no wait outran the chair.
   - The record also leaves open the case where the room is ≤ 0. The assist path then skips with a reason (`internal/loomengine/review.go:71-74`); the parent seat has nothing stated. Plan is left to find the formula.

5. MEDIUM · design · D4/D5 ask lifecycle and the answer verb's "current round"
   - Only await moves an open ask to `timeout`. Nothing closes an open ask when the review table ends, the chair fails or times out, or the chair consolidates without running await. That last case is instruction-only (D7).
   - Counterexample: the chair dies, so the ask of (g, N) stays open, and the parent answers after the deadline. The verb accepts it because the ask is still `open`; D4 checks state, not deadline. The report then lands for a round whose review may already be written, while D4's copy step and D2's "round before held findings" read it inconsistently.
   - With an open (g, N) and a new (g, N+1), the verb's "the slug's current round" does not say which open ask it answers.
   - Fix: Go sets an ask that is still open to `timeout` once the table ends, and the verb names the round it answers.

6. LOW · design · D3, ordering between posting the ask and writing the ask record
   - The record does not say which comes first.
   - Record first, then a crash before the post: the restart finds the record and sends nothing. The parent is never mailed, and the round waits out the full deadline.
   - Post first, then a crash before the write: the parent is asked twice.
   - The stated restart guarantee holds only for one order, and the record does not pick it.

7. LOW · consistency · D6 bound
   - "No code enforces it beyond D4's caller guard" implies the guard narrows what the fork can do. It does not:
     - the guard admits any caller whose strand name equals the parent's (`internal/loomcli/decision.go:693`);
     - the fork shares that name;
     - the guard is only a mistake guard: "guards against a mistaken call, not a forged strand name" (`internal/loomcli/decision.go:11`).
   - So the fork can run every orch-guarded verb on every run, including `decision replace` and `decision store`. The bound should say the fork's reach equals the orch's and is limited by instruction alone.

Tool calls:
1. Grepped webster for fork detection, to test D6's "no fork check" claim one hop past `lym loom`; the reading manifest did not cover it.
2. Grepped the callers of `NotifyParent`, to check D3's claim that PR-Review's ask uses it; the manifest had only the definition.
3. Grepped the loomcli Bouncer `Subject` wiring, to confirm the discussion Bouncer has a `Subject` seam, which the clear and generation paths depend on.

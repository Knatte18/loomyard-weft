```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every bound, guarantee, ordering promise and claim about existing code in the decision record.

**Findings**

1. **BLOCKING** — D4 (and D2): the way forward D4 names cannot be taken while the run holds the entry.
   - Claim: a halt can be cleared with "`lym board upsert` of the field to empty or `off` … then `lym loom resume`".
   - Counterexample: the run's board lock refuses any change to a held entry's fields except `status`. `internal/boardengine/doc.go:204` says a write "touches a field other than status" is refused, and `runlock.go` `scopeChanged` covers every field but Status, Created and Revised. A task halted at PR-Review is still held, so the upsert is refused.
   - This breaks `PATTERN-refusal-way-forward`: the way forward must be one a test can reach.
   - It also means D2's override is frozen once a run starts. The record never says so.
   - The same applies to a beam project from Parent-Align on: the `pull_request` field is not among the fields line 220 leaves editable.

2. **BLOCKING** — D8: the claim that no head reaches the pull request ungated breaks when the agent works during the gate.
   - The gate runs in the task worktree while the agent keeps working there. Counterexample: the tree is clean at head H1, so the `pr-review` gate starts and runs for minutes.
   - Meanwhile the agent edits files, so the gate tests a tree that is changing under it. The landing rows treat a tree that is dirty after the verify as a non-hermetic halt.
   - Or the agent commits H2 during the gate. The push route pushes the branch, not a SHA, so H2 goes to the pull request ungated, and the record's "pushed head" is ambiguous.
   - Fix options: push the gated SHA explicitly, re-check the head after the gate, or gate a snapshot.

3. **MEDIUM** — D9 with D7 and D11: an agent commit made after the approval can land unapproved.
   - D9 returns Done on an approval of the current head H1, and D7 stops the agent only on the way out, after that decision.
   - The agent can commit H2 between the last poll and the stop. Finalize then gates H2, pushes it and merges it.
   - D11's bound says only the merge of the parent and fixer commits land after the approval. H2 lands without approval, which breaks that bound.
   - Fix: stop the agent first, then re-read that the head equals the approved head before Done.

4. **MEDIUM** — D7: stopping the session on a cancelled context contradicts reusing a live session.
   - D7 stops the session on every way out, a cancelled context included, yet says a call that finds the session live reuses it.
   - Any driver interruption (a crash, a batten binary re-step, the supervisor's reinvoke) kills the operator's conversation mid-discussion, and the next entry starts a fresh session without its history.
   - The reinvoke policy in `internal/loomshed/interruptpolicy.go` assumes a re-call finds the live agent and reattaches. With D7 as written there is nothing left to reattach to.
   - The record also leaves the policy's comment that PR-Review "spawns nothing" standing.

5. **MEDIUM** — D9: the record says nothing about a pause or a hibernate during the wait.
   - `internal/shedverbs/loop.go` reads pause coverage between steps through its `Pause` seam. A PR-Review call that waits for days never reaches that boundary.
   - The agent is a live LLM session, and hibernate waits until "every session in the hub is safely paused".
   - Counterexample: `lym hibernate request now` while a run waits at PR-Review. The driver never returns and the agent keeps running, so the hibernate waits indefinitely or cuts the session mid-call.
   - Beam-Await is not a precedent here: it spawns no agent.
   - The wait loop needs to read pause coverage and the agent needs a pause path, or the record needs to state this as a bound.

6. **MEDIUM** — D12: `lym batten cancel` is not the only teardown.
   - The pair and its record can also be torn down by `lym cleanup --apply` of a run that origin has ended (a board removal), by board remove or prune, and by `lym fabric remove`.
   - None of these closes the pull request, so the claim "a cancelled run leaves no open pull request" holds only for the one verb.
   - The proposed `PATTERN-github-auth` text pins the close to `lym batten cancel` alone.
   - `PATTERN-pair-teardown` routes every teardown through `internal/pairteardown`; putting the close there would cover all paths.

7. **LOW** — D11: merging right after a push may be refused routinely.
   - Finalize pushes the task branch and then calls the merge API with that head at once. GitHub can answer "not mergeable" or 405 while it is still computing mergeability for the new head.
   - D11 turns every such answer into a halt that needs an operator resume. The record does not bound a retry, and the parent lock is held during the merge call.

8. **LOW** — D4: `ParseOwnerRepo` accepts only two URL shapes.
   - It accepts exactly `git@github.com:o/r` and `https://github.com/o/r`.
   - A real GitHub origin written as `ssh://git@github.com/o/r.git` or `https://user@github.com/o/r` fails to parse. Under the default `on` such a hub silently skips the pull request instead of halting.

9. **LOW** — D6: re-entry leaves the pull request's title and body stale.
   - After a Finalize reroute back through Describe, re-entry finds the recorded pull request and opens none, but never updates its title and body from the new description.

10. **LOW** — D3: an unreadable parent `beam.yaml` makes the entry gate error.
    - Per `shedengine.EntryGate`, a gate error writes the run failed. The record does not say this.

**Claims I attacked**

| Claim | Result |
|---|---|
| D1: a task's own `beam.yaml` reaches nothing (Finalize never carries per-worktree config to the parent, landingshed `doc.go`) | held |
| D1: CrossPair keeps the mode out of the local layer and `run_settings` | held |
| D2 / D4: the override and its way forward can be changed through `lym board upsert` | broken (#1) |
| D3: one resolver feeds both the entry gate and Finalize | held; an unreadable config fails the run (#10) |
| D4: `ParseOwnerRepo` decides whether a pull request can exist | held for its two shapes; other GitHub URL forms skip (#8) |
| D5: a named account ignores `GH_TOKEN` and has its own cache; the gh `--user` fact | held |
| D6: re-entry finds the recorded pull request and opens no second one | held; title and body go stale (#9) |
| D7: one session per entry, reused when live | broken by the stop on a cancelled context (#4) |
| D8: no ungated head reaches the pull request | broken (#2) |
| D9: Done needs an approval of the current head | broken by the agent-stop ordering (#3) |
| D9: wake interval and poll cadence | held |
| D9: no parent lock is taken | held |
| D9: the wait is safe under pause and hibernate | not addressed (#5) |
| D10: the wait-mark seam exists and is display only | held |
| D11: a GitHub-merged pull request takes the already-landed path (merge-tree `HoldsWork`; origin's ended set is a Bolt `done` that an outside merge does not set) | held |
| D11: Finalize crashing after its own GitHub merge re-enters through the sync and the already-landed path | held |
| D11: only the parent merge and fixer commits land after the approval | broken (#3) |
| D11: the GitHub merge can follow the push without a hitch | at risk (#7) |
| D12: a cancelled run leaves no open pull request | holds for `lym batten cancel` only (#6) |
| D13: no `on_stuck` because every Stuck is a Halt | held |

**Tool calls** (4 of the 6 allowed)

1. Read `internal/githubclient/parseownerrepo.go`, `internal/loomshed/interruptpolicy.go` and `grep -n pause internal/pawlengine/beamawait.go`: the context had the parser's name but not its accepted URL shapes, nor the reinvoke premise or how Beam-Await waits.
2. Grepped the ended-set definition in `internal/fabricengine` and the board run-lock rules in `internal/boardcli/doc.go`: needed to test D11's already-landed claim and D4's upsert way forward.
3. Grepped `scopeChanged` in `internal/boardengine` and `internal/hibernateengine/doc.go`: needed which fields the run lock freezes and what hibernate waits for.
4. Grepped `pause` in `internal/shedverbs/loop.go`: needed whether the shed loop honours a pause only between steps.

```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked each bound, guarantee and existing-code claim in the decision record.

**Claims attacked**

- D1, "every reader takes the state from `internal/runfsm`" and the D11 teardown "reads the FSM state and the act rule": broken (F5).
- D1, an act-rule `act` for a landing past its parent-side merge, on any machine: held as a single-machine rule. The two-adopter case is a pair interaction, left to the interactions lens.
- D2, "no run acts while origin is unreachable": held. A run with nothing owed runs its step, which the second half of the bound admits.
- D2, "a transition's push that meets a held teardown lock waits": held as new behaviour. Today `PushPairAnchored` refuses at once (`internal/fabricengine/pushanchored.go:172-177`), and the record names the change.
- D2, the description of `respawnPairPush`: held (`internal/fabricengine/teardownlock.go:154-158`).
- D3, "a cancel keeps the batten record, in place of today's removal": held (`internal/battencli/cancelcmd.go:56-57`).
- D3, intents written into the batten record beside shed's own writes: held. A precedent exists: `shedverbs/pause.go:126-129` already writes `pause_requested` into `status.json`.
- D3, "only the owner writes a run's records": held for pause and cancel.
- D4 bound, "its result is archived under a tag, not lost and not on the branch": broken (F1).
- D4, "needs no extra fetch per transition": held.
- D4 together with D8, "a release over a live adopt leaves one adopter": broken (F3).
- D6 step 4, "as fabric's removal under an owned-elsewhere verdict does": broken (F2).
- D6, the orch watcher observing a sleep through `pauserecord.SleepWatch`: holds only as new work. Today only the dobby loop, hibernate watch, shuttle run and loop watchdog use it. The record words it as a trigger, so no finding.
- D7 bound, "learns of a hold at its next Bolt pull, at most the dobby's pull interval": broken for verbs outside the dobby (F6).
- D7 upgrade rule, "a standing `adopt` entry makes the first read write one Bolt `adopt` hold": breaks against state an older version left (F4).
- D8, "two machines adopting at once end with one owner": held. Step 1 is one commit, and a rejected push re-reads and refuses.
- D9, "an unregistered pre-upgrade holder refuses once": held.
- D10 bound, "the owner's next mirror replaces" the old owner's late mirror: broken (F7).
- D10, a chunked mirror uploads only the changed last chunk: held. The previous commit's blobs are reachable from the advertised ref.
- D11, "a kept pair's batten record keeps it out of the orphans": held after the upgrade. Before it, a kept pair whose entry was already removed gets no record and is torn down under archive tags. That is lossless, so no finding.

**Findings**

F1. BLOCKING, design, D4 bound and D2 first bullet.
- In a task pair the code side is pushed before the records side, and the takeover entry sits only on records branches.
- Counterexample: the old owner's transition after the takeover pushes its code branch first, and that push lands, since nothing on the code branch rejects it. Its records push is then rejected and the run hands over.
- Its step's code commits are now on origin's code branch, not "archived under a tag, not … on the branch".
- The adopter reset its code tip to origin's at D8 step 2, so its next code push is rejected over a tip it does not contain. No takeover is readable on that branch.
- Evidence: `internal/fabricengine/pushanchored.go:126-128` ("Otherwise the code side is pushed first. The records side is attempted even when the code side failed") and `:212-217`.

F2. MEDIUM, design, D6 step 4.
- D6 rests on "removes the local pair only, as fabric's removal under an owned-elsewhere verdict does", and no such removal exists.
- Fabric's reconcile never hands an owned_elsewhere pair to teardown.
- `internal/pairteardown`, the only teardown route under `PATTERN-pair-remote`, archives and deletes the weft branch on origin. So no local-only route exists to follow.
- Evidence: `internal/fabricengine/doc.go:162` ("an owned_elsewhere entry's pair is never handed to the teardown seam"), and `pattern/PATTERN-pair-remote.md` ("A teardown's push of a weft branch it archives and deletes").

F3. MEDIUM, design, D8 release bound and D4's last bullet.
- Interleaving:
  1. Adopter A re-reads its hold at the start of step 3.
  2. The operator runs `lym adopt --release`.
  3. Machine C adopts, writing its hold, its owner and its takeover entries.
  4. A's step-3 takeover push is rejected. Per D4 it "writes the takeover entry again onto origin's tip … never hands over".
- Result: the records now name A while Bolt names C.
- C's next records push is rejected over a takeover naming another machine, so C hands over and ends its processes. A stops at its next step's hold re-read.
- No machine drives the runs, which breaks "a release over a live adopt leaves one adopter".
- The takeover retry never re-reads the Bolt owner or the hold.
- Evidence: the decision record, D4 last bullet and D8 lines 278-279.

F4. MEDIUM, design, D7 upgrade rule.
- Today every adopt leaves `adopt` entries that "only `lym tender resume` or `lym batten cancel` lifts".
- So a hub that adopted before the upgrade almost surely holds them. Its first read then writes a hub-wide Bolt `adopt` hold, which stops every run on every machine.
- The record names no notice for this hold. The only clears are a green re-run of `lym adopt` here or `--release`, and no agent-read text may name `--release`.
- Counterexample: a machine holding stale `adopt` entries from 2026-11-11 upgrades. Every other machine's runs then wait on a hold no live adopt owns. Their `lym adopt` refuses, naming that machine, and nothing tells the operator why the hub stopped.
- Evidence: `internal/pauserecord/doc.go:21-23`, `pattern/PATTERN-dobby-runs.md` "Adopt pause", and the decision record lines 233-236.

F5. MEDIUM, consistency, D11 first bullet against D1's act rule.
- D11's teardown condition "reads the FSM state and the act rule in place of … the pause record".
- But D1's act rule answers `wait` for every terminal state, and `landed` and `cancelled` are D11's own targets.
- So the act rule cannot tell a paused or held landed run from an unpaused one. Read as a gate, the condition never passes.
- Today's condition excludes a covered run explicitly.
- Evidence: `internal/dobbyengine/ended.go:31-34` (`!run.Covered && !rs.OwnedElsewhere`), and the decision record line 55 ("`wait` when … the state is terminal").

F6. LOW, design, D7 bound.
- "A machine other than the adopter learns of a hold at its next Bolt pull, at most the dobby's pull interval" holds only for the dobby's own passes. `pullBolt` runs only on the dobby's probe tick.
- `lym loom resume` and `lym tender resume <slug>` read the local Bolt; only an observed absence or a hand pass triggers a catch-up.
- On a worktree whose dobby is down, these verbs ask the act rule over a Bolt with no hold in it, for as long as that Bolt stays stale.
- Evidence: `internal/dobbyengine/handover.go:15-31`, and the decision record D6 absence list.

F7. MEDIUM, design, D10 lease bullet and bound.
- The lease retry turns the lease into last-writer-wins. "A mirror whose lease is lost re-reads the ref and pushes again", gated only by the act rule, which reads the local Bolt and lags up to 5 minutes.
- Counterexample:
  1. After the adopt, B mirrors.
  2. Old owner A, before its Bolt pull, mirrors at a turn end. Its lease is lost, so it re-reads and replaces B's mirror with A's.
  3. B's orch is idle, so no "next mirror" comes.
  4. B's orch restarts, for example after a hibernate. Its start sees a mirror naming another machine and restores A's stale transcript over B's newer local one, keeping B's only "beside it".
- The resumed session lacks B's turns, which breaks the bound "the owner's next mirror replaces it".
- Evidence: `internal/dobbyengine/handover.go:16`, and the decision record lines 323-331 and 336-337.

**Tool calls**

1. `internal/dobbyengine/handover.go:15-50` and `ended.go:20-60`: the Bolt pull interval and today's teardown condition, which the manifest had not read.
2. A grep of the `fabricengine` push files for atomic or side order: whether a pair push is atomic across sides.
3. `internal/fabricengine/pushanchored.go:120-230`: the order of the code and records pushes, which the manifest lacked.
4. A grep of `shedengine` status writes and `shedverbs/pause.go`: whether status.json already takes a second writer of intent flags.

```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: attack every claim, bound and ordering promise the decision record states, and report the ones that break.

**Claims attacked**
- D5 "origin never reads the run's records or an ended entry whose parent lacks the run's work": broken (F-a)
- Goal and D5 "only a refusal that retrying cannot fix stops anything", traced one row past Finalize: broken (F-b)
- D3 "An owed push still writes no failed-push record": broken (F-c)
- D5 second landing on a parent whose first push is owed, with one "landing entry" per parent target: broken (F-d)
- Constraint: owner verdict before each push (PATTERN-owner-machine) for the landing and board targets: broken (F-e)
- D3 "clearing that record through its way forward removes the mark": broken (F-f)
- D7 reflog reachability rule and its pin: partly broken (F-g)
- D9 "on every pass whose merge record names a base": broken in a narrow window (F-h)
- D6 "the wait runs between steps, so a waiting run holds no parent lock and no run lock": held
- D6 bound on permanent faults read as network-class: held, since the bound names them
- D6 "a step the Go driver runs writes no halt note": held, though no driven-step marker exists, so the step has to infer the driver from the held driver lock (`loomcli/resume.go:244` reads it)
- D7 hand-over move only when origin's tip is an ancestor of the owed tip: held (`handover.go:257-270`)
- D7 destroy refuses when origin holds commits the checkout lacks: held (`destroy.go`, `moveRemoteTaskBranch`)
- D8 a held push lock fails as owed: held (`pushanchored.go:65-66`, `IsPushOwed` includes `ErrPushLockBusy`)
- D10 no file under the gate packages reads origin: held (the grep found none)
- D12 the loomcli "status push is owed" loop is gone: held (the grep found none)
- D1 a beam parent gets no second target: held, the pusher lock is per records worktree
- D4 `lym fabric sync-status` touches no network: held, as stated
- D13 removes the pair when the tracking refs hold both tips, and as today with pushes off: held

**Findings**

F-a — MEDIUM, design, `_lym/discussion/decision-record.md:133-135` (D5)
- Summary: the ordering promise holds back only the pair target, but batten's transitions after Finalize (Land, Worktree-Teardown, done) commit the prime's records and start the prime's pusher, which no landing entry holds.
- Counterexample: origin's batten record reads Land while origin's parent lacks the work. A reader of origin then takes `battenAtLanding` as `PastParentMerge = true`.
- Evidence: `internal/battencli/commitstatus.go:165-185` (commit, then `StartPush` of the prime's target); `internal/pairteardown/runread.go:204-208, 270-271`.

F-b — MEDIUM, design, `_lym/discussion/decision-record.md:3-6` (Goal), D5
- Summary: right after a local landing on the prime's branch, batten's Land row pulls the prime. A transient fetch failure there ends the row Stuck with "pull failed: …", a blocked stop that D6's network wait never takes.
- Counterexample: Finalize returns done; origin drops before Land runs; the run halts. No decision covers the Land row's pull.
- Evidence: `internal/battenshed/land.go:254-270`; `internal/battencli/land.go:98-104, 168-173`.

F-c — MEDIUM, consistency, `_lym/discussion/decision-record.md:97` (D3)
- Summary: "still writes no failed-push record" is false of today's code. An owed push writes a failed-push record of an owed class (transport, deferred, teardown-lock) unless `OwedUnrecorded` is set, and D1 keeps exactly such a record for webster reset.
- The run reader reads those records as the run FSM's `PushOwed` input. The record never says how that reader behaves once the pusher's owed pushes stop writing records.
- Evidence: `internal/fabricengine/pushanchored.go:291-386`; `pushclassified.go:504-530`; `failedpush.go:185-199`; `pairteardown/runread.go:238-247`.

F-d — MEDIUM, design, `_lym/discussion/decision-record.md:113-114, 122, 130-135` (D5)
- Summary: "the landing entry" is one record of the owed tip per parent target, yet it also holds one run's records commit and makes that run's `done` write.
- Counterexample: landing B onto a parent whose landing A is still owed makes "the parent target then owe the newer tip". Either A's entry is replaced, which releases A's pair hold early and loses A's `done` write, or several entries per target exist, which the record never defines (whose hold, which `done` writes, settled when).

F-e — MEDIUM, design, `_lym/discussion/decision-record.md:281` (Constraints) and D1
- Summary: PATTERN-owner-machine requires pushing a Finalize past its merge only on the owner machine. The constraint keeps the run-scope reconcile for the pair's pusher only.
- The landing target's pusher pushes the prime's code branch and makes the run's `done` board write with no stated owner verdict. Nor is it ordered after the catch-up from origin that a wake requires.
- Evidence: `pattern/PATTERN-owner-machine.md:3, 16-24, 50-53`.

F-f — MEDIUM, design, `_lym/discussion/decision-record.md:88-90` (D3)
- Summary: no writer removes the refused mark.
- Counterexample: the way forward, `lym fabric push`, clears the failed-push record when the push lands (`pushJob.landed` calls `j.clear()`) and touches no ledger. Meanwhile the dobby's ahead check skips a target whose refused mark stands. The mark outlives its record, the bar's minutes count forever, and the all-clear never posts.
- Evidence: `internal/fabricengine/pushclassified.go:463-468`; `runpusher/start.go:122-125`.

F-g — LOW, design, `_lym/discussion/decision-record.md:181, 192` (D7)
- Summary: the rule assumes the local code branch has a reflog. Whether git writes one depends on `core.logAllRefUpdates`, which defaults to true only in a repository with a working tree (git-config(1)). The record pins only the expiry defaults, not this setting or what happens when it is false.

F-h — LOW, design, `_lym/discussion/decision-record.md:210-211` (D9)
- Summary: `mergeParent` writes the merge record only after the after-the-merge-in clean check.
- Counterexample: a pass killed between the merge-in and the record write leaves the earlier record, or none. A resume then reads a stale base or no base, D9's removal does not run, and the before-the-merge-in clean check halts on the leftovers.
- Evidence: `internal/landingshed/finalize.go:1353-1390`.

**Tool calls**
1. Grepped `isOwedClass`, `IsPushOwed`, `FailedPushesOwedBy`, `OwedUnrecorded` and push-lock-busy in fabricengine: the context lacked how owed pushes are recorded and classified today.
2. Grepped the gate packages for origin reads and the "status push is owed" loop, read `pullStopOutcome` and `battencli/commitstatus.go:165-200`: the context lacked these to check D10, D12, the Land pull outcome and the prime's batten pushes.
3. Grepped `dobbyengine/runwatch.go` and the loomcli, shedverbs and sheddriver packages for a driven-step marker: the context lacked the run watch and how a step would know the Go driver runs it.

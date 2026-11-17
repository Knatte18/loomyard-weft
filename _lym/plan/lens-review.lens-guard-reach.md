```yaml
lens: guard-reach
outcome: reported
```

Guard-reach findings follow, one per defect. Each lists severity, card, summary and evidence.

1. **BLOCKING** — Cards 23 and 21, D5 "a subject read as owned elsewhere leaves no process": once the owner pause is gone, nothing hands over a run that turns owned elsewhere while its pass is in flight.
   - Card 23 makes InnerRun's owned-elsewhere verdict release the lock, post a notice and "hold the producer… but pause nothing". It makes loom's pre-push stop "without pausing", and says "the catch-up hands the run over".
   - The dobby owes the catch-up only at a fresh start, on a seen sleep, on a `PassExitOriginWait` exit, on a beam hand-over, and after the owned-elsewhere exit (card 23's `ownedElsewhere`).
   - InnerRun never exits on its owner verdict. It waits in place: `internal/battenshed/innerrun.go:1027` (`if p.ownedBy != "" { p.reportState(w, p.ownerHoldReason()) …`).
   - `PassExitOwnedElsewhere` comes only from a pre-step-loop refusal: `internal/battencli/drive.go:120-131` (`passExitCode`).
   - Before this plan, the owner pause covered the slug and ended the hold. With no pause, the pass, its child and its agents keep running here, and `planPass` only answers none for the listed `hand_over` verdict (`internal/dobbyengine/passes.go:131`).
   - A run under a prime or beam this dobby does not itself read as owned elsewhere is never handed over. Example: a beam's own pawl run, driven by the prime dobby, after that beam is adopted.
   - Fix: either make a listed run whose verdict reads `hand_over`, or a run in flight that InnerRun or pre-push stopped as owned elsewhere, owe the dobby's catch-up; or have InnerRun end the pass with the owned-elsewhere exit. Name the test row that reaches it.

2. **MEDIUM** — Card 22, upgrade wiring: the first read runs only from the prime's dobby, so a machine that does not own the prime never runs it.
   - Card 22 fills `Upgrade` "for the prime's dobby only".
   - `EnsureLive` refuses to start a prime dobby another machine owns (`internal/dobbyengine/start.go:177-182`).
   - On a machine that owns only beams, the `owner` and `adopt` entries stay forever. Card 23 deletes their only lifter (`recheckOwnerPause`, `liftOwnerPause`).
   - So the "every owner entry is dropped" and "standing adopt entry becomes a pause intent" branches are unreachable there.

3. **MEDIUM** — Card 22, D5 "A first read that finds another machine's hold writes none": `UpgradeRead` runs before `CatchUp`'s Bolt fetch and registration.
   - Its owner judgment (`Classify` "through the registry") and its hold read "from the fetched Bolt" both run before the fetch. Card 22 says `catchUp runs UpgradeRead before CatchUp`; card 18 puts the fetch and registration in CatchUp step 1.
   - So it judges ownership on a stale local Bolt, with this machine still unregistered. A run another machine adopted meanwhile can read as owned here and get a pause intent.
   - `WriteHold`'s own pull guards only the hold write, not step 1's intents.

4. **MEDIUM** — Card 10, the copied-file way forward: if `checkHub` runs `Registry.Check` "for the new name with the file's names" using the file's id, refusal 2 (copied `machine.json`) fires on the very verb that is its way forward.
   - That refusal comes before the "writes a fresh id" branch, so that branch can never be reached.
   - The card does not say `Check` runs with the fresh id once the copy is detected. Card 4's refusal 2 has the way forward `lym machine name <new>`, and card 10 places the id judgment after the `Check`.

5. **MEDIUM** — Cards 15 and 21, D5 "drops those entries once it reads no hold": `bolt-hold` entries are dropped only for slugs in the tick's `Scope`.
   - A run left out of `DobbyRuns` when the hold clears keeps its `bolt-hold` entry with no dobby path that drops it. `DobbyRuns` leaves out unreadable records and unsettled code tips: `internal/battencli/dobbyreads.go:193-197, 251-256`.
   - Per card 15, `lym tender resume` lifts it only by hand once the hold is gone.

6. **LOW** — Cards 6 and 10 contradict each other on whether the sweep's owner write carries an identity.
   - Card 6: "A request with no Identity registers nothing, as … the sweep's own pre-registered write make it".
   - Card 10: the sweep request "carries the identity so it writes the current name".
   - Which way the implementer reads it decides whether `writeOwners`' registration `Check` runs on the sweep.

7. **LOW** — Card 17, the "pushes nothing first" branch: a takeover entry on origin is written in production only by adopt, which lands in a later part.
   - In this part the branch is reached only through the testkit writer. It is sound, but it never runs in production until then.

Tool calls:
- `grep` of the PassExit constants and of InnerRun's owner-hold sites. My context lacked the exit-code values and whether InnerRun exits on an owned-elsewhere verdict.
- `sed` of `internal/battenshed/innerrun.go:1015-1040` and `internal/battencli/drive.go:120-135`. My context lacked the InnerRun hold and the pass exit mapping.

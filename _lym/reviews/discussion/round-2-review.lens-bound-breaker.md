```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I tried to break the record's claims and bounds with concrete counterexamples.

**Claims attacked**
- D1 "PushPairAnchored records-first, so the order holds for every caller": **held** for that function's own callers.
- D1 "a flag the holder does not owe … sent records first with D2's takeover read" / AC "a code side flagged by a push whose lock wait gave up never reaches origin before its records side, nor after a takeover": **broken**. The detached pair pushes that stay take the flag and push code first.
- D1 "no run acts while origin is unreachable": **held**. A hand pass's catch-up acts on nothing offline, and a dobby run waits on its owed push.
- D1 "a transition's push that meets a held teardown lock waits; past StatusPushLockWait it turns owed": **held**.
- D2 "takeover is a compare-and-swap; old owner's next transition leaves origin's code branch unchanged": **broken**, by the same detached code-first push (finding 1).
- D2 code-tip give-up with lease and re-read, plus the owner's reset guard: **held**.
- D2 hand-over marks dropped when this machine re-adopts: **held**.
- D3 "a machine id tells two machines apart whatever they are named": **broken**. The comparison is by name, and registration is lazy.
- D3 bound "`--forget-former` changes only this machine's own file and entry": **broken**.
- D3 "lost `machine.json` gets a new id and must register under a new name": **held**.
- D5 "an owed push gets no `bolt-hold` entry, so it lands before an adopter's takeover reads origin": **broken as a guarantee**. D2 covers the consequence.
- D5 upgrade "first read" turns entries into pause intents, "so no run resumes unasked": **broken** in a crash window. No mark or order is stated.
- D5 bound that another machine learns of a hold at its next Bolt pull: **held**, since it is stated.
- D6 two concurrent adopts end with one owner: **held**.
- D6 `--take-hold` racing the old adopter's step 3: **held**, through the per-step hold re-read and D2's rejected takeover push.
- D9 bound "D1 leaves every transition on origin, so nothing pushed is lost" and the rationale "leaves every run in a pushed state": **broken**.
- D10 "never claims a prime": **held**. `EnsureLive` runs only on the started worktree (`internal/orchcli/start.go:388`), and an unowned prime is skipped.
- D8 "`root` is `boardengine.RootSubject`": **held** (`internal/boardengine/owners.go:16`).

**Findings**

1. **BLOCKING · design · D1 (pending-flag retirement, "detached pushes that stay") and D2.**
   - **Summary:** The detached pair pushes the record keeps still push the code side before the records side, with no takeover read. They also consume a code flag that a synchronous push left when its lock wait gave up. This violates the declared `PATTERN-pair-remote` text "the code branch goes along after the records side, both before the run acts" and breaks the D1 and D2 acceptance criteria.
   - **Counterexample:**
     - In a task pair, a transition's `PushPairAnchored` marks `pending.code` and `pending.records` (`internal/fabricengine/pendingflags.go:133`), then gives up on the lock and turns owed.
     - The current holder is the anchor change's detached push. D1 keeps it detached, and it is `SpawnStatusPush(pair, "")` at `internal/fabricengine/anchorchange.go:557`. It owes both sides, so `runHeld` takes both flags (`pendingflags.go:168-173`).
     - Its step pushes the code side first (`internal/fabricengine/coalesce.go:152-157`) and only then the records side. Nothing reads origin's record for a takeover entry before the code push.
     - The same path runs for every fabric commit's fire-and-forget push (`internal/fabricengine/commit.go:150-153`) and for `lym fabric sync` (`internal/fabriccli/spawn.go`). So the old owner's step in flight after a takeover can push code onto origin's code branch, which D2's bound and its criterion "leave origin's code branch unchanged" exclude.
   - **Evidence:** `coalesce.go:139-164`, `pendingflags.go:161-173`, `anchorchange.go:557`, `commit.go:152`.

2. **MEDIUM · design · D3 (registry identity, focus question 3).**
   - **Summary:** The machine id never enters a comparison. Registration happens only at the first owner write or `lym machine name`, and comparisons match the recorded name against this machine's name. So an unregistered machine whose name equals a registered machine's current name reads that machine's owners as its own.
   - **Counterexample:** Machine A is registered as `neve`. Machine B has no `machine.json`, so its hostname rule also gives `neve`, or it has a synced dotfile copy. B starts a dobby and never writes an owner. `dobbyengine.OwnedWorktrees` and `pairteardown.ReadRun` compare names (today `internal/boardengine/owners.go:175`), so B reads A's subjects as owned here and drives them. The refusal fires only when B registers.
   - **Gap:** D3 never says that a comparison checks this machine's id against the entry the name resolves to, or that an unregistered machine reads no owner as its own.
   - **Evidence:** `internal/machinename/machinename.go:133-160` (name from file or hostname, no id); `owners.go:175`.

3. **MEDIUM · design · D3 bound of `lym machine name --forget-former`.**
   - **Summary:** "changes only this machine's own file and entry" understates the reach. Dropping a former name that another machine bears as its current name hands that machine every record still written under the name. That covers an owner the rename sweep missed (offline or failed), a cancel intent's machine, and a takeover entry naming the old name.
   - **Counterexample:** A was renamed from `osl46` while offline, so one owner of A's stayed `osl46`. B registered as `osl46`, and A's registration is refused. A runs `--forget-former osl46`. The owner `osl46` now reads as B on every machine, and B's dobby takes A's subject with no adopt and no takeover entry.
   - **Fix:** State this reach in the bound, or refuse `--forget-former` while any owner on the board names the forgotten name.

4. **MEDIUM · design · D9 bound "D1 leaves every transition on origin, so nothing pushed is lost" and the rationale "leaves every run in a pushed state".**
   - **Summary:** The stop's own hub pause holds an owed push's retry, and D1 says such a retry "waits at the gate like any network call, so a pause holds it". The stop then confirms idle, because nothing is in flight, and kills the run. That transition stays committed only on this machine.
   - **Second path:** A push still in flight when hibernate's wait runs out is cut, because "a confirmation not reached … does not stop the stop".
   - **Effect:** Another machine adopting after the stop does not see that transition. This is the board entry's stated purpose: a state "another start or another machine goes on from".
   - **Evidence:** D1 owed-push retry at the gate (`decision-record.md:54-55`); D9 step 1 (`decision-record.md:309-312`).

5. **LOW · consistency · D5 "its owed push passes the gate and lands before an adopter's takeover reads origin".**
   - **Summary:** Nothing orders the two. The other machine learns of the hold only at its next Bolt pull, and the adopter runs D6 step 3 at once. An owed push that lands later is rejected over the takeover entry and handed over by D2.
   - **Fix:** Reword the sentence as "is not held by the hold", not as an ordering guarantee.

6. **LOW · design · D5 upgrade first read (focus question 3, marks).**
   - **Summary:** The first read uses the legacy entries themselves as its only "not yet done" mark. The record states no order between dropping an `adopt`/`adopt-hold` entry and pushing the run's pause intent and the Bolt hold.
   - **Counterexample:** A crash or offline exit after the drop and before the intent commit leaves the run with neither the entry nor the intent. It resumes unasked, against the decision's stated purpose.
   - **Evidence:** `decision-record.md:194-198`.

**Tool calls**
- Grep of `CoalescePushBothAt`, `CoalescePushRequest`, `SpawnStatusPush` and detached-push callers, plus `anchorchange.go`'s push: the manifest lacked which detached pushes reach a task pair.
- Grep of `RootSubject`, `EnsureLive` callers and `hibernateengine/watch.go` functions: to check the D8, D10 and D9 claims one hop past the record.
- Read of `teardownlock.go:90-112`, `commit.go:135-156` and `fabriccli/spawn.go`: to confirm which push function those detached call sites run.

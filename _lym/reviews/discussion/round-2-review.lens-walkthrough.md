```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: run each decision's mechanism end to end on one concrete case, and check the end state against the decision's own purpose.

**Decisions walked:**
- D1, a transition push whose push-lock wait gives up, followed by an anchor change's detached push in the same pair: **purpose not met**. The code side can reach origin before its records side (finding 1).
- D2, a records push rejected over B's takeover entry while A still owes the code tip, then the code-tip grace gives the tip up, with A's leased push and re-read racing B's re-run step: **purpose met**. The lease, the re-read and the owner's reset-on-reject leave no given-up tip on origin.
- D3, machine X renamed `osl46`→`osl-1046` before the upgrade and not yet registered; a new machine Y whose hostname gives `osl46` makes the first owner write: **purpose not met** (finding 3).
- D4, a dobby wakes from sleep on machine A; run R is now owned by B and has no takeover entry yet (between D6 steps 1 and 3): **purpose met**. A small gap remains in the hand-over mark (finding 4).
- D5, machine B reads an `adopt` hold while run R's push is owed: R gets no `bolt-hold` entry, the act rule answers `push_owed` and then `held`, and the owed push lands before the takeover reads it: **purpose met**.
- D6, a plain `lym adopt` re-run on the machine that already owns the subjects (under its own `adopt-fix` hold, or after the upgrade notice), with the fix task's agent mid-step or a transition push owed: **purpose not met** (finding 2).
- D7, a pre-upgrade kept cancel whose record is in shed state `cancelled` with no intent; the first dobby start writes a finished keep intent: **purpose met**.
- D8, an entry under the prime is printed as `root` at the printing boundary while `hubgeom.ParentOf` keeps the empty slug: **purpose met**.
- D9, a stop with one push owed: the stop ends the loop, and at the next start D4's catch-up at the dobby start pushes the owed commits: **purpose met**.
- D10, a restart after a crash where the hub's prime has no recorded owner and one beam is owned here: the prime is skipped, the beam starts through `lym beam start`, and the terminal attaches: **purpose met**.

**Findings**

1. **BLOCKING · design · D1, the detached pushes kept by the bullet "The anchor change's push … and tender's prime push … stay detached".**
   - **Summary:** a kept detached pair push still sends the code side first, so a code side can reach origin before its records side, and after another machine's takeover. This breaks D1's "the order holds for every caller", the criterion "a code side flagged by a push whose lock wait gave up never reaches origin before its records side, nor after a takeover entry", and the declared `PATTERN-pair-remote` text "after the records side".
   - **Walk:**
     1. A transition's `PushPairAnchored` marks `pending.code` and `pending.records`, and its bounded wait gives up. The flags stay on disk, as D1 intends.
     2. The pair's anchor change calls `SpawnStatusPush(pair, "")`, which carries both `WarpPath` and `WeftPath` (`internal/fabricengine/anchorchange.go:557-562`).
     3. That child runs `coalescePushSides`. Its `runHeld` takes the owed `code` flag (`internal/fabricengine/pendingflags.go:168-172`), and its step pushes the code side before the records side (`internal/fabricengine/coalesce.go:152-163`).
     4. A records rejection there goes through `pushSideDetached` with no takeover read, after the code side has already landed.
   - **Where else:** the same holds for a deferred anchor push that `lym tender resume` pays (D7). D1 reorders only `PushPairAnchored` and retires only the foreign-flag respawn.

2. **MEDIUM · design · D6 step 2 against D4 step 3 and D1's bound.**
   - **Summary:** a `lym adopt` re-run on the machine that already owns the runs archives and resets that machine's own owed transitions and live step work. D4's catch-up would have pushed them, but it only runs later, at step 4.
   - **Walk:**
     1. Step 1's commit for this machine's own hold writes the owner and the registry entry and goes on to step 2. Step 2 then, for each "moved run", "archives local tips origin lacks … resets them to origin's".
     2. On the owner, the local tips origin lacks are its owed transition commits. D1 says "an owed push holds the run at its transition until it lands", and D4's criterion says "none is reset".
     3. Under an `adopt-fix` hold, the fix task's run is live, because step 8 ran `orchcli.BeamUp`. Step 2 resets its branches under a running agent, and nothing ends its processes first.
   - **What D6 never states:** whether a run already owned here counts as "moved". The upgrade's notice (D5) and the `adopt-fix` re-run both send the operator into exactly this path.

3. **MEDIUM · design · D3, "A machine registers at its first owner write or `lym machine name`".**
   - **Summary:** because registration is lazy, a machine renamed before the upgrade can have its former name taken by another machine before the clash refusal can fire. Two machines can then read one owner name as their own.
   - **Walk:**
     1. X's `machine.json` holds `Names: [osl46, osl-1046]` (`internal/machinename/machinename.go:32-37`, `:153-158`). X has made no owner write since the upgrade, so it has no registry entry.
     2. Y resolves `osl46` from the hostname rule and registers first, at an owner write such as `dobbyengine.EnsureLive` claiming an unowned prime. No refusal fires, because nothing registered holds `osl46`.
     3. From then on, the registry reads every owner X recorded as `osl46` as Y, so Y reads X's subjects as owned here.
     4. The refusal comes only at X's later registration (former name equals Y's current name), after Y may already have acted.
   - **What D3 leaves open:** it never says when an existing machine registers after the upgrade, or how an unregistered machine compares its own former names in the meantime.

4. **LOW · consistency · D4 step 4 against D2's hand-over mark and D7's second bullet.**
   - **Summary:** the catch-up does the hand-over steps for a run owned elsewhere but writes no hand-over mark, so D7's mark-driven removal of the local pair never covers a catch-up whose own pair removal failed.
   - **Walk:**
     1. Step 4 ends the processes, archives the tips, posts the notice and removes the pair in local mode. It writes no mark; only D2's in-loop hand-over writes one.
     2. When that local-mode removal fails, for example on a held teardown lock, the pair still has its board entry and batten record. It is therefore no orphan, and it is neither `landed` nor `cancelled`.
     3. No D7 rule then removes it. Only the next absence's catch-up would retry.

Tool calls:
- grep `EndSubjectProcesses` and `runproc.EndSubject`: to check whether D2's in-loop hand-over spares its caller, which the manifest lacked (it spares the caller's pid).
- grep `anchorchange.go` and `tendercli/deps.go` for the push request: to check which sides the kept detached pushes carry, which the manifest lacked.

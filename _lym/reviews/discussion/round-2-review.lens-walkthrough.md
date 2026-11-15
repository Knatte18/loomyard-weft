```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: one concrete case per decision, run step by step, with the end state checked against the decision's own purpose.

**Purpose list**

- **D1** (a landed run whose pair D11 tears down; D3 removes its batten record with the pair): **not met**. The board reads `done` and no batten record is left, so none of the nine states holds.
- **D2** (a transition in a task pair, then a records push rejected by a takeover): **not met**. See D4: the code side is pushed with no order relative to the records push.
- **D3** (`lym tender pause <slug>` on the owner while batten's driver persists the same record): **not met**. The intent can be lost to the driver's next persist.
- **D3** (cancel with keep, offline, then a re-run): **met**. The slug reads `cancelled` until `lym fabric remove`; today's cancel likewise needs `--remove` before the slug runs again.
- **D4** (A owns run R at Implement and B adopts; A's step ends after B's takeover push): **not met**. A's post-takeover code commit lands on origin's code branch.
- **D5** (osl46 renamed before the upgrade; neve registers first): **met** once osl46 makes its first owner write or `lym machine name`.
- **D6** (laptop sleeps, another machine took run R; catch-up step 4): **not met**. The removal it cites does not exist.
- **D7** (B adopts, A reads the hold at its next pull and writes `hold` slug entries): **met**.
- **D7** (upgrade on neve with the `adopt` entries its 2026-11-15 adopt left standing): **not met**. A hub-wide Bolt hold is written and nothing reports it.
- **D8** (`lym adopt root` on B while A keeps pushing R's transitions): **not met**. B's local code branch stays behind origin.
- **D8** (`--release` from another shell while B's adopt is at step 7): **not met**. The adopt stops before step 8's `BeamUp`, so no dobby drives the subjects on B.
- **D9** (A reads itself owned elsewhere; the old orch Claude process was launched before the upgrade): **met**. The refusal names the pid once.
- **D10** (normal move: A's orch idle at 10:08 after a mirror at 10:00; adopt on B at 10:15): **not met**. Turns 10:00–10:08 are lost although nothing crashed.
- **D11** (orphan `rerun`, a kept pair from before the upgrade, a bench pair): **met**. The landed-pair case is folded into D1.

**Findings**

1. **MEDIUM**, design, D4 and D2. `decision-record.md:68-70`, `:147-152`, `:159`.
   - **Summary:** The takeover entry rejects only the records push, so the old owner's code commits made after the takeover still land on origin's code branch. The bound "not on the branch" breaks.
   - **Walkthrough:**
     - B's step 2 resets R's branches and step 3 pushes the takeover on the records branches only (`:262-264`).
     - A's next transition pushes "both sides" (`:69`). Code and records are separate repositories, so the push cannot be atomic, and D2 sets no order between them.
     - A's code push is a fast-forward over B's untouched code branch, so it is accepted. Only the records push is rejected.
     - B's next code push is then rejected. Its run record names B, not another machine, so D2 and D4 replay B's commit over A's stray commit (`:84`, `:153-154`).
   - **Evidence:** the batten record is on `main-weft` and the child record on the pair weft; the code branch is in the code repository (state map §1.1).
2. **MEDIUM**, design, D8 and D6. `decision-record.md:262-266`, `:189-195`.
   - **Summary:** Nothing brings B's local branches up to origin's tip when origin moves ahead of them after adopt step 2. The adopter resumes R at origin's later transition over a stale worktree.
   - **Walkthrough:**
     - A pushes a transition between B's step 2 (fetch and reset) and step 3. A learns of the hold only at its next Bolt pull (`:241`).
     - B's takeover push is rejected. Per `:155`, B writes the entry onto origin's records tip, which fixes the records only.
     - B's local code branch stays at the step-2 tip.
     - Catch-up step 2 only fetches and reads (`:190`). Step 3 only pushes local commits that sit ahead of origin (`:191`). Neither moves a local branch that is behind.
3. **MEDIUM**, design, D6. `decision-record.md:193-195`.
   - **Summary:** Step 4 removes the local pair "as fabric's removal under an owned-elsewhere verdict does", and that removal does not exist.
   - **Evidence:** `internal/fabricengine/doc.go:162` reads "an owned_elsewhere entry's pair is never handed to the teardown seam". `internal/dobbyengine/ended.go` (`strandedEnded`) also excludes `rs.OwnedElsewhere`.
   - **Effect:** the step has no mechanism to run. `pairteardown`, the only teardown route under `PATTERN-pair-remote`, archives and deletes the origin weft branch, which contradicts "origin's branches ... stay".
4. **MEDIUM**, design, D10. `decision-record.md:323`, `:334`.
   - **Summary:** On a normal move, not a crash, the turns since the last mirror are lost. The bound names crashes only.
   - **Walkthrough:**
     - Mirrors run "at most once per mirror interval" and the record names no trailing mirror. A mirrors at 10:00; the 10:08 turn end falls inside the interval, so it is skipped.
     - A stays idle, so no later turn end comes.
     - A reads the takeover, so the act rule answers `hand_over`. That blocks the stop-time mirror (`:324`), and D9 then kills the orch.
     - B restores the 10:00 mirror.
   - **Effect:** this misses D10's purpose, "the orch back as a continuation".
5. **MEDIUM**, design, D7. `decision-record.md:233-236`.
   - **Summary:** The upgrade turns any standing `adopt` entry into a hub-wide Bolt hold, and nothing posts a notice or names a way forward. Every machine's runs wait silently.
   - **Walkthrough:**
     - Adopt entries lift only by hand (`internal/pauserecord/doc.go`). So neve, which still holds entries from its 2026-11-15 adopt of `root`, writes the hold at its first read.
     - Other machines read `wait` for every run.
     - The only exits are a re-run of `lym adopt` on neve or `--release`. Nothing tells the operator either is needed; D8's notice fires only at the fix-try limit (`:275`).
6. **MEDIUM**, design, D3 and D4. `decision-record.md:100-107`, `:147`.
   - **Summary:** The pause intent, cancel intent, cancel finish and takeover entry go into the batten record. The record never says how they survive shedengine's whole-file persist of that same `status.json` while the run's driver is live.
   - **Walkthrough:** `lym tender pause <slug>` writes the intent while InnerRun persists at a row boundary, so one write overwrites the other. A lost intent defeats D3's purpose: the run reads unpaused on every machine.
   - **Evidence:** state map §2.2: `status.json` writer is "`shedengine` only", with `pause_*` set by `shedverbs/pause.go`.
7. **LOW**, design, D8. `decision-record.md:278-279`, `:269`.
   - **Summary:** A `--release` over a live adopt stops it before step 8, so `orchcli.BeamUp` never runs for the subjects. The runs on the adopter have no dobby, which misses "the runs go on by the act rule, with no hand lift".
8. **LOW**, design, D1. `decision-record.md:35-43`, `:105`, `:355`.
   - **Summary:** No state covers a landed run once D11's teardown has removed its batten record (board `done`, no record).
   - **Effect:** `landed` needs the batten record to read `done`, and `queued` needs an empty board status, so the derivation has no answer for an entry readers still list.

**Tool calls**

- Read `internal/dobbyengine/ended.go:1-60`: the `strandedEnded` conditions D11 widens and D6 step 4 relies on were not in my inherited context.

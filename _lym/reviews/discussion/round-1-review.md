---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:128
    summary: D8 isolates only internal/machinename's tests, so the verb tests D5's criteria require write the developer's real machine.json and rename the developer's machine
    fix: In D8, give the verb's file write and read a path seam as well (the same seam machinename's tests use, or one internal/machinecli takes), state that every test of lym machine name hands in a temporary path, and drop the claim that no test outside internal/machinename needs a change; add an acceptance criterion that the verb's tests write the machine file only at the path their seam hands in.
    origin: lens:walkthrough
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:83
    summary: The collision check lets a name another machine holds through, from an origin-only owner or a former name now held by another host, and the stated bounds cover only the write, not the two machines then sharing one identity
    fix: In D5, either run the collision check per hub after the sweep's pull (inside or right before SetOwners, refusing that hub and leaving the file unwritten or reverted), or state as a bound that the verb cannot see owners outside this machine's local Bolts and that a name shared with another machine makes each read the other's owned entries as its own, with the way forward. Apply the same bound to a former name, since the exemption for former names lets a rename back to a name another host now holds through.
    origin: handler
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:89
    summary: D3's named repair fails in the common case, since repairing a hand-broken file back to the name the machine already had is refused by D5's collision check because that name is no longer current or former
    fix: In D5, make the repair over a bad file keep the name the boards record for this machine, for example by recording the last valid name in former_names at every write so a repair can restore it, or by exempting from the collision check a name the file's former_names or last valid name holds; state the case where it cannot be recovered and its way forward, and add a criterion that a repair back to the previous valid name succeeds.
    origin: lens:interactions
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:113
    summary: D6 advises a hub pause for an operator who wants no window case to arise, but a hub pause does not hold the dobby's owned-elsewhere probe or the orch, which the support log admits was never established
    fix: Drop the hub-pause advice from D6 and from the verb's output, or replace it with a hold the code shows covers each listed owner check, citing the evidence (tenderengine/doc.go says a pause never touches the orch; dobbyengine/passes.go holds only an own-scope HoldCaller entry); update the support log's rejected alternative and Q6 to match.
    origin: lens:bound-breaker
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:115
    summary: D6's first recovery misstates the window's effect, since tendercli.PauseOwnedElsewhere also ends the run's processes and a tender resume restarts none of them
    fix: In D6's window bound, state that an owner pause during the window ends the run's processes on this machine, and name the step that restarts the run after the lift (as the code does today), with the evidence it rests on.
    origin: lens:existing-paths
  - id: F6
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:101
    summary: The sweep selects slugs from the hub's local Bolt before SetOwners pulls, so an owner under a former name that only origin holds is never swept, and a hub with none locally is reported as nothing to sweep and never pulls
    fix: In D6, pull each hub's Bolt before reading its owners for the sweep, or have the owner write select former-name owners under its own pull; otherwise state the bound and its way forward.
    origin: lens:walkthrough
  - id: F7
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:63
    summary: No Decision says which package owns the file's path, raw read and former-names accessor, so D3's sole-reader claim, D5's own read and write, D7's former-names input and the sole-source scan in the Notes contradict each other
    fix: In D3, have internal/machinename export the file's path, a raw read that returns the decoded name and former_names without validation, and a reader of the current and former names; state that internal/machinecli reads only through these and writes through internal/state at that path; in D7, take the former names from that reader through DobbyLandings' caller; keep the Notes' scan as pinning that no production file outside internal/machinename names the file.
    origin: lens:walkthrough
  - id: F8
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:43
    summary: Keeping a relative XDG_CONFIG_HOME as given makes the machine file depend on each process's working directory, so the verb and the dobby can read different files, which D1's bound does not cover
    fix: In D2, have the machine file ignore a relative XDG_CONFIG_HOME as standalonestate's state root does, keeping CredentialDir's rule for the GitHub and Slack files, or extend D1's bound to the working-directory case with its way forward; adjust the D2 criterion to match.
    origin: lens:bound-breaker
  - id: F9
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:142
    summary: The Slack constraint calls a rename safe, but after the relay restarts retireEntry archives an old channel under its undated name, so a later rename back to that name hits name_taken and adopt refuses the archived channel
    fix: In the Slack constraint, state the rename-back case and its way forward (renaming the archived channel in Slack, as adopt's refusal names), or state that a former name is not taken back.
    origin: lens:bound-breaker
  - id: F10
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:167
    summary: The criterion says every beam owner equal to the old name reads the new name, but writeOwners skips done entries, so a done beam keeps the old owner
    fix: Make the D6 criterion and D6's sweep sentence name open beam projects only, and say a done entry keeps its recorded owner.
    origin: lens:interactions
  - id: F11
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:92
    summary: D5 bounds the owners left under former names lost to an undecodable file but not the landings D7 then no longer counts
    fix: In D5's lost-name sentence, add that landings finished under a lost name no longer read as this machine's, with its effect and way forward.
    origin: lens:interactions
  - id: F12
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:117
    summary: A tender resume lifted before a hub's sweep has stood skips the deferred push of every pair still read as owned elsewhere, which D6's recoveries do not mention
    fix: In D6's window bound, advise lifting any pause only after that hub's sweep reports done, or name the push that recovers a skipped pair.
    origin: lens:existing-paths
---

### [BLOCKING] Verb tests write the developer's real machine file

**Location:** `_lym/discussion/decision-record.md:128` (D8), against the D5 criteria at lines 162–169.
**Issue:** D8 gives a path seam only to `internal/machinename`'s own tests and says no test environment variable changes.
The test environment sets `XDG_STATE_HOME` but leaves `XDG_CONFIG_HOME` and `HOME` alone on Linux (`internal/testkit/tmuxkit/tmuxkit.go`, `setTempEnv`).
A test of "`lym machine name osl46` … writes the name `osl46`" therefore writes `$HOME/.config/lym/machine.json` and renames the developer's machine for every lym process on it.

### [MEDIUM] The collision check lets a shared machine identity through

**Location:** `_lym/discussion/decision-record.md:83` and its bounds at lines 95–97.
**Issue:** The check reads only each hub's local Bolt before the pull, and exempts this machine's former names.
An owner only origin holds under the new name, or a former name another host has since taken, passes the check.
D5's bound says the replace set keeps the write off such an owner, which is true but misses the harm: once two machines share a name, `boardengine.OwnedElsewhere` (string equality) reads each one's owned entries as the other's own, so either can start, resume and land the other's runs.
No bound states that or names a way forward.

### [MEDIUM] Repair back to the machine's own name is refused

**Location:** `_lym/discussion/decision-record.md:89` (D5), with D3 line 54 and D5 line 83.
**Issue:** D3 names `lym machine name <name>` as the way forward over a bad file.
When a hand edit breaks a file that held `osl46` with owners already swept to `osl46`, the repair drops or loses that name, so `osl46` is neither current nor former, and the collision check refuses it as another machine's.
Removing the file leads to the same refusal; any other name leaves the `osl46` owners unswept.

### [MEDIUM] The advised hub pause does not hold the window's owner checks

**Location:** `_lym/discussion/decision-record.md:113` (D6).
**Issue:** `internal/tenderengine/doc.go` says a pause never touches the orch, and the dobby holds a run only for an own-scope entry carrying `pauserecord.HoldCaller` (`internal/dobbyengine/passes.go:47`, probe at line 191).
Under a hub pause the dobby's owned-elsewhere probe still pauses and ends a run.
The support log (lines 39 and 53) records that whether a pause holds every owner check was not established, yet D6 tells the operator it prevents the cases: a fact the design rests on left unresearched.

### [MEDIUM] Owner pause ends processes, which the named recovery does not restore

**Location:** `_lym/discussion/decision-record.md:115` (D6, first recovery).
**Issue:** `tendercli.PauseOwnedElsewhere` ends the run's processes once the pause stands (`internal/tendercli/doc.go`, Owned elsewhere section).
A tender resume sends carry-on lines only to live sessions (`internal/tenderengine/doc.go`, Resume section), so the recovery D6 names does not bring the run back.

### [MEDIUM] The sweep misses owners only origin holds

**Location:** `_lym/discussion/decision-record.md:101` (D6).
**Issue:** `boardengine.ReadOwners` reads the local `board.json` (`internal/boardengine/ownerset.go:294`), and the pull happens only inside `SetOwners`.
An owner under a former name that only origin holds never enters the slug set, and a hub with no such owner locally is reported as having nothing to sweep, so it is never pulled and the verb exits 0.

### [MEDIUM] No owner for the file's path, raw read and former names

**Location:** `_lym/discussion/decision-record.md:63` (D3), with D5 lines 84 and 89, D7 line 124 and the Notes line 186.
**Issue:** D3 makes `internal/machinename` the only reader of the file, yet D5 has the verb read it itself and write it through `internal/state`, which the leaf cannot import.
D7 needs the former names, but `DobbyLandings` is fed by `machineLandings(machinename.Name, …)` (`internal/beamcli/dobbydeps.go`), and no Decision gives a reader of them.
The Notes' sole-source scan would fail on `internal/machinecli` unless the path comes from `internal/machinename`.

### [LOW] A relative XDG_CONFIG_HOME splits the machine file by working directory

**Location:** `_lym/discussion/decision-record.md:43` (D2), against D1's bound at line 37.
**Issue:** `githubclient.cacheDir` takes a relative `XDG_CONFIG_HOME` as given (`internal/githubclient/cache.go:49`).
The verb then writes `machine.json` under its own working directory while the dobby and orch read under theirs and fall back to the hostname rule.
`standalonestate.stateRootFor` ignores a relative base for state for this reason (`internal/standalonestate/standalonestate.go:154`).

### [LOW] Slack rename back to a former name is not safe

**Location:** `_lym/discussion/decision-record.md:142` (Constraints, Slack).
**Issue:** After the relay restarts, `retireEntry` compares the entry's name with the base name under the new machine name (`internal/slackengine/keeper.go:423`), skips the dated rename and archives the channel under its old undated name.
A later rename back makes a new channel hit `name_taken`, and `adopt` refuses an archived channel (`keeper.go:175`).

### [LOW] Done beams keep the old owner, against the criterion

**Location:** `_lym/discussion/decision-record.md:167`.
**Issue:** `writeOwners` skips a done entry (`internal/boardengine/ownerset.go:188`), so the criterion's "every beam owner" does not hold for done beams.

### [LOW] Lost former names also drop landings from D7

**Location:** `_lym/discussion/decision-record.md:92` (D5).
**Issue:** An undecodable file loses its former names; D5 bounds the owners left under them but not the landings D7 then stops counting.

### [LOW] A lift before the sweep skips pair pushes

**Location:** `_lym/discussion/decision-record.md:117` (D6).
**Issue:** A lift skips the deferred push of every pair `Deps.Owners` reads as owned elsewhere (`internal/tenderengine/doc.go`, Resume section), so lifting before a hub's sweep has stood leaves those pairs unpushed.

## Rejected

- bound-breaker F6 and interactions finding 2 (D3 sole reader against D5's own read): merged into F7.
- walkthrough F3 (D7 has no source for the former names): merged into F7.
- interactions finding 3 (verb tests write the real file): merged into F1, which also raises it to BLOCKING since following D8 as written changes the developer's live machine identity.
- bound-breaker F3 (former-name exemption in the collision check): merged into F2.
- walkthrough F2 and interactions finding 1 (repair refused): merged into F3; kept at MEDIUM rather than BLOCKING, since it needs a hand-broken file and has a recovery through `lym adopt`.
- bound-breaker F2 and walkthrough F4 (relative `XDG_CONFIG_HOME`): merged into F8; LOW, since the variable is rarely relative and the GitHub and Slack files already behave so.
- bound-breaker F4 and interactions finding 4 (done beams): merged into F10.
- Leanness: neither file holds deliberation narrative that belongs in the other; the support log's lens list and ledger are the review trail, and the decision record's bounds each carry a constraint the plan writer needs.
- Acceptance coverage: D1 to D8 each appear as a whole word in a criterion bullet, and each criterion is a test-checkable behaviour or a `Review:` bullet.
- Cited symbols: `githubclient.CredentialDir`, `standalonestate.UserStateDir`, `internal/hubgeom/slackgeom.go` and `internal/slackengine/keeper.go` exist and behave as D2, D3 and the Slack constraint state; every owner check named calls `machinename.Name` per check.

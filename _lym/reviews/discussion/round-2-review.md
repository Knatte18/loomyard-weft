---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 (machinename.ReadFile, machinename.Names), D5 first Bound, D6, D7
    summary: entries of names are never checked against the D4 rule, so an empty or invalid former name enters the replace set and the landing match; an empty one sweeps every ownerless beam and an unowned prime onto this machine
    fix: In D3, state that machinename.Names returns as former names only the entries of names that pass machinename.CheckName (new), dropping an empty or invalid entry; in D5, state that the verb drops such entries when it rewrites the file and says so in its output. Add an acceptance criterion that a names entry that is empty or fails the D4 rule is neither a former name for the D6 replace set nor a name D7 matches.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D5 write step and repair branches, D6 replace set
    summary: the hostname rule's name reaches names only when a file fails to decode, so a name first written by hand then confirmed with lym machine name gets an empty replace set and strands every owner recorded under the hostname name; the collision check's own name set over a bad file is also unstated
    fix: In D5, state that before the collision check the verb takes the hostname rule's name (machinename.FromHostname) as one of this machine's names and appends it to names when absent, on every write, a re-run with the name already set and both repair branches included; the collision check counts it as this machine's. Replace the undecodable-file sentence that the old name recorded is the hostname rule's with this general rule. Add an acceptance criterion that lym machine name osl46 over a hand-written file holding only name osl46 records the hostname rule's name in names and sweeps owners recorded under it.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D3 error cases
    summary: D3 adds machine-file and relative-variable errors with their own way forward, but existing wrappers of the name error append a contradicting hostname fix, so the operator is told to fix the hostname for a bad machine file
    fix: In D3, add that a caller that wraps machinename.Name's error keeps the error's own way forward and adds no hostname fix of its own, and that the machinename errors themselves name the way forward for every failure, the hostname case included. Add an acceptance criterion that a lym batten run refused for a bad machine file names the file's path and lym machine name, and no hostname fix.
  - id: F4
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 no-fallback rationale
    summary: D3 says an error never yields a wrong owner reading, but the dobby's beam owner check reads a name error as not owned elsewhere and goes on, so a bad file does not stop that reader from acting
    fix: In D3, add a Bound that the dobby's beam owner check (internal/dobbyengine/loop.go, beamOwnedElsewhere) reads a name error as not owned elsewhere and keeps the dobby going, while batten's own owner check refuses each pass with the name error, so a bad file stalls the beam's passes rather than letting them run; the way forward is the one D3 names.
  - id: F5
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 shared-name Bound
    summary: the collision check cannot see a live machine that holds the requested name but has recorded no owner on the swept boards, and the shared-name bound names only the pull race and a reused former name
    fix: In D5's Bound that begins A former name is taken to stay this machine's, add the case of a machine that holds the name by hostname or file but has recorded no owner on any swept board, and change by that case or the one above to cover all three cases, keeping the same way forward.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 per-hub outcomes
    summary: a sweep whose replay loses to a competing origin write returns those entries in OwnerWrite.Lost, an outcome D6's report lines do not cover
    fix: Under D6's per-hub bullets, add one for a write that returns entries in OwnerWrite.Lost, reported per entry with the owner that stands and the re-run of lym machine name as the way forward, the verb exiting non-zero.
  - id: F7
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D6 window Bound, first recovery bullet
    summary: a sweep that commits but fails its push leaves the new owner in the local Bolt only, so the dobby probe lifts its owner pause on the local read and the resume re-pauses on origin, repeating until a re-run pushes
    fix: In D6's first recovery bullet, add that a hub whose sweep committed but did not push makes the dobby lift and re-take its owner pause each probe until the re-run of lym machine name pushes the commit, and that the verb's output for that hub names the re-run as the recovery.
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Goal and D3 second bullet
    summary: the Goal says a machine with no name in the file changes no behaviour, but D3 makes a relative HOME or LOCALAPPDATA an error before any file is read, so such a machine with no file now fails every owner check
    fix: Either have D3 return the hostname rule's name when the directory does not resolve to an absolute path and so no file can be read, or add to the Goal's second sentence the exception of a relative HOME or LOCALAPPDATA and its way forward.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 first paragraph
    summary: D1 says the file sits in the directory githubclient.CredentialDir resolves, beside slack.json, but D2 resolves it through standalonestate.AbsoluteUserConfigDir (new), which differs for a relative XDG_CONFIG_HOME
    fix: In D1, replace That is the directory githubclient.CredentialDir resolves today, beside slack.json and credentials.json with a sentence that it is that directory whenever XDG_CONFIG_HOME is unset or absolute, and that with a relative XDG_CONFIG_HOME the GitHub and Slack files stay under it while the machine file resolves under $HOME/.config/lym (D2).
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Auto-mode assumptions
    summary: the section lists D1, D3, D5 and D6 as auto-picks, but the question ledger also records auto-picks for D4 (Q5) and D8 (Q8)
    fix: Change the bullet to D1, D3, D4, D5, D6 and D8 were picked without an operator; the support log's question ledger holds each pick.
  - id: F11
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md Notes for the plan writer
    summary: the PATTERN background edits the two declared changes need, notably the owner setter's second takeover case, sit only in the optional Notes, which the spec says are never a completeness requirement
    fix: Move the three background-change bullets (leaf-packages allowed set gains internal/standalonestate; owner-machine name source is the machine file before the hostname; owner-machine's lym adopt takeover clause gains the rename sweep's replace set) from Notes for the plan writer into Constraints, under the PATTERN-leaf-packages and PATTERN-owner-machine bullets, and keep in Notes only the overview-lines-unchanged hint and the sole-source scan hint.
  - id: F12
    severity: NIT
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 last paragraph and Bound
    summary: D8 says other packages' tests read the developer's machine file, but on Windows tmuxkit.Main already points LOCALAPPDATA at the test's temp directory, so there they read no file
    fix: In D8, add that on Windows tmuxkit.Main's LOCALAPPDATA already points the config directory at the test's temp directory, so those tests read no machine file there and the Bound about a developer's bad file holds only off Windows.
---

### [MEDIUM] Former names are never validated, so an empty one sweeps unowned entries

**Location:** `decision-record.md` D3, D5's first Bound, D6, D7.
**Issue:** D4's rule covers only `name`, and D3 returns `names` with no validation, yet D5 itself sends the operator to hand-edit `names`.
An empty entry puts `""` in D6's replace set.
`boardengine.OwnedElsewhere` (`internal/boardengine/owners.go:175-177`) reads an empty owner as no owner, so the sweep would write this machine onto every ownerless beam and onto a hub's unowned prime.
That breaks D5's bound that the verb never writes an owner recorded under any other name.
D7 would also count every Land entry whose `Machine` is empty as this machine's, and entries written before that field existed decode empty (`internal/shedengine/status.go:118-120`).
The interactions lens raised the same defect.

### [MEDIUM] A hand-written name strands every owner recorded under the hostname name

**Location:** `decision-record.md` D5 write step and repair branches, D6.
**Issue:** The board entry describes a file the operator can edit, and D5 names a hand write as a way forward.
Take an operator who writes `{"name":"osl46"}` by hand, then runs `lym machine name osl46`.
The name is already set, so the verb writes nothing.
`names` is empty, so the replace set is empty, and every `osl-1046` owner stays stranded until `lym adopt`.
That misses the Goal's "keeps every owner it already recorded under the old one".
The interactions lens found the related gap: D5 never says whether the collision check counts the hostname rule's name as this machine's over a bad file.
It also found that the two repair branches treat that name differently.
One rule fixes both: always count the hostname rule's name as one of this machine's names.

### [MEDIUM] Callers' error wrappers contradict D3's way forward

**Location:** `decision-record.md` D3 error cases.
**Issue:** D3's new errors name the file's path and `lym machine name`, or the relative variable.
The callers that wrap them still append a hostname fix:

- `internal/battencli/owner.go:132`: "fix the hostname, then re-run lym batten run".
- `internal/battencli/arm.go:442`: "give the machine a hostname".
- `internal/beamcli/dobbydeps.go:658`, the `machineLandings` seam D7 rewires: "give the machine a hostname".

A bad machine file then refuses every batten pass with two ways forward, and one of them is wrong.
The record is silent on the wrappers' contract.
The finding is about that missing rule, not an enumeration of call sites.
The existing-paths lens raised the same defect.

### [LOW] D3's no-fallback rationale overstates what an error stops

**Location:** `decision-record.md` D3.
**Issue:** `beamOwnedElsewhere` in `internal/dobbyengine/loop.go:552-573` reads a name error as not owned elsewhere and the dobby goes on.
Batten's `thisMachine` (`internal/battencli/owner.go:126-134`) still refuses each pass, so the harm is stalled passes and repeated failures, not a run driven under a wrong owner.
The lens rated this MEDIUM, as a beam another machine owns being driven.
That rating is lowered because batten's own check refuses the pass.
The record should still state the bound.

### [LOW] Collision check is blind to a live machine with no owners on the swept boards

**Location:** `decision-record.md` D5 shared-name Bound.
**Issue:** A machine named `neve` that owns nothing on this machine's hubs passes the check.
Once it records an owner, the two machines read each other's entries as their own.
D5's "by that case or the one above" covers only the pull race and a reused former name.

### [LOW] Lost entries from a sweep have no report line

**Location:** `decision-record.md` D6 per-hub outcomes.
**Issue:** `SetOwners` returns entries a competing origin write kept in `OwnerWrite.Lost`, without an error (`internal/boardengine/ownerset.go:155-160`).
D6 reports only a failed write and a write that records nothing.

### [LOW] A sweep committed but not pushed makes the dobby flap

**Location:** `decision-record.md` D6 window Bound.
**Issue:** On a failed push the owner commit stays local (`internal/boardengine/ownerset.go:278`).
The dobby probe reads the local Bolt with no pull (`internal/dobbyengine/doc.go:173-174`) and lifts its pause.
The resume's verdict is judged against origin (`pattern/PATTERN-owner-machine.md:35`) and re-pauses.
D6's recovery bullet assumes a completed sweep.

### [LOW] Goal's no-change promise fails under a relative HOME with no file

**Location:** `decision-record.md` Goal, D3 second bullet.
**Issue:** Today `machinename.Name` reads only `os.Hostname` (`internal/machinename/machinename.go`).
After D3, a relative `HOME` or `LOCALAPPDATA` errors before any file is read, so a machine with no file changes behaviour.
The walkthrough lens raised the same point.

### [LOW] D1 states one directory where D2 resolves two

**Location:** `decision-record.md` D1.
**Issue:** `cacheDir` takes any non-empty `XDG_CONFIG_HOME` as given (`internal/githubclient/cache.go:48-50`).
`standalonestate.AbsoluteUserConfigDir` (new) ignores a relative one, so the machine file is not beside `slack.json` in that case.
The interactions lens raised the same point.

### [LOW] Auto-mode assumptions omit D4 and D8

**Location:** `decision-record.md` Auto-mode assumptions.
**Issue:** The support log's ledger records Q5 (D4) and Q8 (D8) as auto-picks, but the section lists only D1, D3, D5 and D6.

### [LOW] Required PATTERN background edits sit in optional Notes

**Location:** `decision-record.md` Notes for the plan writer.
**Issue:** The discussion spec makes Notes "never a completeness requirement".
The background edits are what the two declared `changes` bullets consist of, since the overview lines stay as they are.
Among them is the new rule that the owner setter overwrites another machine's owner for the replace set, besides `lym adopt`.
By the rubric's completeness-before-leanness test, that requirement belongs in Constraints.

### [NIT] D8's test-environment claim holds only off Windows

**Location:** `decision-record.md` D8.
**Issue:** `tmuxkit.Main` sets `LOCALAPPDATA` to the test's temp directory on Windows (`internal/testkit/tmuxkit/tmuxkit.go:600-611`).

## Rejected

- bound-breaker F4 (`BOARD_SKIP_GIT` and `BOARD_SKIP_PUSH` skip the pull): those variables switch off git for the board on purpose, so no origin exists to miss. Calling the origin claims false under them is over-flagging a mode the record does not need to bound.

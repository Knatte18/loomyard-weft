---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:80
    summary: D4 keeps the watch memory and owed lines in the run's state entry, which recordOwed deletes every tick once batten reads done, so a landed run loses its owed line and, with no memory, reposts its landed line each tick until the clear
    fix: "In D4, keep the watch memory and owed lines in a state-record map of their own keyed by slug, as LandNotices is (internal/dobbyengine/state.go:46), dropped only by the watch or the done-run clear, never by recordOwed (internal/dobbyengine/passes.go:319-340). In D1, name d.listedRuns as the run list the watch reads, since d.runs drops a done run with no pass in flight (passes.go:184-188). Add an acceptance bullet: a landed run whose batten reads done posts its landed line once across several ticks before the clear."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:70
    summary: "D3's skip reads the landing-notice memory, but that memory is decided and dropped at PastLand (Worktree-Teardown), before the derived state reads landed (batten done), so a land_notice-on run gets both lines; and a cancelled run with land_notice on keeps Requested=true while PastLand stays false, so neither the watch nor the landing notice tells the orch"
    fix: "Rewrite D3's skip so it rests on a fact that outlives the landing notice and covers only what the landing notice sends: the watch copies the run's land_notice reading into its own per-run memory (F1's map) while the run is ahead of Land, and skips only a landed or ended-without-landing reading of a run whose copied reading is on. A cancelled run is never skipped, since landNotices sends nothing for it (internal/battenrecipe/land.go:27-42, internal/dobbyengine/landnotice.go:119-156). Fix D3's rationale and the acceptance bullet on landed runs to match, and add one for a cancelled run with land_notice on that posts its watch line."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:37
    summary: Before Run-Shed a run has no child status, so the child part is empty, which D3's 'the child's state is not running' counts as a stop; every new run posts a spurious stop line at first sight
    fix: "In D1, state that a child part never read (no child status yet) is absent, and in D3 that an absent child part is no stop; only a read child state other than running is. Add an acceptance bullet: a run started before Run-Shed with no child status posts no line under stops."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:66
    summary: A cancel that removes its records (a --remove cancel, or one whose finish finds no record) deletes the batten status inside the cancel command, so dobbyParentRuns drops the run before any tick reads cancelled; the Goal's cancelled stop gets no line and the record does not say so
    fix: "Add to D3 (or the Scope Out list) the stated bound that a cancel which removes the run's records is reported by the cancel's own notice (internal/battencli/cancel.go postNotice) and not by the watch, and adjust the Goal's cancelled bullet to a cancel that keeps its records; or decide a mechanism that catches it, such as the watch posting a line for a run with memory that leaves the list while its last reading was cancelling."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:52
    summary: "D2's stops rule fires on a change 'into a stop' of any one layer, while D3 defines a stop over the whole reading; a batten pause lifting while the child stays blocked turns the derived state paused to halted, a resume that D2 posts, and a stop-to-stop change (child blocked to failed) is not said to post or not"
    fix: "In D2, define the stops rule over D3's whole-reading predicate: a line goes when the new reading is a stop and differs from the memory in any layer's state or in the episode stamp, row moves alone excepted; no line goes when the new reading is no stop. Add acceptance bullets for a child going blocked to failed (one line) and a batten pause lifting over a blocked child (one line, as the reading still stands at a stop with a changed state) so the plan writer has the intended answer."
  - id: F6
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:41
    summary: The run steps run only while scheduling holds (no anchor move, adopt hold or owed catch-up), so the watch posts nothing then, and a catch-up owed after a machine sleep delays stop lines; D1 does not state this bound
    fix: "Add to D1 the bound that the watch runs only on ticks whose scheduling steps run (internal/dobbyengine/loop.go:708-729), so an anchor move, an adopt hold or a catch-up delays its lines until it ends; or decide that the watch step runs outside the scheduling gate, before the done-run clear's gate."
  - id: F7
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:57
    summary: "'A failed per-task read keeps the run's last level' leaves no level for a run whose first read fails, and so leaves open whether that reading becomes the memory and whether D4's first-sight line goes"
    fix: "In D2, state that a run whose run_watch has never read is not judged: its reading neither posts nor becomes the memory until a read succeeds, as the landing notice keeps a run undecided until its first read (internal/dobbyengine/landnotice.go:26)."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:45
    summary: "D1's rationale says a pause seen during the Land lock wait derives as running or landing, but Derive takes paused whenever batten holds a pause intent or reads paused; only a batten block derives as running or landing"
    fix: "In D1's rationale, replace the pause example with a batten block outside Run-Shed, such as a blocked Land row, which derives as running or landing because Halted reads only the child's state (internal/runfsm/state.go:83-97)."
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:113
    summary: A multi-slot tier hold writes one holder record per slot with the same PID and start, so a key of slot, PID and start posts one gate-hold notice per slot
    fix: "In D6, key the gate-hold notice by PID and start, and have one notice name every slot that holder holds (internal/gateslot/pool.go:179-201); fix the matching acceptance bullet."
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:108
    summary: Pool.Holders fails the whole list when any held record is unreadable or unparsable, as a half-written record is, and D6 names a failure path only for the gate config load
    fix: "In D6, state that a failed Pool.Holders read skips the gate check for that tick with one Warn per failure episode, as a failed gate config load does, and add it to the acceptance bullet on the failed config load."
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:106
    summary: "gate_hold_factor and gate_hold_min are new hub keys with no stated range: a negative factor or a zero or negative fallback makes every hold notice at once, and the record says nothing of validating them"
    fix: "In D6, state that the dobby's config load refuses a negative gate_hold_factor and a gate_hold_min below one, naming the key and its range as the way forward, and that the bound of a flood is one notice per hold by the dedup key; add an acceptance bullet for the refusal."
  - id: F12
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:123
    summary: "D6 rests on Go's runtime timers running on CLOCK_MONOTONIC, but pins only clock_gettime(2) for the clock's suspend behaviour; the Go half of the fact has no pin"
    fix: "In D6's rationale, pin the Go half: name the Go runtime source where nanotime reads CLOCK_MONOTONIC on Linux (src/runtime/sys_linux_amd64.s, nanotime1) or a recorded command and output that shows a Go timer not advancing across a suspend, beside the clock_gettime(2) pin."
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:146
    summary: "The PATTERN-tolerant-records constraint names the slot holder record, but that record is YAML written by gateslot and decoded with yaml.Unmarshal, outside the pattern's Go-written JSON scope"
    fix: "In Constraints, keep PATTERN-tolerant-records for the dobby's state record only, and state in D6 that the holder record's new field is YAML decoded without KnownFields (internal/gateslot/pool.go:360-361), so an older record reads as no marker."
---

The record's design holds in outline, and its delivery, gate-config and per-task lookup claims check out against the code.
Two of its mechanisms break on the code they sit on: the watch memory lives in an entry the pass step deletes for every done run, and the landed skip reads a memory that is gone by the time a run derives as landed.

### [BLOCKING] Watch memory in the run's state entry is deleted for every done run

**Location:** `_lym/discussion/decision-record.md:80` (D4)
**Issue:** `drivePasses` drops a done run with no pass in flight from `runs` (`internal/dobbyengine/passes.go:184-188`), and `recordOwed` then deletes `s.Runs[slug]` for every slug not listed (`passes.go:333-336`), ahead of every later step (`internal/dobbyengine/loop.go:710`).
A landed run reads batten done, so each tick wipes its watch memory and owed lines; D4's no-memory rule then reads the landed reading as new and posts it again every tick, and the three-failure drop never counts.
D4's "the done-run clear keeps a run's entry" relies on an entry already gone.
Found by the interactions, existing-paths and walkthrough lenses alike.

### [BLOCKING] The landed and cancelled skip misfires both ways

**Location:** `_lym/discussion/decision-record.md:70` (D3)
**Issue:** `PastLand` turns true at the Worktree-Teardown row (`internal/battenrecipe/land.go:27-29`); on that tick `rememberRuns` decides the notice, and `settleLandNotices` posts it and drops the memory, or drops it at once when off (`internal/dobbyengine/landnotice.go:142-149`, `:171-177`).
The derived state reads landed only once batten reads done (`internal/runfsm/state.go:71-76`), a later tick, so the skip never fires and a `land_notice`-on run gets two landed lines, against the acceptance bullet at line 167.
A cancelled run never turns `PastLand`, so its memory keeps `Requested=true`: the watch skips it and the landing notice sends nothing, so the orch hears of neither.

### [MEDIUM] A run before Run-Shed reads as a stop

**Location:** `_lym/discussion/decision-record.md:37` (D1, D3)
**Issue:** `ReadChild` reports `found=false` while no child status exists (`internal/dobbyengine/runnotice.go:107-108`).
D1's keep-last rule covers only a child that vanished after a reading, so a fresh run's child part is empty, and D3 counts an empty child state as "not running".

### [MEDIUM] A cancel that removes its records is never read as cancelled

**Location:** `_lym/discussion/decision-record.md:66` (D3, Goal)
**Issue:** `clearRunPhase` removes the pair, branch and records inside the cancel command (`internal/battencli/cancel.go:544-565`), and `dobbyParentRuns` lists only slugs whose batten status exists (`internal/battencli/dobbyreads.go:104-111`).
The Goal promises a cancelled line, and the record is silent on this path.

### [MEDIUM] D2's per-layer stop change disagrees with D3's whole-reading stop

**Location:** `_lym/discussion/decision-record.md:52` (D2)
**Issue:** Paused outranks halted (`internal/runfsm/state.go:83-97`), so a pause lifting over a blocked child is a derived change "into a stop" under D2 though it is a resume, and a stop-to-stop change has no stated answer.

### [LOW] The watch is held with the scheduling steps, unstated

**Location:** `_lym/discussion/decision-record.md:41` (D1)
**Issue:** Run steps and the clear run only while `scheduling` holds (`internal/dobbyengine/loop.go:708-729`), so an anchor move, adopt hold or owed catch-up delays every watch line.

### [LOW] No level for a run whose first run_watch read fails

**Location:** `_lym/discussion/decision-record.md:57` (D2)
**Issue:** "Keeps the run's last level" has nothing to keep on a first failed read, and D4's first-sight line depends on the answer.

### [LOW] D1's rationale gives a pause as its example of a batten stop that derives as running

**Location:** `_lym/discussion/decision-record.md:45` (D1 rationale)
**Issue:** A batten pause intent or paused state derives as paused (`internal/runfsm/state.go:83-85`); only a batten block fits the claim.

### [LOW] Multi-slot holds notice once per slot

**Location:** `_lym/discussion/decision-record.md:113` (D6)
**Issue:** `AcquireSlots` writes one record per slot with one PID and start (`internal/gateslot/pool.go:179-201`), so the slot-keyed dedup posts several notices for one hold.

### [LOW] No failure path for Pool.Holders

**Location:** `_lym/discussion/decision-record.md:108` (D6)
**Issue:** One unreadable or unparsable held record fails the whole list (`internal/gateslot/pool.go:353-362`), and D6 states handling only for the gate config load.

### [LOW] The two new hub keys have no stated range

**Location:** `_lym/discussion/decision-record.md:106` (D6)
**Issue:** Only a zero factor has a meaning; a negative factor or a non-positive fallback notices every hold at its start, and no validation is decided.

### [LOW] The Go half of the suspend-clock fact is unpinned

**Location:** `_lym/discussion/decision-record.md:123` (D6 rationale)
**Issue:** `clock_gettime(2)` pins that `CLOCK_MONOTONIC` excludes suspend, but nothing pins that Go's runtime timers, and so the tier timeout, run on that clock.

### [LOW] PATTERN-tolerant-records named for a YAML record

**Location:** `_lym/discussion/decision-record.md:146` (Constraints)
**Issue:** The pattern covers Go-written JSON records; the slot holder record is YAML decoded with `yaml.Unmarshal` (`internal/gateslot/pool.go:360-361`).

## Rejected

- existing-paths 3 (D5 seam must carry `posted` rather than go through `requirePosted`): D5 states the behaviour, and `NotifyRunFrom` already returns `posted` (`internal/orchcli/notify.go:73`); choosing the seam is Plan's, and the rubric does not ask Discussion for call-site enumeration.
- existing-paths 2 bullet on which run list the watch reads: folded into F1's fix, not a separate defect.
- bound-breaker F6 (marker derivable from `Holder.PID`): `runMarker` reads the marker from the environment, and an enclosing gate's `LYM_GATE_TEST_PID` can name a live ancestor rather than the holder (`internal/gatecli/doc.go:580`), so recording the marker is a sound choice.
- Duplicates merged: interactions F1, existing-paths 1 and walkthrough F1 into F1; interactions F2, existing-paths 2, walkthrough F2 and bound-breaker F1 into F2; walkthrough F3 and bound-breaker F2 into F4; interactions F4 and bound-breaker F7 into F7.

Other board entries: `dotnet-gate-fixes`, `beam-member-parent` and `config-keys-nested` agree with the record's Scope Out, Open risks and plan note, and the landed background-pusher and run-agent-resume commits touch no state the record changes.

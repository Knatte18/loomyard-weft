---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:31
    summary: D1 never defines which launches count as plain, so it is open whether a --handoff or --adopt launch over a recorded id emits the hint, and an --adopt launch drops the previous id with no surviving copy
    fix: In D1, after the sentence that introduces the hint, add one sentence fixing the set, for example that every fresh launch that saves an id other than the recorded one emits the hint, which covers a plain launch, a --handoff launch and an --adopt launch naming another id, while a --continue launch never does; then make the Scope In bullet at line 14 and the Acceptance criterion at line 107 use the same wording in place of a plain fresh launch.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:53
    summary: D3's Bound says --continue can never take over a session orch did not launch, adopt or bind, but Open risks line 97 says a provider version can make it resume a subagent's session
    fix: In D3's Bound sentence, qualify the claim to the id the watcher reads from the strand's own run, and state that it rests on the provider naming the parent session id in a subagent's or fork's hook lines, as the matching Open risks bullet records; keep the Open risks bullet as is.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:77
    summary: The question ledger's Q7 and the Turn 5 goal-scope line still say the hint points at --continue, while D1 now names lyx orch stop then --adopt with the previous id
    fix: In support-log.md, change Q7 to ask whether a fresh launch should point back at the previous conversation and resolve it as the envelope hint naming lyx orch stop then lyx orch start --adopt with the previous id inline, since a --continue after the launch would resume the new session; leave the Turn 5 history line as is or append that the hint later moved to --adopt.
---

### [LOW] D1 leaves open which launches emit the previous-id hint

**Location:** `_lyx/discussion/decision-record.md:31`

**Issue:** D1, the Scope In bullet and the Acceptance criterion all say a plain fresh launch with a recorded id emits the hint.
The record uses plain two ways: D1's own paragraph lists `--handoff` among the prompt choices of a launch without `--continue`, while D4 at line 58 and the criterion at line 106 contrast plain with `--adopt`.
So it is open whether a `--handoff` launch emits the hint, and under the narrower reading an `--adopt <other id>` launch replaces the recorded id with no hint.
D4's write runs on that launch too, so the previous id then survives nowhere, which D1's own line 32 says the hint exists to prevent.
The plan writer has to pick, and a test pinned to one reading is wrong under the other.

### [LOW] D3's Bound claims more than the Open risks allow

**Location:** `_lyx/discussion/decision-record.md:53`

**Issue:** The Bound says `--continue` can never take over a session other than one orch launched, adopted or bound.
D4 reads the id from the newest signal of any kind, and the Open risks bullet at line 97 states that a provider version naming a subagent's own id in its hook lines makes the watcher record it, so `--continue` then resumes the subagent's session.
The two sentences contradict each other, and the Bound is the sentence a reader checks for what `--continue` can let through.
Round 1's F4 fix added the Open risks bullet but left the Bound absolute, so this is a new inconsistency, not a regression of that fix.
I checked the code: `readSessionFacts` in `internal/shuttleengine/sessionread.go` takes the session id of the newest signal of any kind, so the risk is real and the Bound has to carry its condition.

### [LOW] Support log's Q7 contradicts D1's hint

**Location:** `_lyx/discussion/support-log.md:77`

**Issue:** Q7 resolves the plain-launch pointer as a hint naming `--continue`, and the Turn 5 goal-scope line at line 43 says the same.
D1 now names `lyx orch stop` then `lyx orch start --adopt <previous id>`.
That is correct, because the plain launch's save replaces the recorded id, so a later `--continue` resumes the new session.
The ledger is the support log's record of each resolved question, and it now records a decision the record no longer makes.

## Focus answers

- Directive 1, where the previous id is read relative to the save: no defect.
  In `internal/orchcli/start.go` the state is loaded at line 212 and the fresh launch's `SaveState` runs at line 254, and the envelope is built from the loaded copy after that.
  D1 requires the previous id from before the save, so the hint names the conversation the operator had before this launch.
  The open part is which launches emit the hint (F1).
- Directive 2, the routes that write `session_id`:
  - Fresh launch: D4, with the id from shuttle's start seam.
  - Adopt and `--continue`: D4 and D2, with the adopted id.
  - Watcher tick: D4, a fold over all the run's signals with a strand-checked save.
  - `start` binding a live strand without launching: D4 clears the id, and the Acceptance criteria bound the case with no shuttle run.
  - A strand relaunched by `lyx reed resume`: the Out bullet and the fourth Open risks bullet.
  I confirmed that `planUpLaunches` in `internal/reedengine/lifecycle.go` never relaunches a strand, so the `reed.Up` call in `start` leaves the orch strand dead after a reboot and `--continue` reaches the relaunch branch.
  I also confirmed that `Engine.Resume` relaunches a strand under its existing guid with its stored `ResumeCmd`, so `adoptStrand` does not clear the id there, and the Out bullet's claim holds that `lyx orch stop` then `--continue` recovers the post-clear id until a watcher ticks.
  The one claim that overreaches is D3's Bound (F2).
- Directive 3, bounds and hint order: D1 orders the watcher-stopping hint first.
  `DecideStart` in `internal/orchengine/launch.go` returns a relaunch for a dead strand even while the watcher is live, so both hints can apply together.
  The `session_id` envelope field and the status line report a value and let nothing through.
  `--continue` is bounded by D3, apart from F2.

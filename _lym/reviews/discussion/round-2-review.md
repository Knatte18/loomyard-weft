---
verdict: BLOCKING
findings:
  - id: F1
    carries: F2
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:140
    summary: "D8 reads an unsettled parent entry's home, or its last desktop, as where the parent's own placement sends it, but a parent under mode off is never moved, a parent whose home is full overflows to the desktop after it, and a homeless parent is placed by mode; a child that takes the lock before its parent's child therefore still anchors on a desktop its parent does not land on and records that anchor as its own home for good (D9), and the 180-second fallback to the current desktop records its anchor as home the same way"
    fix: "In D8, anchor only on a settled parent entry: under the lock, a parent entry that is unsettled sends the child back to the parent wait outside the lock, releasing the lock, and the wait's 180-second bound covers every return to it; delete the sentence giving an unsettled entry's home or last desktop as the anchor. In D8's Bound, or in D9, state what a window anchored on the current desktop past the wait records: either it stays homeless, so its next placement anchors on its parent again, or the fallback desktop becomes its home. Replace the first half of the second D8 acceptance criterion (anchoring on an unsettled entry's home) with: a child whose parent's entry is unsettled under the lock waits again and anchors on the desktop the parent's placement settles it on, including a parent under mode off and a parent whose full home overflowed it."
  - id: F2
    carries: F6
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:182
    summary: "D9's renumber gives every present window of a desktop a new key, so a placement on the desktop an overflowed window landed on rewrites that window's order key, which belongs to its home desktop's sequence; the overflow's kept home key, which F6's fix relies on to return it to its place, is lost"
    fix: "In D9, restrict the renumber to the windows whose home is that desktop: change 'A renumber gives the desktop's present windows' to 'A renumber gives the present windows whose home is that desktop', and in the trigger bullet compare only those windows' current order with their keys. A present window homed elsewhere keeps its home and order key. Extend the fourth D9 acceptance criterion with: a renumber of the desktop an overflowed window landed on leaves that window's home and order key unchanged."
  - id: F3
    carries: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:245
    summary: "The facts D3, D4 and D6 rest on (rootTile, split and manage on a desktop that is not current, the desktop and output move, frameGeometry and clientArea off the current desktop, kscreen-doctor names against workspace.screens names, the two-desktop rootTile probe, a script body running whole once run is called) are still left for later under 'Not yet observed', with a rule that code waits on them, which is the deferred research the rubric flags"
    fix: "Get each listed fact observed now. The read-only facts (kscreen-doctor -j output names against workspace.screens names, and whether rootTile(output, desktop) returns different root tiles for two desktops) move no window: the fixer posts to its parent through lym creel post --parent --expect-reply and asks for each to be run on Neve, giving the script and command. The facts that move windows (a grid built and managed on a desktop that is not current, setting desktops and sendClientToScreen, frameGeometry on another desktop) are put to the parent the same way for the operator to run or approve. Record each answer in Constraints with its command and observed output, and drop the 'before code relies on it' rule. A fact the parent declines to have observed moves to Open risks, naming the decision that rests on it."
  - id: F4
    carries: F13
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:228
    summary: "D12 runs window_command on every exit path of the child, and D9 and D10 make SIGTERM one of them, but runproc.End sends SIGKILL 5 seconds after SIGTERM (internal/runproc/end.go endGrace) while desktophook bounds window_command at 30 seconds, so on that path the command is lost or cut off and the record does not say which"
    fix: "In D12, name the SIGTERM exit: either it runs no window_command, or it runs one that is killed with the child once runproc.End's grace runs out. Add that case to the first D12 acceptance criterion."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: open
    answer: "The anchor now moves under the lock, but an unsettled parent entry's home is not where the parent goes when its mode is off, its home is full or it has no home, so lock order still decides the anchor; carried as F1."
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: open
    answer: "The overflow keeps its home and key, but D9's renumber of the desktop it landed on rewrites that key; carried as F2."
  - id: F7
    status: open
    answer: "The deferral rests on the session not reaching the user session bus, but D2's evidence came from kdotool, which runs KWin scripts over that bus, and the read-only facts move no window. The fixer asks its parent to get them run, and the operator to run or approve the ones that move windows; carried as F3."
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: open
    answer: "Every early exit now runs window_command, but the SIGTERM exit sits inside runproc.End's 5-second grace and the record does not say whether it runs the command; carried as F4."
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: resolved
  - id: F17
    status: resolved
---

### [BLOCKING] An unsettled parent's home is not where the parent lands

**Location:** `_lym/discussion/decision-record.md:140` (D8), with `:150`, `:174-175` (D9) and `:34` (D1).

**Issue:** D8 reads an unsettled parent entry's home desktop as "where the parent's own placement sends it".
Three cases break that claim.
A parent under mode `off` is recorded where KWin opened it, and its old home stays.
A parent whose home is full overflows to the desktop after it, and keeps the full home.
A parent with no home is placed by mode, not on its last desktop.
A child that takes the lock before its parent's child anchors on that desktop, and D9 makes the child's landing desktop its home for good.
Whether the child gets the right anchor still depends on which child takes the lock first, the race F2 was raised for.
The 180-second fallback to the current desktop records its anchor as home in the same way (bound-breaker F-E).
The walkthrough and interactions lenses found the defect on their own.

### [MEDIUM] A renumber on an overflow's landing desktop rewrites its home key

**Location:** `_lym/discussion/decision-record.md:182` (D9).

**Issue:** Suppose a window overflowed from its full home H onto desktop L.
It keeps H as its home and keeps its order key (`:175`), and it sits on L among windows homed on L.
A later placement on L that renumbers L gives "the desktop's present windows" new keys, and the overflowed window is one of them.
Its key belongs to H's sequence, so H's order for that window is lost, and the "no two entries share a key" rule can break on H.
The interactions and walkthrough lenses judged this case held.
They read "present windows" as the windows homed on the desktop, but the text does not say so.

### [MEDIUM] KWin facts are still left for later

**Location:** `_lym/discussion/decision-record.md:245` (Constraints).

**Issue:** The `Not yet observed` list names six facts that D3, D4 and D6 rest on.
It hands their observation downstream, with a rule that code waits for them.
The round-1 deferral cited the session bus, but D2's own evidence comes from `kdotool getwindowpid`, which runs KWin scripts over that bus.
Two of the facts are read-only and move no window.
For the rest, the way forward is the parent, not a later step.

### [LOW] Whether a SIGTERM exit runs window_command is unstated

**Location:** `_lym/discussion/decision-record.md:228` (D12).

**Issue:** "On every exit path" covers the SIGTERM exit that D9 and D10 name.
`runproc.End` sends SIGKILL 5 seconds later (`internal/runproc/end.go` `endGrace`).
`desktophook` lets `window_command` run for 30 seconds (`internal/desktophook/window.go` `productionTiming`).
So the record promises a command it cannot always finish.
The existing-paths lens found the same gap (F-EP2).

## Valid findings not raised this round

This round is incremental, so a new finding is raised only at BLOCKING.
The lens findings below are sound at the severity given, and none reaches BLOCKING.
They stay here for the parent and the next full round:

- walkthrough W1 (MEDIUM): D3's query reports no KWin window identity such as `internalId`, so the apply cannot name a non-lym window that shares its pid with another, as `arrange-to-tiles.sh` does with `w.internalId`.
- walkthrough W2 (MEDIUM): D6 finds an untiled window's zone from zone boxes and padding that D3's query does not report.
- bound-breaker F-A (MEDIUM): D3 never makes the plugin name unique per run, while D10 runs query scripts at once outside the lock; the operator's scripts use `$$`.
- bound-breaker F-B (MEDIUM): `state.UpdateJSON` aborts on an undecodable file, so "an unreadable file reads as empty" does not hold for writes, and a corrupt places file stalls every child for the full parent wait.
- bound-breaker F-F (MEDIUM): D10's bound on the hidden verb omits the hub config it reads and the `window_command` shell command it runs.
- interactions 2 (MEDIUM): a window that appears after the 10-second wait keeps its old last desktop, so the next apply reads it as a hand move and makes KWin's opening desktop its home.
- bound-breaker F-C and walkthrough W4 (LOW): a blocking acquire that succeeds after the 120-second timer, while the child runs `window_command`, holds the lock unused; the Slack holder releases a late acquire.
- bound-breaker F-D (LOW): the first-start bound must also cover the ancestors' own placements, not only the beam starts.
- existing-paths F-EP1 (LOW): nothing tells `lym reed window` which run it runs inside, so D10's `runproc` registration has no owner to register under.

The focus entries on startup order, the per-user lock and tiling under the panel were driven to a verdict.
Startup order: `runBeamStarts` before `openWindows` is stated right, and the first-start fallback is a stated bound.
The per-user lock: `kwinplace.lock` sits beside `launchers.lock` in `standalonestate.UserStateDir` and serializes every hub.
Tiling under the panel: the clamp and the count rule agree with `arrange-to-tiles.sh`.
None of these has a defect beyond those listed above.

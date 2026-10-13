---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md D10 (third paragraph), Constraints (PATTERN-log-levels line)
    summary: D10's failed rename or move logs Warn once per session with a never-cleared session option and no recovery line, which departs from the internal/logger Level policy the Constraints say it follows
    fix: >-
      In D10, replace "a failed rename or move is logged at Warn once per session and at Debug while it persists, and at Debug on psmux, as D2 does; a session option records the warned failure, since each open and attach is its own process." with "a failed rename or move is logged at Warn once per failure episode, at Debug while it persists and at Info when a later check succeeds, and at Debug only on psmux, as D2 does; a session option records an open episode, since each open and attach is its own process, and a successful check clears it." Leave D2's psmux Debug as it is.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3, D10
    summary: D3 and D10 treat the strands' window as Selvage's window, but strandWindowTarget falls back to a window-0 non-own-window strand and then to the session's current window, so D10 can rename and move the overview or an agent window
    fix: >-
      In D10, after "reed checks the strands' window", add the sentence "The check acts only on a window that holds the recorded Selvage pane, alive or dead; when strandWindowTarget resolves through its fallbacks (a non-own-window strand in window 0, which after D1 can be the overview strand, or the session's current window), the check renames and moves nothing and logs at Debug." In D3, replace "It stays the strands' window that strandWindowTarget resolves through the Selvage pane" with "It stays the strands' window that strandWindowTarget resolves first through the Selvage pane; an overview strand's pane is excluded from the resolver's window-0 strand fallback, as an own-window strand's is". Add an acceptance criterion: "With no recorded Selvage pane in the session, an attach or window open renames and moves no window."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 (Attach bullet and Bound)
    summary: The attach select-window changes the session's current window for every attached client, so a second attach moves an operator already in the shell or a live agent window, contradicting D5's own Bound and the shell-counts-as-watched rule
    fix: >-
      In D5's Attach bullet, after "with neither an agent nor an overview it appends nothing and leaves the current window.", add "When another client is already attached to the session, it appends nothing either, so a second attach never moves a viewer; the client listing the pre-flight already takes decides this." Add to the Attach acceptance criterion: "An attach while another client is attached leaves the current window unchanged."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 (second sentence)
    summary: D1 states the overview holds the lowest index whenever it is opened, with no caveat for D2's psmux fallback, where a refused -b opens it at the first free index
    fix: >-
      In D1, after "so it holds the lowest index whenever it is opened.", add "On psmux, where D2's fallback applies, this is best-effort."
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Acceptance criteria (last bullet)
    summary: The migration criterion does not cover the attach paths the D10 placement exists for, Selvage alone in the strands' window and no known client size
    fix: >-
      Extend the last acceptance criterion with "This holds when Selvage is the only pane in the strands' window and when no client size is known."
---

The focus's first question settles in the design's favour.
D5's Attach bullet now appends the selection to every argv `AttachArgv` returns, resolved under the op lock ahead of the layout guards and also with no client size, so the common session where Selvage is alone in the shell window (`len(live) < 2` in `internal/reedengine/attach.go`) still selects the newest live agent's window.
The only remaining drops are a failed lock or state read, and a failed `requireSessionLocked` or `strandWindowTargetFor`, which leave nothing to select or no listing to select from.
D10's placement shares that position, so no early return skips the migration either; the gap is in the acceptance criteria (F5) and in which window the migration may touch (F2).

### [BLOCKING] D10's failure logging departs from the Level policy

**Location:** `_lyx/discussion/decision-record.md` D10, Constraints

**Issue:** `internal/logger/logger.go`'s Level policy: "A failure that repeats each tick logs Warn once per episode, Debug while it persists, and Info at recovery; Episode and EpisodeSet implement this."
D10 retries the rename and move at every window open and attach, so the failure repeats per open, and D10 adopts the episode shape on purpose, but as "Warn once per session" with a session option that records the warned failure and is never cleared.
A failure that recovers and later recurs therefore never warns again, and recovery logs nothing.
The Constraints line claims D10's levels follow `PATTERN-log-levels`, so the design contradicts the constraint it cites.
D10's psmux Debug agrees with D2's Debug for refused `-b` and `-a` insertions; that part needs no change.

### [MEDIUM] The strands' window is not always Selvage's window

**Location:** `_lyx/discussion/decision-record.md` D3, D10

**Issue:** `strandWindowTarget` in `internal/reedengine/windowtarget.go` resolves the Selvage pane's window, then the window of any recorded non-own-window strand pane in window 0, and only then the session's current window (`exactSessionWindowTarget`).
`recordedPaneIDs` in `selvagepane.go` excludes only `AnchorOwnWindow` strands, and `isOwnWindow` tests `d.Anchor == render.AnchorOwnWindow`, so a D1 `overview` strand, which D1 places at the lowest index, is a candidate for the window-0 fallback.
D10's check runs before every window open and in every attach and renames the resolved window `shell` and moves it last.
With Selvage's pane gone from the session's listing (killed, or an old session whose state predates it), the check renames the overview window or the viewed agent window `shell` and moves it after the agents.
D10's Bound, "renames and moves only the strands' window", is true only of the resolver's answer, not of Selvage's window, so it bounds nothing in this case.
D2 already plans for "no shell window resolves", but the resolver never reports that; it returns a window.

### [MEDIUM] Attach selection moves other attached clients

**Location:** `_lyx/discussion/decision-record.md` D5

**Issue:** tmux keeps one current window per session, shared by every client attached to it, and `select-window` in the attach argv sets it for all of them.
`AttachArgv` already lists the attached clients (`warnMismatchedClientsLocked`), so a second client is a case the code knows about.
A second attach, for instance `lyx reed attach` from another terminal into a session where the operator is typing in the shell, moves that operator to the newest agent's window.
That contradicts D5's Bound ("never moves a viewer off the shell or a live agent window"), D5's "the shell counts as watched", and the support log's rejected alternative "A new agent taking focus from the shell. It would steal keystrokes from an operator typing there."

### [LOW] D1 claims the lowest index unconditionally

**Location:** `_lyx/discussion/decision-record.md` D1

**Issue:** D1 opens the overview with `new-window -d -b` and states that it "holds the lowest index whenever it is opened".
D2's psmux fallback drops a refused `-b` to plain `new-window`, which takes the first free index.
D5 and D6 carry the psmux best-effort caveat; D1, the decision the order rests on, does not.

### [LOW] Migration acceptance does not cover the attach paths D10 targets

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria

**Issue:** D10 places the rename and move ahead of the layout guards and the no-client-size return so they run when Selvage is alone in the strands' window or no size is known.
The Selvage-alone and no-client-size clause is in the attach-selection criterion only; the migration criterion ("In a session booted by an older `lyx`, the next attach or window open renames…") does not name either path, so a plan could place the migration after the guards and still meet every criterion.

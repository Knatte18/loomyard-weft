---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:72
    summary: D5's attach selection has no rule for a failed client listing, and its closed list of drops lets the select-window go out blind, which can move another client's viewer off the shell or a live agent window against D5's Bound
    fix: >-
      In D5's Attach bullet, after the sentence ending "the client listing the pre-flight already takes decides this.", add "A failed client listing counts as another client attached: the selection is dropped, logged at Warn." and change "Only a failure to take the lock or read reed state drops it, logged at Warn." to "Only a failure to take the lock, read reed state or list the attached clients drops it, logged at Warn." Add to the Attach acceptance criterion in Acceptance criteria the line "An attach whose client listing fails leaves the current window unchanged."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:136
    summary: D10 says the resolver's window-0 strand fallback can be the overview strand after D1, while D3 excludes the overview strand's pane from that fallback
    fix: >-
      In D10, replace "(a non-own-window strand in window 0, which after D1 can be the overview strand, or the session's current window)" with "(a `top` or `below-parent` strand in window 0, or the session's current window)".
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:152
    summary: The PATTERN-log-levels Constraint names D9 among the Decisions whose Debug and Warn levels follow the policy, but D9 states no log level
    fix: >-
      In the Constraints line starting "`PATTERN-log-levels`:", change "in D2, D5, D8, D9 and D10" to "in D2, D5, D8 and D10".
---

### [MEDIUM] Attach selection on a failed client listing is unbounded

**Location:** `_lyx/discussion/decision-record.md:72`

**Issue:** tmux keeps one current window per session, so a `select-window` chained onto an attach moves every client attached to that session.
D5 guards this with the pre-flight's client listing: with another client attached it appends nothing.
Today that listing is `warnMismatchedClientsLocked` in `internal/reedengine/attach.go`, which on a failed `list-clients` logs at Warn and carries on, and it runs only after the no-client-size early return.
D5 then closes the set of drops: "Only a failure to take the lock or read reed state drops it".
Read together, a failed client listing does not drop the selection, so the attach appends a `select-window` with no knowledge of whether another client is watching.
That attach can move a second client's view off the shell or a live agent window, which D5's Bound ("never moves a viewer off the shell or a live agent window") and the shell-counts-as-watched rule forbid.
The decision on what a failed listing means is left to the plan writer, and the safe reading is not the one the text gives.
D5's text "also when no client size is known" does imply the listing must now run on the no-size path too, which the plan writer can derive; the gap is only the failure case.

### [LOW] D10 contradicts D3 on the overview strand in the resolver fallback

**Location:** `_lyx/discussion/decision-record.md:136`

**Issue:** D3 states that an overview strand's pane is excluded from `strandWindowTarget`'s window-0 strand fallback, as an own-window strand's is (today `recordedPaneIDs` in `internal/reedengine/selvagepane.go` skips own-window strands, and D3 extends that to the overview).
D10's parenthetical says the fallback "after D1 can be the overview strand", which is false under D3.
Behavior is unaffected, since D10 acts on no fallback-resolved window, but the plan writer reads two decisions that disagree about the resolver.
On the focus question itself: with the Selvage-pane condition in D10, the rename and move cannot act on the overview window or an agent window when the Selvage pane is absent, and the Bound holds.

### [NIT] PATTERN-log-levels Constraint cites D9, which sets no level

**Location:** `_lyx/discussion/decision-record.md:152`

**Issue:** D9 skips an apply and its zoom bracket and names no log line, so listing it among the Decisions whose "Debug and Warn levels" follow the policy cites a level that does not exist.
The rest of the line holds for D10: a failed rename or move logs Warn once per episode, Debug while it persists and Info at recovery, matching `internal/logger`'s Level policy and its `Episode` contract, and a session option carrying the open episode across processes is the needed adaptation since each open and attach is its own process.
Attach and window open are one-shot, not polls, so "nothing logs per poll above Debug" is not contradicted by D5's or D10's Warn.

## Focus verdicts

- D10 levels and the `PATTERN-log-levels` line: D10 matches the policy; the Constraint line is true of D10 and over-claims only for D9 (F3).
- D3 and D10 with the Selvage pane absent: the check acts on no overview or agent window and the Bound holds; D10's parenthetical contradicts D3 (F2).
- D5's attach `select-window` and a second client: covered when the listing succeeds; unbounded when it fails (F1).
- The migration acceptance criterion: it states that the rename and move hold when Selvage is the only pane in the strands' window and when no client size is known, which forces D10's placement ahead of the layout guards and ahead of `AttachArgv`'s no-size early return. Not a defect.

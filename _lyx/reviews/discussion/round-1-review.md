---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D8 and Constraints (second bullet)
    summary: D8 declares internal/selfexec a new leaf package with a closed import set, but the Constraints place that set only in its doc.go, outside PATTERN-leaf-packages, whose background is where every leaf package's closed set is declared and which each leaf pins with a leaf_enforcement_test.go
    fix: In decision-record.md Constraints, replace the bullet that begins with internal/selfexec documents its closed import set with two bullets. First, PATTERN-leaf-packages applies to internal/selfexec, so pattern/PATTERN-leaf-packages.md gains an internal/selfexec section stating that it imports only the standard library and internal/logger, and internal/selfexec gets a leaf_enforcement_test.go that pins that allowlist through scankit.AssertImportAllowlist, as internal/segmentcolor does. Second, internal/statuspane/doc.go's import list gains internal/selfexec.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D1 (replace-selection sentence) against D5 Bound and D10
    summary: D1 selects the replacement's window whenever the replaced strand's window was current, so D10's replace of an older top-anchored status strand, whose window is the shell, moves a viewer on the shell to the overview, which D5's bound forbids
    fix: In D1, change the sentence that begins When the replaced strand's window was the session's current window to read - When the replace ends the session's current window (the replaced strand was alone in it), the replacement's window is selected; a replace whose old strand shared a window that stays open, such as a top strand in the shell window, switches nothing. Then add to the second D1-related acceptance criterion - replacing a top loom-status strand while the shell window is viewed leaves the shell window selected.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (second bullet)
    summary: The hidden-window command new-window -d -a names no target, and tmux inserts an -a window after the target window, which defaults to the session's current window, so a hidden window opened while an agent window is viewed lands among the agent windows and breaks D6's premise that hidden windows sit only after the shell
    fix: In D2's second bullet, replace (new-window -d -a) with (new-window -d -a -t <highest-index window>), so the target is the session's highest-index window, read at open time under the op lock.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (third and fourth bullets) and D6 (one-conditional-step sentence)
    summary: The no-shell fallback uses plain new-window -d, which takes the lowest free index, so an own window can fill a gap left by an ended agent or land after hidden windows; D2 states no outcome for this case, while D5's newest-is-highest-index rule and D6's always-one-step claim depend on the order
    fix: In D2's third bullet, replace the plain new-window -d fallback with - when no shell window resolves, an own window opens with new-window -d -a -t <highest-index non-hidden window> and a hidden window as in the second bullet, so start order and hidden-last still hold, logged at Debug. In D6, change Since hidden windows only sit after the shell (D2), one conditional step always reaches a visible window to Since hidden windows sit after the shell (D2), one conditional step reaches a visible window; on psmux, where D2's order is best-effort, a step may land on a hidden window.
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D10 (first paragraph)
    summary: D10 adds a rename and a move-window step before every window open and in the attach pre-flight but states no outcome when either fails or races, such as psmux refusing move-window -a or the window changing between the check and the move, so a failed migration step could fail a window open or an attach
    fix: Add to D10, after its first paragraph - The check runs under the op lock; a failed rename or move is logged at Warn, never fails the window open or the attach, and is retried at the next open or attach. Add a matching Bound line to D10 - the check renames and moves only the strands' window, and only when its name or position is wrong.
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/support-log.md Interview item 6 (leanness bullet) and _lyx/discussion/decision-record.md Open risks
    summary: The consequence that a re-exec'd overview pane keeps a sidecar naming the old build, so the next lyx loom start or lyx orch start replaces it once more, is stated only in the support log, which Plan-Write never reads; without it a plan writer may add a sidecar rewrite to D8 or write an acceptance test that expects no replace after a re-exec
    fix: Add one bullet to decision-record.md Open risks - A re-exec'd overview pane keeps a sidecar naming the build it started with, so the next lyx loom start or lyx orch start replaces it once more; D8 does not rewrite the sidecar.
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Scope, Out, first bullet
    summary: The first Out bullet carries history and deliberation (the logging-cleanup commit hash and the argument that the board's lock gap is harmless), which the plan writer does not act on and which belongs in the support log
    fix: Cut the first Out bullet down to - The hub watchdog's restart on a new binary, which logging-cleanup already retires and replaces with a successor. Move the commit hash and the lock-gap sentence into support-log.md's Interview item 1.
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria, last bullet
    summary: The migration criterion says the strands' window is moved last, but D10 moves it after the last non-hidden window, with hidden windows still after it
    fix: In the last acceptance criterion, change renames the strands' window shell and moves it last to renames the strands' window shell and moves it after the last non-hidden window.
---

### [BLOCKING] selfexec's leaf import set bypasses PATTERN-leaf-packages

**Location:** `_lyx/discussion/decision-record.md` D8 and Constraints

**Issue:** D8 calls `internal/selfexec` "a new leaf package, importing only the standard library and `internal/logger`".
`PATTERN-leaf-packages` covers exactly that case, and its background (`pattern/PATTERN-leaf-packages.md`) is where each leaf package's closed set is declared.
Every listed leaf also pins that set with a `leaf_enforcement_test.go` calling `scankit.AssertImportAllowlist`, as `internal/segmentcolor/leaf_enforcement_test.go` does.
The Constraints place the set in `internal/selfexec/doc.go` alone.
Plan-Write reads only the decision record, so it would land a leaf package outside the pattern's registry with no enforcement test.
The support log's fix for the `statuspane` premise (statuspane is not a leaf) does not apply to selfexec, which is one.

### [MEDIUM] D1's replace selection moves a viewer off the shell

**Location:** `_lyx/discussion/decision-record.md` D1, D5 Bound, D10

**Issue:** D1 says "When the replaced strand's window was the session's current window, the replacement's window is selected."
D10 relies on `lyx loom start` replacing an older build's `top` status strand, and that strand lives in the shell window, which `ReplaceStrand` does not close.
The rule as written then selects the new overview window for an operator typing in the shell.
D5's bound says an automatic switch "never moves a viewer off the shell".
The two Decisions disagree on the one cross-anchor replace D10 makes routine for every older session.

### [MEDIUM] The hidden-window `-a` names no target

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:** `OpenWindow` today targets `exactSessionWindowTarget(session)` (`=session:`), which resolves to the session's current window.
`new-window -a` inserts after the target window, so `new-window -d -a` places the hidden window directly after whatever window the operator is viewing.
D6's single conditional step ("Since hidden windows only sit after the shell") then fails whenever batten opens a hidden window while an agent window is current.
D1 gives its `-b` an explicit target; D2's `-a` needs one too.

### [MEDIUM] The no-shell fallback leaves the window order unbounded

**Location:** `_lyx/discussion/decision-record.md` D2, D6

**Issue:** The focus directive asks whether each fallback leaves a stated, bounded outcome.
The psmux fallback does ("best-effort on Windows and nothing fails").
The no-shell fallback states only its log level.
Plain `new-window -d` takes the lowest free index, so after an agent window ends, the next agent can open before older agents, or after hidden windows.
D5's "highest-index agent window with a live pane ... is the newest live agent" and D6's "always" both assume D2's order, and neither names this exception.
A targeted `-a` after the highest non-hidden window keeps the order without a shell, so the fallback need not degrade at all.

### [MEDIUM] D10's migration step has no failure or race bound

**Location:** `_lyx/discussion/decision-record.md` D10

**Issue:** D10 adds a rename and a `move-window` to every window open and every attach pre-flight.
It does not say whether a failed rename or move fails the open or attach, whether the step runs under the op lock against a concurrent open, or whether it is retried.
The rubric's attack-surface question asks each new routing step to state what it can skip or let through and what bounds it.
D8 states its bound, but D10 states none.

### [LOW] The stale-sidecar consequence is only in the support log

**Location:** `_lyx/discussion/support-log.md` Interview item 6; `_lyx/discussion/decision-record.md` Open risks

**Issue:** `statuspane.StrandCurrent` compares the sidecar's build with the current build, and both `loomcli`'s `ensureStatusStrand` and `orchcli`'s runs-strand ensure write the sidecar only when they add or replace.
After a D8 re-exec, the pane runs the new build while the sidecar still names the old one, so the next start replaces a pane that is already current.
The leanness lens moved this out of the record as harmless.
The completeness test still applies: it tells the plan writer that D8 does not rewrite the sidecar and that tests must not assume otherwise.

### [LOW] Scope Out carries history and deliberation

**Location:** `_lyx/discussion/decision-record.md` Scope, Out

**Issue:** The commit hash `a90ef61d1` and the argument that the board's lock gap is harmless are history the plan writer does not act on.
The exclusion itself is a requirement and stays.

### [NIT] Migration criterion says "last"

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria

**Issue:** D10 moves the strands' window "after the last non-hidden window", and D2 keeps hidden windows after it.
The criterion's "moves it last" asserts a position D10 does not state.

## Rejected

- D5 and D6 with several hidden windows together: once F3's target fix holds, hidden windows all sit after the shell and one conditional step from either end reaches `:^` or the shell, so the premise holds and this is not a defect.
- D5 with the viewed window ending during a replace: only the two overview strands are replaced in production (`loomcli/sharedbootstrap.go`, `orchcli/runs.go`), and D1's rule covers them; the one wrong case is F2's shared-window replace.
- D8's bound: it states the exec runs only the recorded path with the process's own argv, a failed stat never execs, a failed exec is not retried for the same identity, and two consecutive equal stats are required, so the exec path is bounded. Its failure modes are in Open risks.
- D1's refused second `overview` strand: the refusal names the existing strand and the way forward, and a cross-anchor replace into an occupied anchor is not a path any caller takes.
- Acceptance criteria coverage: D3, D7, D9 and D10 each have a criterion; apart from F8, no criterion asserts something no Decision states.
- Auto-mode assumptions repeating Decisions: each line records a self-pick against the board and points at its Decision, which is the section's purpose, so it is not a relocation candidate.
- psmux fallback at Debug rather than Warn: a permanent capability gap that would log on every window open is not a notable recoverable failure, so Debug is defensible under the Level policy.

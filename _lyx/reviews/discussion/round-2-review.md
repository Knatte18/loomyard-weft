---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md#D6
    summary: D6 leaves open what wait the sweep sets after the confirm sweep, so sweepLoop's doubling would grow it from 10 s (20, 40, 50) rather than from the 30 s base the Decision says is unchanged
    fix: In D6's bullet list, add one bullet stating the wait after a confirm sweep, for example "The sweep that ends a pending confirm sets its next wait as if the previous wait were the base, so the backoff resumes from 30 s; a changed strand-name set or a new first miss at that sweep follows the rules above." If a shorter post-confirm backoff is meant instead, state that and name it in the PATTERN-daemon-wakeups sweep line that Scope already lists.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md#Acceptance criteria
    summary: D6's rule that a failed listing while a miss is pending sets the confirm interval as the next wait has no acceptance criterion, though the rationale's per-failure release bound rests on it
    fix: "In Acceptance criteria, after the line ending 'and a failed listing counts as no miss.', add: 'A failed listing after a first miss sets the next sweep at the confirm interval, and the holder is released at the first successful listing after it that misses it again, on the injected clock.'"
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md#D9
    summary: D9's bound says claude shows the bypass-permissions gate only to a strand whose own launch flags ask for bypass, but a settings file whose permissions.defaultMode is bypassPermissions also raises it, so the stated premise is narrower than what the seam lets through
    fix: In D9's Bound, replace "claude shows it only to a strand whose own launch flags ask for bypass permissions, so the seam accepts no permission the strand's command did not already request" with "claude shows it only to a strand whose launch flags or Claude settings ask for bypass permissions, so the seam accepts no permission the strand's own command or configuration did not already request".
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md#D9
    summary: D9 bounds the new exported gate-accept seam by saying its only caller is the llm smoke test's helper, but names nothing that holds that bound, so any production package can call it and play Enter into a pane
    fix: In D9's Bound, after "a production caller is a new decision, not covered by this one", add how the bound is held, either "a seam-enforcement test in internal/shuttleengine pins that no production package outside shuttleengine calls it" (and add that test to the Constraints line for PATTERN-shuttle-provider-seam), or "the bound is held by review alone" if no test is wanted.
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md#D4
    summary: D4's Bound says a non-race exit means no other daemon holds the lock, but a daemon whose start fails before it tries the lock (state directory, lyxtemp root, hub geometry) exits 1 while another client's daemon may be winning the lock
    fix: In D4's Bound, replace "when the spawned daemon cannot answer and no other daemon holds the lock" with "when the spawned daemon cannot answer; a start that fails before it tries the lock ends the wait even if another client's daemon is starting, and that verb returns the daemon-log error".
---

### [LOW] D6 leaves the wait after a confirm sweep open

**Location:** `_lyx/discussion/decision-record.md` D6.

**Issue:** `sweepLoop` in `internal/creelengine/sweep.go` computes the next wait as `min(wait*2, sweepBackoffCeiling)` from the current `wait` after a successful, unchanged sweep.
Once D6's confirm sets that wait to 10 s, the next unchanged sweep doubles from 10 s, giving 20, 40 and then 50 s.
D6 says the 30-second base is unchanged but never says what wait the confirm sweep leaves behind, so the plan writer has to pick between resuming from the base and doubling from the confirm interval.
Both stay above the one-second floor and keep the 60 s bound, but they give different sweep cadences and different text for the sweep line of `pattern/PATTERN-daemon-wakeups.md`, which Scope lists.

### [LOW] D6's failed-listing-while-pending rule has no acceptance criterion

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria.

**Issue:** D6's first bullet says a failed listing while a miss is pending sets the confirm interval as the next wait.
The rationale's bound, that each failed listing after the first miss delays the release by at most the confirm interval, depends on it.
Today `sweepLoop` resets to `sweepBackoffBase` on a failed listing, so this is a change in behavior.
The criteria cover only that a failed listing counts as no miss, and the 60 s line is scoped to listings that succeed, so nothing checks the pending branch.

### [LOW] D9's bypass-gate premise names only launch flags

**Location:** `_lyx/discussion/decision-record.md` D9, Bound.

**Issue:** `gateAcceptNeedles` in `internal/shuttleengine/claudeengine/startup.go` includes `yes,iaccept`, the bypass gate's accepting option, so the seam accepts that gate whenever it shows.
D9 justifies letting it through by saying claude shows the gate only when the strand's launch flags ask for bypass.
Claude also enters bypass mode, and shows the gate, from a settings file whose `permissions.defaultMode` is `bypassPermissions`.
The operator's `~/.claude/settings.json` sets `defaultMode: "auto"` and `skipDangerousModePermissionPrompt: true`, so the smoke test is unaffected.
The seam is general over strands the caller names, though, and its bound should state the real trigger.

### [LOW] D9's single-caller bound has nothing holding it

**Location:** `_lyx/discussion/decision-record.md` D9, Bound.

**Issue:** The new seam is exported from `shuttleengine` and plays the provider's gate-accept keys into a live pane, which accepts folder trust into the operator's Claude config.
D9 bounds it by saying its only caller is the llm smoke test's helper and that a production caller would be a new decision.
The repo holds caller bounds like this with a test (for example PATTERN-agent-filed-issues and PATTERN-shuttle-stop).
D9 names neither such a test nor review as what holds the bound, so the attack-surface bound is stated but not anchored.

### [NIT] D4's Bound overstates what a non-race exit proves

**Location:** `_lyx/discussion/decision-record.md` D4, Bound.

**Issue:** In `internal/creelcli/cli.go`, `daemonCmd` exits 0 on `ErrDaemonRunning` and on a clean stop, and 1 through `refuse` on any other error.
`serveDaemon` can fail before `creelengine.Run` reaches `lock.TryAcquireWriteLock`, through `lyxtemp.ProcessRoot`, `hubgeom.CreelGeometry`, or `Run`'s own `os.MkdirAll` of the state directory.
Such an exit 1 does not prove that no other daemon holds the lock: another client's spawned daemon may be taking it at that moment.
The behavior D4 picks, one last connect and then the daemon-log error, still fits that case; only the Bound's wording claims more than the exit status shows.

## Focus verdicts

- D6 and the `sweepLoop` resets: the Decision names every reset the loop has.
  A changed mail sequence while a confirm is pending keeps the deadline, a changed mail sequence outside a confirm takes the shorter of the base and the time left, and a failed listing sets the base before a first miss and the confirm interval after one.
  The 60 s bound holds under these rules: the current wait is at most the 50 s ceiling and mail never lengthens it, so the first miss comes within 50 s of the removal and the second 10 s after it.
  The 60 s criterion names its condition, that every listing succeeds.
  The gaps are F1 and F2.
- D6 and the changed strand-name set: settled.
  The first bullet says the confirm interval replaces whatever wait the first-miss sweep would set, "the changed-set base reset included".
- D4 and the lock-held, not-yet-answering winner: not a defect.
  The focus names `Serve`, but the function in `internal/creelengine/server.go` is `Run`.
  `Run` takes the lock, then calls `removeUnansweredSocket` and `OpenStore` (which folds the journal) and only then `net.ListenUnix`.
  D4 treats exit 0 as a lost start race, ignores the signal on it and keeps today's backoff for that reason, and an acceptance criterion pins a successful connect in exactly that case.
  The Bound limits the early end to other exits; F5 is a wording nit on that Bound.
- D9's Bound: it names the gates the classification covers (folder-trust and bypass), the one caller, and a per-call trust-list side effect of the strand's working directory.
  F3 corrects the bypass premise and F4 asks what holds the caller bound.
- D1's parse test: not a defect.
  `foldJournal` skips the newest file's last non-blank line when `json.Unmarshal` into `record` fails, and `openJournal` repairs the same file, `names[len(names)-1]`, after `OpenStore` has folded.
  D1 says the cut rule shares the last-line definition and the parse test, and uses `[1]` as an example, which `json.Valid` accepts but which does not unmarshal into a struct, so the plan cannot keep `repairTail`'s current `json.Valid` test.
- Scope's doc list: complete for the surfaces named.
  `internal/creelcli/doc.go` covers the spawn and daemon log today, which D4 changes, and refusal output, which D2 changes.
  D7's adapter mapping and D9's smoke test add no creelcli doc surface.
  `internal/shuttleengine/doc.go` describes `IdleProbe`'s `TooShort` and `Draft` fields and the startup-gate path, which D7 and D9 extend.

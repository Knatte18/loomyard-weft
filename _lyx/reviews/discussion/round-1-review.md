---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:75
    summary: D5's bound that revive types its line only into a session whose state reads an API-error stall at the time of the send is not enforced by the mechanisms D5 picks, for both the guarded send and the orch notice
    fix: "In D5, replace the Bound sentence's state clause with a rule the mechanism enforces: revive's send to a non-orch session refuses, with no idle wait, unless the session's state reads idle-stalled with cause api-error at the send's own ready reading, and a session that reads busy or any other state at that reading is reported skipped and never waited for; for the orch, the watcher drops a queued revive notice at delivery when the orch's state no longer reads idle-stalled with cause api-error, and the report says the notice is conditional. Add two Acceptance criteria: a session that turns busy between revive's state read and its send gets no line once that turn ends; a queued orch revive notice whose orch recovered before delivery is dropped, not typed."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:73
    summary: The revive-versus-pause race leaves to the plan a premise the code already settles, and its absolute criterion cannot hold, since a coverage re-read before the send still leaves the guarded send's ready wait uncovered
    fix: "In D5, replace the sentence beginning 'A pause added while revive runs' with: shuttle's engine send does not consult the pause record, so revive reads coverage for each session immediately before its send and skips a covered one; a pause entry added after that read and before the line lands is the residual window, and the pause's own grace-then-interrupt bounds it, recording the session so resume gives it its line. Replace the criterion at line 148 with: a pause entry live at revive's coverage read for a session keeps the line from that session."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:67
    summary: D5 enumerates sessions the way pause does, which excludes every prime session, yet acts on the orch's session, and leaves the prime's other sessions undecided while D3's status lists every live session in the hub
    fix: "In D5, replace 'It enumerates the covered sessions the way pause does' with: it enumerates the task pairs' sessions the way pause does and, for the hub scope, also reads the orch's session from the prime's orch state; no other prime session is revived, and the report lists none. Add to D3 whether status lists prime sessions other than the orch, and add a revive criterion that a hub-scope revive reads the orch's state and no other prime session."
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:90
    summary: D6's rule that any wall-clock budget above the transport subtracts pause coverage through pauserecord.Excluded names no mechanism a context deadline can follow, contradicts the bound at line 85 that a several-request deadline gets the extension for the held request only, and has no criterion
    fix: "In D6, replace the sentence at line 90 with a concrete rule: a caller whose one context deadline spans several GitHub requests, such as a paginated list, derives a fresh deadline per request instead, so each request's own extension covers its hold; drop the mention of pauserecord.Excluded for GitHub callers. Keep the Bound at line 85 as is. Add an Acceptance criterion: a several-request GitHub caller whose second request is held by a pause completes after resume when the server answers in time."
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:83
    summary: D6 does not say whether the hold itself ends on the caller's cancellation, so the criterion at line 145 reads either as ending at once during the hold or only at release, and today's hold waits on a background context
    fix: "In D6, add one sentence choosing the behavior; recommended: the hold stays blind to the caller's context, so a call cancelled during the hold ends with the cancellation at release, and pauserecord.Network keeps its context-free signature. Reword the criterion at line 145 to: a held GitHub call whose caller's context is cancelled during the hold ends with the cancellation once the hold releases, and one cancelled after release ends at once."
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:103
    summary: D8 re-execs where no cycle step is in flight, but the watcher's hub-pause handling lives in memory, so a re-exec during a hub pause can lose the orch's second interrupt or skip the orch entirely, and the gap also misleads pause and revive, not only refresh and distill
    fix: "In D8, extend the re-exec point to a tick where no cycle step is in flight and no live hub pause entry is unhandled by this watcher, and add to the Bound: a hub pause or a revive in the gap reports no live watcher, and the new image handles a live hub entry from its first tick. Add an Acceptance criterion: the watcher does not re-exec while a live hub pause entry it has not finished handling exists."
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:95
    summary: D7's matching rule can be read two ways, the lyx binary's base name being either the literal lyx or the running binary's own base name, and under the hub being either a string prefix or path containment
    fix: "In D7, replace 'whose base name is the lyx binary's' with 'whose base name, with the (deleted) suffix stripped, equals the base name of the running lyx's own executable', and replace 'whose working directory lies under the hub' with 'whose working directory, with the (deleted) suffix stripped, is the hub directory or a path inside it by path-segment containment'."
  - id: F8
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:56
    summary: D3 opens status to every caller, agents included, bounded only by writing nothing, without stating that it shows an agent other pairs' session states and the full argv of hub lyx processes
    fix: "In D3, append to the sentence at line 56: it shows an agent the hub's session states and the argv of hub lyx processes, nothing a same-user process listing does not already expose, and it reads no transcript or pane content."
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:151
    summary: Two D3 behaviors have no Acceptance criterion, the removal of pause --status and the per-session flag saying whether a live pause entry covers it
    fix: "Extend the status criterion at line 151 with: each session shows whether a live pause entry covers it; and add a criterion: lyx tender pause --status is refused as an unknown flag, and no doc or stencil names lyx pause --status."
  - id: F10
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:26
    summary: The restart-verb Out bullet carries deliberation that belongs in the support log, and its selfreport sentence is neither a Decision nor clearly a requirement on status output
    fix: "In decision-record.md, cut the bullet at lines 25-27 to: A restart verb; a process that can restart safely restarts itself (D8, the reed watchdog, the status panes), and the rest is listed only. Move the sentence 'This narrows the board entry's listing and restart on purpose' into support-log.md's Rejected alternatives restart entry. Drop the selfreport sentence, or, if status must tell the operator so, move it into D7 as: the listing's envelope says that a self-restarting process listed is a defect to report through lyx selfreport create."
  - id: F11
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:137
    summary: The Open risk names no open risk, since net/http does merge Client.Timeout into the request context and PATTERN-github-auth already rules out a GitHub client built outside githubclient
    fix: "Delete the Open risks bullet at lines 137-138 and add to Constraints: PATTERN-github-auth: every GitHub client is built by githubclient, so dropping its http client Timeout in D6 covers every GitHub call. Leave the Open risks section with its heading and a line reading None."
  - id: F12
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:70
    summary: D5's orch notice dedup is a read-then-queue with no lock, so two revives run at once can both queue a revive notice, and the bound states the dedup as absolute
    fix: "In D5, after 'queues none while an undelivered revive notice is already queued', add: two revives run at once may each queue one, which the orch reads as a repeated carry-on line."
---

### [BLOCKING] D5's state bound is not enforced by the guarded send or the notice queue

**Location:** `_lyx/discussion/decision-record.md:75`

**Issue:** D5 bounds revive to typing its line only into a session whose state reads an API-error stall at the time of the send, and the attack-surface rule requires that bound to be one the decision's mechanism enforces.
Shuttle's guarded send, `awaitIdleSession` in `internal/shuttleengine/send.go`, has no state predicate: on a busy session it polls until the session goes idle, up to the send-ready window, and then pastes.
A session that reads an API-error stall at revive's read and then turns busy, through shuttle's own sleep continue (`internal/shuttleengine/hold.go`), a peer's `SendMessage` or the operator, gets the carry-on line after that turn ends, into an `idle-done`, `asking` or differently stalled session, which D5's criterion at line 147 says must not happen.
The orch path has the same gap: `orchengine.QueueNotice` writes a line the watcher types at the orch's next idle, with no state reading at delivery, so an orch that recovered before delivery still gets "carry on".

### [MEDIUM] The revive-versus-pause race is settled already and its criterion cannot hold

**Location:** `_lyx/discussion/decision-record.md:73`

**Issue:** D5 asks the plan to confirm whether shuttle's guarded send holds a send under a live pause entry.
The code already answers no: `Runner.Send` in `internal/shuttleengine/run.go` goes straight to `sendVerified`, and only the `lyx shuttle send` CLI in `internal/shuttlecli/send.go` reads `pauserecord.ReadCoverage`, which `revive` will not pass through.
The fallback, a coverage re-read immediately before each send, still leaves the guarded send's ready wait, up to the send-ready window, between the read and the paste, so the criterion at line 148, that a pause added while revive runs keeps the line from every covered session, is unachievable as written.
The decision should state the residual window and what bounds it.

### [MEDIUM] D5's enumeration excludes the prime but acts on the orch

**Location:** `_lyx/discussion/decision-record.md:67`

**Issue:** D5 enumerates the covered sessions the way `pause` does, and `internal/pausecli/deps.go` builds pause's pairs from every task pair with the prime excluded; pause reaches the orch only through the watcher probe.
D5's second bullet then acts on the orch's session, which that enumeration never reads, and says nothing of the prime's other sessions, while D3 lists every live agent session in the hub.
A plan writer can read revive's hub scope as task pairs only, task pairs plus the orch, or every session in the hub.

### [MEDIUM] D6's outer-budget rule has no mechanism and contradicts its own bound

**Location:** `_lyx/discussion/decision-record.md:90`

**Issue:** Line 85 bounds the extension to the held request only, so a caller issuing several requests under one deadline keeps its original deadline for the rest.
Line 90 then says any wall-clock budget above the transport subtracts pause coverage through `pauserecord.Excluded`, but a `context.WithTimeout` deadline cannot subtract anything once set; `Excluded` measures a span and is used by `pauserecord`'s sleep reading, not by a context.
`internal/fabriccli/openprheads.go` paginates `PullRequests.List` under one `listCtx`, which is the case both sentences claim, so the plan writer gets two rules for one call site and no criterion to settle it.

### [MEDIUM] D6 is silent on cancellation during the hold

**Location:** `_lyx/discussion/decision-record.md:83`

**Issue:** `pauserecord.Network` waits through `WaitUncovered(context.Background(), ...)` in `internal/pauserecord/hold.go`, so today a caller cancelled during a pause stays blocked until release.
D6 says every cancellation other than the deadline still ends the call, and the criterion at line 145 says a call cancelled during the hold ends with the cancellation, without saying whether it ends at once or at release.
Ending at once needs the hold to observe the request context and tell its deadline from its cancellation, which D6's line 84 keeps out of `pauserecord`; ending at release needs nothing new.
The author must pick one.

### [MEDIUM] D8's re-exec point ignores the watcher's in-memory pause handling

**Location:** `_lyx/discussion/decision-record.md:103`

**Issue:** D8's safety argument is that the watcher's cycle state is persisted, but its hub-pause handling is not: `pauseHandled`, `pauseFirstEsc` and `firstTick` are fields of `Watcher` in `internal/orchengine/watcher.go`.
After a re-exec the first-key memory is gone, so `handleHubEntry` in `internal/orchengine/pause.go` skips the orch's second interrupt via `settledWithoutInterrupt`, and a hub entry whose grace ended before the new image's first tick reads settled through `entry.GraceEnd.Before(w.firstTick)`, so the orch is never interrupted.
The gap also makes a hub `pause` report the orch as not interrupted for want of a live watcher (`internal/pauseengine/pause.go`'s `OrchWatcherLive` branch) and makes `revive` report no watcher, while D8's bound names only `refresh` and `distill`.

### [LOW] D7's matching rule has two readings

**Location:** `_lyx/discussion/decision-record.md:95`

**Issue:** "Whose base name is the lyx binary's" does not say whose binary: the literal `lyx`, or the base name of the executable running `status`, which differ under a renamed or test binary.
"Lies under the hub" does not say whether a sibling directory sharing the hub's path as a string prefix counts, and a deleted path carries the `(deleted)` suffix in `/proc/<pid>/exe` and `/proc/<pid>/cwd`, which the rule mentions for neither.

### [LOW] status's open read scope is bounded only by writing nothing

**Location:** `_lyx/discussion/decision-record.md:56`

**Issue:** D3 lets any caller, agents included, run `status` because it writes nothing.
It is a new verb that shows an agent every pair's session states and the full argv of hub lyx processes, and an argv such as `lyx shuttle send <guid> <text>` carries message text.
The decision should state what the read exposes and what bounds it, even if the bound is that a same-user process listing already exposes as much.

### [LOW] Two D3 behaviors have no criterion

**Location:** `_lyx/discussion/decision-record.md:151`

**Issue:** D3 removes `pause --status` with no alias and updates every reference, and it shows for each session whether a live pause entry covers it.
Neither has a matching Acceptance criterion, so the plan has no must-cover scenario for either.

### [LOW] The restart-verb Out bullet holds deliberation

**Location:** `_lyx/discussion/decision-record.md:26`

**Issue:** "This narrows the board entry's 'listing and restart' on purpose" explains a choice against the board entry, which is support-log material.
The requirement the plan writer needs, that no restart verb exists and safe processes restart themselves, survives the cut.
The selfreport sentence is either guidance for the operator, which belongs nowhere in the decision record, or a requirement on `status` output, which then belongs in D7; as an Out bullet it is neither.

### [LOW] The Open risk is not open

**Location:** `_lyx/discussion/decision-record.md:137`

**Issue:** Go 1.26's `setRequestCancel` in `net/http/client.go` merges `Client.Timeout` into the request context through `context.WithDeadline`, so the premise holds and is not a risk.
`PATTERN-github-auth` already requires every GitHub client to come from `internal/githubclient`, and the only production construction site is `NewWithBaseURL` in `internal/githubclient/githubclient.go`.
What remains is a constraint the plan must keep, not a risk.

### [NIT] The orch notice dedup is not atomic

**Location:** `_lyx/discussion/decision-record.md:70`

**Issue:** `orchengine.QueueNotice` takes no lock and has no notice kind, so revive's dedup is a `ListNotices` read followed by a queue.
Two revives run at once can both queue, while D5 and the criterion at line 149 state the dedup without that limit.

## Rejected

- D9's shortcut-guard bound: not a defect.
  `internal/shortcutcli/doc.go` runs every argv of a shortcut in one process, so the exec ends the whole sequence; `a` and `attach` are the single argv `reed attach` and `prime` has it last, and a guard refusing any argv after `reed attach` covers every shortcut that can end in it.
- D6's premise that dropping the http client `Timeout` leaves only the caller's deadline on the request context: sound, per `setRequestCancel` in Go 1.26's `net/http`; the remaining point is filed as F11.
- D6's claim that `landingshed` derives one deadline per call: true for `publish.go`, `finalize.go` and `pullrequest.go`, each a single request under its own `context.WithTimeout`.
- D9's skipped post-run: the root's post-run in `cmd/lyx/main.go` is the exit sweep and `lyxtemp.Remove`, both of which D9 already accounts for.

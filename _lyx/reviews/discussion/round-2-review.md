---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D5 (stale-request bullet) against internal/orchengine/watcher.go tickHandoff
    summary: A request a hub pause held past handoff_timeout_s survives the stale check, but the requested cycle it starts times out on its next tick, so a distill under a pause still never cycles
    fix: In D5's stale-request bullet, add that the handoff phase's timeout clock of a requested cycle (tickHandoff, which starts it at CycleRequestedAt when that predates PhaseEnteredAt) also excludes the time a hub pause entry covered the request, read from the same CoverageOf intervals, or starts at the phase entry for a request a pause held; and change the acceptance criterion to require that a request made under a hub pause longer than handoff_timeout_s runs its cycle to completion after the resume, not only that it is still pending.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1 condition 2, D1 Consequences (shuttle stop bullet), Acceptance criteria (shared-window strand moved)
    summary: Condition 2's widening of presentPanes, the reconcile binding change and the render rule target a shared-window strand outside an own window that reed cannot hold, and the term collides with reed's own shared window
    fix: Rewrite D1 condition 2 to state today's fact that every pane-owning strand is own-window or overview (validateDisplay admits only own-window, overview and hidden, retired anchors decode as own-window, a hidden strand owns no pane), so presentPanes already reads every bound pane session-wide and condition 2 needs no listing, reconcile or render change; delete the presentPanes-widening, reconcile and render sentences, drop the in the shared window clause from the shuttle-stop consequence, and replace the shared-window acceptance criterion with one on an own-window strand whose pane the operator moved to another window of the reed session.
    origin: handler
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 and its acceptance criterion
    summary: D2's server-wide lookup needs the recorded pane id, but reconcile clears the binding of an own-window strand absent from the session in every public reed op, and orch start reconciles before it reads strands, so strand_elsewhere shows only until the next reed op
    fix: Decide in D2 whether reconcile keeps the binding of a strand whose pane the server-wide lookup finds in another session (removal's paneIDsInSession guard already spares that pane) so the placement stays visible to status and start, or state that the placement is reported only by a Status read before the next reconciling reed op and narrow the acceptance criterion to that; either way record in D2 that the #746 observation (%3 in session 0, window bash) came from a tmux list-panes whose server is not established, since pane ids are per server and reed runs on its own -L socket.
    origin: handler
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 (serialization bullet) against internal/orchengine/state.go RequestCycle and internal/orchcli/refresh.go requestCycle
    summary: The compare-and-remove under the state lock does not serialize with the marker's writer, a plain truncating os.WriteFile outside that lock, nor with requestCycle's own no-watcher removal, so a request written after a removal's read can still be removed
    fix: Add to D5's serialization bullet that RequestCycle writes the marker under the orch state lock by an atomic temp-file rename, and that requestCycle's removal when no watcher is live is a compare-and-remove of the request it just wrote under the same lock, so every writer and remover of cycle-request holds the state lock.
    origin: handler
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D5 (withdraw bullet)
    summary: The board entry asks for a verb refusing once the cycle has started, and D5's withdraw never refuses and answers withdrawn false, a departure the record does not list or justify
    fix: Either make lyx orch withdraw refuse, with a way forward naming the running cycle's phase and trigger, when no request is pending because its cycle started, as board section 4 asks; or keep the never-refuse answer and add it to D5 as a departure from the board entry with its reason, and list it under Auto-mode assumptions.
    origin: handler
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D1 Bound of condition 3 (launch grace)
    summary: The grace reads pane age with a ps spawn per shell-foreground strand on every Status, forever for a lingering shell-only strand, though condition 3 runs only on Linux where reed reads /proc itself
    fix: In D1's grace bullet, replace taken with POSIX ps -o etime= by the pane pid's start time read from /proc/<pid>/stat (field 22) against the boot time, as reed's Linux process probes in proctree_linux.go read /proc, so a Status read spawns no process.
    origin: handler
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D7 (Strands skip bullet)
    summary: The skip treats any not-a-git-repository resolve as a teardown, so a worktree with a broken .git link for another cause is skipped and its strands silently lose delivery, contrary to any other failure still fails the call
    fix: In D7, state the bound of the not-a-git-repository skip, that any worktree whose resolve answers it is skipped whatever the cause and its strands get no delivery until it resolves, and log the skip at Warn rather than Info so a non-teardown cause shows, or else skip it only when the worktree's directory or its gitdir is gone.
    origin: lens:bound-breaker
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D5 (distill and refresh pause-notice bullet)
    summary: The notice says the request waits for the hub resume whenever a hub pause holds the cycle machine, but with no live watcher requestCycle removes the marker it wrote, so the notice can describe a request that no longer exists
    fix: In D5's distill and refresh bullet, add that held_by_pause and the note are added only when a live watcher kept the request, and that with no live watcher the verbs answer as today.
    origin: lens:existing-paths
  - id: F9
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D6 Rationale and the #799 and #811 acceptance criterion
    summary: D6 says #811 failed only because a dead registration held the name, but the record gives no evidence that the finished stuck attempt's strand was dead rather than live, which D6 would leave holding the name
    fix: Research the #811 holder's state at the failed attempt (whether burler's stuck fix attempt stops its strand, internal/burlerengine/shuttle.go) and record the answer with its evidence in D6's rationale; if a holder can still be live, add the attempt teardown that removes its strand to D6.
    origin: lens:walkthrough
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 Bound (On tmux the answer is never empty) and Constraints (Fact tmux manual) and Open risks (first bullet)
    summary: The tmux fallback chain is stated twice, in D1's bound and in Constraints, and the first open risk carries review-round narrative about who checked it
    fix: Keep the tmux fact once, in Constraints, and reduce D1's bullet to its consequence (a failed foreground read answers the shell, so the residual is a read racing the foreground group's exit); in the first Open risks bullet keep the fact and its evidence (Claude Code 2.1.308, tmux 3.6, ps -t lists claude as the foreground group) and move checked by the round-1 reviewer and again by its fixer to support-log.md.
    origin: handler
  - id: F11
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D3 Bound
    summary: Two concurrent status calls cannot both pass the start.lock try-lock; the double spawn comes from two calls in sequence before the first watcher takes watch.lock
    fix: Change D3's bound to read that two status calls in sequence, the second before the first one's watcher takes watch.lock, may spawn two watchers, and watch.lock keeps one.
    origin: handler
---

### [BLOCKING] A pause-held request's cycle times out at once

**Location:** `_lyx/discussion/decision-record.md` D5, the stale-request bullet; `internal/orchengine/watcher.go` `tickHandoff` (the clock starting at `CycleRequestedAt`), `tickIdle` (`st.CycleMode, st.CycleRequestedAt = mode, requestedAt`), `tick` (returns on `pauseTick` held before any phase tick).

**Issue:** D5 excludes paused time only from `tickIdle`'s stale-request age.
At the resume a request older than `handoff_timeout_s` is therefore kept, and `tickIdle` enters handoff-requested with `CycleRequestedAt` set to the pre-pause request time.
`tickHandoff` measures the phase timeout from `CycleRequestedAt` whenever it predates `PhaseEnteredAt`, with no pause exclusion, so on the next tick `now.Sub(started) >= HandoffTimeout` holds and it returns to idle with `handoff timed out` before the note gate can pass.
Board section 4's own case, a `distill` under a hub pause, still never cycles, and the acceptance criterion as written passes anyway because it only checks that the request is pending.
All four lenses reached this independently.
The pause exclusion came from round 1's F6 fix, which stopped at the stale removal and did not follow the request into its cycle.

### [MEDIUM] Condition 2 designs for a strand reed cannot hold

**Location:** `_lyx/discussion/decision-record.md` D1 condition 2, the shuttle-stop consequence, the shared-window acceptance criterion; `internal/reedengine/strand.go` `validateDisplay`; `internal/reedengine/render/types.go` anchors and `UnmarshalJSON`; `internal/reedengine/ownwindow.go` `presentPanes`, `isOwnWindow`; `internal/reedengine/selvagepane.go` `planShellWindowMoves`; `internal/reedengine/doc.go` shared-window paragraphs.

**Issue:** D1 claims that today `presentPanes` adds the session-wide listing only for own-window strands, and builds a widening, a reconcile binding change, a render rule, a consequence and an acceptance criterion around "shared-window" strands outside own windows.
No such strand exists: `validateDisplay` admits only `own-window`, `overview` and `hidden`, the retired `below-parent` and `top` decode as `own-window`, a hidden strand owns no pane, and a bound pane left in the shell window is broken out into a window of its own.
Every pane-owning strand is therefore `isOwnWindow`, already read session-wide by `presentPanes` and already kept by reconcile.
In reed's own vocabulary "shared window" means a group's shared own window, so a plan writer reading the record against the code gets a wrong target and an acceptance criterion with no constructible input.
Round 1's F3 introduced this text on the premise that `listStrandPanes` alone lists shared-window strands; that premise holds only for a strand with a non-own anchor, which `validateDisplay` refuses, so this reverses round 1's fix rather than contradicting a settled finding.

### [MEDIUM] Reconcile erases the pane id D2 looks up

**Location:** `_lyx/discussion/decision-record.md` D2 and its acceptance criterion; `internal/reedengine/reconcile.go` `planReconcile` (own-window strand not in `listedByID` goes to `clearedGUIDs`) and `reconcileLocked` (`PaneID = ""`); `internal/reedengine/spawn.go` `launchStrandLocked`, `reconcileApplyPersistLocked`; `internal/reedengine/lifecycle.go` Up and Resume reconcile; `internal/orchcli/start.go` `reedUp` and `ensureRunsStrand` before `Strands`.

**Issue:** Every public reed op except `Status` reconciles first, and reconcile clears the binding of an own-window strand whose pane the session listing does not hold.
`lyx orch start` runs `reedUp` and `ensureRunsStrand`, an add or replace, before it reads strands, so start never sees the pane id, and `lyx orch status` names the placement and `strand_elsewhere` only until any other reed op runs in the prime.
D2's purpose, that the operator learns where a moved pane went, is met only in that window, and the record treats the binding as durable.
The #746 observation itself (`%3 0:bash bash`) reads like the default tmux server: pane ids are per server, reed uses its own `-L` socket, and `0` is tmux's default session name; the record does not establish that a pane in another session of reed's server was observed.

### [MEDIUM] The marker's writer is outside the lock D5 serializes on

**Location:** `_lyx/discussion/decision-record.md` D5, serialization bullet; `internal/orchengine/state.go` `RequestCycle` (plain `os.WriteFile`), `CycleRequested` (an unparsable marker reads pending with a zero request); `internal/orchcli/refresh.go` `requestCycle` (removes the marker with no live watcher).

**Issue:** D5 promises that a request written after a removal's read is never removed by it, by doing every removal as a compare-and-remove under the orch state lock.
`RequestCycle` takes no lock and truncates before it writes, so a refresh can land between the locked re-read and the `os.Remove` and be deleted, and a locked reader can see an empty marker, which `CycleRequested` reads as pending with unbounded age.
`requestCycle`'s own removal when no watcher is live is another unlocked remover.
The acceptance criteria on the race therefore assert a guarantee the decision does not build.

### [MEDIUM] Withdraw departs from the board's refusal without saying so

**Location:** `_lyx/discussion/decision-record.md` D5, withdraw bullet; board entry `orch-strand-cycle` section 4 ("refusing once the cycle has started").

**Issue:** The board entry asks for a verb that withdraws a pending request and refuses once the cycle has started.
D5 says withdraw "never refuses" and answers `withdrawn: false` with the phase.
D1 lists its departures from the operator decision with reasons, but D5 records this one nowhere, neither in D5 nor under Auto-mode assumptions, so the plan writer cannot tell a chosen departure from an oversight.

### [LOW] The grace spawns ps on every read

**Location:** `_lyx/discussion/decision-record.md` D1, the bound of condition 3; `internal/reedengine/proctree_linux.go` (reads `/proc` directly); build tags (reed builds for Linux and Windows only); `internal/orchcli/session.go` `StrandAlive`; `internal/creelcli/delivery.go` `Strands`.

**Issue:** No launch time is stored, so every `Status` read of a shell-foreground strand spawns `ps -o etime=`, and a shell-only strand that lingers past the grace costs a spawn on every read for as long as it stays.
Those reads come from the watcher each tick and from creel's per-worktree listing.
Condition 3 runs only on Linux, where reed already reads `/proc` rather than shelling out, so "POSIX `ps`" adds a spawn for no portability gain.

### [LOW] The not-a-git-repository skip is not bounded to a teardown

**Location:** `_lyx/discussion/decision-record.md` D7, Strands bullet; `internal/creelcli/delivery.go` `Strands`, `worktreeStrands`.

**Issue:** The record says the skip covers a worktree "being torn down", but a not-a-git-repository resolve cannot tell a teardown from a broken `.git` link.
Such a worktree is skipped with an Info line and its strands silently stop getting delivery, while the record says any other failure still fails the call.

### [LOW] The pause notice can describe a removed request

**Location:** `_lyx/discussion/decision-record.md` D5, distill and refresh bullet; `internal/orchcli/refresh.go` `requestCycle`.

**Issue:** With no live watcher, `requestCycle` removes the marker it just wrote, yet D5 adds `held_by_pause: true` and a note that the request waits for the hub resume whenever a live hub entry holds the cycle machine.
The record does not say whether the notice needs a live watcher.

### [LOW] The #811 holder's state is assumed, not shown

**Location:** `_lyx/discussion/decision-record.md` D6 rationale and the #799/#811 acceptance criterion; board entry #811 section; `internal/seatengine/engine.go` `checkName`.

**Issue:** D6 says #811 failed only because a dead registration held the name, but #811's holder came from a finished attempt whose history is `stuck`, and the record does not establish that its strand was dead.
A live holder keeps the name under D6, so `checkName` would refuse as before and #811 would stay open.
This is a fact the design rests on, left unverified.

### [LOW] Duplicated tmux fact and review narrative in the record

**Location:** `_lyx/discussion/decision-record.md` D1 bound ("On tmux the answer is never empty"), Constraints ("Fact (tmux manual …)"), Open risks first bullet.

**Issue:** The tmux fallback chain and its `format.c` and `osdep-linux.c` sources appear in both D1 and Constraints, against say-it-once.
The first open risk names who verified it in which review round, which is deliberation history for `support-log.md`; the version and `ps -t` evidence stays, since the plan writer reads only the decision record.

### [NIT] D3's double-spawn bound names the wrong race

**Location:** `_lyx/discussion/decision-record.md` D3 bound; `internal/lock/lock.go` `TryAcquireWriteLock`.

**Issue:** Under the `start.lock` try-lock, two concurrent `status` calls cannot both restart a watcher.
The double spawn happens when calls run in sequence: the first releases `start.lock` before its detached watcher takes `watch.lock`.

## Rejected

- interactions lens, the watcher's `strand gone` exit is not named among D1's readers: this is call-site enumeration, which the rubric does not flag, and the existing-paths lens traced that exit as the intended effect, since `start` relaunches strand and watcher together.

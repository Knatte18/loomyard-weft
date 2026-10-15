---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/13-creelcli-delivery-adapter.md
    summary: TestHubDelivery is planned untagged but builds a hubforge hub and calls fabricengine.List, which runs git worktree list, so it spawns in the untagged tier PATTERN-test-speed forbids
    fix: In card 13, state that internal/creelcli/delivery_test.go carries //go:build integration, so TestHubDelivery runs in tier 2, and that testmain_test.go stays untagged with only gitkit.HermeticGitEnv and tmuxkit.Main so every tier of the package has its TestMain. No target changes.
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/14-creelcli-verbs.md
    summary: TestCreelCLI is planned untagged but resolves the mode with preflight.ResolveMode, which runs git through lyxcwd.Resolve, and needs a hubforge hub for its hub-mode and subdirectory rows, so it spawns in the untagged tier PATTERN-test-speed forbids
    fix: In card 14, state that internal/creelcli/cli_test.go carries //go:build integration, so TestCreelCLI runs in tier 2 over a hubforge hub, and that its plain-repository rows use a hubforge or gitkit fixture inside that tagged file. No target changes.
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/plan/09-creelengine-delivery.md
    summary: The removed-holder sweep wakes every 30 seconds with no backoff while each wake spawns git (fabricengine.List) and tmux (reed Status) through card 13's Strands, and orch mail withheld under D16 keeps it running indefinitely, against PATTERN-daemon-wakeups' backoff rule
    fix: In card 9, make the sweep back off from a 30-second base, doubling to a named ceiling constant (for example 4 minutes, added to the overview's constants-not-config list) while two consecutive successful sweeps see the same strand-name set, and return to the base on a change of that set, a new post, a new inject or a failed sweep; write the base and ceiling into the What-counts bullet card 9 adds to pattern/PATTERN-daemon-wakeups.md; add a TestSweep_RemovedHolders row reading the sweep waits off the fake clock as 30 s, 60 s, … up to the ceiling and back to 30 s after a post. This departs from D15's fixed 30 seconds because the PATTERN constraint wins.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-creelengine-pool-verbs.md
    summary: Card 7 depends on later cards, since Post wakes card 9's delivery worker and Reply hands a reply to a waiter whose registration is card 10's, and card 9's TestSweep_RemovedHolders answers --wait-reply and --wait waiters that only card 10 registers
    fix: In card 7, make Store own the waiter registry and the wake hook, so a later card only plugs in. Store registers a waiter on a message id, an inject id or a holder, and answers it from the verbs that settle it (reply hand-over, returned-undelivered, inject outcome, mail arriving for a holder). It also exposes a wake notification per receiver on post, inject and release, which card 9's Scheduler subscribes to. Add a TestStore_Mail row where a reply to an EXTERNAL message is handed to an in-process registered waiter. In card 9, drop the reference to card 10 and answer the sweep's waiters through card 7's registry. In card 10, say the protocol binds a waiting connection to card 7's registry.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-creelengine-delivery.md
    summary: Card 9 journals an orch's reminders as withheld although its Scheduler runs no worker for mail muted by the orch rule, so nothing reaches the point where a withheld reminder would be journaled, and card 6's closed event list has no withheld event for cards 7 and 9 to write or for the fold to read
    fix: In card 6, add a withheld event to the event list and the fold. In card 9, state that a withheld event naming creel-notices is journaled once for each change of an orch receiver's unread set (a post, forward or return to it), from the verb rather than from a worker, and that the orch gets no worker and no Idle probe. Extend the orch row of TestScheduler_Reminders to assert one withheld line per new message and zero Idle calls for the orch strand. Card 7's Log and Inbox rows then read that event.
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-creelengine-daemon.md
    summary: The acceptance criterion that a reply journaled while its waiter was disconnected reaches it on reconnect has no row, since TestDaemon_Lifecycle's reconnect rows cover only the injected and returned-undelivered outcomes
    fix: In card 10, add a TestDaemon_Lifecycle row where an EXTERNAL post --wait-reply loses its connection, a strand replies while no waiter is connected (journaled undelivered), and the reconnected waiter receives the reply at once, with the journal holding a delivered event for it.
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-creelengine-pool-verbs.md
    summary: Card 7 adds the inject verb and its privilege rule but its Intent states no bound for what a privileged inject lets through, while D9's bound says it types arbitrary single-line text, slash commands included, into any pane at its next idle
    fix: Add a Bound line to card 7's Intent stating that a privileged sender (EXTERNAL, a program sender or an orch strand) may type any single-line text without a control character, slash commands included, into any non-orch strand's pane at its next idle. State that the TTL (10 minutes by default, 1 hour at most) bounds how stale it lands, and that card 9's pause, hold and orch rules hold it from paused, held and orch panes.
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/10-creelengine-daemon.md
    summary: TestDaemon_Protocol and TestCreelCLI resolve a strand caller only through Linux peer ancestry (a creelfake strand whose PanePID is os.Getpid()), so on Windows, where peer_other.go reports no peer and identity is LYX_STRAND_NAME, those rows resolve as EXTERNAL and fail
    fix: In card 10, and in card 14 for its as-a-strand rows, have every row that calls as a strand also set the Claim's LYX_STRAND_NAME to that strand's name, which agrees with the ancestry on Linux and is the identity off Linux. Skip the row asserting that a LYX_STRAND_NAME naming another strand is refused when runtime.GOOS is not linux, with a reason naming D4's Linux-only ancestry check.
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/11-creel-store-pattern.md
    summary: Card 11's background attributes creelengine naming no tmux command to TestCreelStore_DeliverySeamImports, which scans imports of reedengine and shuttleengine only and cannot see a tmux command
    fix: In card 11, word the background so TestCreelStore_DeliverySeamImports checks the import half, and the no-tmux-command half, like the journal-writer half, is checked by review.
---

### [BLOCKING] Card 13's TestHubDelivery spawns in the untagged tier

**Location:** `_lyx/plan/13-creelcli-delivery-adapter.md`

**Issue:** `pattern/PATTERN-test-speed.md`'s "Untagged tests spawn nothing" section bars `hubforge.NewHub`, `CopyHub`, `SharedHub` and git spawns outside tier-tagged files, and `cmd/lyx/tierpurity_test.go` enforces it.
`TestHubDelivery` runs "over a hub fixture from `hubforge` with a pair", and `hubDelivery.Strands` calls `fabricengine.List`, which runs `gitexec.Run([]string{"worktree", "list", "--porcelain"}, …)` (`internal/fabricengine/worktreelist.go:29`).
Card 15 names its tags explicitly and card 13 names none, so `delivery_test.go` is planned untagged.

### [BLOCKING] Card 14's TestCreelCLI spawns in the untagged tier

**Location:** `_lyx/plan/14-creelcli-verbs.md`

**Issue:** Every row goes through `preflight.ResolveMode`, which calls `lyxcwd.Resolve` (git) at `internal/preflight/predicates.go:111`.
The hub-mode rows and the "subdirectory of a hub worktree" row need a real hub, which `PATTERN-hubforge-fixtures` says comes only from `hubforge`.
Both are spawns the untagged tier forbids, and the card names no build tag for `cli_test.go`.

### [BLOCKING] The removed-holder sweep spawns every 30 seconds with no backoff

**Location:** `_lyx/plan/09-creelengine-delivery.md`

**Issue:** `pattern/PATTERN-daemon-wakeups.md`'s rule: "A wake that spawns a process or captures a pane doubles its wait from a base to a named ceiling while nothing changes."
Card 9's sweep calls `Delivery.Strands` every 30 seconds.
Card 13 implements that with `fabricengine.List` (a git spawn) and each worktree's reed `Status` (a tmux query; card 4 says it reads the `list-panes` listing).
Card 9 adds the sweep to the What-counts list as "every 30 seconds while mail or injects exist", with no backoff.
Under D16 an orch's mail is never reminded and can stay unread indefinitely, so the sweep can run for the daemon's whole life.
D15 sets the fixed 30 seconds, so this is a conflict between the answer key and a listed constraint, and per the explore step the constraint wins.

### [MEDIUM] Card 7 depends on cards 9 and 10

**Location:** `_lyx/plan/07-creelengine-pool-verbs.md`

**Issue:** The spec's "Plan vs. schedule" section: "a card never depends on anything a later card does".
Card 7's `Post` "wakes the receiver's delivery worker (card 9)", and its `Reply` hands a reply to "a waiter registered on that id (the registration is card 10's protocol)".
Card 9's `TestSweep_RemovedHolders` asserts "its waiter answered" and "its `--wait` answered", but only card 10 creates the waiter mechanics.
As written, card 7's reply hand-over and card 9's waiter rows cannot be built or tested at their own card gates.

### [MEDIUM] Withheld orch reminders have no event and no writer

**Location:** `_lyx/plan/09-creelengine-delivery.md`

**Issue:** D16 says `inbox`, `log` and the journal show an orch's reminders as withheld.
Card 9 says the Scheduler runs workers only for "unread mail not muted by a hold or the orch rule", but also that the orch's "reminders are journaled withheld naming `creel-notices`".
With no worker for the orch, the card names no point where that line is written.
Card 6 enumerates the journal's events, and the list has no withheld event, so the line card 9 writes and card 7's `Log` reads is undefined.
Card 6's reader would also ignore it as an unknown event name.

### [MEDIUM] No row for a reply reaching a reconnected waiter

**Location:** `_lyx/plan/10-creelengine-daemon.md`

**Issue:** The acceptance criteria say "a reply or an inject outcome journaled while the waiter was disconnected reaches it on reconnect", and D12 says such a reply is "handed over and journaled as delivered".
Card 10's Intent prescribes this, but `TestDaemon_Lifecycle` reconnects a waiter only for "the injected outcome, and in another row the returned-undelivered outcome".

### [MEDIUM] Inject's privilege bound missing from card 7

**Location:** `_lyx/plan/07-creelengine-pool-verbs.md`

**Issue:** The rubric's attack-surface rule requires the card that adds a verb to state what it lets through and what bounds it.
Card 7 adds `Inject` and the privilege rule, but its only Bound line covers holds.
Card 9 states a bound for reminders only, and card 14's covers sender identity, not what an inject may type.

### [LOW] Strand-caller rows are Linux-only

**Location:** `_lyx/plan/10-creelengine-daemon.md`

**Issue:** Card 10 has a strand caller resolved by scripting "a `creelfake` strand whose `PanePID` is `os.Getpid()`".
Card 8 makes `peer_other.go` report no peer and resolves off Linux from `LYX_STRAND_NAME` alone.
The package also builds and tests on Windows: `internal/proc` keeps `proc_windows_test.go`.
Neither card 10 nor card 14 gates these rows or sets the claim's strand name.

### [LOW] Background overstates what the import scan checks

**Location:** `_lyx/plan/11-creel-store-pattern.md`

**Issue:** The planned background says `internal/creelengine` "imports neither `internal/reedengine` nor any package under `internal/shuttleengine/` and names no tmux command, … checked by `TestCreelStore_DeliverySeamImports`".
The test as specified fails only on those imports.

## Rejected

- Card order, cards 6 to 10 against card 12: card 12 uses card 6's `Geometry` and `SocketName`, and no card from 6 to 10 uses `CreelGeometry` or `lyxdirs.CreelDirName`. Card 6's deferral of the `SocketName` pin to card 12's test is no dependency. Card 13 uses card 5's record-less send and card 4's `PanePID`, both earlier.
- Edit cards 1 to 5: no member's signature changes. `StrandStatus` gains a field, `sendContext` gains a field, and `Check` keeps its signature. So no re-sign arrow is missing, and each Intent's named symbols are listed as member glyphs.
- Cards 2 to 4 are right-sized: each is a separate package's public-surface addition with its own test extension.
- Verify: every targeted package is in the root module and `go test ./...` and `go test -tags integration ./...` cover it. The `tmux` and `llm` tiers are compiled by `go vet`, and the section follows the template's order.
- Test economy: `TestCreelGeometry` follows the one-test-per-constructor precedent of `TestReedGeometry`, `TestBurlerGeometry` and `TestWebsterGeometry`. Card 14's exit-code rows pin the CLI mapping, not card 10's outcomes. The budget raises sum to the planned top-level tests.
- Card 3 does not duplicate an existing exported reader: the only parent-pid parsing lives unexported in `internal/reedengine/proctree.go`, which `creelengine`'s callers may not reach.

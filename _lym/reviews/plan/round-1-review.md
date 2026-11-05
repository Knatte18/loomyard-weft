---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/09-slackengine-relay.md:53
    summary: Card 9's whileAway restart cap can never reach gave-up, since step 3 rewrites RelayState afresh and wipes StartFailures and NextStartAt, and a relay that dies before step 3 leaves the old FirstReadAt, so the tick's respawn loop has no count cap as PATTERN-child-processes requires
    fix: "In card 4's Intent, list RelayState's fields as {PID, StartTime, FirstReadAt, FirstReadRefusal, LastSlackOK, StartFailures, NextStartAt, LastSpawnAt}, so card 9 changes no struct card 4 creates. In card 9: step 3 rewrites only PID, StartTime, FirstReadAt and FirstReadRefusal and keeps the starter's three fields; starter.ensure records LastSpawnAt when it spawns; a whileAway start counts one failure when the relay is down and no accepted first read was written after LastSpawnAt, whether the relay died before or after step 3. Add a TestEnsureRelay row where the fake spawn writes a RelayState as a relay reaching step 3 would and then dies before its first read, five times, ending in gave-up, and a row where a relay dies before step 3 with an earlier relay's FirstReadAt standing, which still counts."
    origin: lens:guard-reach
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/plan/09-slackengine-relay.md:70
    summary: AwaitFirstRead refuses at once on a gone relay's recorded FirstReadRefusal, which only a stale state from an earlier relay can carry, so lym slack away refuses before the relay it just started writes its own state
    fix: "In card 9, AwaitFirstRead takes the time it began and judges only a RelayState whose StartTime is at or after it, reading an older state as not yet written; the refuse-at-once branch applies only to such a state. Add a TestRelayWaits row: a stale state of a gone relay that recorded a refusal before the await began is ignored, and the await returns once a fresh state's FirstReadAt is written."
    origin: lens:guard-reach
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/plan/10-slack-cli.md:49
    summary: The slack group's PersistentPreRunE resolves the location for every verb, but relay runs from the hub directory with --hub-path, which is no worktree, and the card never says how relay gets the prime location that NewProgramClient, DobbyStart and the pause geometry need
    fix: "In card 10, state that the pre-run skips location resolution for relay, as creel's daemon verb resolves none, and that relay builds its prime location from --hub-path through fabricengine.PrimeName and lymcwd.NamedWorktree. NewProgramClient, hubgeom.DobbyStart(prime).Beams, Hub.Paused's pause geometry and hubgeom.SlackGeometry are all built over that location. Add a TestSlackCLI row that runs relay --after with the cwd at a temp hub directory that is no worktree and reaches the relay run."
    origin: handler
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/plan/10-slack-cli.md:45
    summary: The hidden relay verb's stated bound, that a run by hand is stopped by the refused first read, is false for relay --fork, which double-forks a relay out of any pane and so starts one past lym slack start's pane refusal, and false off Linux, where creel places a program claim by environment and never refuses it by pane
    fix: "In card 10's Intent for relay, state what --fork lets through: a process in any pane, a non-orch strand's included, starts the hub's relay as EnsureRelay does. State its bound: it starts only a configured hub's relay, writes no presence, token or setup file, and starts what the next prime dobby tick or bring-up would start. State that off Linux a by-hand run is accepted as SLACK, bounded by creel's environment-only identity there, which D7's trust bound already rests on. Make card 9 step 4's bound say Linux."
    origin: handler
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lym/plan/08-slackengine-outbound.md:35
    summary: outbound.read treats every ErrWaitTimeout from ReadWait as an empty read, but creel's one call timer spans the wait, the handler and the ack confirmation, so a read whose handler posted can also end in ErrWaitTimeout and be logged as idle with a doubled wait
    fix: "In card 8, read records whether its handler ran; an ErrWaitTimeout after the handler ran is a lost confirmation: the record stays, it is not an empty read, and read returns the floor wait when a post succeeded. In card 3, ReadWait's doc says its timeout bounds the wait for mail, the handler and the confirmation together, as creelengine.Client.Call does. Add a TestOutboundRead row where the fake ReadWait runs the handler, which posts, and then returns ErrWaitTimeout, and read keeps the record and returns the floor."
    origin: lens:guard-reach
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lym/plan/09-slackengine-relay.md:49
    summary: RelayDeps.CheckBinary is declared func(beforeExec, failed func()) error, but the production value selfexec.New().CheckWith is func(beforeExec func(), failed func(error)) with no result, so the wiring card 10 names does not type-check
    fix: "In card 9, declare CheckBinary func(beforeExec func(), failed func(error)), dobby's shape, with step 6's failed taking the exec error and logging it before it retakes the lock. Card 10 keeps selfexec.New().CheckWith as its value."
    origin: lens:guard-reach
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lym/plan/03-creel-seams.md:19
    summary: "Card 3 puts internal/creelengine/doc.go under Extends while its change falsifies that doc's Refusal code section, which says the strand-claim refusal is the only coded one and a program claim from a non-orch pane carries no code; resolveSender's own comment says no other refusal wraps a sentinel"
    fix: "In card 3, move `internal/creelengine/doc.go` from **Extends:** to **Edit:**, and say in the Intent that its Refusal code section is rewritten to name both coded refusals, program-claim-in-strand-pane beside strand-claim-outside-pane. Say that resolveSender's doc comment names both sentinels."
    origin: lens:repo-invariants
  - id: F8
    severity: MEDIUM
    class: design
    location: _lym/plan/12-dobby-relay-tick.md:10
    summary: Cards 11, 12 and 13 list their package doc.go under Uses while changing what it describes, so beamcli's Deps.Tick text, fabriccli's anchor-move restart and envelope text, the anchor verb's Long, orchcli's bring-up steps, orchengine's one-line-stencil pointer text and docs/overview.md's beamcli line go stale
    fix: "Card 11: move `internal/orchcli/doc.go` and `internal/orchengine/doc.go` from **Uses:** to **Edit:**; the Intent adds the relay step and its warning to the bring-up steps and says the pointer may carry the presence line. Card 12: move `internal/beamcli/doc.go` to **Edit:** and add `docs/overview.md` under **Edit:**; the Intent says the prime's Deps.Tick runs the carrier's tick, then EnsureRelayWhileAway, and the beamcli overview line names the relay. Card 13: move `internal/fabriccli/doc.go` to **Edit:** and add `internal/fabriccli#anchorCommand` under **Edit:**; the Intent names the relay restart after EndDobbies and relay_start_error in both texts."
    origin: lens:affected-set
  - id: F9
    severity: LOW
    class: design
    location: _lym/plan/09-slackengine-relay.md:59
    summary: Step 6's failed path retakes the lock with TryAcquireWriteLock, so a RelayLive probe holding the lock for a moment makes the old image exit with no relay left, while step 2 already allows ProbeGrace for the same probes
    fix: "In card 9 step 6, retake the lock with lock.AcquireWriteLockWithin(geom.LockPath, lock.ProbeGrace), and exit only when that fails. Extend the existing failed-exec row of TestRunRelay with a lock held only for a probe's grace, which the old image takes back."
    origin: lens:guard-reach
  - id: F10
    severity: LOW
    class: consistency
    location: _lym/plan/07-slackengine-inbound.md:16
    summary: TestInboundText and TestOutboundText each test one unexported helper, inboundText and outboundText, whose behavior is observable through the public-surface tests TestInboundPoll and TestOutboundRead that the same cards add
    fix: "Drop `plan:internal/slackengine#TestInboundText` from card 7's **Create:** and move its rows into TestInboundPoll: a forwarded message whose text holds `<https://x.io/a|label>`, `<https://x.io/a>`, `<#C1|general>`, `<!here>`, `<@U1>` and `&amp; &lt;c&gt;` reaches the mailbox as the URL, the unchanged forms and `& <c>`. Drop `plan:internal/slackengine#TestOutboundText` from card 8's **Create:** and move its rows into TestOutboundRead as the posted text of a prime orch's, a beam orch's and a beam program's mail."
    origin: handler
  - id: F11
    severity: LOW
    class: consistency
    location: _lym/plan/03-creel-seams.md:46
    summary: "The planned ReadWait subtest where a handle error leaves the message unread and is returned covers the same client path as TestCreelCLI's existing subtest: a program client's ReadAll with a failing handler leaves the reply unread for the next ReadAll"
    fix: "In card 3, drop the second planned TestCreelCLI subtest. Keep the ReadWait subtest that blocks until a post arrives, and assert there that the delivered message is acknowledged, which the existing failing-handler subtest's counterpart does not cover for a waiting read."
    origin: handler
  - id: F12
    severity: LOW
    class: consistency
    location: _lym/plan/08-slackengine-outbound.md:40
    summary: Card 8 extends the fake Mailbox declared in card 7's inbound_test.go with scripted reads and acknowledgement records, a change to a file card 8 does not target
    fix: "In card 7, declare the in-memory Mailbox of inbound_test.go with scripted ReadWait answers and a record of acknowledgements from the start. Card 8 then uses it unchanged, and its Intent says uses instead of extends."
    origin: handler
  - id: F13
    severity: LOW
    class: consistency
    location: _lym/plan/10-slack-cli.md:74
    summary: login's refusal of an empty token from standard input has no TestSlackCLI row that reaches it
    fix: "In card 10, extend the login row of TestSlackCLI: an empty standard input refuses, calls no auth.test on the httptest server and writes no token file."
    origin: lens:guard-reach
  - id: F14
    severity: LOW
    class: consistency
    location: _lym/plan/04-slackengine-files.md:57
    summary: SlackGeometry's refusal of an unresolvable credential directory, naming HOME, XDG_CONFIG_HOME or LOCALAPPDATA, has no TestSlackGeometry row that reaches it
    fix: "In card 4, add a TestSlackGeometry row that clears HOME, XDG_CONFIG_HOME and LOCALAPPDATA with t.Setenv and asserts the error names them. The test already sets environment, so it stays non-parallel."
    origin: lens:guard-reach
  - id: F15
    severity: LOW
    class: consistency
    location: _lym/plan/04-slackengine-files.md:39
    summary: Card 4 makes slackengine a told-geometry engine under PATTERN-cwd-resolution, but no card adds slackengine to that background's Bound packages list
    fix: "In card 4, add `pattern/PATTERN-cwd-resolution.md` under **Edit:**, and say in the Intent that its Bound packages list gains `slackengine`."
    origin: lens:repo-invariants
  - id: F16
    severity: NIT
    class: consistency
    location: _lym/plan/10-slack-cli.md:59
    summary: setup's missing_scope refusal names the scopes the Long lists, but only login's Long lists them; setup's Long names none
    fix: "In card 10, have setup's missing_scope refusal name the bot scopes itself, `channels:history` and `channels:read` or `groups:history` and `groups:read`, `users:read` and `chat:write`, as login's Long does."
    origin: handler
---

### [BLOCKING] The tick's restart cap never reaches gave-up

**Location:** `_lym/plan/09-slackengine-relay.md:53`, with the cap at line 66 and its row at line 76.
**Issue:** Card 9 caps the prime dobby tick's respawns to meet `PATTERN-child-processes`' rule that a retry loop around a real spawn caps attempt count, and it keeps `StartFailures` and `NextStartAt` in `RelayState`.
Step 3 writes `RelayState` afresh at every relay start that gets past its lock.
That wipes both fields.
A relay that dies before step 3, at its reparent wait, its lock or its config refusal, leaves the previous relay's `FirstReadAt` standing, so the start counts no failure either.
In production the count never reaches five, and the one-minute wait never grows, so a relay that keeps dying before its first read is respawned on every tick while presence is `slack`.
The test row passes only because the fake spawn writes no state.
Card 9 also adds the two fields to a struct card 4 creates, and names no target for that change.

### [MEDIUM] A stale relay state makes `lym slack away` refuse at once

**Location:** `_lym/plan/09-slackengine-relay.md:70`, used by `_lym/plan/10-slack-cli.md:62`.
**Issue:** A relay that went through the double fork is in no pane, so creel never refuses its first read as a strand-pane claim.
Every other refusal keeps the relay alive and retrying.
So a gone relay with a recorded `FirstReadRefusal` is always an earlier relay, such as one run by hand in a non-orch pane.
`away` calls `EnsureRelay` and then `AwaitFirstRead`, whose first read finds that old state before the new relay has taken its lock and written its own.
It then refuses at once, naming a refusal that has nothing to do with the relay it just started.

### [MEDIUM] `relay` cannot resolve a location from the hub directory

**Location:** `_lym/plan/10-slack-cli.md:49`, and the relay verb at line 65.
**Issue:** Card 9 spawns `slack relay --hub-path <hub>` with the hub directory as its working directory, as dobby's spawn does.
Card 10's group pre-run "resolves the location and fills every seam" for every verb.
`lymcwd.Resolve` refuses a working directory that is not a worktree root (`pattern/PATTERN-cwd-resolution.md:6`), so relay never reaches `RunRelay`.
creel's `daemon --hub-path` verb resolves no location (`internal/creelcli/cli.go:873-905`).
The card also builds `creelcli.NewProgramClient(location, …)` and `hubgeom.DobbyStart(prime)`, and never says where that location or the prime comes from.

### [MEDIUM] The hidden `relay` verb's stated bound is false for `--fork` and off Linux

**Location:** `_lym/plan/10-slack-cli.md:45`; `_lym/plan/09-slackengine-relay.md:55`.
**Issue:** Card 10 bounds the hidden verb by "a run by hand is bounded by the refused first read (D3)".
`lym slack relay --fork`, run by hand from a non-orch strand's pane, carries out the double fork itself.
The relay it starts is out of every pane, so creel accepts its first read.
It is a start that bypasses `lym slack start`'s pane refusal (`_lym/plan/10-slack-cli.md:60`).
Off Linux, `resolveSender` returns through `resolveFromEnvironment` before any pane walk (`internal/creelengine/identity.go:52-54`), so a by-hand run is accepted as `SLACK` there too.
The rubric's attack-surface check asks each new flag's card to state what it lets through and what bounds it.
The guard-reach lens raised the off-Linux half, and this finding merges it.

### [MEDIUM] `ErrWaitTimeout` after a handled read is read as an empty read

**Location:** `_lym/plan/08-slackengine-outbound.md:35`, against `_lym/plan/03-creel-seams.md:38`.
**Issue:** `creelengine.Client.CallWatching` arms one timer for the whole call (`internal/creelengine/client.go:144-146`).
`exchange` runs the handler and then `awaitAckConfirmation` against that same timer (`client.go:274`, `284-287`).
A bounded wait whose mail arrives late, followed by a handler that posts for up to 25 seconds, can return `ErrWaitTimeout` after the posts and the ack were sent.
Card 8 reads that as an idle read: it doubles the wait, so outbound activity does not reset the floor, and it keeps the record without logging.
The record is kept either way, so nothing is posted twice.

### [MEDIUM] `CheckBinary`'s declared type does not match `selfexec.CheckWith`

**Location:** `_lym/plan/09-slackengine-relay.md:49`; `_lym/plan/10-slack-cli.md:65`.
**Issue:** The field is `func(beforeExec, failed func()) error`.
`internal/selfexec/selfexec.go:80` declares `CheckWith(beforeExec func(), failed func(error))`, which returns nothing.
dobby's seam takes that shape.
As written, card 10's wiring does not compile, and step 6's `failed` never sees the exec error.

### [MEDIUM] Card 3 extends a doc whose text its change makes false

**Location:** `_lym/plan/03-creel-seams.md:19`, with the targets `internal/creelengine/doc.go:225-227` and `internal/creelengine/identity.go:46`.
**Issue:** The Refusal code section says the strand-claim refusal "is the only refusal that carries a code".
It also says "a program claim from a non-orch pane … carry no code".
Card 3 gives that program claim a code and only appends a sentence under **Extends:**, which the rubric reserves for adding to a ref without changing what is there.
`resolveSender`'s comment says "no other refusal does", and card 3 edits that function without saying the comment changes.

### [MEDIUM] Cards 11 to 13 leave their package docs and help text stale

**Location:** `_lym/plan/12-dobby-relay-tick.md:10`, `_lym/plan/11-orch-relay-presence.md:27`, `_lym/plan/13-anchor-relay-restart.md:14`.
**Issue:** Each card lists its package's `doc.go` under **Uses:** while it changes what that doc states.
- `internal/beamcli/doc.go:209` says the prime's `Deps.Tick` "is the tick of the `hibernateengine.Carrier`", and its Bound speaks only of the carrier.
- `internal/fabriccli/doc.go:103-110` and the anchor verb's Long (`internal/fabriccli/anchor.go:530`) list what a move restarts and reports, naming only `dobby_start_error`.
- `internal/orchcli/doc.go:12` lists the bring-up steps with no relay step.
- `internal/orchengine/doc.go:35` calls the reload pointer the one-line stencil, to which Go now appends a line.
- `docs/overview.md:229`'s beamcli line names only the hibernate request.

The same method gap recurs across three cards, so this is one `design` finding.

### [LOW] Step 6 retakes the lock without the probe grace

**Location:** `_lym/plan/09-slackengine-relay.md:59`.
**Issue:** `RelayLive` takes the lock for a moment on every whileAway tick and every `status`.
Step 2 waits `lock.ProbeGrace` for that reason; step 6's `TryAcquireWriteLock` does not.
A failed re-exec that meets a probe makes the old image exit with no relay running, until the next start.

### [LOW] Two tests pin one unexported helper each

**Location:** `_lym/plan/07-slackengine-inbound.md:16`; `_lym/plan/08-slackengine-outbound.md:10`.
**Issue:** `TestInboundText` tests `inboundText` and `TestOutboundText` tests `outboundText`, both unexported.
`pattern/PATTERN-test-speed.md`'s test economy section tests an unexported helper through the surface that uses it.
Here those surfaces are `TestInboundPoll` and `TestOutboundRead`, which the same cards add.
`TestOutboundText` already defers its unposted senders to `TestOutboundRead`'s rows.
`lym loom validate-plan --overlay` with both Create lines dropped, together with the other target fixes, reported no blocking finding.

### [LOW] A planned `ReadWait` subtest repeats an existing one

**Location:** `_lym/plan/03-creel-seams.md:46`.
**Issue:** `ReadWait` is `ReadAll` with `Wait: true`.
A handler error goes through the same `exchange` branch (`internal/creelengine/client.go:264-267`).
The existing subtest "a program client's ReadAll with a failing handler leaves the reply unread for the next ReadAll" (`internal/creelcli/cli_test.go:1572`) already pins that branch.

### [LOW] Card 8 changes a test file only card 7 targets

**Location:** `_lym/plan/08-slackengine-outbound.md:40`.
**Issue:** "the fake `Mailbox` extends to hand out scripted reads and record acknowledgements" edits `inbound_test.go`, which is card 7's Create target, and card 8 names no target for it.

### [LOW] `login`'s empty-token refusal has no reaching row

**Location:** `_lym/plan/10-slack-cli.md:74`.
**Issue:** Card 10 says login "refuses an empty one", and no `TestSlackCLI` row feeds it an empty standard input.

### [LOW] `SlackGeometry`'s credential-directory refusal has no reaching row

**Location:** `_lym/plan/04-slackengine-files.md:57`.
**Issue:** The refusal naming `HOME`, `XDG_CONFIG_HOME` or `LOCALAPPDATA` (line 41) is reachable with `t.Setenv`, and no `TestSlackGeometry` row reaches it.

### [LOW] PATTERN-cwd-resolution's bound-package list misses slackengine

**Location:** `_lym/plan/04-slackengine-files.md:39`.
**Issue:** Card 4 cites `PATTERN-cwd-resolution` for slackengine's told geometry.
The background's Bound packages list (`pattern/PATTERN-cwd-resolution.md:32`) names creelengine and dobbyengine, and no card adds slackengine to it.

### [NIT] setup's missing-scope refusal points at a list setup's help does not hold

**Location:** `_lym/plan/10-slack-cli.md:59`.
**Issue:** The refusal "names the scopes the `Long` lists", but in this card only login's `Long` lists them.

## Focus verdicts

- Fidelity, focus item 1: each named piece is carried by a card.
  - The 5-minute answer to a refused creel post is in card 7's poll step and its `TestInboundPoll` row.
  - The 24-hour skip is in card 8.
  - The cap of 5 `ok: false` refusals is in card 8, its `*APIError` kind in card 6.
  - The record kept across the acknowledgement deadline is in card 8, with card 3's confirmed acknowledgement.
  - The `cmd/lym/daemonwakeups_test.go` entry for `relay.go` is in card 9.
  - Every D1–D7 and D9 bullet and every acceptance criterion is carried by a card. D7's Review criteria rest on card 14 and code review. The D2 help text rests on card 10 and code review.
- Card order: no card uses a symbol a later card creates. Card 4 follows card 2, cards 7 to 9 follow card 6, and cards 11 to 13 follow card 9.
- Attack surface: the pane refusals of login, setup, start, away, back and stop state what they let through in card 10. EnsureRelay's never-refuse is stated in the overview's shared decision. The coded refusal in card 3 changes no decision. The gap is the hidden verb (F4).
- Reachability: every refusal named in the focus is reached by a named row, except the two in F13 and F14. The first-read end versus retry has rows in `TestRunRelay`.
- Edit typing: Edit targets name member glyphs, and no member's signature changes, so no re-sign arrow is missing. The ImpactSummaries carry blast-radius conclusions. The one Extends on a rewritten ref is F7.
- Granularity: no card is one per literal symbol. Card 1's two PATTERN texts match the overview's two PATTERN changes verbatim.
- No card leaves a research step, edits this task's board entry, or files or promotes a task. No card carries a `**Verify:**` line.

## Note for the parent: not a finding against the plan

The task's board entry carries a section, "Presence follows where the operator writes (operator, 2026-11-05)".
It changes D6 and D7:
- the relay sets presence to `slack` on an operator's Slack message;
- an operator line typed in an orch's pane sets it back to `terminal`, told apart from lym's own injects;
- reed gets a status-bar toggle button.

Its package list also names `internal/reedengine` and `internal/reedcli`.
The decision record, this round's answer key, carries none of it.
The plan carries none of it either, and the rubric bars any card from taking scope the record does not license.
Whether that addition joins this task needs the record amended through the parent first.

## Rejected

- affected-set lens finding 2: it says `internal/tendercli/deps.go:114`'s `runproc.End` of an owner ends the relay. The lens did not confirm that the owner there can be `runproc.HubOwner`, and the rubric's grounding rule needs a confirmed claim before it can cut or add scope.
- My own check of `PATTERN-daemon-wakeups`' keep-warm cap against the 30-second `AwaitFirstRead` and `AwaitReparent` bounds: D6 fixes 30 seconds. That bound lies under any keep-warm cap, as creel's 30-second `goCallTimeout` does for agents' creel verbs, so it is no departure worth a finding.

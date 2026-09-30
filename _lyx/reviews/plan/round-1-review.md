---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/06-transient-boundary-wiring.md
    summary: git-transport and github-api never reach the producer boundary, because every step-reachable push and GitHub failure is returned as a Stuck verdict, and no card changes those arms
  - id: F2
    severity: BLOCKING
    location: _lyx/plan/06-transient-boundary-wiring.md
    summary: Card 6 claims loom's seed-and-commit bootstrap pushes, but CommitAnchoredPaths never pushes, so the planned bootstrap integration test cannot pass
  - id: F3
    severity: BLOCKING
    location: _lyx/plan/02-leaf-transport-classifiers.md
    summary: The first-non-flag rule reads push.autoSetupRemote=true as the subcommand of every gitrepo push, so the main push path never classifies
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/10-loom-start-resume.md
    summary: resumeParkedDriver re-sends on every Send error, so a delivered-but-unverified resume line can be typed up to about twenty more times, and non-readiness errors wait a minute before refusing
  - id: F5
    severity: MEDIUM
    location: _lyx/plan/02-leaf-transport-classifiers.md
    summary: IsTransient excludes a context deadline, so a timed-out GitHub call is never transient, although D2 counts a transport timeout as transient
  - id: F6
    severity: LOW
    location: _lyx/plan/02-leaf-transport-classifiers.md
    summary: IsTransportFailure adds pull to the D2 op list, and no pull call site keeps a GitError in the chain
  - id: F7
    severity: LOW
    location: _lyx/plan/05-step-envelope-transient.md
    summary: The lyx loom step help text lists the error envelope's keys beside kind, and no card adds transient to it
  - id: F8
    severity: LOW
    location: _lyx/plan/07-step-build-identity.md
    summary: docs/overview.md describes last_step, and card 7 does not add the build identity or binary_changed to it
  - id: F9
    severity: LOW
    location: _lyx/plan/01-shedengine-transient-mark.md
    summary: Card 1's ImpactSummary says card 5 wires a classifier; card 6 is the card that sets Shed.Transient
  - id: F10
    severity: LOW
    location: _lyx/plan/02-leaf-transport-classifiers.md
    summary: Card 2 bundles two independently testable classifiers in two packages
  - id: F11
    severity: LOW
    location: _lyx/plan/09-driver-prompt-park.md
    summary: Card 9 bundles driverResumeLine, a separately testable composer, with the launch-prompt change
  - id: F12
    severity: LOW
    location: _lyx/plan/06-transient-boundary-wiring.md
    summary: No test exercises the battenPreStep Mark wiring
  - id: F13
    severity: LOW
    location: _lyx/plan/10-loom-start-resume.md
    summary: No card names the batten end-to-end acceptance criterion or maps a test to it
  - id: F14
    severity: NIT
    location: _lyx/plan/11-ly-drive-skill.md
    summary: The clean-tree guard adds a _lyx and .lyx path filter that D5 does not state and that has no effect in the warp
---

### [BLOCKING] Transient git and GitHub failures never reach the classifier

**Location:** `_lyx/plan/06-transient-boundary-wiring.md` (also cards 1, 2 and 4, whose classes this leaves unreachable)

**Issue:**
The plan sets the mark only in `Shed.stepLocked`'s `callErr != nil` arm and in the `PreStep` hooks.
Every step-reachable remote git or GitHub failure turns into a `Stuck` verdict or a warning before it gets there:

- `internal/landingshed/publish.go`, step 5: a push failure other than `ErrPushRejected` returns `p.stuckOrCancelled(ctx, "push failed: ...")`.
- The same file, `FindPullRequest` and `PullRequests.Create` failures: `stuckOrCancelled(ctx, publishGitHubErrorReason(...))`.
- `internal/landingshed/finalize.go`'s `pushParent`: a parent push failure is `stuckOrCancelled`, and every GitHub call in the close-PR path only warns.
- `internal/battenshed/create.go`: a `createWorktree` error, which carries `fabricengine.Add`'s `push -u origin` `*GitError` through `%w`, is `Stuck`.
- `internal/battenshed/seamchild.go`'s `PushSeed` and loom's `newCommitStatusSeam` push only warn.

`Stuck` from Publish, Finalize or Worktree-Create persists `blocked` (none of these rows has an `on_stuck`), and card 1 says a verdict never runs the classifier.
So under the plan, `git-transport` and `github-api` are set only by the fakes in cards 1, 5 and 6's tests.
A real unreachable remote or a GitHub 503 still hands back as `blocked`, which misses the Goal ("clears the stops that a single re-step would clear") and D1/D2 in practice.

**Fix:**
Add a card, after the boundary wiring, that edits these producer arms.
Where `shedtransient.Class` classifies the failure as non-empty, the arm returns the failure wrapped with `%w` as a hard error instead of `Stuck`, so the `Shed.Transient` classifier marks it.
Every other failure keeps today's `Stuck` and reason.
The card covers Publish's push, PR query and PR create arms, Finalize's parent-push arm, and batten's Worktree-Create arm.
Admit `internal/shedtransient` to `landingshed`'s and `battenshed`'s import allowlists.
Bundle tests with fakes for both branches of each arm, and add a real-git integration test in `landingshed` that points the push at `http://127.0.0.1:1/...` and pins the real stderr.
Add `./internal/landingshed/...` and `./internal/battenshed/...` to the verify section's integration and smoke lines.

### [BLOCKING] Card 6's bootstrap integration test depends on a push that never happens

**Location:** `_lyx/plan/06-transient-boundary-wiring.md`

**Issue:**
The card says "In loom, the seed-and-commit stage pushes through fabric, so a push that cannot reach the remote now reaches the envelope as `kind: bootstrap, transient: git-transport`."
`seedAndCommitBootstrap` (`internal/loomcli/sharedbootstrap.go`) commits through `fabricengine.CommitAnchoredPaths`.
That function's own doc comment calls it the "narrow, no-push, explicit-paths commit shape", and `CommitWeftPaths` never pushes.
`loomPreStep`'s other sub-steps (lock probe, `ensureStatusStrand`) make no remote call either.
So the planned `integration_transientbootstrap_test.go` never reaches a remote.
With an unreachable push URL it gets a successful bootstrap, not `kind: bootstrap`, and the `pre-receive` case is equally unreachable.

**Fix:**
Drop the false claim and the bootstrap integration test.
State that no current bootstrap sub-step makes a remote call, so the `PreStep` `Mark` wiring is D1/D12's bootstrap boundary held ready for a future remote step, not a path live today.
Move the real-remote integration coverage to the producer card from F1.

### [BLOCKING] `IsTransportFailure` misreads `-c key=val` as the subcommand

**Location:** `_lyx/plan/02-leaf-transport-classifiers.md`

**Issue:**
The rule takes "the first non-flag argument in `Args`" as the subcommand.
`internal/gitrepo/push.go` runs every push as `r.runChecked("-c", "push.autoSetupRemote=true", "push")`, so `GitError.Args` is `["-c", "push.autoSetupRemote=true", "push"]`.
Under the rule, the first non-flag argument is `push.autoSetupRemote=true`, not `push`.
That is `PushRebaseFree`, which is `fabricengine.PushWarpRebaseFreeAt`, which is Publish's `PushBranch`: the one push a loom step makes.
Even once F1 is fixed, that push never classifies, and the card's tests (bare `push`/`fetch` args) would not catch it.
Today no production `gitexec` call passes `-C <dir>`, but the same rule would misread one.

**Fix:**
Define the subcommand as the first argument after git's global options.
`-c` and `-C` each consume the following argument, and any other argument that starts with `-` is skipped.
Add test rows for `-c push.autoSetupRemote=true push` and `-C <dir> fetch` with a transport phrase (true), and for `-c push.autoSetupRemote=true push` rejected as non-fast-forward (false).

### [MEDIUM] The resume loop re-sends a line whose delivery may already have happened

**Location:** `_lyx/plan/10-loom-start-resume.md`

**Issue:**
`resumeParkedDriver` calls `SendDriver` up to about twenty times on any error.
`shuttleengine.Runner.Send` already replays internally (`sendVerified` plays the text `1+sendReplays` times).
Its "sent text never appeared in the pane" error is a verification failure, and `sendVerified`'s own doc comment says a delivered copy can go unseen when the pane churns.
Retrying that error types the resume line again, and possibly many times, into a live driver.
The loop also retries errors that no wait can fix, such as `"not a shuttle strand"`, so those refuse a minute late.
D8 bounds the wait to one condition: the pane not being ready for input.

**Fix:**
Retry only the not-ready refusal.
To make it distinguishable without text matching, `shuttleengine` declares `ErrPaneNotReady` and wraps it into `requireReadyAgentPane`'s existing not-ready error, whose text stays unchanged.
`resumeParkedDriver` retries while `errors.Is(err, shuttleengine.ErrPaneNotReady)` holds, and refuses at once on any other error, leaving the marker in place in both cases.
Add tests: a `shuttleengine` test asserting that the not-ready refusal wraps the sentinel, and a resume test asserting that a non-sentinel error makes exactly one call.

### [MEDIUM] A timed-out GitHub call is never transient

**Location:** `_lyx/plan/02-leaf-transport-classifiers.md`

**Issue:**
`IsTransient` is false for "a `*url.Error` that is ... a context cancellation or deadline".
Publish and `FindPullRequest` wrap each GitHub call in `context.WithTimeout(ctx, publishGitHubTimeout)`.
A slow GitHub therefore fails with `context.DeadlineExceeded`, either bare (go-github returns `ctx.Err()` when the context is done) or inside a `*url.Error`.
D2 names the GitHub class "failed in transport", and for git it counts a timeout as transport.
A per-call timeout is the likeliest transient GitHub fault, and the plan classifies it as not transient.
The step's own cancellation never needs this exclusion: every landingshed arm checks `cancelErr(ctx, ...)` before it returns a failure, so an interrupted step returns the cancellation, not the GitHub error.

**Fix:**
Make `IsTransient` true for `context.DeadlineExceeded` anywhere in the chain, and keep it false for `context.Canceled`.
Adjust the test rows to match: a wrapped and a bare deadline are true, and a `*url.Error` wrapping `context.Canceled` is false.

### [LOW] `pull` goes beyond D2's op list and is unreachable

**Location:** `_lyx/plan/02-leaf-transport-classifiers.md`

**Issue:**
D2 lists the git remote operations as push, fetch, ls-remote and clone, and the card adds `pull`.
No production pull keeps a `*GitError` in its chain:
`gitrepo.Pull` runs through the raw `run` and formats only the exit code, and `pushWithRebaseRetry`'s `pull --rebase` arms format `rebaseGitErr.Stderr` with `%s`.
So the extra op buys nothing and widens a closed list without a decision behind it.

**Fix:**
Drop `pull` from the subcommand set so it matches D2's four operations.

### [LOW] `lyx loom step` help text still lists the old error-envelope keys

**Location:** `_lyx/plan/05-step-envelope-transient.md`

**Issue:**
`internal/loomcli/cli.go`'s `Step` `Long` text says "Every error envelope carries the same three keys beside "kind"".
Card 5 adds `transient` to every error envelope but edits only `shedverbs/step.go` and `CONSTRAINTS.md`.
The decision record's constraints require help text to change in the same commit (CLI/Cobra Invariant and Documentation Lifecycle).

**Fix:**
Card 5 also edits `internal/loomcli/cli.go`, so the `Step` `Long` text says every error envelope also carries `transient`: a transient class name, or empty.

### [LOW] `docs/overview.md`'s `last_step` sentence is not updated

**Location:** `_lyx/plan/07-step-build-identity.md`

**Issue:**
`docs/overview.md` says the status envelope carries `last_step` ("the record `lyx shed step` keeps under `.lyx`").
Card 7 adds `vcs_revision`, `vcs_modified` and `binary_changed` to it, which is an observable CLI change, and lists no doc edit.

**Fix:**
Add a `Prosa` target `docs/overview.md` to card 7 that extends that sentence: `last_step` also carries the recorded build identity and `binary_changed`.

### [LOW] Card 1's ImpactSummary names the wrong card

**Location:** `_lyx/plan/01-shedengine-transient-mark.md`

**Issue:**
The summary says "behaviour is unchanged until card 5 wires a classifier".
Card 5 only carries the class onto the envelope; card 6 sets `Shed.Transient` in `NewShed`.

**Fix:**
Name the boundary-wiring card.

### [LOW] Card 2 bundles two independent classifiers

**Location:** `_lyx/plan/02-leaf-transport-classifiers.md`

**Issue:**
`gitexec.IsTransportFailure` and `githubclient.IsTransient` live in different packages, share no code, and each has its own test file.
Under the one-card-per-independently-testable-unit rule, each gets its own card.

**Fix:**
Split card 2 into a `gitexec` card and a `githubclient` card, and renumber the rest.

### [LOW] Card 9 bundles `driverResumeLine`

**Location:** `_lyx/plan/09-driver-prompt-park.md`

**Issue:**
`driverResumeLine` is a pure composer with its own test rows, and card 10 is its only consumer.
The rubric gives an independently testable symbol its own card even when a single card consumes it.

**Fix:**
Move `driverResumeLine` and its tests into their own card, between the prompt card and the `loom start` resume card.

### [LOW] The `battenPreStep` Mark wiring is not tested

**Location:** `_lyx/plan/06-transient-boundary-wiring.md`

**Issue:**
Card 6 wraps `battenPreStep`'s returned errors in `shedtransient.Mark`, and its tests cover only `NewShed` and the unreachable loom bootstrap case (F2).
`internal/battencli/step_test.go` already drives `battenPreStep` directly for its busy, unseeded and done-slug refusals, and none of those tests is extended.

**Fix:**
Extend `TestBattenPreStep_RunLockHeld_KindBusy`, `TestBattenPreStep_DecodeFailure_KindUnseeded` and `TestBattenPreStep_DoneSlug_KindBootstrap` to assert that `shedengine.TransientOf` of the returned error is empty.
Extend loom's `TestStepCmd_BusyRefusal_BeforeBootstrap` to assert `"transient":""` on the envelope.

### [LOW] No card carries the batten end-to-end acceptance criterion

**Location:** `_lyx/plan/10-loom-start-resume.md`

**Issue:**
The decision record's last acceptance criterion reads: an awaiting child with a parked driver is resumed by batten's `Spawn` (`lyx loom start --no-attach`) after `lyx loom approve`, and a `done` child's driver still exits before teardown.
No card names it.
Card 10's tests exercise the resume branch but not the `--no-attach` return that batten's `Spawn` depends on, and card 9 does not tie its `done` teardown test to batten's `callDone` wait.

**Fix:**
Name the criterion in the resume card and add a test: a live strand with the marker under `--no-attach` returns the success envelope after the one verified send, without attaching.
Name it in the prompt card too, and state that the `done` teardown line is what batten's `callDone` waits on.

### [NIT] The clean-tree guard's path filter is not in D5

**Location:** `_lyx/plan/11-ly-drive-skill.md`

**Issue:**
D5's check is plain: skip when `git status --porcelain` in the warp lists any uncommitted change.
Card 11 adds "lists any change outside `_lyx/` and `.lyx/`".
In the warp, `_lyx` and `.lyx` are links into the weft, and the repository's `info/exclude` excludes both (`/_lyx`, `/.lyx`), so porcelain in the warp never lists weft content.
The filter changes nothing and adds a text rule the decision record did not make.
D5's "never reads the weft" is already satisfied by running porcelain in the warp.

**Fix:**
State D5's rule as written, and say why the driver's own records never count: they live in the weft, which the warp reaches only through excluded links.

## Focus departures

None.
The directives were all reached; the verdicts on the ones that turned up no defect:

- Card 3, `agent-start` coverage: webster's Master starts through `StartGated` (`runnerMasterStarter`), and `websterengine` wraps the start error with `%w` ("webster: start master: %w"), which `shedadapters/webster.go` returns unchanged.
  `ErrNotStarted` therefore already reaches the boundary on that path, and `SingleLLM` and `Burler` are the only `RunGated` callers that turn a died or timeout result into an error.
  Not a defect.
- Envelope key-set tests outside `shedverbs`: `loomcli/step_test.go`, `battencli/step_test.go` and `shedcli/parity_test.go` read single keys and assert no exact error-envelope key set.
  `shedcli/cli_test.go`'s two-key assertion covers the kind-less missing-run refusal, which card 5 does not touch.
  Not a defect.
- Card 7's caller claim: `newStepRecorder` has one production caller (`step.go`) and `lastStepOf` has one (`status.go`), so the ImpactSummary is correct.
  The only other `LastStep` consumer is `loomcli/status_test.go`, and it checks only that the `last_step` key is present.

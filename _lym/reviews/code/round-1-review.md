---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/loomcli/prreview_mail_integration_test.go#prReviewMailDeps
    summary: The new tier1 test hand-writes the Mode, Origin and TaskHead seams, standing in for beam.yaml config and fabric, though it holds a real pair whose wired seams read them; PATTERN-test-stand-ins forbids a change adding such a stand-in.
    fix: "In prReviewMailDeps, stop assigning pr.Mode, pr.Origin and deps.TaskHead by hand. Take Mode and Origin from the same pullRequestDeps(pair, ...) value the test already takes MailAgent from, and read TaskHead from the pair's real HEAD through fabric. Set the state those seams read through each owner: beam.yaml's pull_request through configengine.Set over the beam template, and the pair's origin URL and upstream through fabric or git in the hub fixture. Assert the expected head and mail text from that real head instead of the literal \"head\"."
  - id: F2
    severity: BLOCKING
    class: decision
    origin: lens:code-affected-set
    location: internal/loomcli/prreview_mail_integration_test.go#prReviewMailDeps
    summary: The new test adds uses of two standing fakes whose inventory verdicts forbid new uses, prkit.API (verdict goes) and creelfake.Delivery through startNoticeOrch (verdict real); the halt row's unknown-strand refusal rests on the fake delivery's hand-written strand list.
    fix: "Leave the code in place and defer this finding to the run's parent. The tree has no recorded GitHub stand-in to drive pullrequest.Client over, and no reed-backed creel delivery at tier1. Only the parent can rule whether this task builds them, or whether the board entry's tier1 test over a real creel daemon accepts the two standing fakes. The parent files that ruling as a board note for a new task."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-tests
    location: internal/loomcli/prreview_mail_integration_test.go:295
    summary: Moving the three mail rows out of TestPRReview_Wait dropped the table's shared checks for them, among them no push of the refused head and the whole record compared (GatedHead stays put); the new test holds neither, and no commit carries a Loosened-Check trailer.
    fix: "In prReviewMailDeps, wrap pr.PushTask in a counter returned beside gates, and in TestPRReviewGateFailureMail assert it is 0 for every row. Replace the MailedHead-only record check with a cmp.Diff of the whole record against a literal pullrequest.Record carrying GatedHead \"h1\", the row's strand, guid and session id, and MailedHead reviewHead or \"\"."
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: internal/testkit/envkit/envkit.go:474
    summary: "The board entry moves envkit's session stand-ins with no narrowing, yet LiveStrands (nil, nil) and the batten child's DriverStrand and ReviveStrands stay hand-written reed strand answers; the change description narrows the move to the PR-Review and sampler seams without saying why."
    fix: "Fill ImplementerSeam.LiveStrands in FullEnv from the replayed reed's strand table, as replayedSessions does for SessionLive, and do the same for the batten child seams at envkit.go:657-661 where they read reed. Where a seam is no session stand-in, add a line saying so to the Design decisions of _lym/landing/summary.md instead."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testkit/tmuxrec/record.go:590
    summary: placeholdCall swaps every bound value in a paste's free text, {tmp}, {user}, {host} and {socket} included, so pasted text holding a literal /tmp path or the user name is stored as a placeholder and goes unanswered on a host that binds it differently.
    fix: "In placeholdCall, placehold Stdin only with the path bindings the session derives ({hub}), by passing a Bindings holding just those keys, so other literal text the code sends stays as sent. Add a row to TestPlaceholdCall_HoldsAPastesBufferNameAndKeepsTheNumbersOfItsText whose Stdin holds \"/tmp/x\" and the bound user name and stays unchanged."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/testkit/tmuxrec/recordings_test.go#TestReplay_MatchesAPasteByItsTextWhateverBufferItNames
    summary: Binding placeholders in a recorded stdin (bindCall's Stdin line) has no test, since the paste recording's text holds none and the replay test binds nothing.
    fix: "Add a row to TestReplay_MatchesAPasteByItsTextWhateverBufferItNames that replays a copy of the recorded paste whose Stdin is cut to \"see {hub}/notes\", with Bindings{\"{hub}\": \"/r/h\"}. Send stdin \"see /r/h/notes\" and want it answered."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/testkit/tmuxrec/session.go:118
    summary: newSession's fallback to the shell's bare name, which IdleSession relies on for FullEnv on a host without the shell, has no test.
    fix: "Add a tier0 test in package tmuxrec that builds a session over a reed config whose shell names no executable. Set it through configengine.Set over the reed template in the session's hub before LoadConfig reads it. Assert the session builds with Bindings[\"{shell}\"] equal to that bare name."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/research/no-fake-other-modules/loom-landing.md:103
    summary: The stand-ins inventory, which PATTERN-test-stand-ins names as the list of standing fakes, still lists the removed MailAgent closures of prreview_wait_test.go (line 103) and envkit's noopSamplerSessions (line 200).
    fix: "In docs/research/no-fake-other-modules/loom-landing.md, mark the row at line 103 and the noopSamplerSessions row at line 200 as done, naming the tests that now drive the real modules: internal/loomcli/prreview_mail_integration_test.go, and envkit's sessions over the session-start-stop recording."
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md
    summary: "The Merge danger blast radius names only testkit, yet the diff changes internal/reedengine/io.go, a production file, and tests in landingshed, loomcli and loomrecipe, and envkit's change reaches every FullEnv caller in shedbuild, battenrecipe, loomrecipe and shedrecipe."
    fix: "In _lym/landing/summary.md's Merge danger section, widen the blast radius to the tests of every package building on envkit and tmuxrec, and to reedengine's exported InjectBufferPrefix, which changes no behaviour."
---

### [BLOCKING] The new tier1 test hand-writes beam.yaml and fabric answers

**Location:** `internal/loomcli/prreview_mail_integration_test.go#prReviewMailDeps`

**Issue:** `prReviewMailDeps` sets `pr.Mode` to return `"review"`, `pr.Origin` to a fixed GitHub origin and `deps.TaskHead` to `"head"`.
In production `pullRequestDeps` fills Mode from beam.yaml through `pullRequestModeSeam` and Origin through fabric, and the landing wiring reads TaskHead through fabric (`internal/loomcli/landingdeps.go`).
The test already resolves a real pair from `hubforge` and calls `pullRequestDeps(pair, ...)`, but keeps only `MailAgent` from it.
PATTERN-test-stand-ins: "A test never hand-writes another module's state, records, config or output", and "No change adds a stand-in this rule forbids."
The summary's reason that envkit's remaining closures are a standing fake the inventory lists does not cover these: the test writes them itself.

### [BLOCKING] New uses of prkit.API and creelfake.Delivery, both forbidden by the inventory

**Location:** `internal/loomcli/prreview_mail_integration_test.go#prReviewMailDeps`

**Issue:** The test scripts an open status into `prkit.API`.
The inventory gives `prkit` the verdict `goes`, meaning callers drive `pullrequest.Client` over recorded GitHub answers (`docs/research/no-fake-other-modules/loom-landing.md:187`).
Through `startNoticeOrch` it runs creel's daemon over `creelfake.Delivery`.
That stand-in fills `creelengine.Delivery` for reed's strands and panes, and its verdict is `real` (`loom-landing.md:33`).
The halt row's "creel's own refusal of an unknown strand" therefore depends on `delivery.SetStrands`, a hand-written strand table.
Removing them needs a recorded GitHub stand-in and a reed-backed delivery, and neither exists in the tree.
Whether this task builds them, or whether the board entry's "tier1 loomcli test over a real creel daemon" accepts the two standing fakes, is the parent's call.

### [MEDIUM] The moved mail rows lost their no-push and whole-record checks

**Location:** `internal/loomcli/prreview_mail_integration_test.go:295`

**Issue:** At the review base, the three rows ran through `TestPRReview_Wait`'s shared loop.
That loop asserted the push count, zero here since every row set `noEntryPush` and no `gatePushes` (`prreview_wait_test.go:952-958`).
It also compared the whole record, so `GatedHead` stayed `h1` (`prreview_wait_test.go:959-966`).
The first row's name ended `...AndPushesNothing`.
The new test checks only gate runs, mails, `MailedHead` and the outcome, and envkit's `PushTask` is a bare `return nil` that nothing counts.
A regression that pushed the refused head after mailing it, or advanced `GatedHead`, now passes every test.
No commit in the range carries a `Loosened-Check:` trailer.
The failing `MailAgent` in `prreview_test.go` does keep the no-mail check of the closed, ended and crash rows, with nothing lost.

### [LOW] envkit's remaining reed strand answers are left hand-written

**Location:** `internal/testkit/envkit/envkit.go:474`

**Issue:** `ImplementerSeam.LiveStrands` returns `nil, nil`, and the batten child's `DriverStrand` and `ReviveStrands` (`envkit.go:657-661`) answer fixed values.
Each answers what reed's strand table holds, the same table the moved `SessionLive` seams now read.
The board entry's third change names envkit's session stand-ins without narrowing.
The change description narrows it to "PR-Review and sampler session seams" without saying why the rest are not session stand-ins.
The severity is low because these seams start no session, and the Seen paragraph's framing points at the session runner fakes, which did move.

### [LOW] Placeholding a paste's text rewrites any bound value it holds

**Location:** `internal/testkit/tmuxrec/record.go:590`

**Issue:** `held.Stdin = bindings.placehold(call.Stdin)` rewrites every bound value in free text, among them `{tmp}` (`os.TempDir()`), `{user}`, `{host}` and `{socket}` (`session.go`).
A pasted line holding `/tmp/...` or the recording user's name is stored as a placeholder.
On a host that binds it otherwise the replayed call no longer matches the literal the code sends, and it goes unanswered.
The current recording's text holds none of them, so nothing fails today.

### [LOW] Binding placeholders in a recorded stdin is untested

**Location:** `internal/testkit/tmuxrec/recordings_test.go#TestReplay_MatchesAPasteByItsTextWhateverBufferItNames`

**Issue:** `bindCall` now binds `Stdin`, but `session-paste.json`'s text holds no placeholder, and the new replay test runs with `tmuxrec.Bindings{}`.
Dropping that line passes every test, while a paste of a path under `{hub}` would then never match.

### [LOW] The bare-name shell fallback is untested

**Location:** `internal/testkit/tmuxrec/session.go:118`

**Issue:** `newSession` now binds the configured shell's bare name when `ResolveShellPath` fails, instead of failing the test.
`IdleSession`, and so every `FullEnv` and `LandingDeps` caller, relies on it on a host without the shell.
No test pins that such a host builds a session.

### [LOW] The stand-ins inventory still lists the removed fakes

**Location:** `docs/research/no-fake-other-modules/loom-landing.md:103`

**Issue:** PATTERN-test-stand-ins points at the inventory for the standing fakes and their verdicts.
Its row for the `MailAgent` closures of `prreview_wait_test.go` (line 103) and its `noopSamplerSessions` row for `envkit.go` (line 200) still read as standing, though the diff removes both.

### [LOW] The Merge danger blast radius names less than the diff reaches

**Location:** `_lym/landing/summary.md`

**Issue:** The blast radius reads `testkit`.
The diff also changes `internal/reedengine/io.go`, a production file, though only to export a constant with the same value.
It changes tests in `landingshed`, `loomcli` and `loomrecipe`, and `envkit`'s change runs in every `FullEnv` caller (`shedbuild`, `battenrecipe`, `loomrecipe`, `shedrecipe`).
The `two-way` door holds: no record or board format, config meaning, data or outward push changes.

## Rejected

- code-correctness finding 2, a corrupt recording on a host without the shell: on such a host reed's first start fails, so the tier2 scenario stops at `t.Fatalf` before any recording is written.
- code-tests finding 3, envkit's seams undriven: no test drives a session through `FullEnv` or `LandingDeps` (no caller outside landingshed and loomshed reads those seams), so nothing relies on the fill beyond the non-nil check `NilSeams` already makes.
- Focus check of `IdleSession` dropping the replay: no current caller of `LandingDeps` or `FullEnv` reaches a session seam on any platform, so no unanswered call goes unreported today. The change description's To-check-by-hand names the off-Linux case. Not a defect.
- Focus check of the shell fallback's licence: it is an edit the board entry's third change forces, since `IdleSession` never skips and `FullEnv` runs on hosts without the configured shell. `PlaySession` skips off the recording's OS before `newSession` runs.
- Focus check of the recording's host values: `session-paste.json` holds `{host}`, `{shell}` and pid placeholders. The bare numbers left are tmux's `session_created` time, as in the other recordings, and the paste text is the scenario's constant. No user, path or host value is left.
- `TestPlaceholdCall_HoldsAPastesBufferNameAndKeepsTheNumbersOfItsText` as a per-helper test: `placeholdCall` runs only at record time, which is tier2, so this tier0 test covers a behaviour the public replay surface cannot reach. The existing table varies only format and output, so the case cannot be one of its rows.

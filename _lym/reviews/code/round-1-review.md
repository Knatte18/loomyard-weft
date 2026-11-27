---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/landingshed/config.go:199
    summary: "The LoadConfig doc comment the diff edits names the restored key bare as pr_reviewer, beside the bare conflict, describe and fixer, which PATTERN-config-key-names forbids in a comment."
    fix: "In the LoadConfig doc comment of internal/landingshed/config.go, write every key the edited line names in full: `landing.conflict`, `landing.describe`, `landing.pr_reviewer` and `landing.fixer`, and likewise `landing.describe_review` on the next line."
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/landingshed/prreview_session_test.go#fakePRSessions
    summary: "The diff adds hand-written stand-ins for repository modules, which PATTERN-test-stand-ins forbids a change to add: fakePRSessions simulates shuttleengine.Runner, and the SessionLive, RegisterSession and MailAgent fakes simulate reed's strand table and creel. envkit's new noopSessions (internal/testkit/envkit/envkit.go) puts such a stand-in in a kit."
    fix: "Drive the PR-Review session tests through the real shuttleengine.Runner, standing in only for the LLM session and tmux below it through the repository's shared session kit under internal/testkit (llmkit or shuttlefake, whichever carries the recorded session stand-in), with reed and creel real. Alternatively, move the session-runner behaviour these rows pin into shuttleengine's own tests. Replace envkit's noopSessions with a real runner built over the same kit, or leave the seam to the tests that need it. Add no new hand-written module stand-in."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:678
    summary: "Four places tell the agent to retry a gate failure from outside the change with lym loom pr-continue: the mailed gateRetryHint, the prompt's Ending step 3, pr-continue's Long text and docs/overview.md. The verb also sets the continue flag, so that retry ends the review without the operator's word once the head gates clean. The flag also survives a re-gate that fails, so a later passing commit ends the review by itself."
    fix: "Make the retry route stop implying an ending. Either give pr-continue's mailed-head clearing a path that leaves Continue unset (a regate form of the verb, with its own way forward and test), or reword gateRetryHint, the stencil's step 3, the pr-continue Long text and docs/overview.md. The reworded text must say that pr-continue ends the review once the head passes the gate, so the agent runs it for a retry only on the operator's word."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:652
    summary: "sessionStopWanted stops the session in cases judge then keeps waiting on, so the operator's agent is killed and restarted by session id at every poll. One case is an approval at the gated head while the tree is dirty or the head is past it. The other is the continue flag or an approval at the gated head while the text is stale."
    fix: "Make sessionStopWanted mirror judge's ending conditions. For an open pull request, also require !staleText, plus status.HeadSHA == head and a clean tree for an approval. For the continue flag, also require !staleText. Pass staleText in from wake. Add a wait row: the session is not stopped while an approval at the gated head meets a dirty tree."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:718
    summary: "gateHead records MailedHead even when mailFailure's post fails or no strand is recorded. The head is then never gated again and nobody is told, so the review waits silently, with only a log Warn."
    fix: "Have mailFailure return whether the mail was posted, and set MailedHead only after a successful post, so the next wake gates the head again and retries the mail. Alternatively, halt the row with the gate's reason when no strand is recorded or the post fails."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:715
    summary: "In a wait on an ended review (ReviewEnded, no session kept), a commit that fails the gate is mailed to a stopped strand and gateHead returns nil. judge then returns Done for a clean tree, whatever the head, so an ungated, unpushed head passes PR-Review. Before the diff, that failure halted the row."
    fix: "In gateHead, when rec.ReviewEnded is set, halt on a gate failure with the reason and way forward the row used before the diff, mailing nothing. Add a wait row: a failing commit in an ended review halts and does not end Done."
  - id: F7
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/loomcli/prcontinue.go#prContinueAdmitsStrand
    summary: "The answer key admits orch strands and the run's own PR-Review session, but prContinueAdmitsStrand admits any strand whose role is pr-reviewer: another run's slug, a slugless two-part name or another hub's shortname. Its doc, the refusal text and PATTERN-landing all say the run's session."
    fix: "Admit a pr-reviewer strand only when it is the run's own session: when it equals the record's StrandName, or when its parsed slug equals the run's slug. Since the strand check runs before the record read, pass the run's slug into prContinueVerb from prContinueCmd. Add refused rows for another run's pr-reviewer strand and an unparsable strand to TestPRContinueVerb."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/prreview_session_test.go:637
    summary: "Nothing pins wake's stop-before-read, the tree re-read, or the resume after a stop that leaves the review open. The wait tests' fake reads every strand as live and asserts no stops, so deleting sessionStopWanted's continue or approval branch, or the resume, passes every test."
    fix: "Add TestPRReview_Session rows that assert starts and stops. First, a continue flag on a clean gated tree stops the session before the verdict. Second, an approval at the gated head with the review kept open stops and resumes it, or stops it not at all once F4 is fixed. Third, a tree changed between the first read and the re-read is judged on the re-read."
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/wiring_publish_integration_test.go#TestWireLanding_FillsSamplerSessionSeams
    summary: "No test pins that wireLanding fills Landing.PullRequest.SessionLive and RegisterSession, which checkWiring requires before PR-Review runs. landingdeps_test exempts both, and wiring_seams_test lists them as intentionally nil."
    fix: "Extend the wireLanding wiring test (TestWireLanding_FillsSamplerSessionSeams, or a sibling row) to assert that c.env.Landing.PullRequest.SessionLive and RegisterSession are non-nil after wireLanding, for a recipe with a PR-Review row."
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prsession.go#endSession
    summary: "endSession stops the session on every error the driver re-steps, such as a transient GitHub failure on the entry status read or a failed record commit. The operator's agent is killed mid-exchange and restarted by the re-step."
    fix: "In endSession, leave the session running when the error is one the driver re-steps (a transient failure per shedtransient), as on a cancelled context. Stop it only on a verdict, a halt or a non-transient error. Update Call's doc comment to match."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:266
    summary: "A re-entry on a closed pull request with no continue flag starts a session only for judge to halt and endSession to stop it at once, and every lym loom resume repeats the start."
    fix: "In openPullRequest, set keepsSession for a closed pull request only when rec.Continue is set (the case the summary gives for it), so a closed pull request with no flag halts without starting a session."
  - id: F12
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/prreview_session_test.go:637
    summary: "Several behaviours the diff adds have no test row. They are: ensureSession stopping the strand when the record write fails; mailFailure's failed-post and empty-strand branches; a start failure inside resumeSession becoming a halt; no session for a merged or ended re-entry; the liveWarned path; and a cleared MailedHead making the next wake gate that head again."
    fix: "Add rows to TestPRReview_Session and TestPRReview_Wait for each listed behaviour. Each row asserts the starts, stops, mails or gate runs, and the outcome."
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/beamsetting/beamsetting.go:35
    summary: "The PullRequestReview comments in beamsetting.go and internal/dobbyengine/config.go:39 still say the pull request waits for the operator's own ending of its review. The diff lets the PR-Review session or an orch end it on the operator's word."
    fix: "Reword both comments to say the review ends on the operator's word, given directly on GitHub or through lym loom pr-continue, with a PR-Review session in the task's window."
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shellguard/rows.go:1005
    summary: "The comment line the diff adds is indented one tab short, so the file is not gofmt-clean."
    fix: "Run lym loom fmt internal/shellguard/rows.go so the line takes the three-tab indent of the comment block around it."
  - id: F15
    severity: LOW
    class: design
    origin: handler
    location: contracts/stencils/landing/landing-template-pr-review.md
    summary: "The prompt has the agent change the pull request's title and body by editing the change description. The row then sets them from it with no check, and the prompt never asks the agent to validate its edit, so a term the describe check forbids can reach the pull request."
    fix: "In the stencil's title-and-body bullet, add that after editing the change description the agent runs lym loom validate-description and fixes what it reports before committing."
  - id: F16
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md
    summary: "The Merge danger blast radius names only landing. The diff also changes the agents.yaml role set, which an older binary refuses when a hub file carries a pr-reviewer entry, and the orch contracts and pr-continue admission, which change what an orch does."
    fix: "Widen the summary's Blast radius line to name the agents config role set and the orch's pr-continue use beside landing."
  - id: F17
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/landing/landing-template-pr-review.md:5
    summary: "The stencil's header comment says stencil.FillWith requires every marker non-empty, but RenderPRReviewPrompt fills it with stencil.Fill."
    fix: "Change stencil.FillWith to stencil.Fill in the header comment."
  - id: F18
    severity: NIT
    class: consistency
    origin: handler
    location: contracts/stencils/landing/landing-template-pr-review.md
    summary: "The prompt says lym sets the title and body within a minute of an edit. The wait polls at most once a minute (prPollInterval) and wakes up to a minute apart (prWaitCeiling), so an edit can wait close to two minutes."
    fix: "Reword the bullet to say lym sets them at its next read of the pull request, within about two minutes of the edit."
---

### [BLOCKING] Bare config key name in the LoadConfig doc comment

**Location:** internal/landingshed/config.go:199

**Issue:** The diff edits this line to add `pr_reviewer`, and names it and its siblings without the module prefix.
PATTERN-config-key-names requires a key named in a comment to be written in full, `<module>.<dotted path>`.

### [BLOCKING] New test stand-ins for repository modules

**Location:** internal/landingshed/prreview_session_test.go#fakePRSessions

**Issue:** `fakePRSessions` implements `prSessions`, which `*shuttleengine.Runner` fills in production, so it simulates a repository module.
The `SessionLive`, `RegisterSession` and `MailAgent` fakes do the same for reed's strand table and creel.
`noopSessions` adds another in envkit, a kit that may hold no stand-in for a module.
PATTERN-test-stand-ins: "No change adds a stand-in this rule forbids"; an outside thing, here the LLM session and tmux, is stood in for only below the one module that reaches it.

### [MEDIUM] The gate-retry route ends the review without the operator's word

**Location:** internal/landingshed/prreview.go:678

**Issue:** The mail every gate failure gets tells the agent to run `lym loom pr-continue` for a failure outside the change.
`prContinueVerb` always sets `Continue`, and `judge` ends the review on `Continue` with a clean tree at the gated head (prreview.go:850-854).
The answer key has the session end the review only on the operator's word.

### [MEDIUM] The session is stopped and resumed at every poll

**Location:** internal/landingshed/prreview.go:652

**Issue:** `sessionStopWanted` treats an approval at the gated head as ending the review.
`judge` also requires the local head at that SHA, a clean tree and current text (prreview.go:847-858).
So with an approval and an agent mid-edit, each polled wake stops the session, judges, and resumes it.
The same happens under `Continue` with stale text.

### [MEDIUM] A failed gate mail is recorded as delivered

**Location:** internal/landingshed/prreview.go:718

**Issue:** `mailFailure` logs a failed post and returns.
`MailedHead` is set anyway, so `gateDue` stays false for that head (prreview.go:565) and the wait gives no sign.

### [MEDIUM] A failed head can end an ended review Done

**Location:** internal/landingshed/prreview.go:715

**Issue:** In the `ReviewEnded` wait, `gateHead` now mails a failure and returns nil.
`judge` returns `done()` for `ReviewEnded` with a clean tree on an open or closed pull request (prreview.go:826-827, 838-841), whatever the head.
So a head the gate refused passes PR-Review unpushed; the code before the diff halted here.

### [MEDIUM] pr-continue admits any pr-reviewer strand

**Location:** internal/loomcli/prcontinue.go#prContinueAdmitsStrand

**Issue:** The check is `agentname.MatchesRole` on the role alone, ignoring slug and shortname (agentname.go:314-324).
The answer key and the verb's own doc admit the run's PR-Review session only.

### [MEDIUM] The session stop around the ending read has no test

**Location:** internal/landingshed/prreview_session_test.go:637

**Issue:** The only stop row is the merged verdict.
No row asserts the stop on `Continue` or approval, the re-read, or the resume.

### [MEDIUM] wireLanding's PR-Review seams have no test

**Location:** internal/loomcli/wiring_publish_integration_test.go#TestWireLanding_FillsSamplerSessionSeams

**Issue:** `SessionLive` and `RegisterSession` are filled only in `wireLanding` (sharedbootstrap.go:434-440).
`checkWiring` refuses a nil one, and every inventory test exempts them.

### [LOW] Re-stepped errors stop the operator's session

**Location:** internal/landingshed/prsession.go#endSession

**Issue:** `endSession` spares only a cancelled context.
A transient entry `Status` failure (prreview.go:244) therefore stops the live session the operator is working with.

### [LOW] A closed pull request starts a session only to halt

**Location:** internal/landingshed/prreview.go:266

**Issue:** With no `Continue`, `judge` halts on a closed pull request (prreview.go:835) right after `ensureSession` started a session, and `endSession` stops it.

### [LOW] Session behaviours without a test row

**Location:** internal/landingshed/prreview_session_test.go:637

**Issue:** The behaviours the finding lists carry no test row.
`registerErr` and `liveErr` are never set, and the mail seam never fails.

### [LOW] Stale review-mode comments

**Location:** internal/beamsetting/beamsetting.go:35

**Issue:** The two comments keep the "operator's own ending" wording that the diff changed in the beam template and PATTERN-landing.

### [LOW] rows.go is not gofmt-clean

**Location:** internal/shellguard/rows.go:1005

**Issue:** `gofmt -d` shows the added comment line re-indented by one tab.

### [LOW] The description edit route has no check step

**Location:** contracts/stencils/landing/landing-template-pr-review.md

**Issue:** `syncText` builds the body from the description with `summaryparser.Parse` and the credit block alone.
The prompt invites edits to the description and names no validation.

### [LOW] Blast radius names less than the diff can break

**Location:** _lym/landing/summary.md

**Issue:** The diff changes the agents role list and template, and the orch role contracts and `pr-continue` admission.
The radius names landing only.

### [NIT] Stencil header names the wrong fill function

**Location:** contracts/stencils/landing/landing-template-pr-review.md:5

**Issue:** `RenderPRReviewPrompt` calls `stencil.Fill` (prsession.go); `FillWith` is the include-map variant.

### [NIT] "Within a minute" overstates the text refresh

**Location:** contracts/stencils/landing/landing-template-pr-review.md

**Issue:** `syncText` runs only on a polled wake; `prPollInterval` and `prWaitCeiling` are both a minute.

## Rejected

- code-tests NIT, the `AudienceRole` change has no test: the audience annotation only sorts the help a person reads, and no code decides on it.
- Leaving out the parent-review ask and the findings send-back: the second added entry replaces the parent's role with the operator's word, and the first restores only where the old code still fits.
- An interactive session whatever the autonomy, and mail instead of a halt: the board entry's mode 1 and the added entries ask for the operator's session, and the code before 34e64ac2e mailed the same way.
- The `Loosened-Check:` trailer on 6c9543250: its commit gives the cause, that none of the three packages resolves geometry, which the allowlist guards.
- The dropped `agents-pr-reviewer-role-retired` ID: `hubreconcile` reads markers only for registered IDs (reinterpret.go:52-58), so a hub's leftover marker is inert.

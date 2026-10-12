---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-parent-review-gate-alignment.md:52
    summary: No card says what happens when the agent fails to spawn. Card 6 sends an Aligner.Spawn error up as a gate error and card 9 leaves a Begin error unspecified, which breaks D4's rule that every failure ends in today's path.
    fix: In card 6's Intent step 1, state that an Aligner.Spawn error is logged and recorded through RecordAlignment as an Alignment in state fallback with the error text as Reason. The brief is then rewritten through RenderAlignmentBrief and WriteBrief, and the same evaluation continues into today's delivery path, never returning the error. Add a TestGate_Alignment row where fakeAligner's Spawn returns an error, asserting the delivery prompt is carried, brief.md names the error, and a later evaluation never calls Spawn or Poll again. In card 9's Intent step 7, state that a Begin error writes an alignment note with the error as Reason and ends Done. Add a TestPRAlign row where fakeAlignmentSession's Begin fails, asserting no record, the note, and Done.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/09-pr-align-producer.md:88
    summary: Batten's fallback notice cuts the status error to 200 runes (noticeErrorMax in internal/battenshed/notice.go), and PR-Gate's no-record reason plus its PR URL is already about 165 runes. An alignment clause appended after it loses the report path, so D5's requirement that batten's fallback awaiting notice names the report fails.
    fix: In card 9's PR-Gate bullet, make the alignment clause the start of the reason, with the report path first, for example "alignment report <path> did not settle this pull request - <reason>; " followed by the existing reason. In TestPRGate_Verdicts' new note-matching row, assert that the reason starts with that clause.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-alignment-session.md:51
    summary: Card 5 prescribes two guard branches that no named TestSession row reaches. One is Begin's failed start, which closes the attempt with the start error and returns it. The other is Close returning a StopStrand error.
    fix: Add two TestSession rows in card 5's Tests list. In the first, fakeShuttle's Start returns an error, and Begin returns it while attempt.json is Closed with that error as ClosedReason, and a second Begin with the same key starts nothing. In the second, fakeShuttle's StopStrand returns an error, Close returns it, and the attempt is not marked closed.
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/09-pr-align-producer.md:63
    summary: Several PRAlign.Call branches have no TestPRAlign row. They are step 4's transient GitHub failure (returned), no pull request, and a closed pull request (Done with nothing spawned), plus step 9's moved pull request number and differing TaskHead.
    fix: Add these TestPRAlign rows in card 9's Tests list, all over newPRGateFixture's server. A 5xx answer returns the transient error and spawns nothing. No pull request ends Done and spawns nothing. A closed pull request ends Done and spawns nothing. A TaskHead that differs from the pull request head at apply time writes no record and writes the note.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/plan/04-alignment-records.md:65
    summary: ReadVerdict refuses every escalate without a question, but card 4's stencil bullet asks for a named question only on the operator-decision escalation. An agent escalating on a substantive overlap may leave question empty and be read as invalid instead of escalated.
    fix: In card 4's stencil bullet at line 77, say that every escalate fills question. For a substantive overlap it names the peer and what overlaps; for an operator question it names the question and the record or entry that leaves it open. Add an overlap escalate with a filled question to TestReadVerdict's acceptance rows.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/09-pr-align-producer.md:51
    summary: Card 9 says an empty NotePath is how a darn run's PR-Gate is wired, but its own landingDeps bullet gives every caller, darn included, landingAlignmentDeps with a non-empty NotePath. Card 7 line 44 also credits card 10 with reusing alignmentSettings, though card 9 does.
    fix: In card 9 line 51, reword NotePath's doc to "Empty means no note, which is how envkit and tests that wire no alignment are built; a darn run gets a path but never a note, since its recipe has no PR-Align row". In card 7 line 44, change "Card 10's PR-side wiring reuses it" to "card 9's PR-side wiring reuses it".
  - id: F7
    severity: LOW
    class: design
    location: _lyx/plan/04-alignment-records.md:89
    summary: Card 4 bundles shedrun's AlignmentDir and AlignmentRel into the alignment-package card, though they are public symbols of another package that cards 7 and 9 reuse. The granularity rule gives such symbols their own card, as card 2 does for Fabric.CommitLog.
    fix: Run lyx loom insert-card to add a shedrun-alignment-paths card before card 4. Move its Create targets plan:internal/shedrun#AlignmentDir and plan:internal/shedrun#AlignmentRel and its Edit targets internal/shedrun#TestPathConstructors, internal/shedrun#TestDriveReportsRel and internal/shedrun/doc.go there, together with card 4's shedrun Intent paragraph, and drop them from card 4.
---

### [MEDIUM] A spawn failure skips D4's fallback to today's path

**Location:** `_lyx/plan/06-parent-review-gate-alignment.md:52`, `_lyx/plan/09-pr-align-producer.md:66`

**Issue:** Card 5's `Begin` returns a start error, card 7's `Spawn` passes it on, and card 6's `spawnAlignment` names no handling for it, so the error comes back as a gate error.
`shuttleengine.Gate` returns `(GateResult, error)`, and a returned gate error does not fall back to today's delivery.
That goes against D4's rationale that every failure ends in today's path.
A reed or tmux spawn failure would halt Discussion-Write where today the orch would get the delivery.
Card 9 leaves the same case open at step 7.
D5 limits `PR-Align`'s non-`Done` exits to "an error a producer returns today", and a spawn failure is new.

### [MEDIUM] Batten's fallback notice loses the alignment report path

**Location:** `_lyx/plan/09-pr-align-producer.md:88`

**Issue:** `renderNotice` in `internal/battenshed/notice.go` writes `status.Error` through `oneLine(..., noticeErrorMax)`, which cuts it to `noticeErrorMax = 200` runes.
PR-Gate's no-record reason ("the pull request awaits review; run `lyx loom approve` to land it or `lyx loom reject` to send findings to rework") plus `withPRURL`'s `: <url>` is already about 165 runes.
The card's clause adds 53 runes of prefix, the agent's reason, and then the report path.
Appended, the path falls past the cut, so D5's requirement that batten's fallback awaiting notice names the report fails.
The card says it holds without a test that pins it.

### [MEDIUM] Card 5's start-failure and stop-error guards have no test

**Location:** `_lyx/plan/05-alignment-session.md:51`

**Issue:** `Begin` gets a guard branch: a failed start closes the attempt with the error and returns it.
`Close` gets another: it returns a `StopStrand` error, "since an agent must not run beside a fallback".
None of `TestSession`'s rows reaches either branch: `fakeShuttle` is never scripted to fail `Start` or `StopStrand`.
The failed-start branch is also what makes a retried `Begin` after a restart return a closed attempt, which then polls as `CauseClosed`, so it matters to F1's fallback too.

### [MEDIUM] Several `PRAlign.Call` branches have no test row

**Location:** `_lyx/plan/09-pr-align-producer.md:63`

**Issue:** Step 4 sets three exits: a transient GitHub failure returned through `transientFailure`, and no pull request or a closed one ending `Done` with nothing spawned.
Step 9 refuses to apply when the pull request number moved or `TaskHead` differs.
`TestPRAlign` covers only a head that moved during the wait.
None of the other exits has a named row, though `newPRGateFixture`'s server can produce each input.

### [LOW] An overlap escalation without a question reads as invalid

**Location:** `_lyx/plan/04-alignment-records.md:65`

**Issue:** `ReadVerdict` refuses every `escalate` with no `question`.
The stencil bullet at line 77 asks for a named question only on the second escalation kind, the operator decision the board leaves open.
An agent following it for an overlap escalation may leave `question` empty.
The run still reaches the orch, but as `CauseInvalid` ("no verdict came") rather than as the overlap escalation D3 means.

### [LOW] `NotePath`'s darn wording and card 7's card reference disagree with the plan

**Location:** `_lyx/plan/09-pr-align-producer.md:51`, `_lyx/plan/07-parent-review-alignment-wiring.md:44`

**Issue:** Card 9's `landingDeps` bullet says the darn wiring takes `Alignment` unchanged, so darn's `NotePath` is `note.json` in the pr-gate directory, not empty.
That contradicts the field's own doc line on the same card.
Card 7 says card 10 reuses `alignmentSettings`, but card 9's `landingAlignmentDeps` is the one that does.

### [LOW] shedrun's alignment paths ride inside the alignment-package card

**Location:** `_lyx/plan/04-alignment-records.md:89`

**Issue:** `AlignmentDir` and `AlignmentRel` are exported from `internal/shedrun` and used by cards 7 and 9 in `internal/loomcli`.
Under the rubric's granularity rule, a symbol reused across packages gets its own card, as card 2 gives `Fabric.CommitLog` its own.
Card 4 already spans records, prompt, spec, stencil and the stencil registry.

## Rejected

- D6 `peers-geometry`: the overview resolves the caller through `lyxcwd` instead of D6's `internal/cliwire`. This is not a defect. `internal/cliwire/doc.go` limits that package to webstercli and burlercli and bars `internal/lyxcwd`, and `internal/loomcli/cli.go`'s header makes every `lyx loom` verb hub-only through `lyxcwd.Resolve`. A constraint wins over the decision record's wording, and the Shared Decision says so.
- Attack surface, cards 1, 6, 7, 9: each Intent names what an approve or reject lets through and what bounds it. Card 6 cites the reject cap and `final`'s terminal fail, card 7 the snapshot escalation, and card 9 cites `alignment_pr_approve`, the snapshot and head checks and one agent reject per run. Card 1 explains the Guard class of `skip_alignment` and `alignment_pr_approve`, and why `alignment_timeout_min` stays open: a shorter timeout only routes to the parent. No finding.
- Card 1's coherence check: no hub or test fixture sets `parent_review_wait_min` at or below 20 except `TestLoadConfig_Loads`' 5-minute row, which the card adjusts. `parent_review_wait_min` is local-denied, so only the hub file sets it. Reachable through the named rows.
- Card order: card 6 uses only `parentreview`'s own `RecordVerdict`, not card 8. Card 7 uses cards 1, 4 and 5, card 9 uses cards 7 and 8, and card 4 does not depend on card 2. No back edge.
- Verify coverage: `go test ./...` and `go test -tags integration ./...` cover every targeted package, including `internal/testkit/envkit`, `internal/loomcli`, `internal/battencli` and `internal/landingshed`. The section order matches the plan template's.
- Re-sign arrows: `pingDue`, `ParentReviewBrief` and `PRGate.awaiting` carry them, and every caller of `PRGate.awaiting` is in `PRGate.Call`. `landingDeps` already takes `registry` and `runner`, so its signature is unchanged. `GateConfig`, `Deps`, `approveDeps` and `rejectDeps` change only fields.
- Test budget: the rows for `internal/parentreview` (54 against 50), `internal/loomengine`, `internal/landingshed` and `internal/loomcli` match each card's raise.
- `TestHubPeersDeps_OverRealHub` uses `hubforge.CopyHub` and `AddPair`, which `PATTERN-hubforge-fixtures` allows.
- No `Custom` card. Every `Edit` card's `ImpactSummary` gives a blast-radius conclusion.

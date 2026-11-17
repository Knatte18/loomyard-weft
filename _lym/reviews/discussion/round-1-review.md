---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D4 Bound, and D3 review check site
    summary: >-
      D4's Bound promises that a review passing the gate on its cap keeps its out-of-bound findings for the fixer,
      but D3 puts the round checks in the review check, which is ParseCurrentReview, and that strict parse runs again
      after the gate (acceptReview, handoff.go:55-58) and in Fix (acceptedReview, fix.go:217), so a cap-passed review
      hard-fails the round and the fixer never works its findings.
    fix: >-
      In D3 and D4 state that the round checks (prior, carries, the BLOCKING and file bounds) are a separate review
      gate entry with run-directory context, kept out of ParseCurrentReview, and that the post-gate accept and Fix's
      re-parse run the shape checks alone, so a review passed on the cap is accepted with its out-of-bound findings.
      Add an acceptance criterion that a review passed on the cap with a new non-BLOCKING finding reaches the fixer.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D4 origin parent exemption
    summary: >-
      The origin parent exemption from both D4 rules has no stated bound and any reviewer or fork can claim it,
      since Finding.Origin is free text never validated (verdict.go:73-78) and the parent coverage gate checks only
      that report ids are covered (parentseat.go:94-111), never that an origin parent finding comes from a parent ask.
    fix: >-
      In D4 bound the exemption - it applies only in a round holding a parent-review ask, and only to a finding whose
      non-empty parent_ids all name ids of that ask's report; the review check refuses origin parent on any other
      finding. State what the exemption lets through (findings the parent raised, at any severity and file) and add
      acceptance rows for a refused origin parent finding in a round with no ask and with an invented parent id.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md#D6 a fixer ran in round N, and D1 Go-written fixer-reports
    summary: >-
      D6 counts a round as changed when Go wrote its fixer-report because no fixer was spawned, but D1 gives those
      reports the same frontmatter as a fixer's, with empty dispositions that a spawned gate-findings-only fixer also
      writes, and FixResult.Skipped is never persisted (fix.go:55-57), so the Bouncer has no file telling the cases apart.
    fix: >-
      In D1 give the Go-written fixer-reports a Go-only frontmatter key, such as spawned set to false, that the fix
      check refuses in a fixer-written report; in D6 name that key as the test for a fixer-less round, and add it to
      the D6 acceptance criterion on a round whose fixer was not spawned.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D3 carries, and D4 a finding with no carries key is new
    summary: >-
      The D4 bound can be bypassed through carries - a reviewer marks a fixed, undisputed earlier finding open (no
      answer needed) and links a brand-new LOW finding on an unchanged file to it, and every D3 and D4 check passes.
    fix: >-
      In D3 require a carried finding to keep its earlier finding's location path, and have the review check refuse
      one whose path differs; add that refusal to the D3 acceptance criterion.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md#D3 open decision-class findings carried, D1 ruling optional, D2 with the parent
    summary: >-
      A decision-class finding the parent answered through lym loom decision add and the circling verbs leaves no
      ruling entry, so D2 shows it as still with the parent, D3 offers no status that closes it (withdrawn needs a
      dispute), it is carried open, and routeDecisions (bouncerdecision.go:83-96) re-escalates the settled question
      with cause decision every round.
    fix: >-
      In D2 render beside each decision-class finding round N-1's recorded parent answer (its settled circling
      decision and any Added after Discussion entry since); in D3 let a decision-class finding the parent answered take
      resolved, citing that answer, so it is not carried; add an acceptance criterion that an answered decision finding
      is not escalated again.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D1 fix check
    summary: >-
      D1 adds a fix check to the fix half's gate but does not say what happens when it exhausts its attempts - passing
      on the cap hands D2 and D6 a malformed dispositions list, while failing goes through fixedExit, which archives the
      round's review and fixer-report and spends a counted bounce (burlerfix.go:319-327) over a frontmatter typo.
    fix: >-
      In D1 state the fix check's cap behaviour, for example that it passes on its cap and a report whose frontmatter
      then fails to parse leaves round N-1 incomplete for D2 (today's text, full round) and counts as changed for D6;
      add the matching acceptance criterion.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D6 post-fix target id
    summary: >-
      D6 compares the snapshot with a target id the Bouncer computes live at settle time and records nowhere, so a
      harvest and a replay of the same round after a crash or an edit can disagree on unchanged, breaking the Bouncer's
      rule that settlement reads only on-disk state (bouncer.go:1222, bouncer.go:1390).
    fix: >-
      In D5 and D6 record the post-fix target id in the round's snapshot record through internal/state the first time
      the Bouncer computes it, and have every later settle of that round read the recorded value; add an acceptance
      criterion that a replay after an edit settles as the harvest did.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D6 placement before the circling, soft-cap and hard-cap rules
    summary: >-
      settleUndecided opens with the soft-cap auto-accept (bouncer.go:1355-1360), which D6's placement does not name,
      so whether an unchanged round in the soft-cap window with AutoAccept set escalates stalemate to the parent or is
      auto-accepted is left open.
    fix: >-
      In D6 state where the check sits against the auto-accept step and the escalated-record re-escalation, for example
      after the auto-accept and before the remaining rules, and add the acceptance criterion for an unchanged round at
      the soft cap with AutoAccept set.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D6 settle a stalemate escalation as the other four causes
    summary: >-
      arbiterstop lets the arbiter settle hard-cap, circling and decision escalations and refuses every other cause as
      going to the run's parent (arbiterstop.go:113-116), so a stalemate escalation silently bypasses the arbiter unlike
      circling, and D6 says nothing about it.
    fix: >-
      In D6 state whether the arbiter settles a stalemate escalation, and add the matching acceptance criterion for the
      arbiter's eligibility on a pending stalemate escalation.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md#D4 file bound and Bound paragraph
    summary: >-
      A BLOCKING regression a round's own fix causes in an untouched file, such as a caller broken by a changed
      signature, is refused by the file bound, and the Bound names only a defect round 1 missed in code no fix touched,
      so this loss and its way through are unstated.
    fix: >-
      In D4's Bound state this case and its way through - the reviewer locates such a finding at the changed file and
      line that causes it and names the broken site in its summary - and have the stencil rule say the same.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D2 complete round N-1, and Open risks
    summary: >-
      D2 treats round N-1 as complete when its review and fixer-report are both present, so a legacy fixer-report with
      no frontmatter makes the round incremental with no dispositions to render, contradicting the open risk that an
      in-flight run's next round is full.
    fix: >-
      In D2 make round N-1 complete only when its fixer-report frontmatter parses, so a legacy or unparseable report
      keeps today's prior-rounds text and a full round; the open risk then follows from D2.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md#D4 opening, and acceptance criterion on full rounds
    summary: >-
      D4 makes every round that reviews an amendment full, yet lists only an amendment's first round as full, and the
      archive already makes that round full, so whether an amendment's later rounds are incremental is ambiguous; the
      source of the test, the generation record's Amendment mark (amendment.go:82-89), is left to Plan.
    fix: >-
      In D4 pick one reading - every round of an amendment generation is full, or only its first - name the generation
      record's Amendment mark as the source, and make the D4 acceptance criterion match.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md#D1 every finding but a decision-class one, and D2
    summary: >-
      Decision-class findings leave the fixer's work only when the fix row routes decisions (burlerfix.go:75), but D1
      and D2 state the exclusion unconditionally, so on a non-routing row a fixed decision finding has no disposition
      and is rendered as with the parent.
    fix: >-
      In D1 and D2 condition the exclusion on the round's decision routing, so a non-routing row's fixer gives every
      finding a disposition.
  - id: F14
    severity: NIT
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md#Acceptance criteria, first D3 bullet
    summary: D3 refuses a prior list that repeats an id, but the D3 acceptance criterion names only misses and invents.
    fix: Add repeats to the first D3 acceptance criterion.
---

Round 1 of Discussion-Review over the review-rounds-converge record pair.
The design meets the board entry's three ideas and widens "warp tree" to the review target with the widening stated.
The two BLOCKING findings each break a guarantee D4 states.

### [BLOCKING] Round checks inside the strict parse void D4's cap promise

**Location:** `_lym/discussion/decision-record.md` D4 Bound, D3 review check site.

**Issue:** `ReviewGateEntry` passes on its cap so that "the strict parse after the gate still fails the round" (`internal/burlerengine/verdict.go:251-256`).
`acceptReview` (`internal/burlerengine/handoff.go:55-58`) and `acceptedReview` (`internal/burlerengine/fix.go:209-221`) both run `ParseCurrentReview`.
D3 places the new refusals in that same check, so a reviewer that writes an out-of-bound finding three times hard-fails the round.
D4's Bound says the opposite: that the fixer works such findings.

### [BLOCKING] The origin parent exemption is unbounded

**Location:** `_lym/discussion/decision-record.md` D4.

**Issue:** `Finding.Origin` is free text that no Go check validates (`internal/burlerengine/verdict.go:73-78`).
The parent coverage gate checks only that every report id is covered (`internal/burlerengine/parentseat.go:94-111`).
In round 2, with no parent ask, a reviewer can tag a new NIT on an unchanged file `origin: parent`, and both D4 rules let it through.
This is an escape hatch with no stated bound, which the rubric's attack-surface rule flags.

### [MEDIUM] D6 cannot tell a fixer-less round from a fixer that ran

**Location:** `_lym/discussion/decision-record.md` D6, D1 last bullet.

**Issue:** Go-written and fixer-written reports share one frontmatter shape.
Both can carry empty dispositions: a gate-findings-only fixer writes them too (`internal/burlerengine/prompt.go:228-237`).
`FixResult.Skipped` lives only in memory (`internal/burlerengine/fix.go:55-57`).

### [MEDIUM] carries lets a new finding bypass the D4 bound

**Location:** `_lym/discussion/decision-record.md` D3, D4.

**Issue:** The D3 checks match `carries` links to `open` entries one to one and cap the severity.
They never check that the carried finding is the same defect.
So a new LOW finding on an unchanged file passes once it is linked to any undisputed earlier finding marked `open`.

### [MEDIUM] A parent-answered decision finding re-escalates every round

**Location:** `_lym/discussion/decision-record.md` D1, D2, D3.

**Issue:** A parent's answer through `lym loom decision add` and the circling verbs never becomes a `ruling` entry.
D3 has no status that closes the finding, so it is carried `open`.
`routeDecisions` (`internal/shedadapters/bouncerdecision.go:83-96`) then escalates with cause `decision` again.

### [MEDIUM] The fix check's behaviour at its attempt cap is unstated

**Location:** `_lym/discussion/decision-record.md` D1.

**Issue:** If the fix check passes on its cap, a malformed report reaches D2 and D6.
If it does not, `fixedExit` (`internal/shedadapters/burlerfix.go:319-327`) archives the round and spends a bounce.

### [MEDIUM] The post-fix target id is never recorded

**Location:** `_lym/discussion/decision-record.md` D6.

**Issue:** The id is computed live at settle time.
That breaks the Bouncer's invariant that a harvest and a replay settle alike (`internal/shedadapters/bouncer.go:1222`, `:1390`).

### [MEDIUM] Stalemate placement against the soft-cap auto-accept is unstated

**Location:** `_lym/discussion/decision-record.md` D6.

**Issue:** `settleUndecided` runs the auto-accept first (`internal/shedadapters/bouncer.go:1355-1360`), and D6 names only the circling, soft-cap and hard-cap rules.

### [MEDIUM] Arbiter eligibility for a stalemate escalation is unstated

**Location:** `_lym/discussion/decision-record.md` D6.

**Issue:** `internal/arbiterstop/arbiterstop.go:113-116` refuses every cause but hard-cap, circling and decision.
A stalemate escalation therefore goes past the arbiter, and the record does not say so.

### [MEDIUM] A fix's regression in an untouched file is refused unstated

**Location:** `_lym/discussion/decision-record.md` D4.

**Issue:** The file bound refuses a BLOCKING regression a round's own fix causes in a file the fix did not touch.
The Bound names only defects that round 1 missed.

### [LOW] D2's completeness test contradicts the open risk

**Location:** `_lym/discussion/decision-record.md` D2, Open risks.

**Issue:** D2's test counts a fixer-report with no frontmatter as complete.
That makes the next round incremental, not full as the open risk claims.

### [LOW] Amendment rounds are full by one sentence and incremental by the other

**Location:** `_lym/discussion/decision-record.md` D4.

**Issue:** The generation record's `Amendment` mark holds for the whole generation (`internal/shedadapters/amendment.go:82-89`).
The list of full rounds names only an amendment's first round.

### [LOW] The decision-class exclusion assumes decision routing

**Location:** `_lym/discussion/decision-record.md` D1, D2.

**Issue:** `opts.DecisionsRouted` is set only when `NotifyParent` is set or decisions are escalated (`internal/shedadapters/burlerfix.go:75`).

### [NIT] The D3 acceptance criterion omits repeats

**Location:** `_lym/discussion/decision-record.md` Acceptance criteria.

**Issue:** D3 refuses a `prior` list that repeats an id, and no criterion covers it.

## Rejected

- Relocation pass over the record: the Rationale lines carry each decision's reason and restate no Decision text.
  The Auto-mode assumptions are the required section's one-line picks.
  The open risk states an outcome, which F11 corrects.
  Moving any of them to `support-log.md` would drop nothing a plan writer needs, but it would leave nothing leaner either, so no relocation finding stands.
- D5's premise that `git add -A` under the warp's excludes hashes the fix's change: `.lym/` and `.scratch/` are gitignored, and the run directory under `_lym` is outside the warp tree.
  Per-fix commits leave the working-tree id unchanged, so the tree id and the overlay hash give D6 the same answer in both scopes.
- The bound-breaker's LOW on the harvest and replay of the post-fix id is the same defect as F7 and is merged into it.
- The interactions lens's LOW on the auto-accept placement is the same defect as F8 and is merged into it.
- The walkthrough's MEDIUM on the fixer-ran discriminator, and its LOW on legacy fixer-reports, merge into F3 and F11.

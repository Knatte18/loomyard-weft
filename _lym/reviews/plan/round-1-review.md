---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/02-fixer-report-dispositions.md:40
    summary: >-
      Card 2 has ParseFixerReport decode leniently on unknown keys, against the record's PATTERN-tolerant-records
      Constraint that the agent-written fixer-report frontmatter stays strict; a misspelled top-level dispositions
      key on a round whose fixer owns no finding passes the fix check as an empty list.
    fix: >-
      In Card 2's Intent, replace the sentence "It decodes leniently on unknown keys like the review parser; the
      report is agent-written and stays strict only on what it checks." with: ParseFixerReport refuses a
      frontmatter holding a top-level key other than dispositions and spawned, as checkReviewKeys does for the
      review, naming the stray key and the two legal keys in a way forward to rewrite the frontmatter. Add a row
      to TestCheckFixerReport's refusal list in the Tests paragraph: a frontmatter carrying a misspelled
      dispositions key is refused with that way forward.

  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/01-fabric-worktree-tree.md:19
    summary: >-
      Card 1 builds the working-tree id from an empty throwaway index, so a tracked file that matches an ignore
      pattern drops out of the tree and its edits never change the id, which departs from D5's what git add -A
      would stage and can read a real fix as an unchanged round.
    fix: >-
      In Card 1's Intent, change the WorktreeTreeID build to seed the throwaway index from HEAD first, running
      git read-tree HEAD under the same GIT_INDEX_FILE before git add -A, as newIntegrateBuild in
      internal/fabricengine/integrateweft.go seeds its temporary index, so tracked files stay staged whatever
      the ignore rules say. Add a TestFabric_WorktreeTreeID row: an edit to a tracked file that an info/exclude
      pattern matches changes the id and ChangedPathsBetween lists it. Keep the existing row that an untracked
      ignored file leaves the id unchanged.

  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/11-stalemate-settlers.md:17
    summary: >-
      Card 11 opens the arbiter's eligibility to a new escalation cause but its Intent never states what the
      arbiter can now let through, an accept that ends a stalemated segment with open findings and no parent,
      or what bounds it.
    fix: >-
      Add a paragraph to Card 11's Intent after the Eligibility sentence: with stalemate admitted, the arbiter
      may accept a stalemated segment with findings still open, without the run's parent. Name the bounds:
      the accept passes through acceptRound, whose CarryOver seam carries the open findings into the decision
      record's Open risks where the recipe sets carry_over; budget stays refused, so a spent budget still goes
      to the parent; and the segment stays Awaiting until an accept or a continue is recorded.

  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/10-stalemate-escalation.md:24
    summary: >-
      Cards 10 and 12 miss three texts that enumerate the escalation causes; Card 10 even lists
      internal/shedadapters/doc.go under Extends while that file's cause count becomes false.
    fix: >-
      On Card 10, move internal/shedadapters/doc.go from Extends to Edit and add
      internal/shedadapters#ErrEscalationMalformed under Edit. In Card 10's Intent, say the package doc's
      sentences "It does so for one of three causes. A fourth cause, decision" are rewritten to count the
      stalemate cause, and its in-window settle order gains the stalemate check after the auto-accept. Say also
      that ErrEscalationMalformed's doc and escalation.go's file header, which say four causes, are rewritten
      to five. Do not list the escalation.go self glyph: redundant-file-target refuses it beside the card's
      members of that file. In Card 12's Intent, add that the docs/overview.md paragraph on the review
      segment's escalations names the stalemate escalation beside circling and the hard cap.

  - id: F5
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/08-validate-review-round-checks.md:4
    summary: >-
      Card 8 leaves three validate-review texts false after Card 5 and its own change; the help still says the
      only top-level keys are verdict and findings, and the file header and the validateReviewVerb comment say
      the check reads only the file it is given.
    fix: >-
      Add internal/burlercli#validateReviewVerb under Card 8's Edit. In Card 8's Intent, say the Long help's
      sentence naming verdict and findings as the only top-level keys names prior as the third, the
      validateReviewVerb comment no longer says the check reads only the file it is given, and validate.go's
      file header says the verb also runs the round checks on a review in a run directory. Do not list the
      validate.go self glyph: redundant-file-target refuses it beside the card's members of that file.

  - id: F6
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/10-stalemate-escalation.md:79
    summary: >-
      Card 10's TestBouncer_Stalemate row for an unreadable ledger in the window cannot be set up through
      Call, because recordedVerdict parses the ledger before any settle path and an unreadable one sends the
      round to judgeCall instead.
    fix: >-
      Drop the TestBouncer_Stalemate row "an unchanged second round in the window whose ledger is unreadable
      escalates with cause budget, not stalemate". Keep the stalemated bullet's yield on an unreadable ledger,
      and mark it in that bullet as a fail-loud guard the judged-round read already rules out.

  - id: F7
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/06-prior-findings-block.md:32
    summary: >-
      Card 6 says burler-step-review.md's Prior rounds section keeps the first-round and clean-room text, but
      that section holds only the prior_rounds marker; both texts are Go strings in priorRoundsBlock.
    fix: >-
      In Card 6's Intent, replace "The same section keeps the first-round text and the clean-room text for a
      round with no block." with: priorRoundsBlock keeps its first-round and clean-room text for a nil context;
      the stencil's new rules sit below the prior_rounds marker as unconditional text, worded to apply when a
      prior-findings block is present, since the stencil holds no conditionals.

  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/00-overview.md:34
    summary: >-
      The nil-seams-keep-today Shared Decision says a nil seam keeps every round full, but Card 7 builds the
      round context without the Snapshot seam, so a producer with a nil Snapshot still runs incremental rounds
      under the BLOCKING rule alone.
    fix: >-
      Reword the Decision line of nil-seams-keep-today: a nil BurlerHalfDeps.Snapshot writes no target id and
      no changes file, so every round counts as changed and an incremental round holds the BLOCKING rule alone;
      a nil BouncerConfig.Snapshot or a zero StalemateRounds runs no stalemate check; a nil Profile.Round is a
      full round with today's prior-rounds text.

  - id: F9
    severity: NIT
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/10-stalemate-escalation.md:75
    summary: >-
      Card 10's counts-as-changed row says its variants hold two equal ids, which the no-record and
      no-post-fix-id variants cannot.
    fix: >-
      In that TestBouncer_Stalemate row, attach "though its two ids are equal" only to the spawned false,
      missing-disposition and unparseable-report variants.
---

# Plan-Review round 1

The plan carries every Decision D1 to D6, the arbiter's yield ruling and every Constraint (cards named under Focus below).
One Constraint is carried against its own text (F1), and one fabric verb departs from D5's stated semantics (F2).
The rest are a missing attack-surface bound, doc texts the cards falsify without listing, and test-row and wording slips.

### [BLOCKING] Card 2's fixer-report parser is lenient where the Constraint says strict

**Location:** `_lym/plan/02-fixer-report-dispositions.md:40`

**Issue:** The decision record's `PATTERN-tolerant-records` Constraint says "the review and fixer-report frontmatter are agent-written and stay strict", and `pattern/PATTERN-tolerant-records.md` says agent-written contract files stay strict.
Card 2 instead has `ParseFixerReport` decode "leniently on unknown keys like the review parser".
The review's own current-round parse is strict at the top level: `checkReviewKeys` refuses any key but `verdict` and `findings` (`internal/burlerengine/verdict.go:184-200`).
Under Card 2, a report whose fixer writes `disposition:` or `dispositons:` decodes as an empty list.
On a round where the fixer owns no finding, such as a gate-findings-only fix or one whose findings were all routed, that report passes the fix check, so a slip that should get a re-prompt goes unseen.
A constraint violation is BLOCKING under this round's rules.

### [MEDIUM] Card 1's empty throwaway index drops tracked files that match an ignore pattern

**Location:** `_lym/plan/01-fabric-worktree-tree.md:19`

**Issue:** D5 defines the id as a hash of "what `git add -A` would stage under the warp's excludes".
Against the real index, `git add -A` stages edits to every tracked file, including one an ignore pattern matches.
Card 1 runs `git add -A` into a fresh, empty `GIT_INDEX_FILE`, where every path counts as untracked, so such a tracked file is left out of the tree.
An edit to it then leaves the id unchanged, so a fixer's real fix can read as an unchanged round toward a stalemate, and the changes file omits it, so D4's file bound refuses a finding there.
The repository's own temporary-index code seeds the index before it builds: `newIntegrateBuild` runs `read-tree` first (`internal/fabricengine/integrateweft.go:126`).
This worktree has no tracked-but-ignored file today (`git ls-files -ci --exclude-standard` is empty), but Code-Review runs over every hub's warp, so the verb cannot assume none.

### [MEDIUM] Card 11 adds an arbiter eligibility with no stated bound

**Location:** `_lym/plan/11-stalemate-settlers.md:17`

**Issue:** The rubric's attack-surface question requires the card that adds a routing edge to state in its Intent what the edge lets through and what bounds it.
Card 11 admits `stalemate` to `Eligibility` (`internal/arbiterstop/arbiterstop.go:113` today admits hard-cap, circling and decision).
That routes a stalemated segment to the arbiter, which may accept it with findings still open and no parent involved.
The Intent says only that "the same judgment ends it" and gives the arbiter's accept-or-continue rule; it never names what is let through or the bounds that hold.
The bounds exist in code and D6: `acceptRound` carries open findings into `## Open risks` through `writeCarryOver` (`internal/shedadapters/bouncer.go:1313-1331`), `budget` stays refused, and the segment stays Awaiting until a decision.
The card should state them.

### [MEDIUM] Escalation-cause enumerations that Cards 10 and 12 leave stale

**Location:** `_lym/plan/10-stalemate-escalation.md:24`

**Issue:** Found by the affected-set and repo-invariants lenses and checked against the code.
This is one method gap: the cards update the cause lists they name but miss three more.
- `internal/shedadapters/doc.go:140-142` says an unconverged round escalates "for one of three causes. A fourth cause, decision…".
  A stalemate escalation of an unconverged round makes that false, and the in-window settle order at `doc.go:149-158` gains a step.
  Card 10 lists the file under `**Extends:**`, which the rubric flags for a ref the card rewrites.
- `internal/shedadapters/escalation.go:3` and `ErrEscalationMalformed`'s doc (`escalation.go:37`) say four causes, and Card 10 lists neither.
- `docs/overview.md:1272-1284` lists what the review segment escalates for, and Card 12 edits that file only for the review budget keys line.

The proposed target changes were checked with `lym loom validate-plan --overlay` together with F5's and report no new blocking finding.
The `escalation.go` self glyph is withheld because `redundant-file-target` refuses it beside the card's members of that file.

### [LOW] Card 8 leaves validate-review texts false

**Location:** `_lym/plan/08-validate-review-round-checks.md:4`

**Issue:** From the affected-set lens, merged with repo-invariants F2.
`internal/burlercli/validate.go:32-33` says "The frontmatter's only top-level keys are verdict and findings", which Card 5 makes false; Card 8 edits that help text but names only the round-check and git-repository rewrites.
The `validateReviewVerb` comment (`validate.go:17`) says the check "reads only the file it is given", and the file header (`validate.go:2`) says the verb runs the same parse the review gate runs; after Card 8 the verb reads the run directory and decision record too.
Adding the `validate.go` self glyph is refused by `redundant-file-target`, so the header rewrite goes in the Intent text.

### [LOW] Card 10's unreadable-ledger row cannot be set up through Call

**Location:** `_lym/plan/10-stalemate-escalation.md:79`

**Issue:** From the guard-reach lens, checked against `internal/shedadapters/bouncer.go:505-510`.
`Call` reaches every settle path only after `recordedVerdict` reads the round's verdict and ledger; one that does not read sends the round to `judgeCall`, which rewrites the ledger.
`TestBouncer_Stalemate` drives the Bouncer through `newBouncerFixture`, so its unreadable-ledger-in-window row names an input the judged-round read rejects before `settleUndecided` runs.
`softCapRule` already treats its own unreadable-ledger branch as a fail-loud guard (`bouncer.go:1379-1380`).

### [LOW] Card 6 places the first-round and clean-room text in the stencil

**Location:** `_lym/plan/06-prior-findings-block.md:32`

**Issue:** `contracts/stencils/burler/burler-step-review.md`'s Prior rounds section is the bare `{{.prior_rounds}}` marker (line 36), and the stencil allows no conditionals (lines 15-17).
The first-round and clean-room texts are Go strings in `priorRoundsBlock` (`internal/burlerengine/prompt.go:366-390`).
Card 6's sentence that the stencil section "keeps" them misplaces both texts and leaves open how unconditional stencil rules read on a round with no block.

### [LOW] The overview's nil-seams decision overstates what a nil Snapshot does

**Location:** `_lym/plan/00-overview.md:34`

**Issue:** The Shared Decision says each new seam's zero value gives "every round full".
Card 7's `BuildRoundContext` needs no Snapshot: it reads round N-1's review, snapshot record and fixer-report, and Card 4 writes that record with or without a Snapshot.
So a review producer with a nil `BurlerHalfDeps.Snapshot` still renders the block and runs incremental rounds from round 2 on, with the BLOCKING rule alone.
Only a nil `Profile.Round` is a full round.

### [NIT] Card 10's counts-as-changed row claims equal ids for variants that have none

**Location:** `_lym/plan/10-stalemate-escalation.md:75`

**Issue:** From the guard-reach lens: a round with no record or no post-fix id cannot hold two equal ids, so the row's "though its two ids are equal" applies only to the report variants.

## Focus

- Fidelity: D1 Cards 2, 4, 7 and 10; D2 Cards 6, 7 and 9; D3 Cards 5, 7 and 8; D4 Cards 5, 6 and 7; D5 Cards 1, 3, 4 and 12; D6 Cards 9, 10, 11 and 12; the soft-cap yield entry Card 10 through `designWrongKeys`.
  `PATTERN-refusal-way-forward` is carried by Cards 2, 5, 8, 9, 10 and 12, `PATTERN-config-layers` by Card 12 and `PATTERN-sole-parsers` by Card 2.
  No Decision step or Constraint lacks a carrying card; the tolerant-records Constraint is carried against its text (F1).
- Card order: no card uses a symbol a later card creates.
- Reachability: every refusal in Cards 2, 5 and 8 has a named input and a reaching row, and no upstream check rejects one first; Card 10's unreadable-ledger row does not reach (F6).
- Attack surface: Card 5 states its pass-on-cap bound and its parent-exemption bound, and Card 10 states the stalemate bound; Card 11 states none (F3).
- Edit-card hygiene: every Go `Edit` target that the Intent names is a member glyph, and no Intent changes a signature, so no re-sign arrow is missing.
- Test economy: no card plans a per-symbol test or a test of stencil prose; Cards 6 and 9 assert Go-filled values only.
- Board and task scope: no card edits this task's board entry or files or promotes a task.
- Verify lines: no card carries a `**Verify:**` line.
  The `lens-review*.md` files are Plan-Write's lens-review record and its lens reports, as `twill-lens-report-spec.md` places them; they are not cards, the Card Index names none of them, and they are not a defect.

## Rejected

- Card 3's overlay cache at `.lym/review-snapshot/<run_subdir>` rather than a mirrored `.lym/reviews/...` subpath: existing caches such as `ParentReviewDirName` and `ImplementerDirName` sit under `.lym/<name>` by a `lymdirs` constant the same way, so this follows the repository's practice.
- Card 2's second "fixer report" section in `PATTERN-sole-parsers.md` beside landingshed's: noted by repo-invariants as no broken invariant; the sections name distinct packages and parsers.

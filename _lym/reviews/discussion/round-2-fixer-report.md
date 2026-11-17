# Fixer report — Discussion-Review round 2

Every change is in `_lym/discussion/decision-record.md`; `_lym/discussion/support-log.md` is unchanged.

## Changes

- F1 (BLOCKING), fixed as proposed.
  D2 drops the settled-decision-file source, since one is settled only on an accept, which ends the segment.
  A parent's answer now reaches round N only as an `Added after Discussion` entry, which the block renders once, apart from the findings.
  D3 gives a held `decision`-class finding one of two statuses:
  `resolved` with an `answer` quoting the title of the listed entry that settles it,
  or `open`, in which case it is carried.
  The round-check entry refuses `resolved` when its `answer` quotes no listed title, and it refuses any status but those two.
  The D2 and D3 criteria are rewritten:
  the decision-file criterion is gone,
  and a case is added where one of two held findings is answered and the other stays open.
- F2 (MEDIUM), fixed as proposed.
  D5 records the titles of the added entries, as `writeDecisionSnapshot` does, and says a `replace` fold is why a count fails.
  D2 renders the entries whose titles round N-1's snapshot record lacks.
  The D2 criterion now names "new since the snapshot" and adds a case where a replace folds entries before an add.
- F3 (MEDIUM), fixed as proposed.
  D3 now has the review producer build a round context and hand it to `internal/burlerengine`, and the round-check entry judges the review against that context alone.
  The context holds whether the round is incremental, round N-1's findings with their dispositions and held marks, the listed entry titles, the changes file's paths or none, and the parent-ask ids.
  `validate-review` rebuilds the same context through one exported `internal/shedadapters` builder,
  and a file outside a run directory gets the shape checks alone.
  The self-check reshaped the validate-review sentence; see below.
- F4 (MEDIUM), fixed as proposed.
  D6 records the post-fix target id at the first entry to `settleUnlessRefused`, ahead of `routeDecisions`.
  A snapshot record with no post-fix id counts as changed.
  Two D6 criteria are added: the no-post-fix-id case, and that a round `routeDecisions` settles still records its id.
- F5 (MEDIUM), fixed as proposed, taking the first option the fix key offers.
  D4 drops "no reviewer or fork can claim the exemption".
  It states the bound the design builds:
  in a round holding a parent-review ask, the exemption trusts the reviewer's `origin` and `parent_ids` tags, and this record accepts that bound.
  The Auto-mode bullet says the same.
- F6 (MEDIUM), fixed as proposed, taking the fix key's recommended answer.
  D1 lists `ruling` as a fifth disposition, legal only on a `decision`-class finding of a round that routes decisions.
  D2 shows a `ruling` disposition as the fixer's report of a ruling, never as the parent's answer.
  D3 says only the added entries the block lists answer a held finding.
- F7 (MEDIUM), fixed as proposed.
  D2 counts round N-1 as complete when its fixer-report passes the fix check, or when it is a Go-written report carrying `spawned: false` whose frontmatter parses.
  D1 says the fix check judges only fixer-written reports.
  A D2 criterion is added for a round that follows a skipped fixer.
- F8 (MEDIUM), fixed as proposed.
  D3 says the shape checks admit `prior` as a top-level key and `carries` as a finding key, and that only the round-check entry judges either.
  A D3 criterion is added for the shape checks.
- F9 (LOW), fixed as proposed.
  D3 says that with no changes file a carried finding keeps its earlier location path, and the entry refuses a move.
  A case is added to the D3 criterion on carried findings.
- F10 (LOW), fixed as proposed.
  D6 says the decision escalation sits after the judge and ahead of the refusal check and the settle step.
- F11 (NIT), fixed as proposed: the regression sentence is deleted from D4's Bound.

## Self-check

Changes made from the self-check:
- D2: "It names…" now reads "The block names…".
- D2: the parent notice names `lym loom decision add`, and a ruling never stands in for an added entry.
- D3: `validate-review` keeps its pre-run skip and resolves the worktree only for a file in a run directory.
- D3: with no changes file, a carried finding names the moved site in its summary and fix.
- D6: "every later settle" replaces "every later entry".
- Open risks: a line for the title-collision edge.
- Criteria: one Review line, and the validate-review criterion widened to a file outside any git repository.

The self-check's findings:
- fix-claims, LOW: D6 said "every later entry" where a stands-as-recorded replay calls `settle` directly. Fixed: the sentence says "every later settle".
- fix-consistency, LOW: D2's "It names…" read as if it named the decision file. Fixed: the sentence opens with "The block names".
- fix-way-forward MEDIUM, merged with fix-proposal MEDIUM 2: "resolves its worktree as the other burler verbs do" would abort before the shape-checks fallback. Fixed: validate-review keeps the pre-run skip, resolves the worktree from the file's path only for a file in a run directory, and falls back to the shape checks when the worktree does not resolve.
- fix-proposal MEDIUM 1, merged with fix-way-forward LOW: a parent who answers only by message or by `replace` leaves the finding open, and nothing tells the parent to use `decision add`. Fixed differently: the proposal to count a `ruling` disposition as an answer was not taken, because a `ruling` is the fixer's claim and no parent record backs it. Instead the decision escalation's parent notice names `lym loom decision add`, one entry per answered finding, and D2 states the open outcome. A Review criterion is added.
- fix-proposal, LOW 3: with no changes file, a carried finding points at a stale site. Fixed differently: letting the path move would reopen F9's gap, so the path stays and the carried finding's summary and fix name the site the defect moved to.
- fix-proposal, LOW 4: an entry re-added under a title that a fold removed is not listed. Fixed: an Open risks line records it and gives the way out, adding the entry under a new title.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.

# Decision record — review rounds converge

## Goal

Burler's review rounds stop circling in three ways the board entry names.
A fixer's rebuttal reaches the next reviewer beside its finding, and the reviewer withdraws the finding or answers it.
A round after the first checks the earlier round's findings and raises a new finding only on what the fixes changed,
and only at BLOCKING severity.
The Bouncer stops a segment with a named `stalemate` cause, by a Go check,
once a set number of rounds in a row left the review target unchanged.
The entry names the warp tree; this record reads it as the review target,
so Discussion-Review and Plan-Review, whose fixers edit `_lym` documents, are covered too.

## Scope

In:

- The burler review and fix halves of every review segment the Bouncer drives: Discussion-Review, Plan-Review and Code-Review.
- The fixer-report format, the review-file format, the reviewer's and fixer's stencils, and their Go checks.
- The Bouncer's stalemate check, its escalation cause, the loom config key that sets its count,
  the `lym loom circling` verbs that settle it, and the arbiter's eligibility to settle it.
- Every reviewer engine: Claude and Codex render the same rules.

Out:

- Finalize's fix reviews, which `internal/landingshed` runs outside burler.
- The `circling` cause and its LLM judge: they stay as they are, beside the new Go check.
- The bounce budget, the hard cap and amendment rounds' own scoping, apart from D4's full-round rule.

## Decisions

Round N is the round under work; round N-1 is the round before it.

### D1 — The fixer-report carries a parsed disposition per finding, rebuttals included

The fixer-report gains `---`-delimited YAML frontmatter over its existing prose.
Its key `dispositions` lists one entry per finding of the round's review the fixer owns:
every finding, but a `decision`-class one when the round routes decisions to the parent
(`opts.DecisionsRouted`, set when the fix row notifies the parent or the Bouncer escalates decisions).
The fix producer records that flag as `decisions_routed` in the round's snapshot record (D5),
where the review half and `lym burler validate-review` read it.
On a round that routes no decisions the fixer owns every finding and gives each a disposition.
Each entry has `id` (the review's finding id) and `disposition`, one of `fixed`, `fixed-differently`, `disputed`, `deferred` or `ruling`.
`ruling` is legal only on a `decision`-class finding of a round that routes decisions, when a parent ruling reached the fixer;
it is allowed, never required, since no Go record says which findings a ruling reached.

- A `disputed` entry also carries `ground` (`premise false` or `every fix worse`), `rebuttal` (why the code stands)
  and `evidence`, a non-empty list of `<path>:<line>` locations the rebuttal rests on.
  A dispute resting on verify output cites the test file and line that output comes from;
  that narrowing of `contracts/stencils/burler/burler-step-fix.md`'s evidence kinds is deliberate, since the entry asks for file and line.
- A `fixed-differently`, `deferred` or `ruling` entry carries `reason`.
- The dispute grounds, the disposition order and the deferral rule of the fix step stay as they are;
  the frontmatter states their outcome in a form Go reads, and the prose sections stay for the reader.
- The fixer-reports Go writes itself when no fixer is spawned carry the frontmatter too, with the dispositions their case implies:
  empty when every finding was routed to the parent.
  They also carry the Go-only key `spawned: false`, which tells D6 that no fixer ran;
  a gate-findings-only fixer also writes empty dispositions, so the dispositions alone cannot tell the cases apart.

`internal/burlerengine` is the sole parser of the frontmatter, beside its review parser.
A fix check joins the fix half's gate as the review check joins the review half's:
it refuses a report whose frontmatter is missing or malformed, whose dispositions miss, repeat or invent a finding id,
or that carries `spawned`, a key only Go writes.
The fix check judges fixer-written reports only, so its refusal of `spawned` never meets a Go-written report.
The fix check passes on its attempt cap, as the review gate does, so a frontmatter slip never archives the round or spends a bounce.
A report that then fails the fix check, its frontmatter unparseable or its dispositions incomplete,
leaves its round incomplete for D2, so the next round is full with today's prior-rounds text, and counts its round as changed for D6.

Rationale: the rebuttal must reach the next round by Go, not by the next reviewer finding and reading a prose file,
and only a parsed format lets Go place each rebuttal beside its finding.

### D2 — The next reviewer gets the earlier round's findings, each with its disposition and rebuttal beside it

Round N-1 is complete in the run directory when its review is present, its snapshot record holds `decisions_routed` (D5),
and its fixer-report either passes the fix check, judged with that flag,
or is a Go-written report carrying `spawned: false` whose frontmatter parses.
When it is complete, Go renders a prior-findings block into round N's `prior_rounds` value, replacing today's prior-rounds text.
The block lists every finding of round N-1, `decision`-class ones included,
with its severity, class, location, summary and fix, and under it the fixer's disposition:
for `disputed`, the ground, the rebuttal and the evidence; for the others, the reason.
When round N-1's `decisions_routed` is true, each of its `decision`-class findings is marked as held by the parent;
a `ruling` disposition beside it is shown as the fixer's report of a ruling that reached it, never as the parent's answer.
When it is false, a `decision`-class finding carries its disposition like any other.

A parent's answer reaches round N only as an `Added after Discussion` entry of the decision record, written through `lym loom decision add`.
The block renders once, apart from the findings, every such entry whose digest round N-1's snapshot record lacks (D5).
An entry belongs to the whole record, not to one finding, so the reviewer judges which held finding each entry settles (D3).
A settled decision file is no source: one is settled only on an accept, which ends the segment.
The decision escalation's parent notice names `lym loom decision add`, one entry per answered finding, as the way to answer a held finding;
a ruling sent only as a message, or folded in by `lym loom decision replace`, adds no entry, so its finding stays open and is escalated again.
A fixer-written `ruling` disposition is the fixer's claim, not the parent's record, so it never stands in for an entry.

The block names round N-1's review and fixer-report paths, and D4's changes file when one exists.
A round with no complete round N-1 keeps today's prior-rounds text.

The rules the reviewer follows on the block live in `contracts/stencils/burler/burler-step-review.md` and the review-file format,
outside any engine block, so the Claude and Codex reviewers read the same rules.

Rationale: the board entry asks for the rebuttal beside its finding, for every engine;
Go-rendered text from parsed files is the one channel every engine's reviewer reads the same way.

### D3 — The review file records each earlier finding's status and links each carried finding

When D2's block is rendered, round N's review frontmatter gains a `prior` list, one entry per finding the block lists:
`id` (round N-1's id) and `status`, one of `resolved`, `open`, `outdated` or `withdrawn`.

- `withdrawn` is legal only for a finding round N-1's fixer disputed: the reviewer accepts the rebuttal.
- `open` on a disputed finding needs `answer`: why the rebuttal does not hold, citing what the reviewer read.
- `outdated` means the code the finding named is gone or rewritten so the finding no longer applies.
- A `decision`-class finding the parent held takes one of two statuses.
  It takes `resolved` with an `answer` quoting the title of the added entry in D2's block that settles it,
  and is not carried, so a settled question is never escalated again.
  Otherwise it takes `open` and is carried, so a question still with the parent stays in the review the Bouncer reads.
  Only the added entries D2's block lists answer a held finding; a `ruling` disposition does not.
  Go checks only that the `answer` quotes a listed title, not that the entry settles that finding,
  so a reviewer can resolve a held finding by quoting an unrelated listed entry.
  This record accepts that bound: the block lists only entries added through `lym loom decision add` since round N-1 began,
  so a `resolved` status always rests on an answer recorded for the run in that span.
- Each `open` entry is carried as exactly one finding in round N's `findings`, which names it in a new `carries` key
  (`carries: F3`), so the fixer's flow and every reader of `findings` stay unchanged.
  A carried finding keeps its earlier finding's class, and keeps or lowers its severity, never raises it,
  and keeps its earlier finding's location path, as D4 defines a location's path,
  or moves it to a file D4's changes file touches when the fix moved the defect there;
  so a link cannot pass off a new defect on an unchanged file.
  With no changes file a carried finding keeps its earlier location path and may not move it;
  its summary and fix name the site the defect moved to, so the fixer works the defect where it now is.

The review producer builds a round context and hands it to `internal/burlerengine`:
whether the round is incremental, round N-1's findings with their dispositions and held marks from its `decisions_routed` flag,
the titles of the added entries D2's block lists, the paths the changes file touches or none, and the round's parent-ask ids.
These round checks, with D4's bounds, form a round-check gate entry in `internal/burlerengine/verdict.go`,
separate from the review check `ParseCurrentReview` runs.
The entry judges the review against that context alone, so `internal/burlerengine` reads no `internal/shedadapters` format.
It refuses a review whose `prior` list misses, repeats or invents an id,
holds `withdrawn` on an undisputed finding, lacks `answer` on an open disputed one,
gives a held `decision`-class finding a status other than `resolved` or `open`,
or marks one `resolved` with an `answer` that quotes no title the block listed.
It also refuses a review whose `carries` links do not match its `open` entries one to one,
or whose carried finding changes its earlier finding's class, raises its severity,
moves its location path to a file the changes file does not touch, or moves it at all when there is no changes file.
A review with no D2 block carries neither key, and the entry refuses either there.
The entry joins the review half's gate and passes on its attempt cap, as the review gate does.
`ParseCurrentReview` keeps the shape checks alone, so the accept after the gate (`acceptReview`) and Fix's re-parse (`acceptedReview`)
accept a review that passed the round checks on the cap, with the findings they would have refused.
The shape checks admit `prior` as a third top-level frontmatter key and `carries` as a finding key;
only the round-check entry judges whether either may appear.
`lym burler validate-review` keeps skipping the hub, mode and engine wiring of the other burler verbs.
When the review file sits in a run directory, the verb resolves the worktree through `lymcwd` alone (`PATTERN-cwd-resolution`),
from the seam cwd and never from the file's path: `_lym` is a link out of the worktree,
so a path through it does not lie inside the warp.
It then rebuilds the same context through one exported `internal/shedadapters` builder,
which takes the run directory, the round number, the decision record path and the anchor path the parent-review files sit under,
and reads round N-1's `decisions_routed` flag from its snapshot record (D5);
the verb then runs the shape and round checks.
A file outside a run directory, a seam cwd whose worktree does not resolve,
or a run directory that does not lie under the resolved worktree's own `_lym` run root gets the shape checks alone.

Rationale: the statuses make the re-review's outcome explicit and checkable, and `carries` keeps a still-open finding's identity across rounds
without changing what the fixer reads.

### D4 — A round after the first raises a new finding only on what the fixes changed, and only at BLOCKING

A round is incremental when D2's block is rendered and the run directory's generation record carries no `Amendment` mark.
Any other round is full: a segment's first round, the first round of a new generation (the Bouncer archives the run directory then,
so no complete round N-1 remains) and every round of an amendment generation, since the mark holds for the whole generation.

At the start of an incremental round, the review producer writes a changes file into the round's run directory:
the diff of the review target from round N-1's snapshot (D5) to now.
Its paths are worktree-relative.

In an incremental round, a finding with no `carries` key is new, and D3's round-check entry refuses it unless it is `BLOCKING`.
When the changes file exists, the entry also refuses a new finding whose location's path is not a file the changes file touches.
The location's path is its text up to the first `:`, `#` or space, made relative to the worktree root;
the format tells the reviewer to open every location with the file's path.
When the snapshot's cache is gone and no changes file can be built, the BLOCKING rule holds alone.
A new finding with `origin: parent` is exempt from both rules only in a round holding a parent-review ask,
and only when its `parent_ids` list is non-empty and names ids of that ask's report alone.
The exemption lets through every finding tagged as the parent's, at any severity and on any file: the parent's ask sets their scope.
In an incremental round the entry refuses every other new `origin: parent` finding;
a carried `origin: parent` finding is judged by D3's carries rules alone.
Within a round holding a parent-review ask, the exemption trusts the reviewer's `origin` and `parent_ids` tags,
so a finding of the reviewer's own that it tags with an id of the ask's report passes.
This record accepts that bound: Go cannot tell a finding the parent raised from one tagged with the parent's id,
and the exemption is open only in a round the parent asked to review.
The stencil tells the reviewer to raise a new finding only on a changed hunk; the entry holds the file bound.
A `BLOCKING` regression a round's own fix causes in a file the fix did not touch, such as a caller broken by a changed signature,
is located at the changed file and line that causes it, and its summary names the broken site;
the stencil states that rule beside the hunk rule.
Cluster and assisted rounds keep their fan; forks and assistants get the same rule, and the entry judges the consolidated review.

Bound: the round-check entry passes on its attempt cap, and the strict parse after the gate runs the shape checks alone (D3),
so a review that passes on the cap keeps its out-of-bound findings, and the fixer works them.
Short of the cap, an incremental round raises no non-BLOCKING finding and no new finding on an unchanged file.
A defect round 1 missed in code no fix touched stays unraised by later rounds of that segment.
In source scope an untracked, non-ignored file the fixer leaves counts as touched.

Rationale: a later round that reviews the whole target again finds new nits on code it passed, which is the circling the task removes.

### D5 — Go records a snapshot of the review target once per round

The review producer writes a small snapshot record into the round's run directory before the reviewer first starts, through `internal/state`.
A review half that runs again in the same round keeps the record it finds, so a dead fixer's partial edits never become the round's start.
The record holds the fix scope, a target id, and later the post-fix target id D6 records.
It holds the decision record's `Added after Discussion` entries for D2, each as its title and a sha256 digest
of its heading and body as `discussionparser.AddedDecisionTexts` returns them.
It keys on digests, not on a count:
a `lym loom decision replace` fold removes added entries, so a later answer could sit at or below a stored count,
and an answer added under a folded entry's title is told apart by its body and dated heading.
The fix producer adds `decisions_routed` to the round's record before it spawns or skips the fixer:
the fix row alone knows whether the round routes decisions (D1);
the review row carries neither `NotifyParent` nor `decisions_escalated`.
A record without `decisions_routed` leaves its round incomplete for D2.
The target id depends on the scope:

- `source` scope (Code-Review): the id of the warp worktree's working tree, uncommitted edits included,
  taken through a new general fabric verb that hashes what `git add -A` would stage under the warp's excludes into a tree object,
  without touching the index or refs.
  A second general fabric verb diffs two such tree ids, for D4's changes file.
- `overlay` scope (Discussion-Review, Plan-Review): a sha256 over the target files' sorted worktree-relative paths and bytes.
  A copy of those files goes under `.lym`, at a path `lymdirs` names, so D4 can diff them; Go builds the diff itself.

The record is tracked with the run; the tree object and the `.lym` copy are caches.
A missing cache leaves D4 without a changes file and never fails the round.

Rationale: the ring runs the review half, then the fix half, then the Bouncer, and the reviewer never writes the target,
so the target at the review's start is the target the fixer starts from.
One per-round record serves D4's diff and D6's count;
the working-tree id, not HEAD, keeps an uncommitted fix from reading as no change.

### D6 — The Bouncer escalates a stalemate before the budget runs out

`shedadapters.EscalationCause` gains a fifth value, `stalemate`.
At the first entry to `settleUnlessRefused` for round N, ahead of `routeDecisions`, so for every judged round,
the Bouncer computes the target id through D5's seam
and records it as the post-fix target id in round N's snapshot record, through `internal/state`;
every later settle of round N, a replay included, reads the recorded value,
so settlement reads only on-disk state and a replay after an edit settles as the harvest did.
Round N is unchanged when its post-fix target id equals the target id its snapshot record holds, and a fixer ran in round N.
A round counts as changed when it has no snapshot record, its snapshot record holds no post-fix target id,
its fixer-report carries `spawned: false` (D1), or its fixer-report frontmatter does not parse.
The Bouncer counts the unchanged rounds in a row ending at round N,
starting after the latest earlier round that escalated with cause `stalemate`; an escalation of another cause restarts nothing.
A round whose own pending escalation record holds cause `stalemate` escalates with that cause again, so a replay settles as the first settle did.

The check runs in the Bouncer's unconverged path, after the judge returns a verdict that does not converge, Go guards included.
Within `settleUndecided` it sits after the soft-cap auto-accept, so a round the auto-accept takes is accepted,
and before the escalated-record re-escalation, the soft-cap rule and the circling escalation.
In a round whose soft-cap window is open and whose ledger holds no design-wrong entry, the check yields to the code after it,
so the round settles as today, by `softCapRule`'s accept when no escalation record exists;
the check escalates only a round the existing code would not accept.
The judge still runs and writes its ledger every round, so the ledger chain and convergence stay the judge's.
The decision escalation keeps its place after the judge and ahead of the refusal check and the settle step.
Once the count reaches the loom config key `review_stalemate_rounds`, the Bouncer escalates with cause `stalemate`,
writing the escalation record and parent notice as it does for `circling`, and returns Awaiting.
That escalation is how the segment stops: it runs no further round without the parent.

`lym loom circling accept` ends the segment on the current round, and `continue` runs another round and restarts the count;
both settle a `stalemate` escalation as they settle the other four causes, and `continue` keeps its hard-cap refusal.
The parent notice names both verbs as the way forward.
`internal/arbiterstop` admits `stalemate` beside hard-cap, circling and decision, so the arbiter settles a stalemate escalation as it settles `circling`:
a stalemate is circling that Go detected, and the same judgment ends it.

`review_stalemate_rounds` defaults to 2; 0 turns the check off.

Bound: a stalemate escalation is Awaiting like `circling`, never counts against the bounce budget,
and ends no segment without an `accept` from the parent or the arbiter.

Rationale: a round whose fixer changed nothing is a fact Go can read, so it needs no LLM judgment of its own;
two in a row means a rebuttal round (D1 to D3) settled nothing.

### Added after Discussion (arbiter, 2026-11-17): D6's yield condition is softCapRule's own accept test

Decision: In D6, a round's ledger 'holds no design-wrong entry' when designWrongKeys (internal/shedadapters/bouncerfiles.go) returns no key: no open entry at BLOCKING or MEDIUM severity whose fix tag is not mechanical. The stalemate check yields in a window-open round exactly when softCapRule would take its no-design-wrong accept, so a ledger whose only open design-wrong entries are LOW or NIT also yields; Plan wires the yield through the same designWrongKeys call, not a separate test.

Rationale: Round 3's F5 asked D6 to place the stalemate check against softCapRule's accept; the fixed D6 already says the check escalates only a round the existing code would not accept. softCapRule's accept tests designWrongKeys, which counts only open BLOCKING or MEDIUM non-mechanical entries, so this ruling pins the term to that test and leaves no gap where a LOW design-wrong entry would stalemate a round softCapRule accepts.

## Constraints

- `PATTERN-fabric-boundary`: D5's two verbs are general fabric verbs, a working-tree id and a diff of two ids, shaped for no consumer;
  the Bouncer and the review producer reach them through closures on their configs, as the Bouncer's other seams are wired.
- `PATTERN-sole-parsers`: `internal/burlerengine` is the sole parser of the review file and of the fixer-report frontmatter,
  and the Bouncer and every other consumer read its model.
  Both formats join the entry's background list in a Webster card; the entry's own line is unchanged.
- `PATTERN-cwd-resolution`: `lym burler validate-review` resolves its worktree through `lymcwd` alone, from the seam cwd (D3).
- `PATTERN-lym-files`: D5's overlay copy is a never-tracked cache under `.lym`, named only by `lymdirs`.
- `PATTERN-tolerant-records`: D5's snapshot record is Go-written JSON through `internal/state`;
  the review and fixer-report frontmatter are agent-written and stay strict.
- `PATTERN-config-layers`: `review_stalemate_rounds` is classified `CategoryNone` in loom's `ConfigSpec`:
  its value only moves an escalation earlier or later and lets no gate pass,
  since only an `accept` from the parent or the arbiter ends a stalemated segment, as it ends a circling one.
- `PATTERN-stencil-ownership`: the new rules live in the burler stencils, read through `stencilstore`, outside engine blocks.
- `PATTERN-refusal-way-forward`: the stalemate parent notice and every new refusal of the review, round and fix checks name the way forward,
  and a test reaches each.

## Auto-mode assumptions

- D1: frontmatter on the fixer-report, not a new file; verify-output evidence cites a test file and line.
- D3: an open earlier finding is carried with `carries`, decision-class ones included.
- D3: a held finding's `resolved` status is bounded by the block's added entries, not by which finding an entry addresses.
- D3: a carried finding keeps its class; one whose class proves wrong is marked `outdated` and raised anew within D4's bound.
- D4: Go checks the file bound; the hunk bound is the stencil's.
- D4: `origin: parent` findings are exempt from the incremental bound, only within the round's parent-review ask,
  and the exemption trusts the reviewer's tags there.
- D4: every round of an amendment generation is full.
- D5: the source-scope snapshot is the working tree's id, not HEAD.
- D6: the check runs after the judge, so convergence stays the judge's.
- D6: one loom key `review_stalemate_rounds`, default 2; the existing `lym loom circling` verbs settle it.
- D6: the arbiter settles a stalemate escalation as it settles `circling`.

## Open risks

- An in-flight run that crosses the binary change has rounds with no snapshot record or fixer-report frontmatter:
  by D2 and D6 its next round is full and counts as changed, so it reviews and bounces as today until it holds a complete new-format round.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-3-review.md; fixer report: _lym/reviews/discussion/round-3-fixer-report.md.
  - carries-bypasses-bound (class design, severity MEDIUM, open in rounds 1, 2, 3)
  - resolved-status-unbounded (class design, severity MEDIUM, open in rounds 3)
  - routed-fact-unreachable (class design, severity MEDIUM, open in rounds 3)
  - stalemate-vs-softcap-autoaccept (class design, severity MEDIUM, open in rounds 1, 3)
  - validate-review-worktree-resolution (class design, severity MEDIUM, open in rounds 3)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: a fixer-report whose dispositions miss a finding the fixer owns, repeat one, or name an id the review lacks is refused by the fix check;
  one with a `disputed` entry lacking `rebuttal` or a `<path>:<line>` evidence item is refused; a complete report passes.
- D1: on a round that routes decisions, a `decision`-class finding needs no disposition and may carry a `ruling` entry;
  on a round that routes none, a report missing a `decision`-class finding's disposition is refused.
- D1: the Go-written fixer-reports parse and carry `spawned: false`; a fixer-written report carrying `spawned` is refused by the fix check.
- D1: a fix check that exhausts its attempts passes; the round is not archived,
  and its report, unparseable or with incomplete dispositions, leaves the next round full and counts as changed.
- D2: round N's composed review prompt holds every finding of round N-1, each with its disposition and, for a disputed one,
  its rebuttal and evidence, for both the Claude and the Codex reviewer engine;
  a round with no complete round N-1, or whose fixer-written fixer-report fails the fix check, holds today's prior-rounds text.
- D2: a round after one whose fixer was skipped, its Go-written fixer-report carrying `spawned: false`, renders the block.
- D2: the block lists once every `Added after Discussion` entry whose digest round N-1's snapshot record lacks, and none it holds;
  when a `lym loom decision replace` folds entries and a `lym loom decision add` follows, the new entry is listed,
  as is one added under a folded entry's title with a body of its own.
  A `ruling` disposition is shown as the fixer's report, not as the parent's answer.
- D2: on a segment whose fix row only notifies the parent, with no Bouncer escalation of decisions,
  round N's block marks round N-1's `decision`-class finding as held;
  a round N-1 whose snapshot record lacks `decisions_routed` leaves round N with today's prior-rounds text.
- D3: the round-check entry refuses a review whose `prior` list misses, repeats or invents an id, holds `withdrawn` on an undisputed finding,
  lacks `answer` on an open disputed finding, or whose `carries` links do not match its `open` entries;
  it refuses `prior` or `carries` in a review with no D2 block.
- D3: a carried finding whose class differs from its earlier finding's, whose severity is above it,
  or whose location path moves to a file the changes file does not touch is refused;
  one moved to a touched file passes.
  With no changes file, a carried finding that keeps its earlier location path passes and one that moves it is refused.
- D3: of two held `decision`-class findings where the block lists an entry settling one,
  the review marks that one `resolved` with an `answer` quoting the entry's title, does not carry it, and the Bouncer does not escalate it again;
  it marks the other `open` and carries it.
  The round-check entry refuses `resolved` whose `answer` quotes no title the block listed, and a status other than `resolved` or `open`.
- D3: an incremental review carrying `prior` and `carries` passes the shape checks of `ParseCurrentReview`.
- D3: a review that passes the round-check entry on its cap with a new non-BLOCKING finding is accepted, and the fixer works that finding.
- D3: `lym burler validate-review` on an incremental review in its run directory, its seam cwd in the worktree,
  runs the round checks against the context the `internal/shedadapters` builder rebuilds;
  on a file outside one, with a seam cwd outside any git repository,
  or on a run directory outside the resolved worktree's `_lym` run root, shape checks alone.
- D4: in an incremental round, the round-check entry refuses a new non-BLOCKING finding and a new BLOCKING finding on a file the changes file does not touch,
  and accepts a new BLOCKING finding on a touched file, located by worktree-relative path with a `:line` or `#` suffix,
  and an `origin: parent` finding anywhere whose `parent_ids` name ids of the round's parent-review ask.
- D4: in an incremental round, a new `origin: parent` finding is refused in a round with no parent-review ask,
  and when its `parent_ids` list is empty or names an id the ask's report lacks;
  a carried `origin: parent` finding in a round with no ask passes.
- D4: with no changes file, a new BLOCKING finding on any file passes and a new non-BLOCKING one is refused.
- D4: a segment's first round, a new generation's first round and every round of an amendment generation are full rounds.
- D5: a round's snapshot record holds a target id that changes when a fixer edits a target file, committed or not,
  and stays the same when nothing changed; a review re-run in the same round keeps the first record.
- D5: the changes file of an incremental round lists exactly the target files changed since round N-1's snapshot, for source and overlay scope.
- D6: with `review_stalemate_rounds` at 2 and a judge verdict that does not converge, a second unchanged round in a row escalates with cause `stalemate`;
  one unchanged round does not; a changed round between two unchanged rounds resets the count; 0 never escalates.
- D6: a converged verdict on an unchanged round converges; a round with no snapshot record, whose snapshot record holds no post-fix target id,
  or whose fixer-report carries `spawned: false`, counts as changed.
- D6: a `decision`-class round that `routeDecisions` settles records its post-fix target id.
- D6: a replay of a settled round after a target file is edited reads the recorded post-fix target id and settles as the harvest did.
- D6: an unchanged round at the soft cap that meets every condition of the auto-accept is auto-accepted, not escalated with cause `stalemate`.
- D6: an unchanged round in the soft-cap window, with the auto-accept setting off and a ledger holding no design-wrong entry,
  is accepted by `softCapRule`, not escalated with cause `stalemate`.
- D6: a replay of a round whose pending escalation holds cause `stalemate` escalates with that cause again and runs no further round.
- D6: `lym loom circling continue` after a stalemate runs another round and the count restarts; `accept` ends the segment.
- D6: the arbiter is eligible to settle a pending `stalemate` escalation.
- Review: `contracts/stencils/burler/burler-step-fix.md` tells the fixer to write the D1 frontmatter,
  and the review-file format states D3's `prior` and `carries` keys, the carried finding's class and location-path rules,
  the two statuses of a held `decision`-class finding, `resolved` with an `answer` quoting the settling entry's title or `open` and carried,
  and D4's bound,
  including that a fix's regression in an untouched file is located at the changed line that causes it.
- Review: `lym burler validate-review`'s help text names the round checks it runs on a review in a run directory
  and no longer says the verb needs no git repository.
- Review: the stalemate parent notice names `lym loom circling accept` and `continue`.
- Review: the decision escalation's parent notice names `lym loom decision add`, one entry per answered finding.

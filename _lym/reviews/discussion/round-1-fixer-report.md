# Round 1 fixer-report — Discussion-Review, review-rounds-converge

All 14 findings were fixed in `_lym/discussion/decision-record.md`.
`support-log.md` is unchanged.

## Changes

- F1 (BLOCKING), fixed as proposed.
  D3 moves the round checks (`prior`, `carries`, D4's BLOCKING and file bounds) into a round-check gate entry that has the run directory and round number.
  That entry is kept out of `ParseCurrentReview` and passes on its cap.
  `acceptReview` and `acceptedReview` keep the shape checks alone.
  D4's Bound now points at that split.
  New D3 criterion: a review passed on the cap with a new non-BLOCKING finding reaches the fixer.
- F2 (BLOCKING), fixed differently in one part.
  D4 limits the `origin: parent` exemption to a round holding a parent-review ask, and to a finding whose `parent_ids` list is non-empty and names only ids of that ask's report.
  It states what the exemption lets through.
  New criteria cover a finding refused because the round has no ask, and one refused because it names an invented id.
  The proposal asked the check to refuse `origin: parent` on any other finding.
  I limited the refusal to new findings in an incremental round:
  a full round has no bound to exempt from, and a carried parent finding must stay carriable (fix-proposal self-check finding 1).
- F3 (MEDIUM), fixed as proposed.
  D1 gives Go-written fixer-reports the Go-only key `spawned: false`, which the fix check refuses in a fixer-written report.
  D6 uses that key as the test for a round with no fixer, and its acceptance criterion follows.
- F4 (MEDIUM), fixed as proposed, then widened by the self-check.
  D3 requires a carried finding to keep its earlier location path, as D4 defines a location's path, and the round-check entry refuses one that does not.
  After the self-check, a carried finding may also move to a file the changes file touches (fix-proposal finding 2).
  The D3 criterion covers both cases.
- F5 (MEDIUM), fixed as proposed.
  Beside each `decision`-class finding the parent held, D2 renders the parent's recorded answer:
  round N-1's settled decision file of cause `decision`, and every `Added after Discussion` entry past the count kept in round N-1's snapshot record.
  D5's record gains that count.
  D3 has such a finding take `resolved` with `answer`, so it is not carried.
  The criteria require that it is not escalated again.
- F6 (MEDIUM), fixed as proposed.
  D1's fix check passes on its cap.
  A report that then fails the check leaves the next round full and counts as changed.
  The criterion is added.
- F7 (MEDIUM), fixed as proposed.
  D5's record holds the post-fix target id.
  D6 records it through `internal/state` on the first settle and reads it on every later settle.
  The replay criterion is added.
- F8 (MEDIUM), fixed as proposed.
  D6 places the check after the soft-cap auto-accept and before the escalated-record re-escalation, the soft-cap rule and the circling escalation.
  The soft-cap criterion is added.
- F9 (MEDIUM), fixed as proposed, with the choice the finding left open.
  D6 has `internal/arbiterstop` admit `stalemate`, so the arbiter settles it as it settles `circling`.
  Scope, Bound, the Auto-mode assumptions and the `PATTERN-config-layers` constraint now say so, and the criterion is added.
- F10 (MEDIUM), fixed as proposed.
  D4 and its Bound state that a BLOCKING regression in an untouched file is located at the changed line that causes it, with the broken site named in the summary.
  The stencil rule and the Review criterion say the same.
- F11 (LOW), fixed as proposed.
  D2 counts round N-1 complete only when its fixer-report passes the fix check.
  The open risk now cites D2 and D6.
- F12 (LOW), fixed as proposed.
  Every round of an amendment generation is full, judged by the generation record's `Amendment` mark.
  D4, the criterion and the Auto-mode assumptions follow.
- F13 (LOW), fixed as proposed.
  D1, D2 and the D1 criterion exclude `decision`-class findings only when the round routes decisions (`opts.DecisionsRouted`).
- F14 (NIT), fixed as proposed: the D3 criterion now lists repeats.

## Self-check

- fix-consistency, LOW: `PATTERN-refusal-way-forward` left out the round-check entry. Fixed as proposed.
- fix-consistency, LOW: the round-check entry did not refuse an answered decision finding marked `resolved` with no `answer`. Fixed as proposed, merged with fix-way-forward finding 2.
- fix-consistency, LOW: the Review criterion left out the location-path rule and the `resolved`-with-`answer` rule. Fixed as proposed.
- fix-claims, MEDIUM: "since round N-1's review" could not be computed, because an entry's heading carries only a date. Fixed differently: D5's snapshot records the entry count, and D2 renders the entries past it.
- fix-claims, LOW: the soft-cap auto-accept criterion left out the auto-accept's other conditions. Fixed as proposed.
- fix-way-forward, MEDIUM: a replay of a round with a pending `stalemate` escalation could fall through to CONTINUE. Fixed as proposed: D6 re-escalates with that cause, and a criterion is added.
- fix-way-forward, MEDIUM: no Go check made the reviewer resolve an answered decision finding. Fixed as proposed: the round-check entry refuses any other status, and a criterion is added.
- fix-proposal, MEDIUM: the `origin: parent` refusal also caught carried parent findings. Fixed as proposed: it now refuses only new ones, and a criterion is added.
- fix-proposal, MEDIUM: under the location-path rule, a defect the fix moved could only be marked `outdated`. Fixed differently: a carried finding may move to a file the changes file touches.
- fix-proposal, MEDIUM: the cap rule covered only an unparseable report. Fixed as proposed: D1 and D2 treat any report failing the fix check as incomplete.
- fix-proposal, MEDIUM: arbiter settlement contradicted `PATTERN-config-layers`. Fixed differently: the arbiter choice stays, and the constraint text now names an accept from the parent or the arbiter.
- fix-proposal, LOW: a decision file of any cause counted as the parent's answer. Fixed as proposed: only cause `decision` counts, and a criterion is added.

## Disputed

None.

## Deferred

None.

## Rulings

None.

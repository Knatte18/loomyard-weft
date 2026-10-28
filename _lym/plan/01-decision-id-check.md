# Card 1 — decision-id-check

**Create:**
- `plan:internal/discussionparser#CheckDecisionIDs` -> `func CheckDecisionIDs(record []byte) []Finding`
- `plan:internal/discussionparser#TestCheckDecisionIDs` -> `func TestCheckDecisionIDs(t *testing.T)`

**Edit:**
- `internal/loomshed#ValidateLoomDiscussion`
- `internal/loomshed#ValidateDarnDiscussion`
- `internal/loomshed#TestNewDiscussionGate`
- `internal/discussionparser#TestAppendDecision_AppendsAtEndOfDecisionsSection`
- `internal/discussionparser/validate.go`

**Extends:**
- `internal/discussionparser/doc.go`
- `contracts/specs/loom-discussion-spec.md`
- `internal/loomcli#loomCLI.validateDiscussionCmd`

**Uses:**
- `internal/discussionparser#fencedLines`
- `internal/discussionparser#isAddedDecisionHeading`
- `internal/discussionparser#Validate`
- `internal/loomshed#validDecisionRecord`
- `internal/loomshed#writeDiscussionFixture`

**Intent:**
D1 and D2 of the decision record: a machine-readable decision id and its acceptance coverage.

`CheckDecisionIDs`, in a new file of `internal/discussionparser` (stdlib only, so `TestLeafInvariant_AllowlistOnly` keeps passing), reads the record's bytes and returns its findings with an empty `Path`, which each caller stamps.
It reads the lines of the `## Decisions` section up to the next H2, skipping fenced lines as `fencedLines` marks them.
Each H3 there that matches `### D<n> — <title>` (n a positive integer, title non-empty) yields the id `D<n>`; an H3 that `isAddedDecisionHeading` accepts is skipped; any other H3 is one blocking `decision-heading-malformed` finding naming the heading line, with the detail "way forward: rewrite the heading as `### D<n> — <title>` when it is a decision, or demote it to H4 or plain text when it is not".
An id seen twice is one `decision-id-duplicate` finding naming the id, with "way forward: renumber one of the decisions and every mention of its id".
Then, when the record carries a `## Acceptance criteria` line, every id is looked up as a whole word (bounded by a non-alphanumeric rune or the line's ends, so `D1` does not match `D10`) in the bullet lines of that section up to the next H2; an id no bullet names is one `decision-not-in-acceptance` finding naming the id, with "way forward: name it in the criterion that covers it, or add one".
A range is matched only by the ids it spells.
A record with no `## Acceptance criteria` line gets no `decision-not-in-acceptance` finding, since `Validate`'s `discussion-section-missing` already names the defect; a record with no `## Decisions` line yields nothing.

`ValidateLoomDiscussion` and `ValidateDarnDiscussion` call it after `discussionparser.Validate`, never inside it:
each reads the record with `os.ReadFile` once `Validate` returned no error; a not-exist read adds nothing (Validate already reported the record missing), any other read error is returned.
The loom caller appends the stamped `CheckDecisionIDs` findings after `Validate`'s and before `CheckRelevant`'s, whether or not `Validate` found defects.
The darn caller appends them after `Validate`'s; when `Validate` found defects it returns `Validate`'s plus these and does not run the history check, and on a clean `Validate` it returns these with the history findings appended, so the history check still runs only on a record `Validate` finds clean.
`lym loom decision add` keeps running `Validate` alone through `AppendDecision`, so a record whose decision H3s predate D1 is never refused there.
The file header of `internal/discussionparser/validate.go`, which says the loom gate's other check is `CheckRelevant` and a darn run runs `Validate` alone, is rewritten to say both callers also run `CheckDecisionIDs` beside `Validate`.

Tests:
`TestCheckDecisionIDs`, a table over record texts in the existing tier0 test file of the package: a clean record with `### D1 — x` and `### D2 — y` both named in acceptance bullets; an H3 `### Some decision` (malformed); an `### Added after Discussion (orch, 2026-10-28): t` heading (skipped, no finding); a repeated `### D1 — z` (duplicate); D3 named in no bullet (not-in-acceptance naming D3); D1 named only as `D10` (not-in-acceptance naming D1); a range "D1 to D4" spelling D1 and D4 only (D2 and D3 reported); a `### D9 — x` inside a fenced block (not read); no `## Acceptance criteria` section (no not-in-acceptance finding); an empty-title `### D1 — ` heading (malformed); each finding asserted by its exact Check, the id or heading in its Detail and its way-forward clause in its Detail.
`TestNewDiscussionGate` gains rows: a loom record with a malformed decision heading fails the gate naming `decision-heading-malformed`; a loom record that also misses a required section reports the section finding before the decision finding, in that order; a darn record with a repeated id and every required section fails `NewDarnDiscussionGate` naming `decision-id-duplicate`, calls the history seam once (which returns no versions) and commits nothing; a darn record with a repeated id that also misses a required section fails naming both findings, section first, commits nothing and never calls the history seam (the history seam fails the test when called).
`TestAppendDecision_AppendsAtEndOfDecisionsSection` gains a row whose record's decision H3 is `### Some decision`: `AppendDecision` returns no finding and appends the entry.

Docs: `internal/discussionparser/doc.go` names `CheckDecisionIDs` and that it runs beside `Validate`, never inside it, and not from `AppendDecision`.
`contracts/specs/loom-discussion-spec.md` gains, under the decision record's section, the D1 heading form `### D<n> — <title>` with ids unique, and the D2 rule that each id appears as a whole word in an acceptance bullet, for loom and darn records alike, `Added after Discussion` entries carrying no id.
`validateDiscussionCmd`'s Long help gains a sentence naming the decision-id and acceptance checks on loom and darn runs.

**ImpactSummary:** two loomshed discussion checks gain appended findings; their callers (both discussion gates, `validate-discussion`) keep their signatures, and records without `### D<n>` decision headings that carry only `Added after Discussion` H3s are unaffected.

**Commit:** `1: Check decision ids and their acceptance coverage in the decision record`

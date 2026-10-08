---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:81
    summary: D3 does not say whether a recorded start missing from the repository, or recorded starts with no common ancestor, take the archive-only path or refuse
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:109
    summary: D4 lists left uncommitted paths only for a standalone reset to start and the archive-only path, while the Acceptance line promises it for every standalone reset and a pair reset to start leaves untracked files unlisted
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:280
    summary: The D7 Acceptance line names the frontmatter, framing and verify section as refused overview changes but omits Shared Decisions and Rename mechanic, which D7 pins
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:50
    summary: D1 says every target keeps the missing-commit, ancestor and dirty-tree guards, but D3 drops all three for the archive-only path of the start target
---

### [MEDIUM] D3's archive-only path has no closed trigger set

**Location:** `_lyx/discussion/decision-record.md:81-83`
**Issue:** D3 sends `reset --to start` to the archive-only path "when no batch recorded a start, or the recorded start is not an ancestor of HEAD".
`PlanReset` has two more outcomes before a start can be used: a recorded commit missing from the repository (`missingCommitsClause`), and recorded starts that share no single oldest commit and no common ancestor (`resolveResetSHA`'s octopus merge-base failure).
D3 then says the archive-only path "drops the missing-commit, ancestor and dirty-tree refusals", which implies the missing-commit case archives without moving, but the trigger sentence does not list it.
The no-common-ancestor case appears nowhere.
The plan writer has to pick between archiving and refusing for these two states.
The choice matters: refusing sends the operator to a way forward that no longer exists (`run --fresh` is retired from the loom path by D3), and archiving loses nothing but the run record's place.
**Fix:** Close the trigger set in D3: name every state of the start target that archives without moving (no recorded start, recorded start missing, recorded start not an ancestor, starts with no common ancestor), or state which of them still refuse and with what way forward.

### [LOW] D4 and the Acceptance line disagree on which resets list the paths they left

**Location:** `_lyx/discussion/decision-record.md:109`, `_lyx/discussion/decision-record.md:275`
**Issue:** D4 lists the left uncommitted paths in the envelope for "a standalone reset to start, or an archive-only one in either mode".
The Acceptance line promises the listing for "Standalone `lyx webster reset`", for any target.
A pair-mode reset to start that moves the branch discards the run's tracked changes but leaves untracked files, including files the earlier run created, and the next run starts over them; D4's pair row says they stay, but the sentence naming where the listing applies leaves this case out.
The archive-only path, the one the focus asked about, is stated correctly.
**Fix:** State once which resets list left paths (every reset that ends with `--to start`, plus every standalone reset), and make D4 and the Acceptance line say the same.

### [LOW] The D7 Acceptance line pins fewer regions than D7

**Location:** `_lyx/discussion/decision-record.md:280`
**Issue:** D7's Decision, Bound and Scope "Out" agree that everything outside the Card Index is pinned: the frontmatter, the framing, `## verify:`, `## Shared Decisions` and `## Rename mechanic`.
I checked this against `planparser.splitFraming` (the index is the lines from `## Card Index` to the next `## `) and `loom-plan-spec.md` (the plan-level sections follow the index): the whole-file-minus-index hash covers all five, so the mechanism matches the Decision.
The Acceptance line lists only "frontmatter, framing or `## verify:` section", so a test written from it can leave the other two unpinned.
**Fix:** Reword the Acceptance clause to "a change outside the Card Index (frontmatter, framing, `## verify:`, `## Shared Decisions`, `## Rename mechanic`)".

### [LOW] D1's guard claim conflicts with D3

**Location:** `_lyx/discussion/decision-record.md:50`
**Issue:** D1: "Every target keeps `PlanReset`'s existing guards (merge in progress, foreign branch, missing commit, target not an ancestor of HEAD, foreign dirty paths)".
D3: the archive-only path of `--to start` drops the missing-commit, ancestor and dirty-tree refusals.
Read together, the two decisions contradict for the start target.
**Fix:** Add "except `start`'s archive-only path, D3" to D1's sentence.

## Settled focus points

- D7 guard region: matches what stays refused (F3 is an Acceptance wording gap only).
  The rationale states no bound, which is correct; the Decision, Bound and Scope "Out" state the same one.
- D7 begun-card index lines: the claim holds.
  `rebaselineBatches` compares `batchCardIDs`, which is `%02d-%s` of `Card.Number` and `Card.Slug`; both come from the index entry.
  The index line's intent is `Card.Summary`, read only by `RenderBatchIndex`, and the card file stays pinned by `CardHashes`, so the Bound's "number and slug pinned, intent not" is true and states what is let through.
  Not a defect.
- D6 re-edit of a rendered amendment: stated (`rendered` resets to false, a record with any unrendered entry is forced failed, a done record clears the list).
  Not a defect.
- Attack surface: D1 (held lock), D3 (archive-only), D4 (standalone keep-reset), D6 (forced failure and ladder rung), D7 (overview index), D9 and D12 each state what they let through and what bounds it.
  The one open bound is F1.

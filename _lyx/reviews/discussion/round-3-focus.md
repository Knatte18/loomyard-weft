---
round: 3
focus:
  - "D7 in `_lyx/discussion/decision-record.md`: settle whether the guard region it pins for a mid-run overview change matches what stays refused. Check against `planparser.splitFraming` in `internal/planparser/parse.go` (framing is the prose between the title and `## Card Index`) and `contracts/specs/loom-plan-spec.md` (the integration verify is the plan-level `## verify:` section after the Card Index, beside `## Shared Decisions` and `## Rename mechanic`) that the frontmatter, `## verify:`, Shared Decisions and Rename mechanic are pinned, and that D7's rationale, Bound, Scope 'Out' and the Acceptance line state one and the same bound."
  - "D7 begun-card index lines: settle whether the mechanism that holds the index change (`rebaselineBatches` and `batchCardIDs` in `internal/websterengine/state.go`, which compare only `NN-slug` ids) can make the stated Bound and Acceptance claim about a begun card's index line true, or whether the claim is narrowed to number and slug or backed by a named mechanism."
  - "D3 and D4 reset trees: settle whether D4's per-mode list states the tree left by every reset path D3 defines, including the archive-only path that drops the dirty-tree refusal and moves nothing, in pair and standalone mode, and whether the envelope's listing of left paths is stated for it."
  - "D6 amendment record: settle whether the decision states what a rebaseline does when the named card already has an `AmendedCards` entry with `rendered: true` (a recovery live on the first amended text, then a second edit), so the recovery's done record cannot clear a mark while the code follows older text than the plan carries."
  - "Attack surface of the new guard paths: for each verb, flag or routing edge a Decision in the record introduces or loosens (the overview index landing in D7, the archive-only reset in D3), settle whether the Decision states what it can now skip or let through and what bounds it."
---

Rationale: the first focus entry targets the gating defect class; the others follow the open findings through their decisions.

---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md#decisions (Decision 1)
    summary: Decision 1 counts the gap after a recorded interrupt marker as working, though the marker ends the turn and task-run transcripts carry it
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md#decisions (Decisions 2, 4, 7)
    summary: The record names no rule for whether orch transcripts, including orch+sub forks charged wholly to a run, enter the idle union
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md#decisions (Decision 8)
    summary: Decision 8 widens the weft default with no stated bound on reading a slug against the wrong hub's weft; only the failure case is named, under Open risks
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md#decisions (Decision 3) and #open-risks
    summary: The earliest-transcript-line span start is the common case for archived runs, and for a reused slug it escapes the clip Open risks relies on
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md#decisions (Decision 9)
    summary: Decision 9 leaves the shedrun helper's shape open and states neither run-id validation nor what a seed or status that fails to decode does
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md#auto-mode-assumptions and Decision 6
    summary: Auto-mode assumptions restate Decisions, and Decision 6's rationale re-argues the rejected start-timestamp alternative
---

### [MEDIUM] Interrupt markers end a turn but Decision 1 counts the gap after them as working

**Location:** `_lyx/discussion/decision-record.md`, Decision 1; Auto-mode assumptions bullet 1; Open risks bullet 2.

**Issue:** Decision 1 makes every interval whose first line is a user line working time.
When a session is interrupted, Claude Code writes the last in-flight assistant line with `stop_reason: null` and then a user line whose text is `[Request interrupted by user]`.
That user line ends the turn, but under Decision 1 the interval from it to the session's next line counts as working, however long the session then sits.
This is not rare: across the hub's task-run project directories, 65 transcripts outside the prime carry an interrupt marker, and in the sampled prime transcript every `null`-ending assistant line followed by a user line was followed by this marker.
The record treats the case as an unavoidable risk ("counts the gap until its next line as working"), but the marker is recorded, so the rule can end the turn there without a threshold.
The other first-line kinds are determined: lines without a parseable timestamp drop out, equal timestamps give zero-length intervals, ordering is per transcript before the union, and the Auto-mode assumption and Open risks agree with each other and with the Decision's text.
Only the interrupt line is misclassified.

**Fix:** In Decision 1, treat a user line whose text is the interrupt marker as a turn end, like an assistant line whose `stop_reason` is set and is not `tool_use`, and add an acceptance criterion for it.
Narrow the Auto-mode assumption and Open risk to what stays undetectable: a crash with no marker.

### [MEDIUM] No rule for orch transcripts in the idle and role-time unions

**Location:** `_lyx/discussion/decision-record.md`, Decisions 2, 4 and 7; Scope "Out".

**Issue:** Decision 4 computes idle over "every transcript's working intervals in the run" and says idle holds parent review.
For tokens, the tool charges an orch sub-agent whose first prompt names exactly one run slug wholly to that run, and the orch's review forks are such sub-agents.
A plan writer reading "transcripts in the run" through the tool's existing attribution can include them, which turns parent review into working time and contradicts Decision 4's own definition of idle.
The record says only that the orch row's time cells are blank (Decision 7, Scope "Out"); it never says the orch session and its sub-agents stay out of every union.
With orch excluded, Decision 7's "union of working time plus idle is the span" holds per run, since both are computed over the same transcript set and clipped to the same span, and it holds in "All runs" as long as each run's union is computed per run before summing.
Neither condition is written down.

**Fix:** Add to Decision 4: the union covers only the run's own worktree transcripts and their sub-agents; the orch session and every orch sub-agent, including one charged wholly to the run for tokens, stay out, so parent review counts as idle.
Add to Decision 7 that "All runs" sums per-run unions and idles, never a union across runs.

### [MEDIUM] Decision 8 widens the weft default without stating what it lets through

**Location:** `_lyx/discussion/decision-record.md`, Decision 8; Open risks bullet 4.

**Issue:** The weft default now applies whenever `-weft` is empty, which is a routing edge the rubric's attack-surface question covers.
The Decision states neither what the edge can now let through nor what bounds it.
With `-hub` naming another hub and no `-weft`, each named slug is looked up in the current worktree's weft sibling.
A slug absent there falls silently into the no-status fallback, and a slug that exists in both hubs reads the wrong hub's status, seed and span with no warning.
The one consequence the record names, a named-slug run outside a resolvable worktree now failing, sits only under Open risks, not in the Decision.

**Fix:** State in Decision 8 what the default can now let through (a slug read against the current worktree's weft, not the `-hub`'s) and its bound: for example, the default applies only when `-hub` is also defaulted, or a named slug with no archive tag in the resolved weft is reported by name.
Move the resolvable-worktree failure into the Decision as part of that bound.

### [LOW] The transcript-line span start is the common case and escapes the clip

**Location:** `_lyx/discussion/decision-record.md`, Decision 3; Open risks bullet 3.

**Issue:** `Seed.StartedAt` landed on 2026-10-10, and 82 of the 108 archive tags in the prime's weft carry a seed without `started_at`, so Decision 3's fallback to the earliest transcript line is the usual case for archived runs, not an edge.
Open risks bullet 3 says clipping to the archived span keeps a reused slug's older sessions out.
Under the fallback the span start is itself the earliest transcript line, so a reused slug's older session moves the span start back and the clip no longer bounds anything.
No archived slug in the prime's weft has more than one tag today, and in the sampled runs the earliest line precedes the first history entry's end, so `elapsed sums to the span` holds and no interval goes negative; the record states neither condition.

**Fix:** In Decision 3, bound the fallback start so an older run's lines cannot set it: for example, the earliest line of a session still active at the first history entry's end, or that end itself when no line precedes it.
Correct Open risks bullet 3 so it claims the clip only where the span start does not come from the transcripts.

### [LOW] Decision 9's helper is open and unbounded

**Location:** `_lyx/discussion/decision-record.md`, Decision 9.

**Issue:** "a helper for bytes read from git or for a run id without alias resolution" leaves the choice between two shapes to the plan writer.
The run id comes from a tag ref name, which is input from outside the tool, and `PATTERN-shed-run-directory` requires `ValidateRunID` before any run id is joined onto an anchor; the Decision does not carry that bound into the new helper.
No archive tag in the prime's weft is `self`-only, so dropping the legacy `self` fallback loses nothing today, but the record does not say the helper drops it.
The existing seed decoder rejects unknown keys and `status.json` is decoded into `shedengine.Status`; the record does not say whether an archive whose seed or status fails to decode aborts the report or takes the no-status fallback.
The bound against a second decoder holds as written, since decoding stays inside `shedrun`.

**Fix:** Name one shape in Decision 9: a `shedrun` decoder over seed bytes and a run-id-relative path pair for `seed.json` and `status.json` that validates the run id and applies no alias.
State the decode-failure behavior and add it to the acceptance criteria.

### [NIT] Restated assumptions and a re-argued alternative in the decision record

**Location:** `_lyx/discussion/decision-record.md`, Auto-mode assumptions; Decision 6.

**Issue:** Auto-mode bullets 2, 4, 5 and 6 restate Decisions 2, 4, Scope and 10 without adding a requirement.
Bullet 1 carries the one requirement not elsewhere, the interrupted-turn rule, which F1 moves into Decision 1.
Decision 6's rationale argues what a start timestamp would have bought, which is the support log's rejected alternative restated in the record.
Completeness-before-leanness: the rule "no start timestamp is added" and the note that `Publish` and `Finalize` show as whole rows stay in Decision 6; only the counterfactual narrative moves.

**Fix:** Cut the restating Auto-mode bullets, and reduce Decision 6's rationale to the fact that history entries already record a finish time per row, leaving the counterfactual to the support log's Rejected alternatives.

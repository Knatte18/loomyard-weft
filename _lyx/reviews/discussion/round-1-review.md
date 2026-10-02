---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md — Decisions 9 and 10
    summary: The done-only fold reverts a newer status on any entry that was already done in the legacy file at migration, contradicting its own rationale
  - id: F2
    severity: LOW
    location: _lyx/discussion/decision-record.md — Decision 7
    summary: The tier rule refuses a dependency on a done entry at a higher tier, which blocks nothing and serves no purpose
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md — Acceptance criteria
    summary: Committed defaults have no check - new entry without tier gets 3, without type gets feature, abandoned renders in its tier and survives prune
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md — Decision 13
    summary: The hidden notes aliases change behavior (list and remove reach every entry, a retried promote-note is refused) and the record does not say so
  - id: F5
    severity: LOW
    location: _lyx/discussion/support-log.md — Interview turn 7, Rejected alternatives, Question ledger item 14
    summary: The support log still records hard line breaks as the pick after the hub review replaced Decision 21
  - id: F6
    severity: NIT
    location: _lyx/discussion/decision-record.md — Goal
    summary: The never-name-it-Bolt rule is a binding constraint sitting in Goal with provenance the support log already records
---

Both artifacts were read in full against the rubric, the writer's stencil, and the board code they describe (`internal/boardengine/store.go`, `internal/boardcli/cli.go`).
Each focus directive is answered below; none was departed from.

### [MEDIUM] The legacy done-mark fold can revert a newer status

**Location:** `decision-record.md`, Decision 9 (rationale) and Decision 10; `support-log.md`, Rejected alternatives "Fold every legacy status...".

**Issue:** Decision 9 folds every legacy `done` entry whose slug is in the store with another status, on every load, and claims this "cannot ... revert a newer status".
That holds only for entries that were not yet `done` when migration ran.
An entry already `done` in `tasks.json` at migration stays `done` in the frozen legacy file until `retire-legacy`.
If the new binary later moves it to any other status — `set-status` clears or sets status freely (`Store.SetStatus` in `store.go` takes any `*string`), so reopening or marking it `abandoned` is a normal write — the next load folds it straight back to `done`, and the next write persists the revert.
The same happens to a new entry created later under a slug that was `done` in the legacy file.
The fold is meant to absorb exactly one kind of event — an old binary marking done after migration — but as specified it re-applies every historical done mark forever.
No acceptance criterion catches this.

**Fix:** Make each legacy done mark apply once.
`board.json` records the slugs whose legacy done mark it has already absorbed (seeded at migration with every legacy slug already `done`);
the fold applies only legacy done marks not yet recorded, records each one it sees whether or not the slug is present, and `retire-legacy` drops the record.
Correct the rationale, Decision 1's shape, and add an acceptance criterion: an entry `done` at migration and later set to another status keeps that status across loads.

### [LOW] The tier rule should exempt done dependencies

**Location:** `decision-record.md`, Decision 7.

**Issue:** Decision 7's stated purpose is that "a tier-1 entry is never blocked by something not yet concretized".
A `done` dependency blocks nothing, yet the rule as written refuses promoting an entry whose finished dependency sits at a higher tier, or demoting a done entry below its dependents.
Done entries keep their tier (Decision 18 renders them under Done "whatever their tier"), so a finished tier-3 `design` entry that a feature depends on would force a pointless promotion of the finished entry.

**Fix:** State that a dependency whose status is `done` is exempt from the tier comparison, and add the exemption to the validation acceptance criteria.

### [LOW] Acceptance criteria miss committed defaults

**Location:** `decision-record.md`, Acceptance criteria.

**Issue:** Decision 3 commits a new entry without `tier` to tier 3, Decision 4 a new entry without `type` to `feature`, and Auto-mode assumptions place `abandoned` entries in their tier (not under Done) and keep them through `prune`.
None of these has a check: Validation covers only refusals, the README golden test does not name `abandoned`, and the `prune` criterion says only what it removes.
These are exactly the observable behaviors a plan could get wrong without a test noticing; the tier-3 default is also an Open risk the record names.

**Fix:** Add criteria for the two creation defaults, for an `abandoned` entry rendering in its tier section with its status tag, and for `prune` leaving `abandoned` entries in place.

### [LOW] Hidden aliases change behavior silently

**Location:** `decision-record.md`, Decision 13.

**Issue:** The old `notes` verbs operated on `notes.json` alone: `notes list` listed only notes, `notes get` and `notes remove` found only notes (`b.ListNotesBrief`, `b.GetNote`, `b.RemoveNote` in `cli.go`).
Run as the top-level verbs over one store, `notes list` returns every entry and `notes remove` can delete a tier-1 entry.
Separately, the old `promote-note` was idempotent on retry, while Decision 14's `promote` refuses an equal target, so a retried `promote-note` on an entry already at tier 1 now fails.
Decision 13 implies the first change only through "over the same store" and does not mention the second; the plan writer has to infer both and the help text the aliases carry.
The envelope key is not a concern: the old `notes get` already returned `{"task": ...}`.

**Fix:** State in Decision 13 that the aliases see the whole store, and decide the retry case — `promote-note` on an entry already at tier 1 succeeds as a no-op, keeping the old idempotence.

### [LOW] Support log contradicts Decision 21

**Location:** `support-log.md`, Interview turn 7, Rejected alternatives "Render each body line as its own paragraph", Question ledger item 14.

**Issue:** The hub review replaced Decision 21 (hard breaks out, body unchanged), and its entry says so, but ledger item 14 still reads "Resolved, auto-pick: hard line breaks in the rendered design doc only".
Ledger items 1 and 17 mark their hub-review renames; item 14 does not.
The rejected alternative's rationale ("hard breaks keep paragraphs grouped") argues for the option the hub rejected, and hard breaks themselves are missing from Rejected alternatives.
The interview turn is history and may stay, but unmarked it reads as the current pick.

**Fix:** Mark turn 7 and ledger item 14 as replaced by the hub review with the current resolution, add hard line breaks to Rejected alternatives with the hub's reason, and rewrite the per-line-paragraph entry's rationale without endorsing hard breaks.

### [NIT] Naming rule belongs in Constraints

**Location:** `decision-record.md`, Goal.

**Issue:** "never the name of the tracker, its store, its skill or its README" is a binding rule on every name the plan writer picks, sitting in Goal behind a provenance tag "(operator, 2026-10-02)".
The support log's Hub review entry already records that provenance.
The completeness-before-leanness test passes for the move: the rule itself is extracted into Constraints, only the provenance leaves.

**Fix:** Move the rule to Constraints without the provenance; Goal keeps a short clause saying the module is "board", not "Bolt".

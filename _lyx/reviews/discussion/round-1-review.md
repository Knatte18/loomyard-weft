---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D10
    summary: import and close take a bare issue number, so a pull request number, an already closed issue or a board-recorded open issue can be commented on and closed; the Bound names only the repo
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D10
    summary: the recorded-and-open retry leaves open which slug the comment names, whether a hand-recorded D9 issue is closed, and whether a retry repeats a comment already posted
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D10
    summary: the fold target's allowed kind and status, and the fate of title, brief and labels when into is given, are not stated
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:D12
    summary: D12 rationale says every hub migrates without an operator step, but D3 refuses every later write, including status writes, to a migrated entry whose area label is not in that hub's config
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D6
    summary: refusing an unknown --label filter stops the operator from listing entries that carry a label since removed from config, the entries D3 says must be relabelled; the path-only Board case is unstated
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D12
    summary: migration can yield duplicate labels, which D3 refuses, and the type-name token drop does not say whether legacy or configured type names are meant
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D8
    summary: the Bound does not say what path or stdin the flag reads, or what happens when the payload itself arrives on stdin
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D9
    summary: upsert of issues does not say whether it replaces or appends, nor whether duplicates are refused
---

### [MEDIUM] Intake mutations are unbounded by issue state and type

**Location:** `decision-record.md`, D10 Bound.
**Issue:** `import` and `close` take a bare number.
GitHub's issue endpoints accept a pull request number, so `close` could comment on and close a pull request, although `list` excludes them.
`close` also has no stated check for an issue that is already closed or that a board entry records.
The Bound says only that no repo but the inbox is touched.
**Fix:** state that `import` and `close` refuse a pull request and an already closed issue, that `close` refuses an issue a board entry records, and add a matching acceptance criterion.

### [MEDIUM] The retry rule is under-specified

**Location:** `decision-record.md`, D10 Bound; Open risks.
**Issue:** The comment names "the entry's slug", but for a retry the payload slug may differ from the recording entry.
An issue recorded by hand through D9 and still open is closed by `import` with a completed comment, which the operator may not intend.
A retry after a comment that succeeded and a close that failed posts the comment again.
**Fix:** a retry names the recording entry's slug and ignores any other payload slug; it is reachable only by an explicit `import` of that number, and `list` never offers it; the comment is skipped when one naming the slug already exists.

### [MEDIUM] Fold semantics are open

**Location:** `decision-record.md`, D10.
**Issue:** `into` may name a done entry or a task, and the record does not say whether either is allowed.
`title`, `brief` and `labels` beside `into` are neither used nor refused.
**Fix:** `into` accepts any entry of either kind and status, validated like any write under D3; `title`, `brief` and `labels` beside `into` are refused.

### [MEDIUM] Migration promise contradicts write-time validation

**Location:** `decision-record.md`, D12 Rationale against D3.
**Issue:** Only loomyard gets its area list (D13).
In any other hub a migrated entry with a prefix label is refused on every write, including a claim or done transition mid-run.
"Without an operator step" is therefore false there.
**Fix:** state in D12 that a hub with area labels needs a `board.yaml` edit the refusal names, and that the refusal applies to status writes too.

### [MEDIUM] Unknown-label filter hides orphaned entries

**Location:** `decision-record.md`, D6.
**Issue:** D3 leaves an entry holding a removed label refused until relabelled, but `list --label <removed>` is refused as unknown, so the operator cannot select those entries.
With a path-only Board there is no config to check against.
**Fix:** a filter label is refused only when it is in neither config list and no entry carries it; a path-only Board does not validate the filter.

### [LOW] Migration label edge cases

**Location:** `decision-record.md`, D12.
**Issue:** A prefix token repeating an added label (for example `[undecided]` on a `design` entry) yields duplicates that D3 refuses.
"A type name" does not say which names.
**Fix:** drop duplicates, keeping first occurrence; type names are the legacy values and the configured types.

### [LOW] `--body-file` read bound

**Location:** `decision-record.md`, D8.
**Issue:** The Bound covers validation only.
**Fix:** the flag reads exactly the named path or stdin with the operator's own permissions; it is refused when the payload also comes from stdin.

### [LOW] `issues` write semantics

**Location:** `decision-record.md`, D9.
**Issue:** Replace or append is unstated; D3 states it for `labels`.
**Fix:** `issues` replaces the whole list on upsert, and duplicates are refused.

## Relocation and exclusion

None found: Decisions carry Decision and Rationale only, and the deliberation sits in `support-log.md`.
Checked against the acceptance criteria: the `Other` section matches D7, and the `tier` refusal matches D1 and D4, since `upsertAllowedKeys` rejects unknown keys.
D2 against D12 raises no finding: the old tier rule forbade a task depending on a note, so migration cannot produce one, and a migrated note's `depends_on` is emptied.

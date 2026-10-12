---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:85 (D4 severity rule) and :97 (D5 severity rule)
    summary: The severity marker D4 places mid-sentence in the fork boilerplate's last line takes a noun phrase, but D5's Plan severity value is a multi-clause directive, so the Plan fork line renders garbled
    fix: In D4, state that the severity rule marker replaces only the parenthesised phrase a severity (`BLOCKING`, `MEDIUM` or `LOW`) so Discussion keeps its bytes, and that every host value must read as a noun phrase in that slot. In D5, rewrite the severity rule as such a phrase, for example a severity (`BLOCKING` when the plan as written fails its verify or ships a wrong behavior, `MEDIUM` when a decision, criterion or branch goes unreached or untested, `LOW` otherwise), with the evidence as the `path:line` or glyph read, and move report defects only, never a passed check into a separate host value or into the Plan spawn point of D3. Add an acceptance criterion that the twill Plan prompt's fork boilerplate last line carries the Plan severity rule as one well-formed sentence.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:211-212 (Acceptance criteria for fabric-vocabulary-name)
    summary: D9's refuse and pass lists leave rule branches unreached - the doc-root branch of the bare-token rule, the `cmd/` branch of the host-geometry rule, and the host-geometry pass outside `internal/` and `cmd/`
    fix: In the refuses criterion add a member with a side token and no test prefix in a package under a doc root such as `cmd/`, and a host-geometry identifier under `cmd/`. In the passes criterion add a host-geometry identifier in a package outside `internal/` and `cmd/`, such as one under `tools/`.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:56 (D1 rationale)
    summary: D1's rationale rests on a false premise - a present loom.yaml lacking a key does not yield the zero value, because configengine fills missing template keys in memory, and discussion_producer, which D1 claims to mirror, refuses an empty value
    fix: Rewrite D1's rationale to say an absent key takes the template's `single` through configengine's in-memory fill, and that an explicit empty value reads as `single`, unlike `discussion_producer`, which refuses it; keep the decision and its criterion.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:198 (Open risks, fork give-up)
    summary: The give-up risk says the writer lands an unreviewed plan, but Plan-Review still reviews it, and no Decision ties the risk to that bound
    fix: Rewrite the risk as a fork giving up after 15 empty waits of up to five minutes each, against this repository's 120-minute plan timeout, leaves the plan without its lens review, recorded in `lens-review.md`; the Plan-Review segment and the plan gates still run on it. Add the same sentence to D2's Bound line.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:134 (D8)
    summary: D8 names a routing edge - a worktree's burler.yaml redefining `twill-plan` changes the lenses - without stating what that lets through and what bounds it
    fix: Add a Bound line to D8 - a redefined fan changes only the lens texts the forks review through; the fan resolver refuses an empty fan, one past its maximum size and one naming an undefined lens, and the forks stay read-only per D2.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:92-102 (D5) and _lyx/discussion/support-log.md:301 (Q9)
    summary: The requirement that D5 and D8 carry the bench's host and lens texts verbatim lives only in the support log's Q9, which the plan writer never reads
    fix: Add one sentence to D5 and one to D8 stating that the listed texts are rendered as written, since the bench file they come from is unreadable to the plan writer.
---

### [MEDIUM] Plan severity rule does not fit the marker slot D4 defines

**Location:** `_lyx/discussion/decision-record.md:85` and `:97`

**Issue:** The fork boilerplate's last line in `contracts/stencils/twill/twill-directive.md` reads "Return your findings only as your final message, each with a severity (`BLOCKING`, `MEDIUM` or `LOW`), the section or decision it concerns, a one-line summary and the evidence."
D4 makes "a severity (`BLOCKING`, `MEDIUM` or `LOW`)" the host value, and the acceptance criteria require Discussion's prompt to change only on the reviewed-files line, so the slot stays mid-sentence.
D5's Plan value starts "report defects only, never a passed check; …" and ends "evidence is the `path:line` or glyph read".
Substituted into that slot it yields "each with report defects only, never a passed check; `BLOCKING` when …, the section or decision it concerns, …", which no fork can parse as one instruction.
The other two new values fit their slots: the spawn point opens the procedure and the rejection rule is a whole sentence in step 6.

### [MEDIUM] D9's criteria leave rule branches unreached

**Location:** `_lyx/discussion/decision-record.md:211-212`

**Issue:** D9's bare-token rule fires on a package "under `internal/` or a doc root of the scan", and its host-geometry rule fires "under `internal/` or `cmd/`", which matches the scan's `hostScope` in `internal/lyxcwd/enforcement_test.go` (`!isTest && (inInternal || first == "cmd")`).
The refuse list reaches only the `internal/` half of each branch, and the pass list never shows a host-geometry identifier outside its scope.
A plan could implement both rules for `internal/` alone and meet every criterion.
The other branches agree with the rule text: the test prefix, configsync's identifier rule, `Rename` `New` handles, the `Type.Method` last segment and the unparsable handle all have a row.

### [LOW] D1's rationale rests on a false premise

**Location:** `_lyx/discussion/decision-record.md:56`

**Issue:** `internal/configengine/config.go`'s `load` states "A present file's missing template keys are filled from the template in memory and logged, under both policies", so a `loom.yaml` written before the key loads `single` from the template without D1's empty-reads-as-`single` rule.
That rule only matters for an explicit `plan_producer: ""`, which `discussion_producer` refuses (`internal/loomengine/config.go:726-730`), so "mirrors `discussion_producer`" is also wrong on this point.
The decision itself is harmless; the support log's Turn 3 adopted it on the same premise.

### [LOW] Fork give-up risk overstates what lands

**Location:** `_lyx/discussion/decision-record.md:198`

**Issue:** The fork boilerplate gives up after 15 `ready: false` waits, each run with a 300000 ms timeout, so about 75 minutes, against the tracked `plan_timeout_min: 120`.
The risk says the writer "lands an unreviewed plan", but the Plan-Review perch reviews every plan whatever the lens review did, and D2's bound already names the plan gates.
The give-up is recorded only as a risk with no bound stated beside it.

### [LOW] D8's redefinable fan has no stated bound

**Location:** `_lyx/discussion/decision-record.md:134`

**Issue:** D8 states that redefining `twill-plan` in a worktree's `burler.yaml` changes the lenses, a routing edge the rubric's attack-surface rule requires a bound for.
The bound exists in code, not in the record: `burlerengine.ResolveFan` refuses an unknown fan, an empty fan, one past `maxClusterN` and one naming an undefined lens.

### [LOW] Verbatim requirement for D5 and D8 texts lives only in the support log

**Location:** `_lyx/discussion/decision-record.md:92-102` and `_lyx/discussion/support-log.md:301`

**Issue:** Q9 says the round-2 directive is readable only from the hub prime's `.scratch`, and that D5 and D8 carry its texts verbatim.
`Plan-Write` never reads the support log, and D5 presents its host values as semicolon-joined lists without saying they are the text to render, so a plan writer may paraphrase them.

## Focus verdicts without a finding

- D5 and Goal: the rejection rule matches the Goal's BLOCKING-or-MEDIUM scope, and the twill prompt criterion names the plan host's rejection rule, so dropping it fails a criterion.
- D2: fork authorization states its bound (forks read-only and without the Agent tool by stencil, the plan gates bound what lands).
- D6 and D7: both state their bounds; the criteria cover the index check, fingerprint, prior-plan block, rotation and commit. `stalePlanRotation` moves every top-level `.md`, so D7's rotation needs no new enumeration.
- D10: the bound is stated; its leaf row and the "only the predicates" rule are review-enforced, as `PATTERN-leaf-packages` is for every leaf row. Keeping the skipped roots (`_lyx`, `.lyx`) in the test keeps the new package clear of `PATTERN-lyxdirs-single-declarer`.
- D4: `{{.reviewed_files}}` occurs once in the directive stencil, so the Discussion difference is bounded to one line, and the missing-value criterion covers the three new values.
- Relocation: no deliberation narrative in the record needs moving; D4's parenthetical justifies host-specific values rather than re-arguing a rejected alternative.

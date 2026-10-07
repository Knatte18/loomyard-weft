---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:10
    summary: Goal says the plan gate refuses any card ordering a weft sibling path or fabric spelling, but D13 scans the spelling only in Verify, verify section and fenced blocks, and its Bound does not list the inline span and prose pass-through as a miss
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:177
    summary: D13 Bound says the check does not look at absolute paths or dot-dot escapes, but the sibling-name rule it refuses matches exactly those spellings, including the motivating relative weft path of the card-batching incident
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:207
    summary: Constraints omit PATTERN-fabric-vocabulary, which bars the weft and warp tokens from planparser, its tests, the plan spec row, the template steer and websterengine tests that D13 to D15 require
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:174
    summary: D13 claims the audit's exact-path branch matches nothing the separator-prefixed name does not, but the branch has no name-end anchor and matches a longer sibling such as a task worktree whose name extends the weft name
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:149
    summary: D11 keeps findings of multi-profile and restarted runs out of every profile line but does not say what the line's findings per card divides by, so it can mix a subset numerator with the line's own card count
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:87
    summary: D5 says never a multi-card batch over budget after a resume, which is a statement about the estimate and reads stronger than the in-fork drift the Open risks admit
---

### [MEDIUM] Goal states the plan gate's guarantee without D13's bound

**Location:** `_lyx/discussion/decision-record.md:10`, against D13 (lines 167–177) and Open risks (line 239).
**Issue:** The Goal says a card "can no longer order a command naming the weft repo's sibling path or a fabric-driving spelling without the plan gate refusing it".
D13 refuses the spelling only in a `**Verify:**` value, the `## verify:` section and fenced blocks.
A card that orders `` `lyx weft …` `` in an inline code span, or in prose, passes by design, and that is the usual way a card gives a command inside its instruction text.
D13's Bound lists what the matcher misses (`bash -c "…"`, variables, other path spellings) but not this scope exemption, and Open risks mentions only a path in an inline span being refused.
Attack-surface question: the new check is a guard, and the Goal presents it as total while the Decision bounds it.
**Fix:** Word the Goal's sentence to the bound: the plan gate refuses a card whose `**Verify:**` value, verify section or fenced block orders such a command, and a weft sibling path also in an inline span.
Add to D13's Bound that a spelling in an inline span or prose passes the gate and the implementer audit is the guard for it.

### [MEDIUM] D13 Bound contradicts the sibling-name rule on absolute and relative paths

**Location:** `_lyx/discussion/decision-record.md:177`, against line 169 and Rationale line 179.
**Issue:** The Bound says the check "does not look at absolute paths or `..` escapes".
The refused form is "a name ending in the weft suffix after a separator", and the motivating case is `../card-batching-weft`, which is a `..` path; an absolute `/hub/x-weft` matches the same rule.
A plan writer reading the Bound can implement the check to skip any path with a leading `..` or `/`, which stops it refusing the incident that motivated it.
**Fix:** Say what is meant: the check refuses a path only when it contains a name ending in the weft suffix, and does not refuse an absolute path or a `..` escape that names no such sibling.

### [MEDIUM] Constraints omit PATTERN-fabric-vocabulary

**Location:** `_lyx/discussion/decision-record.md:207` (Constraints), against D13, D14 and D15.
**Issue:** `TestEnforcement_FabricVocabulary` fails any file outside the owner set (`fabricengine`, `fabriccli`, `weftname`, `gitkit`, `hubforge`, `boardengine`, `configsync`) that carries the bare token weft or warp, beyond the closed `vocabExempt` spellings.
The scan covers every `_test.go` in the module, non-test Go under `internal/`, and `.md` and `.yaml` under `contracts/`.
D13 to D15 put exactly that vocabulary where the scan looks: a check in `internal/planparser` and its fixtures (a sibling-path card, `lyx weft`, `lyx warp` spellings), the `card-fabric-reference` row in `loom-plan-spec.md`, the committed-data steer in `loom-template-plan.md`, and `websterengine` tests for the reader list over weft paths.
Only `lyx weft sync`, `lyx warp checkout` and `lyx.exe weft push` are exempt, so the acceptance cases as worded cannot be written as literals in those files.
The record names the constraint's neighbours (`PATTERN-cwd-resolution`, `PATTERN-fabric-git`) but not this one, so the plan writer meets it as a failing `go test` after the cards are written.
**Fix:** Add a Constraints entry for `PATTERN-fabric-vocabulary`: the weft and warp tokens stay on the fabric side; `planparser`, its tests, the spec row and the template steer say "fabric repo" and build weft-named inputs through the fabric side's matcher or exempt helpers, as the existing audit tests do.

### [LOW] D13's exact-path equivalence is not exact

**Location:** `_lyx/discussion/decision-record.md:174`, against `internal/fabricengine/refscanner.go` (`NewRefScanner`).
**Issue:** The record says the audit's exact-path branch "matches nothing the separator-prefixed name does not, so the audit's matches stay unchanged".
The exact-path alternative is the current weft worktree's full path with no name-end anchor, so it also matches a longer sibling whose name extends the weft name (a task worktree `x-weft-push` next to a worktree `x`), which the separator-prefixed alternative rejects on purpose.
Dropping the branch to share only the location-free patterns would change the audit's matches in that case, contradicting the Constraint that the audit's matches do not change.
**Fix:** State that the audit keeps its exact-path branch, which needs the worktree location the plan check lacks, and that the shared rule is the three location-free patterns; drop the equivalence claim.

### [LOW] D11 leaves the findings-per-card denominator open

**Location:** `_lyx/discussion/decision-record.md:149`, with lines 145–150 and acceptance line 257.
**Issue:** Per-batch figures, including `cards`, go to each batch's own profile, so a multi-profile run adds its cards to every profile line it touches, while its findings and a restarted run's findings enter no line.
The line's "Webster-Review findings per card" then has a numerator over only the single-profile, non-restarted runs; the record does not say whether the denominator is those runs' cards or the line's `cards`, which includes the excluded runs' cards and would understate the figure.
**Fix:** State that the profile line's findings per card is the sum of findings over the sum of cards of the runs that contribute findings, and that the line shows how many runs that is.

### [NIT] D5's "never a multi-card batch over budget" is an estimate-level claim

**Location:** `_lyx/discussion/decision-record.md:87`.
**Issue:** After a resume the position is too high, so the estimate is conservative about the start; that does not rule out a measured batch over `budget`, which Open risks line 235 admits for the in-fork coefficients.
**Fix:** Word it to the estimate: "never a multi-card batch whose estimate exceeds `budget`".

## Focus verdicts

- D14 reader list: closed, eight named members (`cat`, `head`, `tail`, `ls`, `wc`, `grep`, `jq`, `cut`), joined only by pipes, `;` and `&&`, with output redirection, substitution and backgrounding excluded.
  None of the eight writes a file or runs a program under those limits, so the Bound's "no listed reader can write" holds. Not a defect.
- D14 Bound sentence on the batch's commits: it claims only that the start lies in HEAD's history, nothing is uncommitted and no listed reader writes, and says it does not inspect the commits. The stated evidence supports that. Not a defect.
- D13 inline span: the record gives a reason for the asymmetry (a span documenting a command opens with its spelling, a span naming a path is as likely an argument as a quotation). Settled; the leftover gap is the bound wording in F1.
- D13 shared rule: the record names the rule (separator-prefixed name, bare word, spelling), says the audit's flags do not change and that `planparser` receives a matcher through a seam, as `websterengine` does. Open points are F2 to F4.
- D11 multi-profile run: the contribution rule is stated and the acceptance criterion tests it; the open point is F5.
- D11 `--fresh` run: `--fresh` mints a new run guid, so more than one guid in the weft history identifies it; the record excludes it from every profile line and lists it in its own marked row. Settled.
- D11 card count with no recorded partition: sourced from the run's plan commit. Settled.
- Goal, D4, D5 guarantees: the over-budget lone card is shown by the batch's recorded `Estimate` and position (D4), and `accept-audit --batch` is scoped to the closed reader list; the two overstatements are F1 and F6.

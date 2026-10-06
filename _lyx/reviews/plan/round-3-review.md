---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/00-overview.md (prune-procedure step 1)
    summary: Step 1 says only card 19 rewrites the committed report, but the rerun and splice is card 20 after the split and renumbering.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/00-overview.md (ownership-boundary, prune-procedure step 6) and _lyx/plan/20-rerun-measure-after.md
    summary: The reason set is stated as exactly four classes, but the skills-in-one-turn keep reason matches none of them, so card 20's check rejects a keep the plan itself demands.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/00-overview.md (prune-procedure step 8)
    summary: The loop exit condition requires only that a candidate carries a keep directive, not a reason of a valid class, so arbitrary annotation can end the loop.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/16-git-geometry-prune.md
    summary: Card 16 bundles seventeen packages across fabric CLI, hub geometry, git access, filesystem, lock and process concerns, too many for one reviewer to judge a prune against.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/19-friction-leaf-prune.md
    summary: Card 19 calls batcher and shell friction or leaf packages and owns the leaf closed-set rule, while leaf packages gitkit and standalonestate sit in card 16 with no such rule.
---

### [BLOCKING] Stale card number in the prune procedure

**Location:** `_lyx/plan/00-overview.md`, `prune-procedure` step 1.
**Issue:** Step 1 says `-out` never points at the committed report, "which only card 19 rewrites (Decision 9)".
Card 19 is `friction-leaf-prune`; the card that splices into `docs/benchmarks/test-redundancy.md` is card 20 (`rerun-measure-after`).
A prune card that follows the stale pointer is sent to a card that does no splice.
**Fix:** Name the rerun card by its slug and its current number after any renumbering.

### [MEDIUM] The skill-loading keep reason is outside the closed reason set

**Location:** `_lyx/plan/00-overview.md` (`ownership-boundary`, `prune-procedure` step 6), `_lyx/plan/20-rerun-measure-after.md`.
**Issue:** Step 6 says the reason "is exactly one of four classes", and card 20 checks that each listed candidate's reason "is one of the four classes".
`ownership-boundary` and card 14 require a candidate among the skill-loading tests to be kept with `//testtiming:keep` naming `skills-in-one-turn`.
That reason is a parent addition to the decision record and fits none of the four Decision 10 classes: it pins no assertion, names no core file, receives no assertion, and is not a `Removable: no` covering-test tie.
Card 20 would flag a keep the plan requires, or a writer would stretch a class to fit.
**Fix:** State the skill-loading reason as a fifth, named class in step 6 and in card 20's check, bounded to the tests `ownership-boundary` lists.

### [MEDIUM] The loop's exit condition does not require a valid reason

**Location:** `_lyx/plan/00-overview.md`, `prune-procedure` step 8.
**Issue:** Step 8 repeats "until the scratch list shows only candidates that carry a keep directive".
Step 6 does define the closed reason set and says a candidate with none is deleted or folded, never annotated, but the exit test names only the directive.
A prune card reading step 8 alone can annotate a candidate with arbitrary text and exit, and the plan states the class check only at card 20, after every package card has committed.
**Fix:** Make step 8's exit condition "every listed candidate carries a directive whose reason is one of the classes of step 6", so the per-card loop enforces what card 20 later checks.

### [MEDIUM] Card 16 is too many concerns for one review

**Location:** `_lyx/plan/16-git-geometry-prune.md`.
**Issue:** The card targets seventeen packages: the fabric CLI and `hubforge` fixtures, hub and standalone geometry, `lyxcwd`, `pairteardown`, git access (`gitexec`, `gitkit`, `gitrepo`, `gitoracle`, `githubclient`), and `fslink`, `fsx`, `lock`, `proc`, `weftname`.
Its Intent already lists six separate invariants it must keep (`PATTERN-hubforge-fixtures`, `PATTERN-mutation-record`, `PATTERN-gitrepo-client-boundary`, `PATTERN-pair-teardown`, `PATTERN-test-isolation`, the windows-only files).
The slowest in-scope fixture-merge work (`fabriccli`) shares a card with pure unit-test packages, so one reviewer cannot judge the prune against one concern.
**Fix:** Split into a hub and geometry card (fabric CLI, `hubforge`, `hubgeom`, `standalonegeom`, `standalonestate`, `lyxcwd`, `pairteardown`, `weftname`) and a git and filesystem card (`gitexec`, `gitkit`, `gitrepo`, `gitoracle`, `githubclient`, `fslink`, `fsx`, `lock`, `proc`), renumbering the later cards, their files, `Commit:` subjects and every cross-card reference.

### [LOW] Card 19's package labels and the leaf rule do not match the leaf set

**Location:** `_lyx/plan/19-friction-leaf-prune.md`.
**Issue:** The card calls its packages "the friction packages and the leaf packages", but `batcher` and `shell` are neither (`PATTERN-leaf-packages` lists `gitkit`, `modelspec`, `tokenvocab`, `buildinfo`, `standalonestate`, `pattern`, `friction`).
It also holds the rule that leaf packages keep their closed import sets, while two leaf packages, `gitkit` and `standalonestate`, are in card 16, which states no such rule.
**Fix:** Name `batcher` and `shell` as the registry and seam packages in card 19, and carry the closed-set rule on each card that holds a leaf package.

## Focus departures

None.
Focus 1 is answered by F2 and F3: step 6 states the closed set and the delete-or-fold rule, but the exit condition and the skill-loading reason leave it open.
Focus 2: the Card Index, file names, `Commit:` subjects and `Applies to:` ranges agree today; F1 is the one stale reference, F4 the one card too wide, and card 18's CLI and IDE packages are a small cohesive group, not a defect.

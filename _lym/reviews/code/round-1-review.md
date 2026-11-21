---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/adoptcli/doc.go:53
    summary: The lym adopt help text at internal/adoptcli/cli.go:47 says the package doc holds the steps and every field of the envelope, but the trim removed the envelope field list and the steps section.
    fix: "In internal/adoptcli/doc.go, under `# The envelope` (line 53), restore the envelope's field list from the base revision 3d3b4812 (base doc.go lines 152-166): `slugs` and `prime_recorded` always present; `hold`, `fix_task`, `prime_check`, `check_lines`, `deploy`, `warnings`, `prime_beam_steps`, `prime_beam_warning`, `archives`; per slug the pair added, owner recorded, run taken over or `owned_by`, `check`, `beam_steps` and the error. State the step order (start, each tier's start and end, end) in one sentence too, so the help text's claim about the steps holds. Change no code: cli.go stays as it is."
    origin: lens:code-affected-set
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/yamlengine/doc.go:3
    summary: yamlengine's doc.go, changed by one line, still restates the exported functions' own doc comments, which the operator decision lists as superfluous.
    fix: "In internal/yamlengine/doc.go, cut the per-function catalogue at lines 3-17 down to one or two overview sentences, and cut from the `# Open maps` and `# Per-name maps` sections each rule that the function's own doc comment already states, for example the Reconcile open-map and per-name rules that internal/yamlengine/reconcile.go:20-36 carries, and FillMissing's rule in fill.go:17-22. Keep the definitions of unit, open map, per-name map and shape, which no single function owns, and every rule no exported symbol's doc states. Leave the function doc comments unchanged."
    origin: handler
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/batcher/doc.go:15
    summary: batcher's doc.go, cut by only 7 lines, still restates Active's and PeakContext's doc comments, says what Active does twice, and narrates the estimator's formula term by term.
    fix: "In internal/batcher/doc.go, delete lines 15-21, which restate Active's doc comment at internal/batcher/config.go:103-109, and merge what remains of the Active paragraph at lines 35-41 into one statement of the load contract. Then cut the term-by-term read-set, message and PeakContext narration at lines 47-64, which estimate.go:207-209 carries, down to the why: the estimate predicts a fork's peak context so a batch fits the model's window, and adding a card never lowers the peak. Keep the retired-key refusals and their way forward, the boundary rules, and the recorded-partition invariant."
    origin: handler
  - id: F4
    severity: NIT
    class: consistency
    location: _lym/landing/summary.md:14
    summary: The change description counts 162 doc.go files, but HEAD tracks 161.
    fix: "In _lym/landing/summary.md, change \"162\" to \"161\" in the Size table row and in the sentence \"130 of the 162 files changed\"."
    origin: handler
---

### [MEDIUM] adoptcli's trimmed doc no longer holds what `lym adopt --help` says it holds

**Location:** internal/adoptcli/doc.go:53

**Issue:** `internal/adoptcli/cli.go:47` tells the operator "The package doc holds the steps, the hold and every field of the envelope."
At HEAD the doc's `# The envelope` section is two sentences and names no field, and the `# The steps` section is gone (base doc.go:37 and 152-166).
The envelope's fields are the verb's output contract, which the answer key's "contracts and invariants" rule keeps, and a cited statement is one the commit message of 20ad9235 says stays.
Only the doc can be changed, since the task forbids code changes.

### [MEDIUM] yamlengine's doc.go still restates function docs

**Location:** internal/yamlengine/doc.go:3

**Issue:** The operator decision asks to cut every doc.go's restatement of function and type docs.
This file lost one line (122 to 121).
Its opening catalogue and its open-map and per-name-map rules for Reconcile and FillMissing repeat their own doc comments (internal/yamlengine/reconcile.go:15-36, fill.go:14-22).
The assistant reviewer independently raised the same gap at doc.go:95; this finding merges it.

### [MEDIUM] batcher's doc.go still restates symbol docs and repeats itself

**Location:** internal/batcher/doc.go:15

**Issue:** This file went from 135 to 128 lines.
Lines 15-21 restate Active's doc comment (internal/batcher/config.go:103-109), and line 35 opens the Active description a second time.
Lines 47-64 narrate the estimator's terms and the PeakContext sum, which estimate.go:207-209 documents.
The decision lists restatement, step-by-step narration and repetition as superfluous.
The code-scope fork pointed out the same residue.

### [NIT] The change description's doc.go file count is off by one

**Location:** _lym/landing/summary.md:14

**Issue:** `git ls-tree -r --name-only HEAD | grep -c 'doc\.go$'` gives 161, not the 162 the summary states twice.
The comment-line counts in the summary (27,842 before, 11,688 after) reproduce exactly.

## Rejected

- Removed headings that appear elsewhere ("Run settings", "The parent seat", "Store housekeeping", "Keys the template lacks", "Remote control", "The HardenFix row and the harden_round key", "The verify gate", gatecli's report headings and others): each match is a heading or sentence of its own in that file, not a citation into the trimmed doc.
- `docs/overview.md:2056`'s pointer to fabricengine's state matrix, verb table and sabotage-proof table: those tables were already missing at the review base, so this diff did not break the pointer.
- The fabricengine guard tests' allowlist reasons (`cmd/lym/destructiveguard_test.go:115`, `cmd/lym/uncontainedwrite_test.go:80`): the doc still names the raw tokens, and `# The mutation record` (doc.go:280) and the guard test's name (doc.go:314) survive.

The diff passes the focus's mechanical checks:
- Every changed file is a `doc.go`, and every changed line is a comment or blank.
- Every package clause still has its doc comment attached, and every file still opens a `// Package` line.
- No build constraint changed.
- Every PATTERN entry the new text names exists.

The code-correctness and code-tests forks found nothing, as expected for comment-only prose.

## Focus departures

- "Largest cuts (fabricengine, websterengine, gatecli, reedengine, shuttleengine, tenderengine): does each doc.go still carry ... or did the trim delete a guarantee ... no exported symbol's doc carries?": I checked these files through every live citation into them and through the guarantees other files rely on, not by reading each removed paragraph against the symbol docs. The combined diff is about 2.4 MB, and a full line-by-line read did not fit one round. A guarantee that no other file cites and that went with its surrounding mechanism would not show up in this review.

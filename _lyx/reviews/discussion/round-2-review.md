---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:163
    summary: The planned PATTERN-lyxdirs-single-declarer amendment calls the script's head variable block the one non-Go site spelling _lyx or .lyx, but recipes, template YAML, specs and stencils already spell them, so the amendment would land a false sentence in PATTERN.md
    fix: "In decision-record.md Constraints, first bullet, replace the sentence beginning \"The task amends that entry's line\" with one that drops the \"one non-Go site\" claim and states the actual exemption, e.g. \"The task amends that entry's line in `PATTERN.md`, in the same commit as the script, to admit a shell tool under `tools/` that cannot import `internal/lyxdirs`, provided it spells each literal once, in a variable block at the script's head.\" Keep `PATTERN.md` in Scope In. In support-log.md, Round 2 entry, record the premise check (non-Go tracked files such as `contracts/recipes/loom-recipe.yaml`, `internal/fabricengine/template.yaml` and the specs under `contracts/specs/` already spell `_lyx/`)."
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:33
    summary: D1 bounds what --work clearing can delete but not what --out's move-into-place can overwrite; an existing directory at --out receives the report inside it while the tool prints the --out path, and an existing unrelated file is replaced silently
    fix: "In decision-record.md D1, after the sentence \"The report is written to a temporary name beside `--out` and moved into place only on success, so a failed run leaves the previous report intact.\", add \"`--out` names a file: an existing directory at `--out` exits 1 naming `--out`, writing nothing; an existing file there is replaced.\" In Acceptance criteria, extend the unwritable-`--out` criterion with \"an `--out` naming an existing directory exits 1 naming `--out`\"."
---

### [MEDIUM] PATTERN amendment rests on a false "one non-Go site" premise

**Location:** `_lyx/discussion/decision-record.md:163` (Constraints, `PATTERN-lyxdirs-single-declarer` bullet)

**Issue:** The Constraint plans the amendment and Scope In lists `PATTERN.md`, so the focus question about planning the PATTERN change in the same task is answered yes.
The amendment's wording is the defect: it admits the script's head block "as the one non-Go site that spells those literals".
`git ls-files` shows tracked non-Go files that already spell `_lyx/` or `.lyx/`: `contracts/recipes/loom-recipe.yaml` (e.g. line 120, `_lyx/discussion/decision-record.md`), `contracts/recipes/darn-recipe.yaml`, `internal/fabricengine/template.yaml`, `internal/shuttleengine/template.yaml`, `.gitignore`, and specs and stencils under `contracts/specs/` and `contracts/stencils/`.
`internal/loomshed/planwrite.go:184` reads the invariant as covering "production path-construction context" in Go.
Landing the amendment as written puts a sentence in `PATTERN.md` that is false the day it lands, and a later reader would take those existing sites for violations.
The plan writer copies this wording, since the Constraint is the only source it reads for the amendment.

### [LOW] `--out` overwrite has no stated bound

**Location:** `_lyx/discussion/decision-record.md:33` and `:39` (D1)

**Issue:** The focus asks, for every Decision that deletes or overwrites a path named by an operator flag, what the operation can reach.
For `--work` the answer is complete: the `.forkaudit-work` marker gates every deletion, deletion is limited to the tool's own named intermediates, and the root, hub, `--out`-directory and ancestor refusals plus the `--out`-inside-`--work` refusal sit on top; a directory that is empty or marked cannot hold the repo root, the hub or the report, so the bound holds.
For `--out` D1 says only that the report is moved into place on success and that an unwritable `--out` exits 1.
With an existing directory at `--out`, a move places the report inside it under the temporary name while the tool prints `--out`'s path, so the printed path is wrong; an existing unrelated file at `--out` is replaced with nothing in the record saying so.
Replacing a file is ordinary `-o` behavior and is fine once stated; the directory case needs a refusal.

## Verdicts on the focus questions with no finding

- Batch resolution (D7): the search is restricted to the fork's own run's refs, `<run>-weft` local or under `origin/` and `archive/<run>/*`.
  Checked against the substrate: `glyph-audit-tool-weft` is a worktree of `loomyard-weft`'s store, which holds every pair's `<slug>-weft` branch, its `origin/` copy and the `archive/<slug>/<sha>` tags, and an archive tag's tree carries `_lyx/plan/` cards.
  Within one run, `internal/websterengine` runs batches one at a time (its `doc.go` holds parallel execution as the board entry `webster-parallel-execution`), so a run's own batches cannot share a spawn window.
  Not a defect.
- Run names (D2): matching is exact against `<encoded hub>-<run>`, and a prefix never matches.
  The allowed set, lowercase letters, digits and `-`, excludes `.`, `..` and `/`, and Claude Code's encoding leaves it unchanged; the project directories under `~/.claude/projects` for this hub all have the form `-home-knatte-Code-loomyard-LYXHUB-<slug>`.
  Not a defect.
- Trace values as arms (D3): the bound is `[a-z0-9_-]+`, not a marker arm name, under a `trace:` prefix that already keeps it apart from classifier arms; a value outside the bound leaves the fork `unknown` and is listed once.
  Acceptance criteria cover `raf` and `|`.
  Not a defect.

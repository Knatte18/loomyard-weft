---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D2/D3
    summary: D3 carry and --set under an open map are undefined when the existing value is a legacy list while the template is a map
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D3
    summary: D3 does not state what MissingKeys now lets through for a wrong-shaped open-map value or what a mistyped --set label name does
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D7
    summary: D7 leaves open whether carried issues pass through validateWrite and what cross-entry duplicates mean
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D9
    summary: D9 states no precedence when an argument is both a subcommand and a module, and nothing for bare config with --set or --print
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:Scope
    summary: Scope In omits the picker removal (D9, A1) and the allowlist deletion (D11) that the Decisions carry
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:R3
    summary: R3 repeats the burler maps already named under Scope Out
---

### [MEDIUM] Legacy list under an open map has no defined reconcile or --set behavior

**Location:** `decision-record.md`, D2 and D3 (focus 2).
**Issue:** D2 says nothing writes the list shape and reconcile carries it unchanged "(D3)".
D3 carries the existing value at P whole, so `reconcile --apply` keeps a list-shaped `labels` and does not clash with the map template; that half holds.
`--set labels.x=...` on such a file is not covered.
D3 says `Set` accepts any key `P.<name>`, but under a sequence there is no mapping to add a key to.
The record leaves open whether `Set` refuses with a way forward, converts the list to a map, or errors opaquely.
R1 and R2 do not cover it, and no acceptance criterion pins it.
**Fix:** add to D3 (or D2) the outcome for `Set` under a list-shaped P, and an acceptance criterion for it.
Refusing with a message that tells the operator to rewrite the value as a map keeps D2's decode-only bound intact.

### [MEDIUM] D3 does not state what an open map now lets through

**Location:** `decision-record.md`, D3 (focus 1).
**Issue:** D3 states the declared-paths-only bound and scalar-only `Set`.
It does not state two things the open-map rule lets through.
First, `MissingKeys` is satisfied by the presence of key P "whatever its value", so `labels: foo` or `labels: {a: {b: c}}` passes config load; D1 says validation checks names only and never says the decode refuses a non-map, non-sequence value or a non-scalar description.
Second, `--set labels.quarrry=...` silently creates a new valid label instead of reporting a typo, which is the exposure the focus names; the bound ("declared paths only") limits where, not what.
**Fix:** state in D3 that a typo under P creates a label, accepted by design, and state in D1 or D3 that `boardengine` decode refuses a P value that is neither a map nor a list and a non-scalar description, naming `board.yaml` and the key.

### [LOW] D7 leaves validation order and cross-entry duplicates open

**Location:** `decision-record.md`, D7 (focus 3).
**Issue:** `validateWrite` refuses a non-positive or duplicated issue within one entry, and the current `MergeTasks` validates the patched incoming before removal.
D7 dedupes, so the within-entry rule cannot trip on the carry, but the Decision does not say the carry happens before `validateWrite` so the validated entry is the final one.
`validateWrite` does not enforce uniqueness across entries.
A carried number can therefore sit on a third entry that already records it, and D7's rationale that intake "sees each issue recorded by exactly one entry" holds only for the removed entries' numbers.
**Fix:** state that the carry is applied before validation, and either bound the rationale to the removed entries or say a number also recorded by an untouched entry is carried as-is.

### [LOW] D9 leaves precedence and flag combinations open

**Location:** `decision-record.md`, D9 (focus 4).
**Issue:** the refusal fires for an argument that is "neither a subcommand nor a `configreg` module".
The Decision does not say what happens when an argument is both, for example a module later named `reconcile`; today the subcommand wins in cobra routing, but the record is silent.
Bare `lyx config --print` and `lyx config --set k=v` still reach `dispatch` and resolve a cwd, and D9's "resolves no cwd" covers only the bare listing.
**Fix:** say the subcommand wins and a module cannot share its name, and say bare `--print` stays on its present path.

### [LOW] Scope In omits two Decisions

**Location:** `decision-record.md`, Scope (focus 6).
**Issue:** the In list carries "the CLI tree exceptions still recorded", which covers D9 and D10 in part.
The picker removal (D9, A1) is a behavior removal not named in Scope, and the findings-allowlist deletion (D11) is not named either.
The picker removal is defensible, since the bare listing contradicts a picker and the support log records the alternative, but a removal should be a Scope item.
D5's absent-target outcome is stated (exit 1 against empty stdout, exit 0 for an empty body), so a caller can tell the two apart; that point needs no change.
The D4 claim "reads `board.yaml` only" is consistent with R4: the load goes through `configengine.Load`, which D3 extends.
**Fix:** add the picker removal and the allowlist deletion to Scope In.

### [NIT] R3 repeats a Scope Out bullet

**Location:** `decision-record.md`, Open risks R3 (focus 7).
**Issue:** the Out bullet already says burler's `lenses` and `fans` have the same reconcile exposure and are not asked for.
R3 restates that and adds deliberation ("one line each") that carries no requirement for the plan writer.
**Fix:** delete R3, or cut it to the one fact the Out bullet lacks.

## Premise checks (focus 8)

- `cliTreeFindings` holds exactly the `config#bare`, `config#bogus` and `selfreport#bogus` entries in `cmd/lyx/clitree_test.go`; the premise holds.
- No `notes` group or `promote-note` verb remains in `cmd` or `internal`, and `ee1ffa4b`'s message says they were removed; the premise holds.
- This worktree's `_lyx/config/board.yaml` holds `types` and `labels` as lists; the premise holds here.
- `lyx config` bare reaches `menu` through `dispatch` after cwd resolution, as D9 describes.

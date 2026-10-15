---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:163-165 (Constraints, first bullet)
    summary: the planned PATTERN amendment only adds a shell exception, so the amended line would still say no other production file spells the literals, which tracked non-Go files already do
    fix: In the first Constraints bullet, replace the sentence beginning "The task amends that entry's line" with the amended line's intended content, scoped to what the invariant governs. Write it as "Naming `_lyx` or `.lyx` in a Go path - only `internal/lyxdirs` declares those literals, and no other Go production file spells them; a shell tool under `tools/` that cannot import it spells each literal once, in a variable block at the script's head." Keep the `-weft` sentence unchanged. Keep the Scope In line as is.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:39-40 (D1, report write and --out)
    summary: D1 says an existing file at --out is replaced and that a temporary name beside --out is used, but not what else that write can reach, how the temporary name is chosen, or what bounds the replacement
    fix: In D1, after "an existing file there is replaced", add that the write reaches exactly two paths, the file named by `--out` and one temporary file created with `mktemp` in `--out`'s directory (so it never takes the name of an existing sibling), the latter removed on failure; that nothing outside `--work` is otherwise created, replaced or deleted; and that `--out` is not checked against tracked or foreign files, its bound being the explicit flag and the caller's own permissions, as with a shell redirect. Add `mktemp` to the list of required commands in D1. Add an acceptance criterion that a failed run leaves no temporary file beside `--out`.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:38 (D1, --work refusal list)
    summary: the refusal checks (equals or ancestor of the repo root, hub or --out's directory; --out inside --work) do not say whether paths are compared canonically, so symlinks or `..` can slip past them with only the marker rule left as the bound
    fix: In D1's third bullet under the work directory's clearing, state that all these comparisons use canonical paths (`realpath -m`, symlinks and `..` resolved, a not-yet-existing path resolved through its nearest existing ancestor), and that the marker rule is the bound if a comparison is ever wrong, because a directory without the marker is never cleared. Add `realpath` to the required commands. Add an acceptance criterion that a `--work` given as a symlink to the repo root, and one spelled with `..` to reach `--out`'s directory, each exit 1 naming `--work` and delete nothing.
---

### [LOW] The planned PATTERN amendment leaves the line false for non-Go files

**Location:** `_lyx/discussion/decision-record.md:163-165`, Constraints, first bullet.
**Issue:** The focus question is whether the amendment states only true things.
The Constraint's own sentences are true: the script does read `_lyx/webster/state.json`, `_lyx/plan/` and `.lyx/logs`, and no tracked file under `tools/` spells those literals today, so the head block would be the first shell site there.
The round-2 repair also removed the false claim that the head block is the one non-Go site, and the premise it replaced is confirmed: `contracts/specs/*.md`, `contracts/stencils/**`, `internal/fabricengine/template.yaml` and `tools/sandbox/*.md` spell `_lyx/` or `.lyx/`.
What remains is the amended `PATTERN.md` line itself.
The current line reads "no other production file spells them", and the Constraint only appends a shell exception to it.
A reader of the amended line would take the shell tool as the sole exception, which the tracked non-Go files contradict.
`PATTERN-lyxdirs-single-declarer` governs path construction in Go production code, and no scan enforces it over non-Go files, so the amendment should say that scope instead of adding a second exception to a sentence that is already too wide.
This is a wording gap, not a design error, which is why it is LOW.

### [LOW] `--out` replacement and its temporary file have no stated reach or bound

**Location:** `_lyx/discussion/decision-record.md:39-40`, D1.
**Issue:** D1 handles the two named cases correctly: an existing directory at `--out` exits 1 naming `--out` and writes nothing, and an existing file is replaced.
It does not say what the replacement can reach.
It can replace any file the caller can write, including a tracked source file, with no bound beyond the explicit flag.
That is acceptable for an operator-named output path, the same as a shell redirect, but the attack-surface rule asks the Decision to say it, as D1 does for `--work` with its marker.
The temporary name "beside `--out`" is a second written path, and its name, its collision behavior against an existing sibling and its removal on failure are all unstated.
A fixed name such as `<out>.tmp` would silently overwrite or leave behind a sibling file, and the current acceptance criterion only checks that the previous report survives.
Naming the two reachable paths and the `mktemp` choice closes this without adding a guard.

### [LOW] The `--work` refusal checks do not say how paths are compared

**Location:** `_lyx/discussion/decision-record.md:38`, D1.
**Issue:** `--work` is the one flag that deletes, and D1 bounds it well: the `.forkaudit-work` marker, deletion of named intermediates only, and a refusal list.
The refusal list compares `--work` with the repo root, the hub and `--out`'s directory, and tests whether `--out` lies inside `--work`.
The record does not say the comparison is on resolved paths.
A lexical comparison misses a symlink to the repo root or a `..` spelling, and the guard is then bypassable.
The marker rule still prevents harm in the repo-root case, since the root is non-empty without the marker, so this is not BLOCKING.
But the rubric asks a bypassable guard to state what bounds it, and the record leaves the bound implicit.
Stating canonical comparison, and the marker as the backstop, makes the refusal list testable.

## Rejected

- The `--work` clearing bound itself (marker, empty-or-marked, named intermediates only) is sufficient for what it can delete; no finding.
- Absence of a restriction on which existing file `--out` may replace, for example only a prior report, is not raised as a finding; F2 asks only that the reach and its bound be stated.
- The `.scratch/` default paths are not a `PATTERN-lyxdirs-single-declarer` literal and need no amendment.

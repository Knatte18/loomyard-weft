---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: PATTERN.md:12
    origin: lens:code-affected-set
    summary: PATTERN-hub-containment one-liner lacks the declared workspace-file clause and disagrees with its pattern/ background
    fix: 'In PATTERN.md line 12, replace the entry text with the declared text "Linking hub-level containers or writing the hub workspace file: from the hub only, never into a worktree." keeping the background link; if PATTERN.md then exceeds its headroom line, shorten another entry the task already touched rather than this one.'
  - id: F2
    severity: LOW
    class: consistency
    location: PATTERN.md:31
    origin: lens:code-pattern-fit
    summary: PATTERN-fabric-write-containment one-liner is a shortened variant of the declared text
    fix: 'In PATTERN.md line 31, set the entry text to the declared text "Writing under `_launchers`, or the hub workspace file, from `fabricengine`: via an `os.Root` at the hub, never raw `os.MkdirAll`, `os.WriteFile` or `fslink`." subject to the same headroom check as F1.'
---

### [MEDIUM] PATTERN-hub-containment one-liner lacks the declared clause

**Location:** PATTERN.md:12
**Issue:** The task declares a change to this entry, so the entry is judged by the declared text, which adds "or writing the hub workspace file".
The merged one-liner still reads "Linking hub-level containers: from the hub only, never into a worktree."
`pattern/PATTERN-hub-containment.md` carries the workspace-file bullet, so the overview line and its background disagree.
The parent's only `PATTERN.md` edit (`PATTERN-shed-recipe-registry`) merged cleanly beside the task's edits, so the gap is not a merge artifact; it sits in the shared file the focus directive asks about.

### [LOW] PATTERN-fabric-write-containment one-liner is a shortened variant of the declared text

**Location:** PATTERN.md:31
**Issue:** The line reads "Writing `_launchers` or the workspace file from `fabricengine`: ... never raw `os.MkdirAll`/`os.WriteFile`/`fslink`."
The meaning matches the declared text, but the declared wording is the one the entry is judged by.
Commits `268316623` and `ca54dd5df` shortened it, apparently for the file's size headroom.

## Rejected

- The parent's `Batchifier` removal against task code: no task code, doc or test references the removed row; remaining mentions are batcher config vocabulary or the contract's note that the row is retired.
- The parent's `PATTERN-shed-recipe-registry` change against task code: the task registers and looks up no producers, and `Names` still exists.
- Test economy: the parent's and the task's tests cover disjoint behaviors in disjoint packages, and the two sides never call each other, so no redundancy or uncovered interaction exists.
- Task portal and containment rework against new parent callers: the parent touches none of `fabricengine`, `ideengine`, `hubforge` or `fslink`.

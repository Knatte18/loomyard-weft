# Round 3 fixer report

All edits are in `_lyx/discussion/decision-record.md`.

## Changed

- F1: Decision 2 now says a `Removable: no` candidate is never deleted and is folded or kept with a directive; Decision 10 gains a fourth reason class for it.
- F2: Decisions 1, 2 and 4 protect a core-named test and a receiving test from being folded or merged away; a fold or merge may use one as the host under its own name.
- F3: Decision 2 and the acceptance criteria require the PR to list every deleted candidate with its receiver or "no unique assertion", and each receiver to exist in the final tree.
- F4: Decision 12 defines the figure: `-covermode=set`, statements covered over total, non-test files directly in the package's own directory, exact match.
- F5: Decision 9's no-tests clause is removed.
- F6: Decision 5 fixes the comment spelling `// tiermove: needs shared fake <name>`; an acceptance criterion checks it and the PR entry.
- F7, F8, F9: the added scenario decision is now Decision 13 in the numbered form, with bounds (stop on a failed step, state-changing steps named, alone-rerun on the whole scenario, assertions and protected tests kept, per-family before and after measurement listed in the PR); Decision 7 names merges; an acceptance criterion covers merged families.

## Deferred

None.

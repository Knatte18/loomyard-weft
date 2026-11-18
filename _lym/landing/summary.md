# Comment trim second pass: measurement of what the rewrite left

This change makes no edit to the repository.
The board entry asks for a check before any new comment pass, so this description reports the check and a recommendation.
Figures come from the tree at the task's base commit, non-test `.go` files under `internal` and `cmd`.
A comment line is a line whose first non-blank characters are `//`; code lines exclude blanks and comments.

## Did the rewrite cover `doc.go` files?

Yes, partly.
Of the `doc.go` files under `internal` and `cmd`, 89 were modified since 2026-11-08, in 124 commits.
Most of those commits were feature work that appended to the doc, not trims.
The files are still long: together they hold 27,301 comment lines, about a quarter of all comment lines in production code.

The longest `doc.go` files, with the package's non-comment code lines beside them:

| Package | `doc.go` lines | Package code lines |
| --- | --- | --- |
| `fabricengine` | 2474 | 22826 |
| `websterengine` | 1284 | 7827 |
| `gatecli` | 1273 | 9079 |
| `reedengine` | 1174 | 7147 |
| `shuttleengine` | 1075 | 8830 |
| `shuttleengine/claudeengine` | 865 | 6565 |
| `loomcli` | 861 | 14325 |
| `boardengine` | 859 | 7347 |
| `creelengine` | 841 | 6440 |
| `shedadapters` | 753 | 5891 |
| `orchengine` | 752 | 2757 |
| `landingshed` | 678 | 5396 |

By doc-to-code ratio, `orchengine` (27 %), `dobbyengine` (16 %) and `landingshed` (13 %) stand out; `fabricengine` is the largest in absolute lines.

## Comment share overall and by package

Production code holds 103,762 comment lines against 268,614 code lines: a 27.9 % comment share.
Among packages with at least 300 code lines, the highest shares are:

| Package | Comment lines | Code lines | Share |
| --- | --- | --- | --- |
| `gitexec` | 248 | 317 | 43.9 % |
| `shedengine` | 1148 | 1523 | 43.0 % |
| `shellguard` | 251 | 346 | 42.0 % |
| `mergeresolve` | 224 | 356 | 38.6 % |
| `shuttleengine` | 5501 | 8831 | 38.4 % |
| `reedengine` | 4297 | 7148 | 37.5 % |
| `logger` | 363 | 623 | 36.8 % |
| `shedrun` | 526 | 925 | 36.3 % |
| `loomrecipe` | 364 | 639 | 36.3 % |
| `orchengine` | 1545 | 2758 | 35.9 % |

The shrink study's 20 to 31 % band for the seven most patched modules holds: `websterengine` 32.8 %, `landingshed` 32.2 %, `burlerengine` 32.9 % and `shedadapters` 31.9 % sit at its top, and several smaller packages exceed it.

## Recommendation

A narrow second pass on `doc.go` only is worth doing.
The rewrite did not shrink the package docs, and the largest ones restate what the function docs already say, as the study found for `landingshed`.
Start with the packages above by absolute `doc.go` size, `fabricengine`, `websterengine`, `gatecli` and `reedengine`, then `orchengine` and `landingshed` by ratio.
A pass over function and inline comments in `gitexec`, `shedengine`, `shellguard` and `mergeresolve` is a separate, smaller decision, since their shares are highest but their absolute counts are small.

## Design decisions

- The entry's decision record holds only the entry's body, with no `Added after Discussion` entries, no scope and no acceptance criteria; I treated the check as the whole task and made no trim.
- Any trim pass follows once the operator picks packages from the lists above; it needs its own board entry.

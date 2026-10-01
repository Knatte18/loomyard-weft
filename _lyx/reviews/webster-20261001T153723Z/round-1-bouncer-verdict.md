---
verdict: BLOCKING
rationale: "F1, F3, F4 and F5 are fixed and build, vet and both test suites pass at 085c17adb, but F2's fix is incomplete: grepOpensPager matches only an --open prefix, so git grep --op=CMD and --ope=CMD (unique abbreviations git accepts, verified to run CMD on git 2.53) still class as read-only, against its own doc's claim that any abbreviation counts"
---

# Webster-Review round 1 — judge verdict

## Range

`product.parent` is `main` in `_lyx/shed/refusal-audit/status.json`, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
The reviewer's report states the same range.
This verdict concentrates on cards 44–46, commit `8c4b50f2b` and the five fix commits `a235907f7`..`085c17adb`, since two earlier passes approved the range through card 43.

## The report

The round-1 report is sound.
Each of its five findings is a real defect against card 44's fail-closed intent or against `CONSTRAINTS.md`'s Comment Line-Break Convention, and its settled focus questions (the empty-command branch, the card 44 test rows, the card 46 Reason and blocked persist, the card 45 sync-failure test) hold up on re-reading.
It has no `## Focus departures` section.

## The fixes

Verified at HEAD:
`go build ./...`, `go vet ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./...` all pass.

- **F1** — fixed.
  `programName` trusts a path-spelled program only when `path.Dir` (which cleans the path) is a system bin directory, so `./cat`, `/tmp/tools/grep` and a `..`-escaping path are mutating.
- **F2** — incomplete; see below.
  The `-c` and `--config-env` branches are correct, and they sit before the subcommand so `git log -c` is unaffected.
- **F3** — fixed.
  `fi`, `done`, `for`/`select` headers, `[[` and `read` all class as read-only, and a loop body or word-list substitution that runs `rm` stays correctness.
- **F4** — fixed for every shape the report named.
  `skipOptions` skips each wrapper's value word, xargs gains the GNU long value options, and the shell branch stops at its first operand.
- **F5** — fixed.
  Every line the report named is now one sentence per line.
  The WebsterProducer bullet's first two lines in `shedadapters/doc.go` still wrap mid-phrase, but they are unchanged from `main` and therefore legacy that the convention forbids rewrapping.

## Blocking finding

### [MEDIUM] `git grep` pager abbreviations still fail open

**Location:** `internal/websterengine/audit.go`, `grepOpensPager`.
**Issue:** the function counts a long option only when it starts with `--open`, and its doc says "so does any `--open` abbreviation".
git's option parser accepts any unique abbreviation of `--open-files-in-pager`, and `--op` is already unique (`--o` alone is ambiguous with `--or` and `--only-matching`).
In a scratch repository on git 2.53, `git grep --op=./pg.sh hello` and `git grep --ope=./pg.sh hello` both ran `./pg.sh`.
A `go test -overlay` probe at HEAD (no tree file touched) reports `mutatingCommand` false for `git -C /fabric grep --op=./pg.sh state` and `--ope=...`, and true only for `--open=...`.
So the fix for F2 leaves the exact fail-open F2 named reachable through a shorter spelling, and its doc comment states a property the code does not have.
**Fix:** treat a long option as the pager option when the part before any `=` is at least `--op` and is a prefix of `--open-files-in-pager`;
correct the doc comment to match, and add correctness rows for `--op=` and `--ope=`.

## Card mechanical checks

Cards 44–46 carry only `Edit` and `Prosa` groups, so no `Rename` or `Delete` group check applies.
Card 44's Edit group deletes `mutatingSegment`, `editsInPlace` and `mutatingTools`, and a repo-wide grep at HEAD finds no remaining reference.

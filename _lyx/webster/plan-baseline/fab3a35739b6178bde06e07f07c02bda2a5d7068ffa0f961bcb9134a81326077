# Card 5 — git-reader-flag-rows

**Edit:**
- `internal/fabricengine/refscanner_test.go`

**Uses:**
- `internal/fabricengine#historyForm`
- `internal/fabricengine#gitForms`
- `internal/fabricengine#readOnlyGitRemote`
- `internal/fabricengine#readOnlyGitBranch`
- `internal/fabricengine#readOnlyGitSegment`

**Intent:**
D4: the exact-match flag lists and the two prefixes (`--format=`, `--pretty=`) in `refscanner.go` stay the bound, and a row per reader form pins that a writing, executing or config-touching flag is rejected.

The review against git's documentation finds no listed flag that writes, executes or touches config: the history flags format or select output, `ls-remote`'s select refs, `rev-parse`'s print values, `diff`'s and `status`'s shape output, `remote get-url`'s and `remote show -n`'s print names, the `branch` listing flags list, `cat-file`'s modes print object data, and `config --get` and `--get-regexp` read; `git status` may refresh the index's stat cache, which the audit documentation already records.
So this card changes no production code.
The implementer repeats the review while writing the rows and, if a flag is found that writes, removes it in this card and says so in the commit.

Rows of `TestIsReadOnlyCommand`, one per form where none is present, each rejected: `git show --output=x`, `git diff --output=x`, `git remote set-url o u`, `git branch -m a b`, `git config --get --file f k`.
The rows for `log` (`--output=x`) and `ls-remote` (`--upload-pack=x`) already exist and are not duplicated.

**ImpactSummary:** test rows only.

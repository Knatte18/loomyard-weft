---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/burlerengine/cluster_test.go:325
    summary: No row of TestCommandRunsMutatingGit pins the --format value skip in gitTagListsOnly, so dropping --format from gitTagValuedListOptions breaks nothing.
    fix: Add one row to the table in TestCommandRunsMutatingGit next to the --sort row, for example {"tag --format value starting with a dash is not mutating", "git tag -l --format -x", false}.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: scope
    location: internal/burlerengine/cluster.go:83
    summary: The -- branch of gitTagListsOnly with list mode already set has no row; the only -- row is git tag v1 -- --list, which takes the false path.
    fix: Add one row to the table in TestCommandRunsMutatingGit, for example {"tag list option before a double dash is not mutating", "git tag -l -- v1", false}.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: scope
    location: internal/burlerengine/cluster_test.go:274
    summary: The tag-writing options -u/--local-user, -F/--file and -e/--edit, and the long spellings --delete, --annotate, --sign and --force, have no row, so deleting one from gitTagWritingOptions breaks nothing.
    fix: Add rows to the table in TestCommandRunsMutatingGit that pair a list option with each untested writing option, for example "git tag -l -u key v1", "git tag -l -F msg.txt v1", "git tag -l -e v1" and "git tag -l --delete v1", each expected true.
    origin: lens:code-tests
---

### [LOW] No row pins the --format value skip

**Location:** internal/burlerengine/cluster_test.go:325
**Issue:** `gitTagValuedListOptions` names `--sort` and `--format`, but only `--sort` has a row.
Removing `--format` from the map leaves every row green.

### [LOW] No row pins a list option before `--`

**Location:** internal/burlerengine/cluster.go:83
**Issue:** `gitTagListsOnly` returns `listMode` at `--`.
The sole `--` row has `listMode` false.
A change to `return false` passes every row.

### [LOW] Tag-writing options without a row

**Location:** internal/burlerengine/cluster_test.go:274
**Issue:** One enumeration gap, not several: only `-d`, `-a`, `-s`, `-f` and `--message=` have rows, so the guard's refusal of the other entries in `gitTagWritingOptions` is unpinned.

## Rejected

- Abbreviated or unlisted writing options beside a list option (`--mess`, `--forc`, `--trailer`, `--create-reflog`) letting a tag be created: probed with real git, which rejects them or creates no tag.
- Exception leaking to other subcommands: the `subcommand == "tag"` guard confines it, and `branch` and `stash` keep the subcommand-only rule.
- Summary or `doc.go` claims not borne out by the diff: the option sets, the unchanged halt message and the fork-audit paragraph all match the diff.

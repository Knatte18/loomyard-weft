```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-premised findings' searches (F3, F4, F8, and where F5 and F6 touch docs) and check the fix covers every member.

**Findings**

- **LOW** — `internal/fabricengine/doc.go:1225`, "The hub pull" section. The `Bolt.Pull` sentence still says "runs the pull-first path a board write runs, so … reports the same skip and way forward".
  - F8 removed this wording from the `Bolt.Pull` and `PullHub` code comments, but its twin in `doc.go` is unchanged.
  - Evidence: `doc.go:1225` still has "the pull-first path"; the `bolt.go` comment now states the skip contract inline.
  - `doc.go` is the package-level doc, where the self-contained-comment rule does not bind, so this is only a cross-reference left in its old form.
- **LOW** — `internal/fabricengine/doc.go:1228-1229`. The doc says "a side that stops never keeps the next one from running" and lists stopped pairs, but says nothing new for the F3 and F4 behaviour.
  - Missing: a pull honoring SkipGit leaves every side up to date, and a beam whose origin record is unreadable is a stopped beam side instead of an error.
  - `PullHub` now reports both, and CLAUDE.md says a changed observable behaviour updates the module doc in the same commit.
  - The earlier doc text did not claim the old error behaviour, so nothing in it is wrong now.

**Searches that found no gaps**

- F4: `localBeamLocations` has no remaining reference in any `.go` or `.md` file. `localBeamCandidates` has a single caller, `PullHub`, which handles `beam.err`.
- F3: SkipGit is handled in `hubpull.go` only in `pullPairSide`, which returns before `Open`. `Bolt.Pull` already honored it, so both side kinds are covered, and the CLI layer never reads it.
- F8: no other comment in `hubpull.go` names `PullThenCommitWritten`, `Pull` or the pull-first path. The `pullPairSide` doc names none either.
- F6/F2: the moved, stopped-then-moving, SkipGit, bad-record and dirty-board/unreachable-weft rows each came from the fix.
  - The review's "beam owned by another machine" row was not added. `Origin` has no machine field, so there is nothing to pin.
- Unchecked: F1, F5 and F7 are not set-premised, so I did not search them.

**Tool calls**
1. A grep for `localBeamLocations` across `.go` and `.md` files, plus the `PullThenCommitWritten` / pull-first / `SkipGit` greps over `hubpull.go`, `bolt.go` and the CLI `hubpull.go`. My context lacked the post-fix state of those files, and this one call covered several searches.
2. `sed -n 1222,1245p internal/fabricengine/doc.go`, to read the hub-pull module-doc section, which was not in my context.

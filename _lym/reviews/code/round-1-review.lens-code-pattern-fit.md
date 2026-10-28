```yaml
lens: code-pattern-fit
outcome: reported
```

Task: judge the diff's fit with the repository's rules (PATTERN, CLAUDE.md), listing each rule I checked as held or broken.

**Rules checked**

- `PATTERN-parent-lock`: held. Commit `65d012cc` carries a `Pattern-Departure: PATTERN-parent-lock` trailer that names the same reason as the decision record. The pull takes the branch's lock for one pair only (`hubpull.go:134-138`).
- `PATTERN-fabric-git`: held. Git runs only in `internal/fabricengine` (`hubpull.go`, `bolt.go`). The CLI passes only the lock seam.
- `PATTERN-gitrepo-client-boundary`: held. The diff adds no raw git call. `acquireBranchLock` still goes through `gitrepo.New(...).CurrentBranch()`.
- `PATTERN-cli-cobra`: held. `--hub` is a flag on the existing `pull` subtree, not a new module.
- `PATTERN-cliwire-sole-wiring`: held. `runHubPull` takes the location the existing wiring resolved and adds no new resolution.
- `PATTERN-cwd-resolution`: held. `fabricengine` uses `lymcwd.NamedWorktree` and `PrimeName`, and no engine imports `lymcwd` newly in a way the pattern forbids.
- `PATTERN-pair-parent`: held. No parent is picked, and the beam list reads origin records via `ReadOriginFor`.
- `PATTERN-refusal-way-forward`: held. A stop's detail carries the way forward, and the error text says "each stopped side's detail names its way forward".
- `PATTERN-test-speed`, `PATTERN-test-isolation`: not checked against the test files. They are outside my 6-call budget and belong to the code-tests lens.
- Declared PATTERN changes landed: nothing to check. The diff to `PATTERN.md`, `pattern/` and `docs/` is empty, and the directive shows no declared changes.
- `CLAUDE.md` "Docs land in the same commit": held. `internal/fabricengine/doc.go` has the `# The hub pull` section and `internal/fabriccli/doc.go` has `# Hub pull`.
- `docs/overview.md`: held. The decision record says it changes only if a module row changes, and none does.
- `CLAUDE.md` "Docs land in the same commit", plain `lym fabric pull` unchanged: held. The non-hub branch in `weft_verbs.go` is untouched.
- Amendment focus (`fabricengine/doc.go`): held.
  - Parent paragraph: lines 1133-1134 carry the parent's change once. Line 1133 has "hub file splices"; line 1134 has the beam-file sentence.
  - Task section: `# The hub pull` sits under its own heading at line 1223, between the snapshot paragraph and `# Changing an anchor`.
  - No duplicate or dropped parent lines.
- Amendment focus (doc against code): held.
  - `fabricengine/doc.go`: its hub pull section agrees with `hubpull.go`.
    - Skip when no upstream: `hubpull.go:125`.
    - `SkipGit`: `hubpull.go:116`.
    - Unreadable origin record as a stopped beam: `hubpull.go:184-186`.
    - Records side unreachable as stopped: `hubpull.go:146`.
  - `fabriccli/doc.go`: its Hub pull section agrees with `hubpull.go` and `weft_verbs.go`.
    - Run id `fabric-pull`: `acquireBranchLock(..., "pull")` gives `"fabric-" + verb`.
    - Error envelope carrying all sides: `errWithRecordFields` with `fields`.
- Amendment focus (lock order): held. `Bolt.Pull` takes the board write lock and then `pullFirst` takes the push lock. The pair lock is released by a `defer` per pair and never held across sides.
- Amendment focus (shared lock seam): held. The `anchordeps.go` change adds a `verb` argument. The anchor path still passes `"anchor"`, so its run id `fabric-anchor` and wait notice are unchanged. The removed `anchorLockOwnerRunID` constant is no longer referenced in the diff, but I did not grep the rest of the repo for other users.
- Commit messages and departures: held. The only trailer names a real entry that forbids the plain way, and no other commit needs one.

**Findings:** none.

**Tool calls**
1. `git log 26c17888..HEAD` with bodies, plus a `git diff --stat` of `PATTERN.md`, `pattern/` and `docs/`. My context held no commit bodies or trailers and no diff of those paths.

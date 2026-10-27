```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the search behind each finding whose premise is a set. Only F5's premise (a read that makes a directory before `ReadJSON`) is a set, and the fix leaves its other members in place.

**Findings**

- **LOW** — `internal/loomengine/seed.go` (`VerifySeedOwnership`, `MkdirAll` removed). The fix covers only the one site the review named.
  - Eight other sites still call `os.MkdirAll(filepath.Dir(&lt;status lock path&gt;))` directly before `state.ReadJSON` on a status file:
    - `internal/beamcli/deps.go:96`
    - `internal/loomcli/resume.go:51`
    - `internal/loomcli/circling.go:222`
    - `internal/battencli/startguard.go:297`
    - `internal/battencli/wire.go:656`
    - `internal/tendercli/deps.go:297`
    - `internal/tendercli/deps.go:607`
    - `internal/hibernatecli/deps.go:128`
  - Each is a pure read that does nothing but read the status afterwards, and `ReadJSON` now reads without the lock when the lock directory is absent. So they are redundant in the same way, and they contradict the premise F5 states: "a read never creates a directory, anywhere", which `internal/state/doc.go` now says too.
  - Three sites carry a comment or a failure branch that depends on the mkdir:
    - `resume.go` has the comment "which a pair re-created from its branch lacks".
    - `wire.go` and `circling.go` have failure branches written for the mkdir.
  - Removing them there would need those comments and branches removed with them.
  - I could not tell whether the review excluded these on purpose. It scoped F5 to `VerifySeedOwnership` because it runs in the run's own worktree.

**Unchecked**

- F1 (other `LoadState` readers), F2, F3 and F4. Their premises are single sites or missing tests, not sets.

**Calls made**

1. Grep for `state.ReadJSON[` with 6 lines of context before each, filtered to `MkdirAll`, to find the sites that still make a directory before a read. My context did not hold them.
2. A `sed -n` loop over the eight hits, to confirm each is a pure read before `ReadJSON` and to see what depends on its mkdir.

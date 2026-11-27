```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, which traces each change in the diff to the board entry's goal points.

**Decisions** (the decision record alone is the answer key; it has no `Added after Discussion` entries)
- G1: "Worktree-Create retries a network failure of a remote lookup a few times with backoff before it blocks."
- G2: "the stop question reads an absent branch on a run whose Worktree-Create never succeeded as the normal fresh state."

**Changes and what they trace to**
- C1 traces to G1. `internal/fabricengine/add.go` adds `originProbeBackoffDelays` and `retryOriginProbeInTransit`, wraps the warp and weft probe goroutines in them, and updates the `BackoffSleep` and `Add` doc comments.
- C2 is forced by C1. `internal/fabricengine/add_remoteleftover_integration_test.go` adds `TestAdd_RetriesOriginProbesThatFailInTransit` and updates the file header.
- C3 traces to G2. `internal/battencli/drive.go` changes `primeShuttle`: before the pair exists it no longer resolves the task worktree, so the log no longer carries the `absentTaskWorktreeRefusal` text "durable status says it was already created". That text was the false premise.
- C4 traces to G2. `internal/battencli/drive.go` adds the fresh-state sentence to `battenTacklerRecipe` and `battenArbiterRecipe`.

**Decisions and the changes that build them**
- G1 is built by C1, with C2 as its test. C1 covers only the two pre-flight probes, and the issue's reported `ls-remote` of the weft branch is one of them. See finding 1 for the rest.
- G2 is built by C3 and C4.

No change traces to nothing, and no commit in the range carries a `Pattern-Departure:` trailer.

**Findings**

1. LOW · scope · `internal/fabricengine/add.go:288-307` (and the Worktree-Create closure, `internal/battencli/wire.go:1132-1217`)
   - Summary: G1 asks that Worktree-Create retry a network failure of "a remote lookup" before blocking. The diff retries only Add's two pre-flight probes. Worktree-Create's other remote lookups still return a transport failure on the first attempt:
     - `beamForkParent` calls `TaskBranchOnOrigin`, which reaches `ls-remote` through `originTaskBranch` and `firstOnOrigin`/`remoteHeadTip` (`wire.go:1179`, `pairbranch.go:238`).
     - `resolveWeftBranch` calls `remoteHeadTip` and then a fetch (`weftbranch.go:41`, `:54`), reached when the weft is live and only on origin.
     - Add fetches the adopted warp branch (`add.go:526`).
     - Add fetches to fast-forward the weft branch (`add.go:596`, `:604`).
   - On the reported case, a fresh task outside a beam, none of these run before the probes. So the issue's own failure is covered, and the limit is the goal's general wording.
   - Fix: wrap these lookups in the same retry, or state the narrower scope in the change description, which names only "Add's two pre-flight lookups".
   - Evidence: `internal/fabricengine/weftbranch.go:41-56`, `internal/fabricengine/pairbranch.go:233-247`, `internal/battencli/wire.go:1179`, `internal/fabricengine/add.go:526,596,604`, `_lym/landing/summary.md:10`.

Tool calls: none; the inherited context held everything this lens needed.

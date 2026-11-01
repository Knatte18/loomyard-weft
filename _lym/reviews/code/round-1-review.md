---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabriccli/anchordeps.go:258
    summary: The anchor move's restart loop and beamStart have no test; the testing.Testing guard returns before any of their logic runs
    fix: In internal/fabriccli/anchordeps.go split the label and argument choice (root for the prime, the slug otherwise) and the join of per-worktree failures into a helper that takes the process run as an injected function, and have beamStart pass the real exec call. Add one table-driven test in the fabriccli tests that drives the helper with a fake run and asserts the prime maps to root, a beam maps to its slug, a failure of one worktree does not stop the others, and the joined error carries each worktree's label and the child's output.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/orchcli/start.go:479
    summary: BeamUpWorktree's prime-versus-beam location choice and its hand-off to BeamUp are unpinned, as are the nil default of batten's pairStart and pawl's StartDobby wiring
    fix: In internal/orchcli/start.go move the location choice of BeamUpWorktree into a small function and test that it returns the prime for the prime's name and the resolved worktree for a beam, in a row of a test in internal/orchcli. The nil default of pairStart in internal/battencli/cli.go and the StartDobby wiring in internal/loomcli/pawlwiring.go need no test of their own once that function is pinned.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/orchcli/doc.go:11
    summary: The package doc still says BeamUp starts the dobby alone through dobbyengine.EnsureLive, which reads against the new PATTERN-dobby-orch-pair entry
    fix: In internal/orchcli/doc.go line 11 reword the sentence so the dobby start is described as the first step of the pair that BeamUp brings up with the runs strand and the orch, dropping the word alone.
---

## Verdict

The diff makes the change the amended board entry asks for.
Every non-test path that starts a dobby now goes through `orchcli.BeamUp`, `orchcli.BeamUpWorktree` or a `lym beam start` child.
The only remaining non-test callers of `dobbyengine.EnsureLive` and `SpawnDetached` are `BeamUp`'s own dobby step (`internal/orchcli/start.go`) and the `hubgeom.DobbyStart` seam it reads, so no path starts a dobby without its orch.
No finding is BLOCKING.

Checked against the focus directive:

- `BeamUpWorktree` matches `lym beam start` for the prime and for a beam: `lymcwd.ResolveWorktree` over `fabricengine.WorktreePath`, as `adoptcli` does, then `BeamUp` with `Prepare` false.
  `BeamUp` is idempotent when the dobby and orch are live, so calling it when the dobby is already live is safe.
  `dobbyStartError` returns nil for started and live and an error naming the worktree for refused, failed and unreported.
- `testing.Testing()` guards in production code are an established pattern in this repository (`orchcli/spawn.go`, `battencli/cli.go`, `adoptcli/deps.go` and others), and no PATTERN entry forbids them; the coverage cost is F1 and F2.
- `ensureParentDobby` runs the owner and landing-hold refusals before the pair start, and its errors still name `lym beam start`.
- The anchor move's `fail` path releases the parent lock and closes the change before it restarts dobbies, so the child's origin reconcile meets no lock; the success path was reordered to match.
  A child's failure comes back with its JSON error envelope, which `finishVerb` writes to stdout, in the error text.
- The new `PATTERN-dobby-orch-pair` entry covers the child-process route; the sibling `PATTERN-dobby-starts-runs` carries neither a `(test)` marker nor a background file, so their absence is no breach.

### [MEDIUM] Anchor move restart has no test

**Location:** internal/fabriccli/anchordeps.go:258
**Issue:** The loop over worktrees, the `root` label for the prime, `cmd.Dir`, and the child's output in the failure text are new behavior.
The anchor tests fake `EnsureDobbies` wholesale, and `beamStart` returns before reaching any of it under test.

### [LOW] BeamUpWorktree's location choice is unpinned

**Location:** internal/orchcli/start.go:479
**Issue:** `BeamUpWorktree` returns `ErrBeamUpUnderTest` at once in a test binary, so the prime-versus-beam resolution is never reached.
The nil default of `pairStart` and pawl's `StartDobby` wiring depend on it and are reached by no test either.

### [NIT] Package doc still says alone

**Location:** internal/orchcli/doc.go:11
**Issue:** The sentence says `BeamUp` starts the dobby alone through `dobbyengine.EnsureLive`, which reads against the new rule that a dobby never starts alone.

## Rejected

- Lens code-correctness F1 (an orch is left running when the dobby is refused or fails): a dobby that does not start only warns in `BeamUp` by its documented design, and `BeamUpWorktree` is specified as `lym beam start` does; the error it returns is the caller's signal.
- Lens code-correctness F2 (a lone dobby remains when the orch step fails): speculative and inherent in the dobby-first order `lym beam start` already has; the failure is reported to the caller.
- Lens code-affected-set F1 (new entry has no `(test)` marker or background file): the sibling entry `PATTERN-dobby-starts-runs` and others have neither, and the constraints document does not require them.
- Lens code-affected-set F3 (entry needs a carve-out for the `hubgeom` seam and `BeamUp`'s step): the entry names `orchcli.BeamUp` as the way, and its step is part of that route; the seam is wiring, not a start.

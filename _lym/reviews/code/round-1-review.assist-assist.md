---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/fabriccli/roleguard.go:45
    summary: Resolving the normal .lym junction changes the apparent owner to the records sibling and still refuses real hardener sandboxes.
    fix: Identify the owning code worktree before canonicalizing its sandbox root in inOwnSandboxRoot. Compare the canonical target with that canonical root instead of inferring ownership from the parent of the resolved .lym directory. Extend TestSandboxedRoleGuard with the normal code-to-records .lym junction and require admission through the mounted command tree.
    origin: lens:code-affected-set
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/fabriccli/roleguard.go:53
    summary: A sandbox under another hub's worktree with the same slug passes the ownership check.
    fix: Pass the originating session's owning worktree identity, including its hub, into the sandbox containment check. Compare the target against that worktree's canonical sandbox root rather than accepting any candidate whose basename equals the strand slug. Add a refusal row for two different hubs containing the same worktree slug.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/fabriccli/roleguard_test.go:238
    summary: The new admission table does not pin canonical path containment or relative clone destinations.
    fix: Extend TestSandboxedRoleGuard with literal expected outcomes for existing inward and outward symlinks, a symlinked owner root, traversal outside the root, a nested sandbox-shaped lookalike, and relative --into destinations inside and outside the allowed root. Use filesystem fixtures in the existing mounted-command test and assert admission or refusal; keep the production-junction regression with F1.
    origin: lens:code-tests
---

### [BLOCKING] Normal wired sandboxes remain refused

**Location:** `internal/fabriccli/roleguard.go:45`

**Issue:** The ordinary root-anchor layout makes the new exception reject the scenario the Goal requires.
`hardencli.resolveTarget` supplies the code worktree root at `internal/hardencli/cli.go:115`.
`hardenshed.SandboxRoot` appends `.lym/harden-sandbox` at `internal/hardenshed/sandbox.go:25`.
Fabric wires `.lym` into the records sibling through `internal/fabricengine/junction.go:46–52,202`.
Thus `/hub/task/.lym/harden-sandbox/container/hub/prime` resolves under `/hub/task-weft/.lym/harden-sandbox`.
The guard then resolves the enclosing `task-weft` worktree and compares its name with `task`, returning false.
Both ordinary verbs and clones into an existing sandbox directory retain the original refusal.

The filesystem inspection confirms this junction shape in the review worktree:
`.lym` links to `/home/knatte/Code/loomyard-LYMHUB/harden-sandbox-fabric-verbs-weft/.lym`.
The new test instead creates a real `.lym` directory in `sandboxHubDir`, so its passing rows miss this interaction.
The correctness and affected-set lenses independently reported this defect; their reports are consolidated here.

### [MEDIUM] A matching slug does not establish the originating worktree

**Location:** `internal/fabriccli/roleguard.go:53`

**Issue:** For `ly:slug:webster`, a target under another hub's `slug/.lym/harden-sandbox` passes the new check.
With a real `.lym` directory there, both comparisons succeed:
the resolved worktree is named `slug`, and its path equals the candidate path the traversal itself selected.
Neither comparison establishes that it belongs to the originating session's hub.
The guard passes only `name.Slug` at line 130 and discards the parsed hub shortname for this decision.

This is possible without changing the strand variable or fabricating a sandbox:
a worktree anchored in a subdirectory has its wired `.lym` at that anchor,
while the hardener's sandbox root remains at the worktree root.
See `internal/fabricengine/junction.go:51` and `internal/hardenshed/sandbox.go:25`.
Another hub can therefore have a legitimate root-level sandbox and the same task slug.
The Goal requires refusal outside the session's own sandbox, not admission to any same-named worktree's sandbox.

### [MEDIUM] Canonical containment and relative destinations lack regression rows

**Location:** `internal/fabriccli/roleguard_test.go:238`

**Issue:** The new table covers ordinary directory names and absolute clone destinations only.
No row distinguishes resolving an outward symlink from accepting its lexical sandbox prefix.
Removing the symlink resolution would leave the table's asserted outcomes unchanged.
The table also leaves the guard's relative `--into` interpretation unpinned.
The existing `TestCommand_CloneDestination` uses omitted or absolute `--into` values and runs without the parent guard.
These are behavior gaps in the new permission boundary, not requests for helper-level tests.
The production-junction fixture gap is included in F1 rather than counted twice.

## Other focus conclusions

- `cloneDestination` preserves the prior empty, absolute and relative calculation.
  The handler and guard call the same helper with the same context cwd and flag.
- Ordinary mutating verbs derive their hub from cwd.
  `pull --hub` changes scope within that hub; add/remove slugs and anchor arguments remain scoped to it.
  The existing detached-push and teardown-child exceptions precede the new branch and are unchanged.
  Reader and merge-task exceptions also preserve their previous order.
- Existing outward symlinks and paths resolving outside the sandbox are refused by canonical traversal.
  A failed worktree resolution refuses admission.
  A nested sandbox-shaped directory whose parent is not a matching worktree returns false immediately.
  An existing symlinked worktree root is canonicalized consistently when the entire target exists;
  the normal `.lym` junction remains the separate failure in F1.
- On a nonexistent target, the fallback ignores symlinks in existing ancestors.
  No escaping write was established: current clone probes require the destination directory to exist.
  `probeWeftBinding` creates its probe beneath cwd, and `readOriginLayout` runs git with that cwd.
- The refusal retains actionable hubforge and failed-run guidance.
  The negative rows use literal booleans for another worktree, a non-worktree lookalike and the outer worktree.
  They do not cover another hub with the same slug.
- The package documentation and stencil sentence explain the requested behavior and introduce no separate scope change.
  The Root-Cause trailer names the strand-only guard that the diff changes.
  No PATTERN changes were declared, and no assertion was loosened in the reviewed diff.

## Validation limits

Reviewed range: `a042662792534f728cc1069f23d9830755b71af8..b6b1bf3460c3be246c68f16ef6402d179ea78ce0`.
Read both supplied diff pieces, the range's commit messages, the decision record, board entry T8 and landing summary.
All lens reports were saved before consolidation.

Attempted `timeout 3600s lym gate test --wait --tier 0 ./internal/fabriccli -- -run '^TestSandboxedRoleGuard$' -count=1`.
The command exited before running tests because the gate could not write its holder record under `_bolt/.lym/gate`.
The filesystem sandbox reported a read-only filesystem there.
No test pass or live fabric mutation is claimed; findings rest on the code paths and read-only filesystem evidence.
No target file was changed.

## Rejected

- A separate escaping-write finding for a missing destination beneath a symlink is unsupported by the current clone path.
- The test lens's proposed real hubforge fixture is unnecessary for the filesystem-only guard regression.
  A fixture representing the normal junction can stay in the existing tier-0 behavior test.
- Duplicate reports of the normal `.lym` ownership failure are consolidated into F1.

---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/fabriccli/roleguard.go#inOwnSandboxRoot
    summary: In a real task worktree `.lym` is a link to the `-weft` records sibling's `.lym`, so EvalSymlinks lands the walk in `<slug>-weft/.lym/harden-sandbox`, ResolveWorktree names `<slug>-weft`, and the guard refuses every verb in the round's own sandbox hub, which is the case the Goal asks to let through.
    fix: "In inOwnSandboxRoot, judge ownership against the canonical sandbox root of the strand's own worktree instead of inferring the owner from the parent of the resolved `.lym`. One way: walk the target as written (cleaned, not resolved) to find the candidate `<worktree>/.lym/harden-sandbox`, require lymcwd.ResolveWorktree(worktree) to name the slug at that path, then require EvalSymlinks of the target (deepest existing ancestor, see F3) to lie at or under EvalSymlinks of that candidate root. A target already written in its resolved `-weft` form must also map back to its owner, for example through lymcwd.ResolveWithin's records-sibling mapping. In roleguard_test.go, make fakeWorktree's `.lym` a directory link (through internal/fslink) to a `<name>-weft/.lym` in a fake records sibling, as fabric wires it, so the OwnSandboxHub and IntoOwnSandboxRootFromItsWorktree rows run through the production layout."
    origin: assist:assist
  - id: F2
    severity: LOW
    class: design
    location: internal/fabriccli/roleguard_test.go#TestSandboxedRoleGuard
    summary: Several guard behaviours have no row that fails when they break, namely a relative clone `--into`, the `WorktreePath() == worktree` check, symlink resolution of an existing target, and the path-flag refusal staying ahead of the new sandbox pass.
    fix: "In the sandbox subtest of TestSandboxedRoleGuard add rows with literal expected outcomes for a clone with relative `--into .lym/harden-sandbox/hub-3` run from the own worktree (passes); a cwd at `<own>/sub/.lym/harden-sandbox/hub-1` (refused); a cwd reached through a link pointing into the own sandbox hub (passes) and a link inside the sandbox pointing out of it (refused); and `pull --weft-path /x` run from the own sandbox hub (refused with the path-flag refusal)."
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/fabriccli/roleguard.go#inOwnSandboxRoot
    summary: A target that does not exist yet is compared as written with no symlink in its existing ancestors resolved, so a worktree reached through a symlinked path refuses a legitimate `clone --into` its own sandbox, and a link planted inside the sandbox lets the lexical check pass for a destination that lands outside it.
    fix: In inOwnSandboxRoot, replace the EvalSymlinks-or-Clean fallback with resolving the deepest existing ancestor of dir through filepath.EvalSymlinks and re-joining the not-yet-existing tail, so the guard compares the directory the clone writes to; update the doc comment's sentence about a dir that does not exist yet to match.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/fabriccli/roleguard.go#inOwnSandboxRoot
    summary: Ownership is checked by worktree name alone, so a sandbox under a worktree of the same slug in another hub on the machine passes for this strand.
    fix: Pass the strand's hub identity into the containment check (the parsed name's hub shortname, or the own worktree's Location) and require the candidate worktree's hub to be the strand's own hub before admitting the target; add a refusal row with two hub directories holding a worktree of the same slug.
    origin: assist:assist
  - id: F5
    severity: LOW
    class: consistency
    location: internal/fabriccli/fabric.go:7
    summary: The fabric.go file header still says the guard refuses every mutating verb from a task session's strand name, but the guard now passes any verb aimed under the role's own sandbox root and reads the call's paths too.
    fix: Rewrite fabric.go line 7 to say the guard refuses a task session's mutating verbs unless the directory the verb acts in lies under its own worktree's sandbox root.
    origin: lens:code-affected-set
  - id: F6
    severity: NIT
    class: consistency
    location: internal/fabriccli/doc.go:124
    summary: The Merge section's line that the guard refuses these verbs from a task session names only the merge-task writer as an exception, while the merge verbs now also pass in a hub under the role's own sandbox root.
    fix: Change the doc.go Merge line to say the guard refuses these verbs from a task session outside a hub under its own sandbox root, keeping the merge-task writer sentence after it.
    origin: lens:code-affected-set
---

### [BLOCKING] The guard still refuses verbs in a real hardener's own sandbox hub

**Location:** `internal/fabriccli/roleguard.go#inOwnSandboxRoot`

**Issue:** In the hub layout fabric wires, a task worktree's `.lym` is a directory link into its records sibling.
Observed in this hub: `harden-fabricengine/.lym -> /home/knatte/Code/loomyard-LYMHUB/harden-fabricengine-weft/.lym`, and `readlink -f harden-fabricengine/.lym/harden-sandbox` gives `.../harden-fabricengine-weft/.lym/harden-sandbox`; this review's own worktree has the same link.
`inOwnSandboxRoot` resolves the target with `filepath.EvalSymlinks` first (`roleguard.go:45`), so the walk finds the sandbox root at `<hub>/<slug>-weft/.lym/harden-sandbox` and takes `<hub>/<slug>-weft` as the worktree.
`lymcwd.ResolveWorktree` then reports `WorktreeName` `<slug>-weft` (the records sibling has its own `.git` gitfile), which never equals the strand's slug, so the pass at `roleguard.go:130` never fires.
Every mutating verb run from a round's sandbox hub, and every clone into an existing sandbox directory, is still refused: the Goal's one requested change does not take effect.
The test misses it because `sandboxHubDir` makes `.lym` a plain directory inside the fake worktree.

### [LOW] Guard rows missing for relative --into, the worktree-path check, symlinks and branch order

**Location:** `internal/fabriccli/roleguard_test.go#TestSandboxedRoleGuard`

**Issue:** Every clone row passes an absolute `--into`, so judging the raw `into` instead of `cloneDestination(cwd, into)` would pass the suite.
No row places a sandbox root under a subdirectory of the own worktree, so dropping `location.WorktreePath() == worktree` (`roleguard.go:53`) passes too.
No row uses a link, so removing the symlink resolution leaves every outcome unchanged.
No row runs a path-flag call from the own sandbox hub, so moving the sandbox pass above the path-flag refusal would pass.

### [LOW] A not-yet-existing target skips symlink resolution of its ancestors

**Location:** `internal/fabriccli/roleguard.go#inOwnSandboxRoot`

**Issue:** When `EvalSymlinks(dir)` fails, the guard falls back to `filepath.Clean(dir)` (`roleguard.go:45-48`) and keeps every symlink in the existing ancestors.
`ResolveWorktree` resolves symlinks (`lymcwd.go:202-211`), so for a worktree reached through a linked path `location.WorktreePath()` differs from the unresolved `worktree` and a legitimate clone into the own sandbox is refused.
In the other direction, `--into <sandbox>/link/new` with `link` pointing outside passes the lexical check while `CloneHub` builds `HubPath(dest, name)` at the link's target (`fabricengine/clone.go:209`).
The doc comment's "a link counts where it lands" holds only for an existing target.
Severity is LOW: the escaping case needs a link planted inside the sandbox, and the clone's probe directory under the destination (`recordsprobe.go:63`) likely fails first when the destination is missing.

### [LOW] A same-named worktree in another hub passes the ownership check

**Location:** `internal/fabriccli/roleguard.go#inOwnSandboxRoot`

**Issue:** The check compares `location.WorktreeName` with `name.Slug` and the path with the walk's own candidate (`roleguard.go:53`); neither ties the candidate to the strand's hub.
A strand `ly:slug:webster` therefore passes in a sandbox under `<otherhub>/slug/.lym/harden-sandbox`.
The harm is bounded, since the target is still another run's throwaway sandbox hub, so this is LOW rather than the assistant's MEDIUM.

### [LOW] fabric.go header describes the old guard

**Location:** `internal/fabriccli/fabric.go:7`

**Issue:** The header reads "The guard refuses every mutating verb from a task session's strand name before the verb runs", which the diff's new pass at `roleguard.go:130-132` makes false.

### [NIT] doc.go Merge section names only the merge-task exception

**Location:** `internal/fabriccli/doc.go:124`

**Issue:** "The sandboxed-role guard refuses these verbs from a task session." is now true only outside the role's own sandbox root; the new "Sandboxed-role guard" section states the exception, so the reader is only briefly misled.

## Focus conclusions

- `inOwnSandboxRoot`, paths outside the root: `..` segments collapse through `Clean` or `EvalSymlinks` and are refused; an existing outward link is refused; a lookalike non-worktree directory is refused; a nested sandbox root of a differently named sandbox worktree fails closed. A cwd in a sandbox hub container but outside any sandbox worktree passes the guard, but the verb's own `lymcwd.Resolve` climbs to the real worktree and its strict cwd gate refuses, so nothing acts on the real hub.
- `inOwnSandboxRoot` and `ResolveWorktree`: a failed resolve fails closed; a worktree reached through a symlink fails for a missing target (F3) and, through the standard `.lym` link, for every target (F1).
- `verbTargetDir`: every guarded verb but clone resolves its hub from the context cwd through `resolveWarpLocation`, so the cwd is the right directory; `add --parent`, `remove <slug>` and `pull --hub` act inside the hub that cwd resolves; the hidden teardown child passes ahead of the new branch, as before.
- `cloneDestination`: empty, absolute and relative `--into` give the same result as the removed inline switch, and the guard and the clone call it with the same cwd and flag.
- `sandboxedRoleRefusal`: the new pass sits after the teardown-child pass, the path-flag refusal and the reader and merge-task passes, so it widens none of them; the refusal text keeps the hubforge and status FAILED way forward and names the sandbox root.
- `TestSandboxedRoleGuard`: rows cover the real-hub (own worktree outside its sandbox), another worktree's sandbox and lookalike-directory refusals with literal booleans; the gaps are F1's layout and F2.
- Scope: the stencil line and the `doc.go` section describe the requested behaviour for the round agent and the package reader, and are licensed. The `Root-Cause:` trailer names the defect the diff removes, the strand-only judgement.

## Rejected

- code-scope's LOW that the pass is open to every task role rather than only hardener round roles: the Goal's operative rule is the location, its own sandbox root, with the refusal kept everywhere else; only a hardener run builds a sandbox root, so the role-agnostic pass admits nothing the Goal refuses.
- code-correctness's MEDIUM severity for the missing-target symlink case is lowered and kept as F3; its false-refusal half on the standard layout is subsumed by F1.
- The assistant's MEDIUM test-gap finding is merged into F2, its junction fixture into F1's fix.

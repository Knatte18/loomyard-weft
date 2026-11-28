---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/clone.go#ignoredWork
    summary: ignoredWork counts the ignored files lym itself writes as work, so a plain --reset of any finished or used hub is refused while it holds nothing of the operator's.
    fix: "In `ignoredWork`, skip every ignored path lym itself writes into the three stores, each through a named constant, not a literal. In the prime code store that is the anchored paths of fabric's warp exclude list (`Config.WarpExcludePaths`, default `CLAUDE.local.md AGENTS.override.md .heddle .claude/settings.local.json` in internal/fabricengine/template.yaml), read from the hub's fabric config. It is also the IDE's `.vscode/` config that internal/ideengine/spawn.go keeps out of git through info/exclude. In the Bolt checkout it is `fabriclayout.BoardWriteLockFile`, `fabriclayout.BoardPushLockFile` and the board render manifest `.board-rendered.json`. That manifest is boardengine's unexported `renderManifestFile`, and fabric must not import boardengine, so move its name into a fabriclayout constant both read. Whether the other lock lines in the records store's info/exclude (`.gitrepo-push.lock`, `*.swaplock`, `*.json.lock`, `settle-record.patch.lock`) are still written by current code is unverified: check their writers and skip each one that is. Then make three edits that state the narrowed rule: the `ignoredWork` doc comment, the internal/fabricengine/doc.go paragraph (\"Every ignored path counts but links and lym's own never-tracked directories\"), and the summary's \"Every ignored path counts as work\". Add rows to the `TestCloneOneURL_JoinsTheNamespacedHub` reset tests that pin the skips. A hub holding a rendered `CLAUDE.local.md`, a `board.lock` in `_bolt`, a `.weft/` directory and an ignored link, and nothing else, passes a plain `--reset`. Adding an operator-ignored `secret.env` beside them is still refused."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/clone.go#storeWorkOriginLacks
    summary: Commits reachable only from the Bolt checkout's own HEAD (a detached HEAD, a paused rebase) are removed by a plain --reset unnamed, since the commit count runs in the records store, whose HEAD is not Bolt's.
    fix: "In `storeWorkOriginLacks`, when `ownsRefs` is false, still count the commits the checkout's own HEAD alone holds. Run `git rev-list --ignore-missing --count HEAD --not --branches --tags --remotes` in that dir, and when the count is not 0 add `<n> commit(s) of <dir>'s HEAD that origin lacks`. Branches and tags are already counted once in the records store, so this adds no double count. Add a row to the reset refusal table in `TestCloneOneURL_JoinsTheNamespacedHub`: in `h.BoardDir()`, `checkout --detach` and commit a file, then assert the refusal names it."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/clone.go#checkResetTarget
    summary: A store whose git read fails, such as a Bolt .git file pointing at a gitdir that is gone, now makes --reset refuse with no way forward, and --discard-work cannot pass it, where the old code read only the code store.
    fix: "In `checkResetTarget`, when `hubWorkOriginLacks` returns an error, return it with a way forward, as `existingHubError` does: `reset: what the hub at <hubPath> holds cannot be read: <err>; way forward: <moveAsideWayForward(hubPath)>`. Add a reset test row that replaces `_bolt/.git` with a file naming a missing gitdir and asserts that the refusal carries `way forward: move it aside`."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/fabricengine/clone.go#hubWorkOriginLacks
    summary: The board entry is a bug, labelled bug, asking to stop --reset losing work, and no commit of the review range carries a Root-Cause trailer.
    fix: "Give the fix commit of this round a `Root-Cause:` trailer naming the defect the diff removes: `hubWorkOriginLacks` read only the prime code store, so `--reset` removed the records store's and the Bolt checkout's unpushed commits, uncommitted changes and ignored files unread. Rewrite no existing commit."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: tools/sandbox/SANDBOX-CORE-SUITE.md:244
    summary: The S6 step still says --reset makes repeated sandbox clones idempotent, but a finished clone leaves its module-config commit unpushed on the records branch, which plain --reset now refuses.
    fix: "Edit tools/sandbox/SANDBOX-CORE-SUITE.md:244 to say that a repeated sandbox clone over a hub it made before takes `--reset --discard-work`, since the earlier clone left its module-config commit unpushed. Or say it takes plain `--reset` once that hub's records are pushed."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabriccli/fabric.go:100
    summary: No test drives the CLI's --discard-work flag through runCloneWithReset into CloneOptions.DiscardWork, so a dropped GetBool or mapping passes every test.
    fix: "Add a row to an existing internal/fabriccli clone command test. Run the cobra `clone` command with `--discard-work` and without `--reset`, and assert the envelope's error is `--discard-work only widens --reset; way forward: pass --reset with it, or leave it out`. That refusal comes before any git, so the row fits tier 0. Whether an existing fabriccli tier-0 command test can host the row is unverified; if none can, add one with `tiergate.Require(t, 0)`."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/clone.go#storeWorkOriginLacks
    summary: The cap of ignoredPathsNamed named paths and the trailing \"and N more\" count have no test; the one ignored-file row names a single path.
    fix: "Add a row to the reset refusal table in `TestCloneOneURL_JoinsTheNamespacedHub`. It ignores and writes seven files `a.env` to `g.env` in the prime, and the refusal must name `ignored files in <prime>: a.env, b.env, c.env, d.env, e.env and 2 more`, with the literal order git lists them in."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-affected-set
    location: internal/fabricengine/clone.go#RefuseHubWithoutBolt
    summary: RefuseHubWithoutBolt still names a plain --reset clone, which is now refused when the hub's records store holds unpushed commits, as a used hub whose Bolt was lost does; the test had to push the records first to keep the named clone working.
    fix: "In `RefuseHubWithoutBolt`, call `hubWorkOriginLacks(l.HubPath)`. When it reports store work and no use, build each named clone with `DiscardWork: true` and name the work it loses. Otherwise keep the plain `--reset` form. Update internal/fabricengine/doc.go's RefuseHubWithoutBolt sentence to match. Add a `TestCommand_HubWithoutItsBoltNamesTheResetClone` row with the records store left unpushed that asserts the `--reset --discard-work` clone is named."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/clone.go:1027
    summary: storeWorkOriginLacks reads any Lstat error on <store>/.git, such as EACCES, as no repository, so that store's work is removed unnamed.
    fix: "In `storeWorkOriginLacks`, return nil only when `errors.Is(err, os.ErrNotExist)`. Return any other Lstat error as `look for the repository of <dir>: <err>`, which `checkResetTarget` then carries with the way forward F3 adds."
---

### [BLOCKING] lym's own ignored files count as work

**Location:** `internal/fabricengine/clone.go#ignoredWork`
**Issue:** The board entry asks for a refusal over "ignored files that hold work".
`ignoredWork` skips only links, `.lym` and `.weft`.
On this machine's live hub, `git ls-files --others --ignored --exclude-standard --directory` lists these entries:
- In the prime: `CLAUDE.local.md` and `AGENTS.override.md` (regular files from heddle's render, which `CloneAndWire` runs through `NotifyWorktreeMade`), `.claude/` and `.vscode/`.
- In `_bolt`: `.board-rendered.json`, `board.lock`, `board.push.lock`, `.gitrepo-push.lock` and `board.json.swaplock`.

Every one of them would be named as work.
So a plain `--reset` of a hub that has finished a clone and rendered heddle is refused, and the idempotent re-clone always needs `--discard-work`.
That flag then discards the operator's real ignored files along with lym's.
The doc comment's "every ignored path counts but lym's own" and scenario F12's precondition do not hold on a real hub.

### [MEDIUM] Commits only the Bolt checkout's HEAD holds are lost unnamed

**Location:** `internal/fabricengine/clone.go#storeWorkOriginLacks`
**Issue:** The Bolt checkout is read with `ownsRefs: false`, so its commits are counted only through the records store.
There `rev-list HEAD --branches --tags` resolves the records store's own HEAD.
A detached HEAD or a paused rebase in `_bolt` holds board commits no branch reaches, and `--reset` removes them without naming them.

### [MEDIUM] A store that cannot be read blocks --reset with no way forward

**Location:** `internal/fabricengine/clone.go#checkResetTarget`
**Issue:** A failed `git status` in the records store or `_bolt` returns `reset: read the status of …` before `DiscardWork` is consulted, and names no way forward.
A Bolt `.git` file left dangling is one such case.
Before this diff only the code store was read, so such a hub could be reset.

### [MEDIUM] No Root-Cause trailer on a bug fix

**Location:** `internal/fabricengine/clone.go#hubWorkOriginLacks`
**Issue:** The entry is labelled `bug`, and its title names `--reset` removing unpushed work as the defect.
None of f3b82b33d, cbb379580 or fcf76fd71 carries a `Root-Cause:` trailer.

### [MEDIUM] SANDBOX-CORE-SUITE still promises an idempotent --reset

**Location:** `tools/sandbox/SANDBOX-CORE-SUITE.md:244`
**Issue:** A finished clone commits its per-worktree module configs on the records branch with no push (`internal/fabriccli/clone.go`, `CloneAndWire`).
The summary concedes that a plain `--reset` of that hub is refused.
The core suite's S6 step was left saying `--reset` keeps repeated sessions idempotent.

### [MEDIUM] The CLI flag's pass-through is untested

**Location:** `internal/fabriccli/fabric.go:100`
**Issue:** Every new test sets `CloneOptions.DiscardWork` directly.
None runs the `clone` command with `--discard-work`, so the `GetBool` and the `cloneFlags` mapping are unpinned.

### [LOW] The ignored-path cap and count are untested

**Location:** `internal/fabricengine/clone.go#storeWorkOriginLacks`
**Issue:** `ignoredPathsNamed` and the " and %d more" suffix are reached by no row.
`IgnoredFileInThePrime` writes one path.

### [LOW] RefuseHubWithoutBolt can name a clone that is refused

**Location:** `internal/fabricengine/clone.go#RefuseHubWithoutBolt`
**Issue:** A clone killed before its Bolt checkout leaves no unpushed records commit, so the plain form holds there.
But the refusal fires for any hub without a Bolt repository, a used one included, and then the named plain `--reset` clone is refused over the records store's unpushed commits.
`reconcile_missingbolt_integration_test.go` had to push the records before the named clone would apply.

### [LOW] Any Lstat error reads as no repository

**Location:** `internal/fabricengine/clone.go:1027`
**Issue:** `storeWorkOriginLacks` returns nil for every `Lstat` error on `<store>/.git`, not only a missing one.
An unreadable store's work is then removed without being named.

## Rejected

- code-scope S2 (`cloneFlags` struct is unrequested scope): rejected. Adding the flag had to change `runCloneWithReset`'s signature, and the struct is the shape of that forced edit, with no behaviour of its own.
- code-tests LOW (the `ignoredWork` skips are pinned only incidentally): rejected as a separate finding. F1's fix adds the rows that pin the skips once they are narrowed.
- code-scope S1 and code-affected-set F1: merged into F1, the same defect.

---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Merge danger
    summary: The two-way door and the blast radius `runpusher` leave out two things. PruneLedger now deletes ledger entries in a new state, a leftover directory without `.git`, and their owed hand-overs, dropped landings and held records go with them. The changed TargetOf and TargetRefused answers also reach callers in beamcli, battencli and loomcli.
    fix: "In `_lym/landing/summary.md` under `## Merge danger`, set the door to one-way, since an entry PruneLedger drops is not restored by a revert, and name that deletion. Widen the blast radius to the push ledger and the callers of `TargetOf`, `TargetRefused` and `Start`. These are `internal/beamcli` (`restartHubPushers`, `refusedPushWayForward`), `internal/battencli` (`DobbyRuns`, the status commit's `StartPush`) and `internal/loomcli` (the status commit's `StartPush`, `startRunPusher`, `parentPushState`)."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/runpusher/targetreads.go:86
    summary: A regular file at the worktree path makes `Lstat(<path>/.git)` fail with ENOTDIR. `worktreeGone` returns that as an error, which the old directory stat never did. TargetOf, TargetRefused, the start and PruneLedger then fail on it on every call, and PruneLedger keeps the entry for good. The error path also has no test row.
    fix: "In `worktreeGone`, read `errors.Is(err, syscall.ENOTDIR)` as gone, the same as `fs.ErrNotExist`, since a file at the path holds no worktree. In `internal/runpusher/seams_test.go`, add a `removals` row named `ThePathIsARegularFile` that replaces the worktree directory with a regular file and expects gone with no error. In `TestPruneLedger`, add an entry whose `Worktree` cannot be read, under a directory made untraversable through `permkit.Chmod`, and expect the entry kept and an error returned."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/runpusher/parentseams.go:59
    summary: "`pairOf` still judges a pair standing from its directory alone, so a leftover directory without `.git` reads as standing in the parent seams. Those seams then run git on it, and `HoldPair` writes a ledger entry for it. The rest of the package, and the change description's \"A pair stands only while `<worktree>/.git` exists\", read that directory as gone."
    fix: "In `pairOf`, replace the `os.Stat(pair.WorktreePath())` read with `worktreeGone(pair.WorktreePath())`, so that standing is `!gone`, and return its error. Update the doc comment to say a pair stands while its worktree holds its `.git` entry."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/runpusher/run.go:58
    summary: The `WorktreeGone` field doc still says it reports whether the worktree no longer exists. The seam now also answers true for a directory that exists without its `.git` entry.
    fix: "Change the doc at `internal/runpusher/run.go:58` to: `// WorktreeGone reports whether the worktree no longer stands: its path is gone, or a removal left a directory there without its .git entry.`"
  - id: F5
    severity: LOW
    class: consistency
    origin: handler
    location: "_lym/landing/summary.md#Design decisions"
    summary: The change description says listing worktrees needs the repository, which a removed pair's path no longer reaches. Fabric already lists them from the prime, in `fabricengine.PairWorktreeStands`, so that premise is false. The real reason for the `.git` stat is left unstated.
    fix: "In `_lym/landing/summary.md`, rewrite the bullet that opens with **The `.git` entry decides, not the git worktree list.** Say that `fabricengine.PairWorktreeStands` lists the worktrees from the prime, and that the start path instead stats `.git` because the read needs no prime location and runs no git on each tick's start. Drop the sentence claiming that the list cannot be reached."
---

### [MEDIUM] The merge-danger door and blast radius name less than the diff changes

**Location:** `_lym/landing/summary.md#Merge danger`

**Issue:** The rubric makes a `two-way` call a finding when the diff deletes data.
`PruneLedger` (`internal/runpusher/targetreads.go:141-162`) now also removes the ledger entry of a directory left without `.git`.
It keeps an entry only when it holds landings, so the entry's `HandOvers`, `Dropped` and `HeldRecords` go with it, and reverting the code does not restore them.
The blast radius names only `runpusher`.
But the new `TargetOf` answer, a pair with nil error, and the new `TargetRefused` answer, false, change behaviour in `beamcli/dobbydeps.go:246-275,539-574`, `battencli/dobbyreads.go:323`, `battencli/commitstatus.go:168-210`, `loomcli/wiring.go:214-226,1409-1418` and `loomcli/landingdeps.go:1042`.

### [LOW] `worktreeGone` turns a file at the path into a permanent error

**Location:** `internal/runpusher/targetreads.go:86`

**Issue:** `os.Lstat(filepath.Join(path, ".git"))` on a path that is a regular file returns ENOTDIR, which `errors.Is(err, fs.ErrNotExist)` does not match.
The old `os.Stat(path)` read such a path as standing.
Now every read through `worktreeGone` fails, and `PruneLedger` keeps the entry and logs a prune failure each tick.
No test row drives a failing read, so `PruneLedger`'s rewritten failure branch (`targetreads.go:153-154`) is unpinned too.
The code-tests lens raised the missing test rows separately; they are merged here because one fix covers both.

### [LOW] The parent seams' `pairOf` keeps the old directory-only presence read

**Location:** `internal/runpusher/parentseams.go:59`

**Issue:** `pairOf` returns `standing` from `os.Stat(pair.WorktreePath())` (`parentseams.go:60-70`).
Every other presence read in the package now goes through `worktreeGone`, which reads a directory without `.git` as gone.
For such a directory the parent seams run git reads and write a held entry, the kind of read on a removed path that this change removes elsewhere.

### [LOW] The `WorktreeGone` field doc misstates the seam

**Location:** `internal/runpusher/run.go:58`

**Issue:** The doc reads "reports whether the worktree no longer exists".
`run.go:98` now wires the seam to `worktreeGone`, which also answers true when the directory still exists without its `.git` entry (`targetreads.go:84-85`).

### [LOW] The design-decision rationale for the `.git` stat rests on a false premise

**Location:** `_lym/landing/summary.md#Design decisions`

**Issue:** The bullet says "Listing worktrees needs the repository, which a removed pair's path no longer reaches."
`fabricengine.PairWorktreeStands` (`internal/fabricengine/remove.go:987-1001`) lists the linked worktrees from the prime, so the removed pair's path never needs to reach the repository.
It already tells a leftover directory from a registered worktree.
The `.git` stat is a defensible choice, but the description gives a reason the code does not bear out.

## Rejected

- code-correctness finding 2 (NIT): `TargetOf` returns a filled target beside its error. Every caller drops the value on error, and Go defines no value on an error return, so nothing is wrong.
- code-correctness finding 3 (LOW): with this change, a beam pair whose worktree was removed outside `internal/pairteardown` while landings stand loses its start warning. Teardown refuses in that state, `PruneLedger` keeps the entry, and the push summary's owed notice still reports it. The board entry asks for exactly this silent skip.
- The new `gitEntryName` constant looks like a second declaration beside `cliwire`'s `gitDirName`. `.git` is git's fixed name and cannot go stale, so the named-constant rule in `PATTERN-code-conventions` does not reach it.

Focus notes, all driven this round:
- The reported path, `DobbyRuns` → `restartPusher` → `TargetOf`, reaches the pusher only while the directory stands, because `PairPresent` reads it. That case is the leftover directory, and the change now skips it silently. `settleCodeTip` still runs beside it for such a run. For a landed run, though, it reads origin's records branch from the prime and runs no git on the pair path.
- Creation, adoption and hand-over make the pair through `git worktree add`, which writes `.git` with the directory, so no live pair lacks the entry.
- `lym gate test ./internal/runpusher` passed: 68 passed, 0 failed.

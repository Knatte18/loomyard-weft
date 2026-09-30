---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/02-fabric-anchored-exclude.md
    summary: ExcludeAnchoredDir's check-ignore runs through the raw gitexec.RunGit, adding an unmarked third raw site to fabricengine and failing cmd/lyx's pinned Checked-Call guard
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/05-batten-wire-open.md
    summary: The batten integration tests that reach the real OpenIDE assert nothing about it, so a SpawnDriven that fails is only a warning and still passes a clean teardown
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/02-fabric-anchored-exclude.md
    summary: PathTracked is an independently testable query unrelated to info/exclude, bundled into the exclude card and into gitexclude.go, whose header scopes that file to exclude mutation
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/02-fabric-anchored-exclude.md
    summary: No card tests the acceptance criterion that the driven exclude write holds fabricengine's exclude lock; exact-content assertions pass just as well without the lock
  - id: F5
    severity: LOW
    location: _lyx/plan/03-ide-spawn-driven.md
    summary: The shared resolution helper is specified as resolving worktreeDir and color only, but Spawn's prime-workspace branch also needs primeName and primeErr
  - id: F6
    severity: LOW
    location: _lyx/plan/01-vscode-attach-chain.md
    summary: Card 1 creates TaskChainInteractive and passes it from Spawn, but its Create list declares only TaskChain and TaskChainAttachOnly
  - id: F7
    severity: LOW
    location: _lyx/plan/03-ide-spawn-driven.md
    summary: SpawnDriven writes the repo-wide exclude line even when tasks.json is tracked and nothing gets written, hiding new files under a tracked .vscode/ in every worktree
  - id: F8
    severity: LOW
    location: _lyx/plan/00-overview.md
    summary: The verify section skips cmd/lyx, whose repo-wide guard tests scan the targeted packages, so F1's guard failure would pass verify
---

### [BLOCKING] Raw `gitexec.RunGit` in `ExcludeAnchoredDir` breaks the Checked-Call guard

**Location:** `_lyx/plan/02-fabric-anchored-exclude.md`, the `git check-ignore` bullet.

**Issue:**
The card tells the implementer to run `git check-ignore -q` "through `gitexec.RunGit`" so it can tell exit 0 from exit 1.
CONSTRAINTS.md's gitexec Checked-Call Invariant allows the raw form only at pinned, `//gitexec:raw`-marked sites.
`cmd/lyx/checkedcall_test.go` pins `"internal/fabricengine": 2` (`weftRepoExists`, `weftBranchExists`), and `weftwiring.go`'s header calls those two "the whole of internal/fabricengine's pinned raw-site allowance".
A third, unmarked `gitexec.RunGit` in `gitexclude.go` fails `TestCheckedCallInvariant_RawSitesMarkedAndPinned`.
The plan's `## verify:` does not run `./cmd/lyx/...`, so this passes verify and fails only at the full-suite gate.
The raw form is not needed:
`gitexec.Run` returns a `*gitexec.GitError` carrying `ExitCode` whenever git ran and exited non-zero, so exit 1 is `errors.As(err, &gitErr) && gitErr.ExitCode == 1`.

**Fix:**
Specify `gitexec.Run`: nil error means ignored (exit 0), a `*gitexec.GitError` with `ExitCode == 1` means not ignored, and any other error is returned wrapped.
State that no raw site is added, so the pinned count stays 2.

### [MEDIUM] The batten integration tests never check that the driven path ran

**Location:** `_lyx/plan/05-batten-wire-open.md`, the `testmain_integration_test.go` bullet.

**Issue:**
The card says the lifecycle and records integration tests "then run the real driven path against real pairs", and "their teardown must still find the pair clean".
Two problems.
First, `OpenIDE` failures are warning-only by design, so if `SpawnDriven` fails early (for example on the exclude step) nothing is written, teardown finds the pair clean and every test stays green.
The clean-teardown assertion only means something once the test has shown that `.vscode/` was actually written.
Second, not every lifecycle test reaches a spawn.
`TestBattenIntegration_FourRowRun_SeedsChildCommitsAndTearsDown`'s status stub answers `done`, so `Call` never spawns and never opens.
The one test that reaches `OpenIDE` and then tears down is `records_integration_test.go`'s, through `callAwaiting`'s approval re-spawn.
The fresh-spawn lifecycle tests (status stubs answering `false, nil`) reach it through `Call`.
The card leaves this as a conditional "fix it if it breaks" and states no positive assertion.

**Fix:**
Name `records_integration_test.go` as a target.
After the approved step, it asserts that the pair's anchor `.vscode/tasks.json` exists and decodes to the single attach task, and that `git status --porcelain` in the pair is empty.
Its existing teardown assertions then show that removal succeeds with `.vscode/` present.
State that the other integration tests stay unmodified and must pass.

### [MEDIUM] `PathTracked` needs its own card

**Location:** `_lyx/plan/02-fabric-anchored-exclude.md`.

**Issue:**
The rubric's granularity rule says an independently testable symbol gets its own card even when one card is its only consumer.
`PathTracked` is a read-only index query with its own three test cases and no relation to `ExcludeAnchoredDir`: neither calls the other.
The card also puts it in `gitexclude.go`, whose header says the file "owns every read-modify-write of a git repository's `.git/info/exclude`", which a `git ls-files` query is not.

**Fix:**
Split `PathTracked` into its own card with its own file (`internal/fabricengine/pathtracked.go`) and its own integration-tagged test file.
Card 2 keeps `ExcludeAnchoredDir` only.
Renumber the later cards to match.

### [MEDIUM] The exclude-lock acceptance criterion has no test

**Location:** `_lyx/plan/02-fabric-anchored-exclude.md`, the tests paragraph; also `03-ide-spawn-driven.md`.

**Issue:**
The decision record's criterion reads: "The driven path's exclude write holds `fabricengine`'s exclude lock, and preserves every existing line of the shared `info/exclude`."
Card 2 covers the second half with exact-content assertions, and card 3 with its custom-line check.
The first half rests only on the intent's instruction to route through `mutateGitExclude`.
None of the listed tests would fail if `ExcludeAnchoredDir` wrote the file directly with `writeFileAtomically` and skipped the lock.

**Fix:**
Add a card 2 test that takes the exclude lock itself (`lock.AcquireWriteLock` on `gitExcludeLockFileName` beside the resolved exclude path).
It then starts `ExcludeAnchoredDir` in a goroutine, asserts the call has not returned and the file is unchanged while the lock is held, releases the lock, and asserts the call returns `changed` with the line added.
`flock` conflicts between two opens in one process, so this is deterministic in-process.

### [LOW] The shared helper must return the prime's name and error

**Location:** `_lyx/plan/03-ide-spawn-driven.md`, step 1.

**Issue:**
Step 1 extracts "that shared resolution" (worktree path, `PrimeName` with warn-and-degrade, `PickColor`) into one helper both functions call.
`Spawn`'s prime-workspace branch (`primeErr == nil && slug == primeName`) needs both `primeName` and `primeErr` after that resolution.
The card never says the helper returns them.
A helper returning only `worktreeDir` and `color` would make the ImpactSummary's claim that `Spawn`'s observable behaviour is unchanged false, or push the implementer into resolving `PrimeName` twice, which would log the warning twice.
The two edits to `Spawn` (card 1's `WriteConfig` argument and card 3's extraction) compose, since they run in sequence.

**Fix:**
Specify the helper's returns: the worktree path, the color, and the prime's name with a flag saying whether it resolved.
`Spawn` keeps its prime branch keyed on those values, and the warning is logged exactly once, inside the helper.

### [LOW] Card 1's Create list omits `TaskChainInteractive`

**Location:** `_lyx/plan/01-vscode-attach-chain.md`, `**Create:**`.

**Issue:**
The intent declares two constants, and card 1's own edit of `Spawn` passes `TaskChainInteractive`, but only `TaskChain` and `TaskChainAttachOnly` are declared.
The target list does not match what the card creates.

**Fix:**
Declare `plan:internal/vscode#TaskChainInteractive` with head `const TaskChainInteractive TaskChain = 0`.
List it in the `ide-spawn-driven` card's `Uses`, since that card edits `Spawn`, whose body keeps passing it.

### [LOW] Do the tracked-file check before the exclude write

**Location:** `_lyx/plan/03-ide-spawn-driven.md`, steps 2–3.

**Issue:**
Step 2 writes the exclude entry before step 3 checks whether `tasks.json` is tracked.
When it is tracked, `git check-ignore` reports the directory not ignored (verified: with `.vscode/tasks.json` committed and `.vscode/` in `.gitignore`, `git check-ignore -q -- .vscode/` exits 1), so step 2 appends `/<anchor>/.vscode/` to the shared exclude file.
Step 3 then writes nothing.
The line serves no purpose, and it hides any new, meant-to-be-committed file under the repo's tracked `.vscode/` (a new `launch.json`, say) in every worktree, prime included.
Step 2's own rationale ("before anything is written") only applies when something will be written.

**Fix:**
Swap the steps.
When `tasks.json` is tracked, warn, skip both the exclude write and `WriteConfig`, and launch.
Otherwise, write the exclude entry, then call `WriteConfig`.
The tracked-file test also asserts the shared exclude file is unchanged.

### [LOW] Verify does not run the repo-wide guards in `cmd/lyx`

**Location:** `_lyx/plan/00-overview.md`, `## verify:`.

**Issue:**
The checked-call, tier-purity, hermetic-env and spawn-observability guards live in `cmd/lyx`'s untagged tests and scan every targeted package's source.
Verify runs the targeted packages but not `cmd/lyx`, so a guard regression like F1 passes verify.

**Fix:**
Add `./cmd/lyx/...` to the untagged `go test` line.

## Focus verdicts

- Card 1 target list: incomplete, see F6.
- Card 2 granularity: split, see F3.
- Card 2 `check-ignore`: verified against a scratch repo.
  A directory that does not exist yet gives exit 1 when not ignored and exit 0 when an exclude line covers it, as the card states.
  A directory with tracked content gives exit 1 even when `.gitignore` lists it (see F7 for the consequence).
  An operator's global excludes counts as "git already ignores it", which matches the decision record's "or left alone when git already ignores it".
  Tests are unaffected, since every targeted test package runs `gitkit.HermeticGitEnv()`.
  The one implementability defect is the raw call (F1).
- Exclude lock criterion: not carried by any test, see F4.
- Cards 1 and 3 on `Spawn`: they compose, provided the helper returns the prime's name and error, see F5.
- Card 5 integration tests: see F2.
- Card 4 against the Run-Shed criteria: not a defect.
  Bounce accounting is `OutputPointer.BudgetExempt` plus the outcome, so asserting outcome, `Reason` and `BudgetExempt` covers "identical".
  Clearing the once-marker on `found == false` before the spawn is what the decision record prescribes ("removes any once-marker before that spawn").
  It differs from the done-seen clear, which is keyed on the post-spawn state, but for a good reason:
  the once-marker must be gone before `openIDEOnce` runs in the same `Call`.

## Focus departures

- "Checked and left out: … card 3's exclude-before-tracked-check order, which writes the entry even when the repo tracks `tasks.json`, falls under the repo-wide exclude risk the decision record already accepts."
  Departed, raised as F7.
  The accepted risk is an exclude line that hides an untracked `.vscode/` the driven path created.
  In the tracked case the driven path creates nothing, so the line has no purpose, and it hides new files inside a directory the repo deliberately tracks.
  Reordering the steps costs nothing.
